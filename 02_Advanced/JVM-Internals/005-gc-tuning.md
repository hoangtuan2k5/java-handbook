# GC Tuning

## What is it

`GC tuning` là quá trình quan sát allocation pattern, heap usage, pause distribution, và collector behavior rồi mới chỉnh cấu hình JVM một cách có giả thuyết.

Nó không phải trò “thêm vài flag thần kỳ”. Tuning tốt luôn bắt đầu từ triệu chứng runtime thật, rồi nối triệu chứng đó với memory behavior thật.

## How I used to misunderstand it

Mình từng nghĩ cứ tăng `-Xmx` là sẽ hết vấn đề GC.

Thực tế tăng heap đôi khi chỉ trì hoãn triệu chứng. Nếu nguyên nhân là leak, cache quá lớn, object retention, collector không hợp workload, hoặc container memory limit không đủ, heap lớn hơn chỉ làm lỗi đến muộn hơn hoặc làm pause đắt hơn.

## How it actually works

Tuning GC nên đi theo một vòng lặp nhỏ:

```text
Symptom
  -> collect logs/metrics
  -> form one hypothesis
  -> change one thing
  -> compare before/after
```

Những tín hiệu đáng nhìn nhất thường là:

| Tín hiệu | Có thể gợi ý gì |
| --- | --- |
| Allocation rate cao | App tạo nhiều object ngắn sống |
| Young GC quá dày | Young generation chịu áp lực lớn |
| Post-GC baseline tăng dần | Có leak hoặc retention bất thường |
| Pause tail xấu | Collector goal, object graph, hoặc concurrent cycle có vấn đề |
| Full GC xuất hiện lặp lại | Old generation pressure hoặc sizing sai |

Một contrast rất hay dùng để tránh chỉnh sai hướng:

| Triệu chứng | Phản xạ dễ sai | Cách nghĩ tốt hơn |
| --- | --- | --- |
| Heap gần đầy | Tăng `-Xmx` ngay | Hỏi thêm post-GC còn bao nhiêu, GC overhead bao nhiêu |
| Pause cao | Đổi collector ngay | Kiểm tra pause distribution, object lifetime, allocation burst |
| GC nhiều | Kết luận “heap thiếu” | Có thể workload đang tạo rác quá nhanh |
| Process bị OOMKilled | Chỉ nhìn heap chart | Phải nhìn cả native memory, metaspace, direct buffer, thread stack |

Điểm cốt lõi là: mỗi lần tuning nên có câu kỳ vọng rõ ràng, ví dụ “nếu tăng heap hoặc đổi pause target, P99 pause phải giảm nhưng post-GC baseline không được tiếp tục tăng”.

## Code example

```java
List<byte[]> cache = new ArrayList<>();

for (int i = 0; i < 1000; i++) {
    cache.add(new byte[1024 * 256]);
}
```

Nếu đoạn code này chạy theo burst hoặc cache giữ dữ liệu quá lâu, chỉ tăng heap có thể giúp app chịu được lâu hơn nhưng không sửa policy giữ object. Đây là lý do tuning phải đi cùng hiểu biết về object lifetime, không chỉ heap size.

## When to use / when NOT to use

Hãy dùng mental model này khi:

- đã có GC log, metrics, hoặc symptom runtime rõ ràng,
- app chạy đúng chức năng nhưng latency hoặc memory behavior không ổn,
- cần cấu hình JVM theo budget của container hoặc host.

Không nên tune chỉ vì nghe nói một flag nào đó “best practice cho mọi app”. Không có tuning phổ quát tách rời workload.

## How this connects to real Java projects

Spring Boot app thường chạy trong container, nên heap chỉ là một phần của tổng process memory. Ngoài heap còn có metaspace, direct memory, code cache, thread stack, native overhead, và đôi khi memory của thư viện ngoài JVM heap.

Vì vậy tune cho Spring service phải nhìn toàn process. Một app có thể chết vì `OOMKilled` dù heap chưa đầy nếu tổng memory vượt limit của container.

## Gotchas

- Tăng heap có thể giảm frequency nhưng tăng cost của collection.
- Average pause đẹp chưa nói gì nhiều về tail pause.
- Đổi nhiều flag cùng lúc gần như phá hỏng khả năng học từ kết quả.
- Không phân biệt `heap pressure` với `retention problem` rất dễ đưa ra tuning sai.

## Check yourself

- Vì sao tuning nên bắt đầu từ logs và metrics thay vì từ flag list?
- `Post-GC baseline` tăng dần gợi ý điều gì khác với chuyện heap chỉ tạm thời đầy?
- Khi nào tăng `-Xmx` là hợp lý, và khi nào chỉ đang che triệu chứng?
- Vì sao tuning trong container phải nhìn cả non-heap memory?
- “Change one thing at a time” giúp gì khi debug GC?

## Exercises

### Exercise 1: Recommend Heap Change

Độ khó: Easy

Đề bài:
Cho `maxHeapMb`, `peakUsedMb`, và `gcOverheadPercent`. Hãy trả về `"increase-heap"` nếu `peakUsedMb * 100 >= maxHeapMb * 90` và `gcOverheadPercent >= 15`; trả về `"investigate-leak"` nếu `peakUsedMb * 100 < maxHeapMb * 90` nhưng `gcOverheadPercent >= 15`; ngược lại trả về `"keep-current"`.

Ví dụ 1:

Đầu vào:
```text
maxHeapMb = 1024
peakUsedMb = 980
gcOverheadPercent = 18
```

Đầu ra:
```text
"increase-heap"
```

Giải thích:
Heap gần đầy và GC overhead cao cùng lúc, nên hướng xử lý đầu tiên là tăng heap budget.

Ràng buộc:

- `1 <= maxHeapMb <= 1048576`
- `0 <= peakUsedMb <= maxHeapMb`
- `0 <= gcOverheadPercent <= 100`

### Exercise 2: Count Young Gen Pressure Spikes

Độ khó: Medium

Đề bài:
Cho `int[] allocationBurstMb` và `int youngGenCapacityMb`. Một spike xảy ra khi `allocationBurstMb[i] > youngGenCapacityMb`. Hãy đếm số spike.

Ví dụ 1:

Đầu vào:
```text
allocationBurstMb = [64, 120, 80, 200]
youngGenCapacityMb = 100
```

Đầu ra:
```text
2
```

Giải thích:
Burst ở index `1` và `3` vượt sức chứa vùng trẻ.

Ràng buộc:

- `0 <= allocationBurstMb.length <= 100000`
- `0 <= allocationBurstMb[i] <= 1048576`
- `1 <= youngGenCapacityMb <= 1048576`

### Exercise 3: Find First Unsafe Pause Target

Độ khó: Medium

Đề bài:
Cho `int[] pauseTargetsMs` và `int[] observedPauseMs` có cùng độ dài. Hãy trả về index đầu tiên mà `observedPauseMs[i] > pauseTargetsMs[i]`. Nếu mọi observed pause đều đạt target, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
pauseTargetsMs = [50, 50, 50, 50]
observedPauseMs = [40, 48, 67, 45]
```

Đầu ra:
```text
2
```

Giải thích:
Pause ở index `2` vượt mục tiêu đầu tiên.

Ràng buộc:

- `0 <= pauseTargetsMs.length <= 100000`
- Hai mảng phải có cùng độ dài
- Nếu không có violation, trả về `-1`

## Links

- [[004-GC-algorithms]]
- [[006-JVM-flags]]
- [[008-heap-dump-and-analysis]]
- [Oracle GC Tuning Guide](https://docs.oracle.com/en/java/javase/21/gctuning/index.html)
- [Factors Affecting Garbage Collection Performance](https://docs.oracle.com/en/java/javase/21/gctuning/factors-affecting-garbage-collection-performance.html)
- [Unified JVM Logging, `-Xlog`](https://docs.oracle.com/en/java/javase/21/docs/specs/man/java.html#enable-logging-with-the-jvm-unified-logging-framework)
