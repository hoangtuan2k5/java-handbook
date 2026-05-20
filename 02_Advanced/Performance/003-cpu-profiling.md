# CPU Profiling

## What is it

`CPU profiling` là cách đo xem thread đang dùng CPU ở đâu, method nào xuất hiện nhiều nhất trong stack, và hotspot thật nằm ở tầng nào của ứng dụng. Nó giúp phân biệt code nào đang active compute với code nào chỉ đang chờ lock, I/O, hoặc sleep.

Trong Java, CPU profiling thường dùng `sampling` vì overhead thấp hơn và phản ánh runtime thật tốt hơn cho production-like workload. Kết quả hay được xem qua flame graph, call tree, top methods, hoặc per-thread timeline.

## How I used to misunderstand it

Mình từng nghĩ method đứng đầu danh sách profile là chỗ cần tối ưu ngay. Nhưng method đó có thể chỉ là caller tổng quát, còn vấn đề thật nằm sâu hơn trong stack hoặc ở branch hiếm nhưng rất đắt.

Mình cũng từng lẫn CPU time với request latency. Một request chậm chưa chắc đang dùng nhiều CPU. Nó có thể chờ database, lock, network, hoặc queue. CPU profile chỉ trả lời tốt khi bottleneck thực sự là computation hoặc spin.

Một hiểu nhầm nữa là thấy app chậm rồi mở đúng CPU profile là đủ. Nhiều case thực ra cần `wall-clock`, lock, hoặc I/O signal chứ không chỉ mỗi `RUNNABLE` stacks.

## How it actually works

Sampling CPU profiler lấy mẫu stack trace của thread theo chu kỳ. Nếu một method xuất hiện trong nhiều sample, khả năng cao nó đang chiếm nhiều thời gian CPU hơn. Đây là phép đo xác suất theo tần suất xuất hiện, không phải đồng hồ tuyệt đối cho từng dòng code.

Điểm mạnh của sampling là overhead thấp và ít làm méo workload. Điểm yếu là nếu profile quá ngắn hoặc sample rate không phù hợp, hotspot nhỏ hoặc không đều có thể bị bỏ lỡ. Vì vậy phải đo đủ lâu và đọc profile cùng context traffic.

Khi xem flame graph, chiều rộng thường biểu diễn tổng thời gian sample, không phải số lần method được gọi. Một method gọi ít nhưng rất đắt vẫn có thể rất rộng. Ngược lại, method được gọi nhiều nhưng cực ngắn chưa chắc nổi bật.

### CPU vs wall-clock vs lock signals

| Nếu bạn muốn biết | Tín hiệu phù hợp hơn | Vì sao |
|---|---|---|
| Thread đang đốt CPU ở đâu | CPU sampling profile | Tập trung vào compute hotspot |
| Request chậm vì chờ gì | wall-clock profile hoặc trace | Bao gồm cả thời gian chờ |
| Thread bị block ở đâu | lock / thread state profile | CPU profile có thể không cho thấy rõ blocking |
| App vừa CPU vừa allocation nặng | CPU + allocation profile | Nhiều hotspot thật ra là object churn |

### Reading a flame graph

```text
khung rộng ở gần đỉnh
  != không tự động có nghĩa là nhiều lời gọi
  == thường có nghĩa là chiếm nhiều sampled time

stack cao
  != không tự động có nghĩa là xấu
  == thường chỉ là call path sâu
```

## Code example

```java
long total = 0;

for (String value : values) {
    total += Long.parseLong(value.trim());
}
```

Nếu vòng lặp này chạy trên dataset lớn, CPU profiler có thể cho thấy thời gian tập trung ở `trim`, `parseLong`, hoặc allocation liên quan tới chuỗi trung gian. Mục tiêu là tìm chỗ CPU thực sự bị đốt thay vì đoán bằng cảm giác.

Nếu flame graph rộng ở parsing nhưng allocation profile cũng sáng lên, quyết định sửa có thể là giảm object churn chứ không chỉ rewrite thuật toán parse.

## When to use / when NOT to use

Dùng CPU profiling khi:

- process dùng CPU cao liên tục hoặc có spike lặp lại
- endpoint chậm vì compute, parsing, serialization, regex, hoặc compression
- bạn nghi có hotspot do algorithm hoặc object churn

Không mong CPU profiler giải thích đầy đủ các case chậm do blocking I/O, DB wait, hay thread contention nếu bạn không bật thêm góc nhìn phù hợp như lock hoặc wall-clock profile.

## How this connects to real Java projects

Trong Spring Boot service, CPU hotspot hay nằm ở JSON serialization, validation, mapping DTO, proxy chain quá dày, template rendering, hoặc custom data transformation. CPU profiler giúp bóc tách xem thời gian ở framework frame chỉ là đường đi hay chính converter, interceptor, hay application code mới là hotspot.

Nó cũng rất hữu ích với scheduled jobs và batch step trong Spring, nơi một vòng lặp lớn hoặc một parser tốn CPU có thể làm cả worker pod bị throttle dù số request không cao.

### Practical smell detection

| Symptom | Smell nên nghĩ tới |
|---|---|
| CPU tăng khi payload lớn | serialization, validation, mapping, regex |
| CPU cao nhưng DB không bận | application loop, parsing, compression |
| CPU thấp mà latency vẫn cao | blocking I/O, connection pool wait, lock contention |
| Flame graph nhiều `String` / `char[]` helper | text processing hoặc object churn |

## Gotchas

- Profile quá ngắn có thể bỏ lỡ hotspot chỉ xuất hiện theo đợt.
- Flame graph rộng không có nghĩa method đó bị gọi nhiều, mà thường là nó chiếm nhiều thời gian sample.
- CPU profile không thay thế lock profile hoặc wall-clock profile cho bài toán blocking.
- Chạy trong debug mode hoặc với logging quá dày có thể làm shape CPU khác production.
- Hotspot ở framework frame chưa chắc là lỗi framework. Phải nhìn xuống leaf frame và workload thật.

## Check yourself

- Vì sao request latency cao chưa chắc đồng nghĩa CPU hotspot?
- Một frame rộng trong flame graph thường biểu diễn điều gì?
- Khi nào nên dùng thêm lock profile hoặc wall-clock profile thay vì chỉ nhìn CPU?
- Vì sao sampling profile cần chạy đủ lâu?
- Nếu CPU hotspot đi cùng allocation hotspot, hướng tối ưu có thể khác gì so với chỉ nhìn compute thuần?

## Exercises

### Exercise 1: Find Hottest Stack Frame

Độ khó: Easy

Đề bài:
Cho `String[] frameNames` và `int[] sampleCounts`. Hãy trả về tên frame có số sample lớn nhất. Nếu có nhiều frame cùng số sample lớn nhất, trả về frame xuất hiện sớm nhất. Nếu mảng rỗng, trả về chuỗi rỗng `""`.

Ví dụ 1:

Đầu vào:
```text
frameNames = ["parse", "serialize", "compress"]
sampleCounts = [30, 80, 80]
```

Đầu ra:
```text
"serialize"
```

Giải thích:
`serialize` và `compress` cùng cao nhất, nhưng `serialize` xuất hiện trước.

Ràng buộc:

- `0 <= frameNames.length <= 100000`
- Hai mảng phải có cùng độ dài
- `0 <= sampleCounts[i] <= 100000000`

### Exercise 2: Count Blocking Hotspots

Độ khó: Medium

Đề bài:
Cho `String[] threadStates`, `int[] sampleCounts`, và `int minSamples`. Một hotspot blocking được tính khi `threadStates[i]` là `"BLOCKED"` hoặc `"WAITING"` và `sampleCounts[i] >= minSamples`. Hãy đếm số hotspot như vậy.

Ví dụ 1:

Đầu vào:
```text
threadStates = ["RUNNABLE", "WAITING", "BLOCKED", "TIMED_WAITING"]
sampleCounts = [300, 50, 120, 90]
minSamples = 100
```

Đầu ra:
```text
1
```

Giải thích:
Chỉ sample `BLOCKED` vượt ngưỡng và được tính là blocking hotspot.

Ràng buộc:

- `0 <= threadStates.length <= 100000`
- Hai mảng phải có cùng độ dài
- So sánh state theo đúng chuỗi đã cho

### Exercise 3: Detect Sampling Bias

Độ khó: Medium

Đề bài:
Cho ba cờ `tooShortRun`, `debugMode`, và `noWarmup`. Hãy trả về:

- `"high"` nếu có ít nhất hai cờ là `true`
- `"medium"` nếu có đúng một cờ là `true`
- `"low"` nếu cả ba cờ đều là `false`

Ví dụ 1:

Đầu vào:
```text
tooShortRun = true
debugMode = false
noWarmup = true
```

Đầu ra:
```text
"high"
```

Giải thích:
Có hai lý do khiến CPU profile dễ bị lệch khỏi hành vi steady state.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"low"`, `"medium"`, hoặc `"high"`
- Không cần mô phỏng profiler thật

## Links

- [[001-profiling-tools]]
- [[002-memory-profiling]]
- [[004-benchmarking-with-JMH]]
- [[../JVM-Internals/003-JIT-compiler]]
- [[../JVM-Internals/006-JVM-flags]]
- [async-profiler](https://github.com/async-profiler/async-profiler)
- [Brendan Gregg, Flame Graphs](https://www.brendangregg.com/flamegraphs.html)
- [Java Flight Recorder overview](https://docs.oracle.com/en/java/javase/21/jfapi/introduction-jfr.html)
