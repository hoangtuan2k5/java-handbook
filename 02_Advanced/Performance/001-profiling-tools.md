# Profiling Tools

## What is it

`Profiling tools` là nhóm công cụ giúp mình nhìn thấy chương trình Java đang tiêu tốn thời gian, CPU, memory, allocation, lock, hay I/O ở đâu trong runtime thật. Thay vì đoán bằng mắt hoặc chỉ nhìn log, profiler cho mình evidence về hành vi thực tế của process.

Điểm quan trọng nhất là: profiler không phải một tool duy nhất. Nó là một nhóm góc nhìn khác nhau lên cùng một hệ thống đang chạy.

`Java Flight Recorder (JFR)` mạnh về low-overhead runtime events. `async-profiler` mạnh về CPU, allocation, lock profiling theo kiểu sampling. `VisualVM` và `JDK Mission Control` tiện để mở snapshot, timeline, heap dump, và xem dữ liệu theo hướng trực quan.

### Quick distinction

| Câu hỏi | Tool / approach thường hợp | Vì sao |
|---|---|---|
| App chậm, chưa biết lý do | `JFR` | Overhead thấp, nhìn rộng nhiều loại signal |
| CPU cao, muốn flame graph | `async-profiler` | Sampling tốt, đọc hotspot nhanh |
| Nghi allocation churn | `async-profiler` allocation mode hoặc `JFR` allocation events | Thấy object nào đang được tạo nhiều |
| Nghi memory leak | Heap dump + `VisualVM` / MAT / JMC | Cần retained path, không chỉ CPU sample |
| Muốn mở dữ liệu theo timeline trực quan | `JDK Mission Control` / `VisualVM` | Dễ đọc cho điều tra ban đầu |

## How I used to misunderstand it

Mình từng nghĩ profiler là một nút bấm thần kỳ, mở lên là biết ngay bug ở đâu. Thực tế profiler chỉ cho tín hiệu. Người phân tích vẫn phải biết mình đang hỏi câu hỏi gì, ví dụ đang săn CPU hotspot, memory leak, startup bottleneck, hay lock contention.

Mình cũng từng nghĩ cứ attach profiler vào production là nguy hiểm. Điều đó chỉ đúng với một số tool hoặc mode có overhead cao. Có những lựa chọn như `JFR` hoặc `async-profiler` đủ nhẹ để dùng trong incident thật nếu mình hiểu trade-off.

Hiểu nhầm phổ biến nữa là trộn `profiling`, `benchmarking`, và `tuning` thành một việc.

- `profiling` trả lời: app đang tốn ở đâu
- `benchmarking` trả lời: implementation A hay B nhanh hơn trong workload đã kiểm soát
- `tuning` trả lời: sau khi đã có evidence, mình đổi config hoặc code gì để cải thiện

## How it actually works

Profiler không “đọc suy nghĩ” của JVM. Nó thu thập tín hiệu theo cơ chế cụ thể.

- `sampling profiler` lấy mẫu stack trace định kỳ rồi suy ra phần nào của code xuất hiện nhiều nhất
- `instrumentation profiler` chèn thêm hook vào method enter hoặc exit nên chi tiết hơn nhưng thường tốn overhead hơn
- event-based recording như `JFR` ghi lại các runtime event quan trọng với overhead thấp hơn nhiều tool instrumentation nặng

Vì cơ chế khác nhau, output cũng khác nhau.

- CPU profile trả lời: thread đang bận ở đâu
- allocation profile trả lời: object nào được tạo nhiều nhất
- memory analysis trả lời: object nào đang bị giữ lại quá lâu
- lock profile trả lời: thread chờ monitor hoặc `park()` ở đâu

Nếu dùng sai tool cho sai câu hỏi, mình có dữ liệu nhưng vẫn không ra kết luận đúng.

### Mental model: profiling flow

```text
Symptom
  |
  v
Question you are really asking
  |
  +--> CPU hotspot? ---------> CPU profiler / flame graph
  |
  +--> Too many objects? ----> Allocation profiling
  |
  +--> Memory not released? -> Heap dump / retained analysis
  |
  +--> Threads blocked? -----> Lock / thread profiling
  |
  +--> Need broad picture? --> JFR
```

Trong Java production, thứ quan trọng là chọn tool có overhead phù hợp, đo trong thời gian đủ dài, rồi đọc kết quả theo context hệ thống. Một flame graph cao không tự động có nghĩa là bug. Có khi đó chỉ là nơi công việc hợp lệ đang diễn ra.

## Code example

```java
List<String> payloads = new ArrayList<>();

for (int i = 0; i < 100_000; i++) {
    payloads.add("user-" + i);
}
```

Đoạn code này vừa tạo nhiều `String`, vừa tạo nhiều allocation ngắn hạn.

- `CPU profiler` có thể cho thấy thời gian nằm ở concatenation hoặc resize
- `allocation profiler` lại cho thấy object churn mới là tín hiệu đáng chú ý
- heap dump sau đó có thể cho thấy phần lớn object đã chết rồi, nghĩa là đây là churn chứ không phải leak

Cùng một đoạn code nhưng mỗi profiler cho một góc nhìn khác nhau.

## When to use / when NOT to use

Dùng profiling tools khi:

- service chậm nhưng chưa rõ là CPU, memory, lock, hay I/O
- benchmark ra số lạ và bạn cần evidence thay vì phỏng đoán
- production incident cần snapshot runtime càng gần sự thật càng tốt

Không dùng một profiler duy nhất cho mọi bài toán. Cũng không nên kết luận từ profile quá ngắn, profile lấy trong debug mode, hoặc profile lấy khi traffic không giống production.

## How this connects to real Java projects

Trong Spring Boot app, profiler rất hữu ích để phân biệt chậm ở controller, serialization, ORM, cache, proxy, hay downstream call. Ví dụ request chậm có thể không nằm ở business logic mà nằm ở `Jackson`, `Hibernate`, connection pool wait, hoặc excessive logging filter.

Khi chạy trên Spring, stack trace thường có nhiều proxy và framework frame. Điều đó không có nghĩa framework là thủ phạm. Mình phải nhìn tiếp xuống dưới xem CPU thực sự đang đốt ở query mapping, JSON conversion, reflection, hay code ứng dụng.

### Smell detection cheat sheet

| Symptom trong app Spring | Smell nên nghĩ tới trước |
|---|---|
| CPU tăng khi response lớn | `Jackson` serialization, DTO mapping, compression |
| Memory tăng dần sau traffic | cache không eviction, persistence context giữ entity quá lâu |
| Thread pool bận nhưng CPU không quá cao | blocking I/O, connection wait, lock contention |
| Startup chậm | classpath scan, bean creation, reflection, config loading |

## Gotchas

- Dùng sai profiler cho sai câu hỏi làm mình mất thời gian đọc dữ liệu không liên quan.
- Profiling quá ngắn dễ bỏ lỡ periodic spike hoặc warmup effect.
- Instrumentation nặng có thể làm workload đổi shape, từ đó kết quả không còn phản ánh runtime thật.
- Flame graph rộng không tự động là bug, đôi khi đó chỉ là đoạn code đang xử lý phần lớn công việc hợp lệ.
- Có evidence rồi mới tuning. Đừng đổi JVM flags hoặc viết lại code trước khi biết bottleneck nằm ở đâu.

## Check yourself

- Khi app chậm, vì sao nên phân biệt rõ giữa `profiling`, `benchmarking`, và `tuning`?
- Nếu nghi object churn nhưng chưa nghi leak, vì sao allocation profile hợp hơn heap dump đơn lẻ?
- Vì sao `JFR` thường là lựa chọn tốt cho điều tra production-safe ban đầu?
- Một flame graph rộng nói lên điều gì, và không tự động nói lên điều gì?
- Trong Spring Boot, vì sao framework frame đứng cao trên stack chưa chắc là thủ phạm thật?

## Exercises

### Exercise 1: Choose Profiler For Incident

Độ khó: Easy

Đề bài:
Cho `String symptom`, `boolean prodSafeRequired`, và `boolean needAllocationData`. Hãy trả về tên profiler phù hợp nhất theo luật sau:

- trả về `"jfr"` nếu `prodSafeRequired` là `true` và không cần allocation detail sâu
- trả về `"async-profiler"` nếu `needAllocationData` là `true`
- ngược lại trả về `"visualvm"`

Ví dụ 1:

Đầu vào:
```text
symptom = "latency spike"
prodSafeRequired = true
needAllocationData = false
```

Đầu ra:
```text
"jfr"
```

Giải thích:
Case này ưu tiên công cụ có overhead thấp và an toàn hơn cho runtime thật.

Ràng buộc:

- `symptom` có độ dài từ `0` đến `100`
- Chỉ cần áp dụng đúng luật chọn tool đã cho
- Không cần mô phỏng profiler thật

### Exercise 2: Count Actionable Profiles

Độ khó: Medium

Đề bài:
Cho `int[] durationsMillis`, `boolean[] capturedOnProdSafeSettings`, và `int minDurationMillis`. Một profile được xem là actionable nếu thời lượng capture lớn hơn hoặc bằng `minDurationMillis` và được thu trong mode production-safe. Hãy đếm số profile actionable.

Ví dụ 1:

Đầu vào:
```text
durationsMillis = [500, 5000, 1500, 300]
capturedOnProdSafeSettings = [true, false, true, true]
minDurationMillis = 1000
```

Đầu ra:
```text
1
```

Giải thích:
Chỉ profile thứ hai theo index `2` vừa đủ thời lượng vừa được capture bằng cấu hình an toàn.

Ràng buộc:

- `0 <= durationsMillis.length <= 100000`
- Hai mảng phải có cùng độ dài
- `0 <= minDurationMillis <= 1000000`

### Exercise 3: Find First Missing Observation

Độ khó: Medium

Đề bài:
Cho `String[] requiredSignals` và `String[] capturedSignals`. Hãy trả về tín hiệu đầu tiên trong `requiredSignals` chưa xuất hiện trong `capturedSignals`. Nếu tất cả đã có, trả về chuỗi rỗng `""`.

Ví dụ 1:

Đầu vào:
```text
requiredSignals = ["cpu", "allocation", "locks"]
capturedSignals = ["cpu", "locks"]
```

Đầu ra:
```text
"allocation"
```

Giải thích:
`allocation` là tín hiệu đầu tiên bắt buộc nhưng chưa được thu thập.

Ràng buộc:

- `0 <= requiredSignals.length <= 100000`
- `0 <= capturedSignals.length <= 100000`
- So sánh chuỗi theo đúng nội dung

## Links

- [[003-CPU-profiling]]
- [[002-memory-profiling]]
- [[004-benchmarking-with-JMH]]
- [[../JVM-Internals/003-JIT-compiler]]
- [[../JVM-Internals/005-GC-tuning]]
- [Java Flight Recorder overview](https://docs.oracle.com/en/java/javase/21/jfapi/introduction-jfr.html)
- [JDK Mission Control](https://www.oracle.com/java/technologies/jdk-mission-control.html)
- [async-profiler](https://github.com/async-profiler/async-profiler)

