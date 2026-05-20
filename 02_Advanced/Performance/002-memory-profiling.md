# Memory Profiling

## What is it

`Memory profiling` là quá trình đo và phân tích cách ứng dụng cấp phát, giữ lại, và giải phóng object trong heap. Mục tiêu không chỉ là biết heap đang lớn bao nhiêu, mà còn hiểu object nào đang sống quá lâu, object nào tạo quá nhiều, và vì sao GC phải làm việc nặng.

Nó thường liên quan tới `heap dump`, allocation profile, retained size, dominator tree, GC logs, và timeline về heap usage. Đây là nhóm kỹ thuật quan trọng khi nghi có memory leak, excessive object churn, hoặc pause time tăng vì pressure trong heap.

## How I used to misunderstand it

Mình từng nghĩ memory problem chỉ tồn tại khi app `OutOfMemoryError`. Thực tế nhiều hệ thống chưa bao giờ crash nhưng vẫn chậm dần vì old generation phình lên, GC chạy dày hơn, và latency bị kéo dài.

Mình cũng từng nhìn một heap dump rồi chỉ tập trung vào object count. Điều đó chưa đủ. Có những type xuất hiện rất nhiều nhưng retained size nhỏ, trong khi một cache map hoặc graph object không quá đông lại giữ phần lớn heap sống.

Một hiểu nhầm khác là trộn `memory profiling` với `CPU profiling`. CPU profile trả lời app đang bận ở đâu. Memory profile trả lời object nào đang được tạo hoặc giữ lại quá nhiều. Hai tín hiệu này thường liên quan, nhưng không thay thế nhau.

## How it actually works

Memory profiling có hai câu hỏi lớn.

1. `allocation rate`, tức chương trình đang tạo object nhanh tới mức nào
2. `retained memory`, tức object nào đang giữ lại các object khác khiến heap không thu hồi được

Hai câu hỏi này liên quan nhưng không giống nhau.

Allocation rate cao thường tạo nhiều GC pressure, đặc biệt với object ngắn hạn. Retained memory cao thường gợi ý cache không giới hạn, listener không được remove, static collection giữ reference, hoặc object graph bị giữ ngoài ý muốn.

Heap dump cho mình ảnh chụp tại một thời điểm. Allocation profile lại cho mình nhịp độ tạo object theo thời gian.

### Allocation vs retained

| Câu hỏi | Tín hiệu cần nhìn | Công cụ hay dùng | Smell thường gặp |
|---|---|---|---|
| App tạo object quá nhanh? | allocation rate, young GC pressure | `async-profiler` allocation, `JFR` allocation events | string concat trong loop, DTO churn, boxing nhiều |
| App giữ memory quá lâu? | retained size, dominator tree, GC root path | heap dump, MAT, JMC | cache không limit, singleton giữ state, listener quên cleanup |
| Heap tăng sau mỗi batch? | timeline heap usage giữa các lần chạy | JFR, metrics, repeated dump comparison | object sống qua nhiều cycle ngoài ý muốn |

Khi đọc heap dump, `retained size` thường quan trọng hơn `shallow size`. Một object map nhìn shallow size nhỏ nhưng có thể dominator cả một graph lớn. Vì vậy memory profiling không chỉ là “type nào nhiều nhất” mà là “đường nào đang giữ heap sống”.

### Memory investigation flow

```text
Heap looks big
  |
  +--> Is allocation rate high? ----> look for churn / temporary objects
  |
  +--> Is retained memory high? ----> look for long-lived references
  |
  +--> Is old gen growing over time? -> suspect leak or unbounded cache
```

## Code example

```java
private static final Map<String, byte[]> CACHE = new HashMap<>();

public static void remember(String key, byte[] value) {
    CACHE.put(key, value);
}
```

Đoạn code này không sai về mặt cú pháp, nhưng nếu key tăng mãi và không có eviction thì heap sẽ giữ `byte[]` lâu dài.

Memory profiler hoặc heap dump sẽ cho thấy `CACHE` là dominator giữ retained memory lớn. Nếu mình chỉ nhìn CPU profile, mình có thể không thấy gì quá rõ vì bottleneck ở đây là lifetime của object, không phải compute hotspot.

## When to use / when NOT to use

Dùng memory profiling khi:

- heap usage tăng dần sau nhiều request hoặc job
- GC pause time tăng nhưng CPU business logic không quá cao
- service bị restart vì memory pressure hoặc container limit

Không chỉ nhìn tổng heap rồi kết luận có leak. Cũng không nên so retained size giữa các snapshot khác workload mà không có cùng traffic pattern hoặc cùng thời điểm warmup.

## How this connects to real Java projects

Trong Spring app, memory issue thường xuất hiện ở cache tự chế, request/response object quá lớn, `Hibernate` persistence context giữ entity quá lâu, hoặc bean singleton giữ reference tới data đáng ra chỉ sống theo request. Những thứ này ít khi lộ ra nếu chỉ nhìn controller code.

Khi phân tích batch job hoặc scheduled task trong Spring, memory profiling giúp thấy object nào bị giữ qua nhiều lần chạy, ví dụ collector trung gian, map tổng hợp, hoặc event listener không được reset.

### Spring memory smell map

| Triệu chứng | Nơi nên nghi trước |
|---|---|
| Old gen tăng dần sau traffic | `@Cacheable` key space quá lớn, singleton state giữ request data |
| Batch job xong mà heap không về baseline | collection trung gian sống quá lâu, persistence context chưa clear |
| Full GC nhiều nhưng CPU app không quá cao | retained objects lớn, cache hoặc graph sống lâu |
| Nhiều `byte[]` hoặc `String` to | payload lớn, serialization, buffering, file / network data giữ lại |

## Gotchas

- Object count cao chưa chắc là thủ phạm, retained size mới thường quyết định pressure thật.
- Heap dump chụp sai thời điểm có thể làm mình nhầm object sống tạm với leak thật.
- Cache có hit rate tốt vẫn có thể là memory leak nếu không có size limit hoặc eviction policy.
- Allocation rate cao và retained memory cao là hai vấn đề khác nhau, đừng trộn chúng thành một.
- Heap lớn hơn chỉ làm lỗi xuất hiện muộn hơn nếu retained path gốc vẫn còn nguyên.

## Check yourself

- Vì sao `allocation rate` cao chưa chắc đồng nghĩa `memory leak`?
- Khi nào `retained size` quan trọng hơn `object count`?
- Nếu heap tăng dần sau mỗi batch run, bạn nên hỏi câu gì trước: app tạo quá nhiều object hay app giữ object quá lâu?
- Vì sao memory issue trong Spring hay nằm ở cache, singleton state, hoặc persistence context?
- Nếu CPU không quá cao nhưng GC pause tăng, vì sao memory profiling là hướng hợp lý?

## Exercises

### Exercise 1: Find Peak Heap Growth Step

Độ khó: Easy

Đề bài:
Cho `int[] usedHeapMb`, trong đó mỗi phần tử là heap đang dùng sau một lần lấy mẫu theo thời gian. Hãy trả về chỉ số `i` sao cho mức tăng từ `usedHeapMb[i - 1]` sang `usedHeapMb[i]` là lớn nhất. Nếu mảng có ít hơn `2` phần tử, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
usedHeapMb = [120, 150, 151, 210, 220]
```

Đầu ra:
```text
3
```

Giải thích:
Mức tăng lớn nhất là từ `151` lên `210`, xảy ra tại chỉ số `3`.

Ràng buộc:

- `0 <= usedHeapMb.length <= 100000`
- `0 <= usedHeapMb[i] <= 1000000`
- Nếu có nhiều đáp án, trả về chỉ số nhỏ nhất

### Exercise 2: Count Retained Leak Candidates

Độ khó: Medium

Đề bài:
Cho `String[] objectTypes`, `int[] retainedSizesKb`, và `int thresholdKb`. Một object type được xem là leak candidate nếu retained size tại cùng chỉ số lớn hơn hoặc bằng `thresholdKb`. Hãy đếm số candidate.

Ví dụ 1:

Đầu vào:
```text
objectTypes = ["SessionCache", "byte[]", "String"]
retainedSizesKb = [9000, 200, 150]
thresholdKb = 500
```

Đầu ra:
```text
1
```

Giải thích:
Chỉ `SessionCache` vượt ngưỡng retained size đáng ngờ.

Ràng buộc:

- `0 <= objectTypes.length <= 100000`
- Hai mảng phải có cùng độ dài
- `0 <= thresholdKb <= 100000000`

### Exercise 3: Classify Heap Pressure

Độ khó: Medium

Đề bài:
Cho `int oldGenUsagePercent`, `int allocationRateMbPerSec`, và `boolean fullGcFrequent`. Hãy trả về:

- `"high"` nếu `fullGcFrequent` là `true` hoặc cả `oldGenUsagePercent >= 85` và `allocationRateMbPerSec >= 500`
- `"medium"` nếu chưa phải `high` nhưng một trong hai điều kiện `oldGenUsagePercent >= 85` hoặc `allocationRateMbPerSec >= 500` đúng
- ngược lại trả về `"low"`

Ví dụ 1:

Đầu vào:
```text
oldGenUsagePercent = 88
allocationRateMbPerSec = 520
fullGcFrequent = false
```

Đầu ra:
```text
"high"
```

Giải thích:
Old generation cao và allocation rate cũng cao, nên heap pressure được phân loại là cao.

Ràng buộc:

- `0 <= oldGenUsagePercent <= 100`
- `0 <= allocationRateMbPerSec <= 100000`
- Chỉ trả về `"low"`, `"medium"`, hoặc `"high"`

## Links

- [[001-profiling-tools]]
- [[003-CPU-profiling]]
- [[../JVM-Internals/008-heap-dump-and-analysis]]
- [[../JVM-Internals/005-GC-tuning]]
- [Java SE Troubleshooting Guide, Diagnose Memory Leaks](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks004.html)
- [Eclipse Memory Analyzer Tool](https://eclipse.dev/mat/)
- [JDK Mission Control](https://www.oracle.com/java/technologies/jdk-mission-control.html)

