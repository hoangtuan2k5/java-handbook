# Heap Dump and Analysis

## What is it

`Heap dump` là ảnh chụp trạng thái heap tại một thời điểm. Nó giúp bạn nhìn object graph thật đang nằm trong memory, thay vì chỉ nhìn metric tổng quát như heap used hoặc GC count.

`Heap dump analysis` là quá trình dùng histogram, dominator tree, `retained size`, và GC roots để trả lời câu hỏi quan trọng nhất:

“Object nào đang giữ memory, và ai đang giữ nó sống?”

## How I used to misunderstand it

Mình từng nghĩ heap dump chỉ đáng tạo khi app đã nổ `OutOfMemoryError`.

Thực tế nếu đợi tới lúc đó mới nhìn thì nhiều khi đã quá muộn, file dump khó thu thập, hoặc môi trường production đã tự restart mất dấu vết. Heap dump hữu ích cả khi app chưa crash nhưng `post GC baseline` cứ tăng dần hoặc old generation cứ phình lên bất thường.

## How it actually works

Heap dump cho bạn snapshot của object graph, trong đó thường quan tâm nhất là:

| Góc nhìn | Dùng để trả lời gì |
| --- | --- |
| Histogram theo class | Type nào chiếm nhiều object hoặc nhiều bytes |
| Dominator tree | Nhánh nào giữ phần memory lớn nhất |
| GC roots path | Từ đâu mà object còn reachable |
| `Shallow size` vs `retained size` | Object tự nó to, hay nó chỉ là “nắp chai” giữ cả graph lớn |

Một flow phân tích leak rất hay dùng:

```text
Heap usage rises
      |
      v
Take dump
      |
      v
Check biggest retained components
      |
      v
Follow GC root path
      |
      v
Find owner: cache, static field, ThreadLocal, session, listener...
```

Contrast cực quan trọng:

| Khái niệm | Nghĩa là gì |
| --- | --- |
| `Shallow size` | Memory của chính object đó |
| `Retained size` | Tổng memory sẽ được giải phóng nếu object đó và phần graph chỉ reachable qua nó biến mất |

Đây là chỗ rất nhiều người kết luận sai. Một object có `shallow size` nhỏ vẫn có thể là dominator cực lớn nếu nó giữ cả collection hoặc graph lớn phía sau.

## Code example

```java
Map<String, byte[]> cache = new HashMap<>();

for (int i = 0; i < 10_000; i++) {
    cache.put("user-" + i, new byte[1024]);
}
```

Nếu `cache` này sống cùng application và không có eviction, heap dump thường cho thấy một dominator như `HashMap`, bean singleton chứa map đó, hoặc một root path đi từ Spring context tới collection này.

Điều quan trọng không phải chỉ là “nhiều `byte[]`”, mà là “ai đang giữ những `byte[]` đó sống”.

## When to use / when NOT to use

Hãy dùng heap dump khi:

- heap sau GC không quay về baseline cũ,
- old generation tăng dần theo thời gian,
- nghi ngờ memory leak hoặc retention bất thường,
- cần biết collection, cache, hoặc object graph nào đang giữ memory.

Không nên tạo heap dump lớn bừa bãi trên production nhạy latency nếu chưa hiểu chi phí disk, thời gian dừng, và rủi ro dữ liệu nhạy cảm trong dump.

## How this connects to Spring

Spring apps dễ giữ memory ở singleton bean, cache không giới hạn, request/session data sống quá lâu, Hibernate persistence context, listener, scheduler, hoặc `ThreadLocal` trên thread pool.

Heap dump rất mạnh trong bối cảnh này vì nó giúp nối symptom “memory tăng” với chính bean hoặc collection đang dominator memory, thay vì chỉ kết luận chung chung là “Spring app bị leak”.

## Gotchas

- Dump có thể rất lớn và chứa dữ liệu thật của người dùng hoặc secrets.
- Nhìn `shallow size` mà bỏ qua `retained size` rất dễ chẩn đoán sai.
- Một snapshot duy nhất chưa chắc đủ, nhất là khi cần so sánh trước và sau load.
- `HeapDumpOnOutOfMemoryError` hữu ích, nhưng phải chuẩn bị sẵn disk, path, và quy trình bảo mật.

## Check yourself

- Vì sao heap dump thường hữu ích trước cả khi app nổ `OutOfMemoryError`?
- `Retained size` khác gì với `shallow size`?
- Khi thấy nhiều object lớn, vì sao câu hỏi tiếp theo phải là “ai giữ chúng sống”?
- Dominator tree giúp gì mà histogram đơn thuần không cho thấy hết?
- Nếu chạy trong production, bạn cần nghĩ thêm gì ngoài chuyện kỹ thuật phân tích?

## Exercises

### Exercise 1: Find Largest Retained Component

Độ khó: Easy

Đề bài:
Cho `String[] componentNames` và `int[] retainedSizesMb`. Hãy trả về tên component có `retained size` lớn nhất. Nếu có nhiều component cùng lớn nhất, trả về component xuất hiện sớm nhất. Nếu mảng rỗng, trả về chuỗi rỗng `""`.

Ví dụ 1:

Đầu vào:
```text
componentNames = ["cache", "session-store", "report-buffer"]
retainedSizesMb = [120, 80, 120]
```

Đầu ra:
```text
"cache"
```

Giải thích:
`cache` và `report-buffer` cùng lớn nhất, nhưng `cache` xuất hiện trước.

Ràng buộc:

- `0 <= componentNames.length <= 100000`
- Hai mảng phải có cùng độ dài
- Nếu rỗng, trả về `""`

### Exercise 2: Count Suspicious Duplicate Strings

Độ khó: Medium

Đề bài:
Cho `String[] values` và `int minOccurrences`. Một string bị xem là suspicious duplicate nếu xuất hiện ít nhất `minOccurrences` lần. Hãy đếm có bao nhiêu giá trị khác nhau thỏa điều kiện này.

Ví dụ 1:

Đầu vào:
```text
values = ["prod", "user", "prod", "prod", "cache", "user"]
minOccurrences = 2
```

Đầu ra:
```text
2
```

Giải thích:
`"prod"` và `"user"` là hai giá trị khác nhau xuất hiện đủ nhiều lần.

Ràng buộc:

- `0 <= values.length <= 100000`
- `1 <= minOccurrences <= 100000`
- So sánh string phân biệt hoa thường

### Exercise 3: Detect Leak Trend

Độ khó: Medium

Đề bài:
Cho `int[] postGcUsageMb` là heap usage ngay sau mỗi lần GC toàn cục quan sát được. Hãy trả về `true` nếu dãy này tăng nghiêm ngặt ở mọi bước và có ít nhất 2 điểm dữ liệu, ngược lại trả về `false`.

Ví dụ 1:

Đầu vào:
```text
postGcUsageMb = [300, 340, 390, 450]
```

Đầu ra:
```text
true
```

Giải thích:
Baseline sau GC cứ tăng dần, đây là một tín hiệu đơn giản cho leak trend.

Ràng buộc:

- `0 <= postGcUsageMb.length <= 100000`
- `0 <= postGcUsageMb[i] <= 1048576`
- Dãy có dưới 2 phần tử thì trả về `false`

## Links

- [[../../01_Core/Memory/001-GC]]
- [[../../01_Core/Memory/002-Memory-leak]]
- [[005-GC-tuning]]
- [Oracle Troubleshooting Guide, Troubleshooting Memory Leaks](https://docs.oracle.com/en/java/javase/21/troubleshoot/troubleshooting-memory-leaks.html)
- [Eclipse Memory Analyzer (MAT)](https://eclipse.dev/mat/)
- [`jcmd` Tool Guide](https://docs.oracle.com/en/java/javase/21/docs/specs/man/jcmd.html)
