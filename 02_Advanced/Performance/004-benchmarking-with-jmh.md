# Benchmarking with JMH

## What is it

`JMH` là `Java Microbenchmark Harness`, thư viện chuẩn để viết microbenchmark cho JVM. Nó giúp mình đo performance của một đoạn Java code theo cách ít bị lừa hơn bởi warmup, JIT, dead code elimination, constant folding, và nhiều tối ưu runtime khác.

Nó đặc biệt hữu ích khi mình muốn so sánh hai implementation nhỏ, ví dụ hai cách parse chuỗi, hai cấu trúc dữ liệu, hoặc hai chiến lược tạo object. Với JVM, đo bằng `System.nanoTime()` trong một vòng lặp thủ công rất dễ sai, còn JMH sinh ra để xử lý chính các bẫy đó.

## How I used to misunderstand it

Mình từng nghĩ benchmark chỉ cần chạy method nhiều lần rồi chia trung bình là đủ. Thực tế cách đó thường đo cả warmup, đo code bị JIT tối ưu mất, hoặc đo một workload không hề giống production.

Mình cũng từng xem chênh vài phần trăm trong benchmark là chân lý. Với JVM, benchmark phải có fork, warmup, measurement ổn định, và phải đọc kết quả cùng độ biến thiên. Nếu không, số đẹp chỉ là nhiễu được trình bày rất gọn.

Một nhầm lẫn lớn nữa là dùng benchmark để trả lời câu hỏi đáng ra thuộc về profiler. Benchmark trả lời implementation A hay B nhanh hơn trong setup đã kiểm soát. Nó không tự nói production đang chậm ở đâu.

## How it actually works

JMH tạo harness riêng để chạy benchmark dưới điều kiện kiểm soát. Nó tách warmup và measurement, có thể chạy nhiều `fork`, cô lập state, và cung cấp `Blackhole` để tránh compiler xóa mất đoạn code vì kết quả không được dùng.

Warmup rất quan trọng vì JIT cần thời gian để quan sát và tối ưu code. Fork cũng quan trọng vì nó giúp giảm ảnh hưởng từ trạng thái JVM trước đó. Measurement mode như throughput hay average time giúp mình chọn đúng cách đọc số theo câu hỏi đang hỏi.

Điểm cốt lõi là benchmark phải phản ánh bài toán thật. Nếu microbenchmark quá nhỏ, quá synthetic, hoặc tách rời object graph và traffic thật, kết quả có thể đúng về mặt harness nhưng vô ích cho production decision.

### Profiling vs benchmarking vs tuning

| Hoạt động | Câu hỏi chính | Output điển hình |
|---|---|---|
| Profiling | App đang tốn ở đâu? | flame graph, allocation view, heap dump |
| Benchmarking | A hay B nhanh hơn trong workload kiểm soát? | throughput, average time, percentiles |
| Tuning | Sau khi có evidence, mình đổi gì để cải thiện? | config change, code change, rerun measurement |

### JMH pitfall checklist

| Pitfall | Vì sao nguy hiểm | Dấu hiệu |
|---|---|---|
| Không warmup đủ | Đang đo code chưa vào steady state | run đầu chậm bất thường |
| Không fork | Bị nhiễu từ JVM state trước đó | kết quả đổi mạnh giữa benchmark cases |
| Không dùng result hoặc `Blackhole` | code có thể bị dead code elimination | benchmark quá đẹp một cách đáng ngờ |
| Input quá synthetic | số đẹp nhưng không giúp production | implementation thắng benchmark nhưng app không nhanh hơn |
| So chênh rất nhỏ | đang đọc nhiễu như tín hiệu | variance lớn, score overlap |

### Benchmark reading flow

```text
Có cần so sánh implementation A với B không?
  |
  +--> Có thể cô lập pure JVM logic không? --> JMH phù hợp
  |
  +--> Cần full endpoint / DB / network? --> dùng load test, không phải JMH
```

## Code example

```java
@Benchmark
public int sumLengths() {
    int total = 0;
    for (String value : values) {
        total += value.length();
    }
    return total;
}
```

JMH sẽ lo phần warmup, measurement, fork, và cách gọi method benchmark. Phần mình cần nghĩ kỹ là state đầu vào có thực tế không và kết quả trả về có đang bị compiler tối ưu mất hay không.

Nếu benchmark này dùng dataset quá nhỏ hoặc luôn cùng một input đơn giản, kết quả có thể che mất chi phí thật của branch prediction, allocation, cache locality, hoặc parsing logic trong workload production.

## When to use / when NOT to use

Dùng JMH khi:

- bạn so sánh hai implementation Java nhỏ hoặc trung bình
- bạn cần kiểm tra tác động của allocation, parsing, collection choice, hoặc concurrency primitive
- bạn muốn có benchmark JVM đáng tin hơn vòng lặp tự viết

Không dùng JMH để thay thế load test toàn hệ thống. Cũng không nên lấy kết quả microbenchmark rồi suy ra trực tiếp endpoint latency nếu benchmark không chứa các chi phí thật như serialization, network, DB, hoặc framework lifecycle.

## How this connects to Spring

Trong hệ Spring, JMH hữu ích khi tách một đoạn logic thuần Java ra khỏi framework để đo riêng, ví dụ mapper, validator, parser, cache key builder, hoặc custom serializer. Nó giúp trả lời xem bản thân đoạn code có đáng tối ưu không trước khi nghi ngờ cả framework.

Ngược lại, nếu mục tiêu là hiểu HTTP throughput hay end-to-end latency của Spring Boot app, JMH không thay thế được load test. Nó chỉ đo rất tốt một lát cắt hẹp của logic JVM.

### Practical smell detection

| Tình huống | Dùng gì trước |
|---|---|
| So sánh hai hàm parse thuần Java | `JMH` |
| Endpoint chậm chưa rõ lý do | profiler trước |
| Muốn biết app chịu bao nhiêu request thật | load test |
| Benchmark thắng nhưng app không nhanh hơn | quay lại profile toàn hệ thống |

## Gotchas

- Benchmark không có warmup thường đo nhầm trạng thái chưa được JIT tối ưu.
- Không dùng `Blackhole` hoặc giá trị trả về có thể khiến code bị dead code elimination.
- Chạy không fork dễ bị ảnh hưởng bởi trạng thái JVM từ benchmark trước.
- Microbenchmark đúng chưa chắc đồng nghĩa production cải thiện, vì bottleneck hệ thống có thể nằm ở nơi khác.
- Chênh lệch nhỏ mà variance lớn thường không đủ để ra quyết định kỹ thuật lớn.

## Check yourself

- Vì sao `JMH` phù hợp để so sánh implementation A và B, nhưng không tự trả lời production đang chậm ở đâu?
- Warmup và fork đang bảo vệ benchmark khỏi những loại sai lệch nào?
- Khi nào một microbenchmark đúng về mặt kỹ thuật nhưng vẫn vô ích cho quyết định production?
- Vì sao `Blackhole` hoặc giá trị trả về có thể quan trọng trong benchmark?
- Nếu endpoint Spring Boot chậm vì DB wait, vì sao `JMH` không phải câu trả lời đầu tiên?

## Exercises

### Exercise 1: Count Warmup Violations

Độ khó: Easy

Đề bài:
Cho `int[] warmupIterations` và `int minWarmup`. Mỗi benchmark case được xem là vi phạm nếu số warmup iteration nhỏ hơn `minWarmup`. Hãy đếm số case vi phạm.

Ví dụ 1:

Đầu vào:
```text
warmupIterations = [5, 10, 2, 8]
minWarmup = 5
```

Đầu ra:
```text
1
```

Giải thích:
Chỉ case có `2` warmup iteration thấp hơn ngưỡng yêu cầu.

Ràng buộc:

- `0 <= warmupIterations.length <= 100000`
- `0 <= warmupIterations[i] <= 100000`
- `0 <= minWarmup <= 100000`

### Exercise 2: Compare Benchmark Scores

Độ khó: Medium

Đề bài:
Cho `double throughputA`, `double throughputB`, và `double regressionThresholdPercent`. Nếu `throughputB` thấp hơn `throughputA` ít nhất `regressionThresholdPercent` phần trăm, trả về `"regression"`. Nếu `throughputB` cao hơn `throughputA` ít nhất ngưỡng đó, trả về `"improvement"`. Ngược lại trả về `"similar"`.

Ví dụ 1:

Đầu vào:
```text
throughputA = 1000.0
throughputB = 880.0
regressionThresholdPercent = 10.0
```

Đầu ra:
```text
"regression"
```

Giải thích:
`throughputB` thấp hơn `throughputA` 12 phần trăm, vượt ngưỡng regression.

Ràng buộc:

- `0.0 <= throughputA, throughputB <= 1000000000.0`
- `0.0 <= regressionThresholdPercent <= 100.0`
- Chỉ trả về `"regression"`, `"improvement"`, hoặc `"similar"`

### Exercise 3: Validate Benchmark Setup

Độ khó: Medium

Đề bài:
Cho ba cờ `usesBlackhole`, `forksEnabled`, và `measuresSteadyState`. Hãy trả về `"valid"` nếu cả ba cờ là `true`. Nếu đúng hai cờ là `true`, trả về `"warning"`. Ngược lại trả về `"invalid"`.

Ví dụ 1:

Đầu vào:
```text
usesBlackhole = true
forksEnabled = false
measuresSteadyState = true
```

Đầu ra:
```text
"warning"
```

Giải thích:
Benchmark setup có một thiếu sót quan trọng nhưng chưa hỏng hoàn toàn.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"valid"`, `"warning"`, hoặc `"invalid"`
- Không cần chạy benchmark thật

## Links

- [[001-profiling-tools]]
- [[003-CPU-profiling]]
- [[../JVM-Internals/003-JIT-compiler]]
- [OpenJDK JMH project](https://openjdk.org/projects/code-tools/jmh/)
- [JMH samples](https://github.com/openjdk/jmh/tree/master/jmh-samples)
- [JMH source repository](https://github.com/openjdk/jmh)
