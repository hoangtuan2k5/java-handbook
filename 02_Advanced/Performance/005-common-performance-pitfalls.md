# Common Performance Pitfalls

## What is it

`Common performance pitfalls` là các mẫu tưởng vô hại nhưng thường làm Java app chậm đi rõ rệt khi vào dữ liệu lớn hoặc traffic thật. Chúng không phải bug cú pháp, mà là bug về cost model, ví dụ allocation quá nhiều, N+1 queries, boxing không cần thiết, logging quá nặng, hoặc cache dùng sai cách.

Điểm khó là nhiều pitfall chỉ lộ ra khi scale tăng. Code vẫn đúng về mặt business, test vẫn pass, nhưng throughput giảm, latency tăng, GC nặng hơn, hoặc database bị bắn quá nhiều query.

## How I used to misunderstand it

Mình từng nghĩ performance problem chỉ đến từ thuật toán quá tệ như O(n²). Thực tế nhiều hệ thống chậm vì hàng chục chi phí nhỏ cộng lại, mỗi request không quá tệ nhưng nhân với traffic lớn thì thành bottleneck thật.

Mình cũng từng tối ưu vi mô rất sớm, trong khi bỏ qua pitfall lớn hơn như query lặp trong loop hoặc tạo object vô nghĩa ở hot path. Những lỗi kiểu này thường đáng sửa hơn nhiều so với tranh luận vài nanosecond giữa hai API.

Một hiểu nhầm nữa là thấy pitfall note rồi cố sửa hết. Danh sách pitfall chỉ nên giúp mình đặt giả thuyết nhanh hơn. Xác nhận cuối cùng vẫn nên đến từ profiling, benchmarking, hoặc measurement thực tế.

## How it actually works

Pitfall hiệu năng thường xuất hiện khi code che giấu chi phí thật. Một vòng lặp nhìn đơn giản nhưng bên trong gọi ORM lazy load nên sinh N+1 queries. Một dòng nối chuỗi trong loop tạo ra cả đống object tạm. Một cache tưởng giúp tăng tốc nhưng lại không có eviction nên giữ heap sống quá lâu.

Điểm chung là cost không nằm trên bề mặt API. Muốn tránh pitfall, mình phải nghĩ bằng mental model về allocation, I/O round trip, lock contention, object lifetime, và số lần công việc thực sự được lặp lại.

Khi đã có profile hoặc benchmark, pitfall note kiểu này giúp mình map từ triệu chứng sang nguyên nhân quen thuộc. Nó không thay thế profiling, nhưng giúp đặt giả thuyết đúng nhanh hơn.

### Symptom to smell map

| Triệu chứng | Smell nên nghĩ tới trước | Tool xác nhận hợp lý |
|---|---|---|
| Latency tăng khi dữ liệu lớn | thuật toán xấu, N+1, serialization nặng | profiler + query metrics |
| CPU tăng mạnh | regex, parsing, concat, boxing, stream pipeline ở hot path | CPU profile |
| GC tăng nhưng feature không đổi | object churn, cache sai, payload lớn | allocation + memory profile |
| DB bị bắn nhiều query lẻ | lazy loading, query trong loop | SQL log, APM, profiler |
| Benchmark đẹp nhưng app không nhanh hơn | đo sai tầng, bottleneck nằm chỗ khác | end-to-end measurement |

### Decision shortcut

```text
Thấy một performance smell
  |
  +--> Nó có nằm trên hot path không? ---------- không --> tạm thời để yên
  |
  +--> Mình đã có evidence chưa? ---------- chưa --> đo trước
  |
  +--> Smell này có lớn và lặp lại không? ---------- có --> ưu tiên sửa
```

## Code example

```java
String result = "";

for (String part : parts) {
    result += part;
}
```

Đoạn code này trông ngắn gọn nhưng tạo rất nhiều `String` trung gian khi `parts` lớn. Trong hot path, pitfall này vừa tăng CPU vừa tăng allocation pressure. `StringBuilder` thường là lựa chọn hợp lý hơn cho trường hợp nối chuỗi lặp.

Điểm đáng nhớ là đây không chỉ là pitfall về CPU. Nó còn là pitfall về memory churn. Nhiều performance smell thực ra đụng nhiều resource cùng lúc.

## When to use / when NOT to use

Dùng checklist các pitfall này khi:

- endpoint bắt đầu chậm sau khi dữ liệu tăng
- batch job đột nhiên ăn nhiều CPU hoặc memory hơn dự kiến
- bạn đã thấy profile nhưng cần map kết quả sang nguyên nhân quen thuộc

Không tối ưu mọi pitfall một cách mù quáng. Nếu code không nằm trên hot path hoặc chưa có evidence, đừng biến code đơn giản thành phức tạp chỉ vì sợ một cost nhỏ chưa chắc quan trọng.

## How this connects to real Java projects

Spring app gặp pitfall rất thường ở `N+1 queries` với `Hibernate`, conversion hoặc validation chạy lặp lại, excessive logging trong filter/interceptor, cache annotation dùng với key quá lớn, hoặc collection xử lý quá nhiều trong controller/service layer.

Nhiều pitfall còn bị che bởi abstraction. Ví dụ một getter trên entity có thể kích hoạt lazy load. Một `toString()` trong log debug có thể kéo theo serialize object graph lớn. Vì vậy performance review trong Spring nên nhìn cả framework behavior chứ không chỉ business method.

### Common Spring smells

| Smell | Hậu quả thường gặp |
|---|---|
| `N+1 queries` | request chậm, DB load tăng mạnh |
| logging debug trong loop | CPU tăng, I/O tăng, benchmark méo |
| `@Cacheable` key space quá lớn | memory pressure, GC nặng |
| mapping DTO lặp nhiều lớp | CPU và allocation tăng |
| stream / boxing ở hot path lớn | cost cộng dồn rõ khi scale |

## Gotchas

- N+1 query thường không lộ ra với dataset nhỏ nên rất dễ lọt code review.
- Cache không giới hạn có thể biến tối ưu thành memory leak.
- Excessive boxing hoặc stream pipeline không phải lúc nào cũng xấu, nhưng ở hot path lớn thì cost cộng dồn đáng kể.
- Logging mức debug trong vòng lặp lớn có thể làm benchmark và production latency sai khác hẳn.
- Smell list chỉ là hypothesis generator, không phải bản án cuối cùng.

## Check yourself

- Vì sao nhiều performance problem thực tế không đến từ một bug lớn mà đến từ chi phí nhỏ lặp lại?
- Khi nào một performance smell nên được bỏ qua tạm thời?
- Vì sao `N+1 queries` và string concat trong loop đều là ví dụ của “cost bị che bởi API”?
- Nếu benchmark đẹp nhưng production không nhanh hơn, bạn nên nghi điều gì?
- Trong Spring, vì sao lazy loading hoặc logging có thể biến một đoạn code nhìn vô hại thành bottleneck?

## Exercises

### Exercise 1: Find First N Plus One Endpoint

Độ khó: Easy

Đề bài:
Cho `String[] endpointNames`, `int[] queryCounts`, và `int queryLimit`. Hãy trả về tên endpoint đầu tiên có số query lớn hơn `queryLimit`. Nếu không có endpoint nào vượt ngưỡng, trả về chuỗi rỗng `""`.

Ví dụ 1:

Đầu vào:
```text
endpointNames = ["/users", "/orders", "/reports"]
queryCounts = [3, 18, 6]
queryLimit = 10
```

Đầu ra:
```text
"/orders"
```

Giải thích:
`/orders` là endpoint đầu tiên vượt ngưỡng query cho phép trên mỗi request.

Ràng buộc:

- `0 <= endpointNames.length <= 100000`
- Hai mảng phải có cùng độ dài
- `0 <= queryLimit <= 1000000`

### Exercise 2: Count Excessive String Concats

Độ khó: Medium

Đề bài:
Cho `int[] concatCountsPerLoop` và `int threshold`. Mỗi loop được xem là excessive nếu số lần nối chuỗi trong loop lớn hơn hoặc bằng `threshold`. Hãy đếm số loop excessive.

Ví dụ 1:

Đầu vào:
```text
concatCountsPerLoop = [1, 50, 3, 100]
threshold = 10
```

Đầu ra:
```text
2
```

Giải thích:
Loop thứ hai và thứ tư có số lần concatenation vượt ngưỡng đáng lo.

Ràng buộc:

- `0 <= concatCountsPerLoop.length <= 100000`
- `0 <= concatCountsPerLoop[i] <= 1000000`
- `0 <= threshold <= 1000000`

### Exercise 3: Classify Cache Misuse

Độ khó: Medium

Đề bài:
Cho ba cờ `unboundedCache`, `noEviction`, và `mutableValues`. Hãy trả về:

- `"high"` nếu có ít nhất hai cờ là `true`
- `"medium"` nếu có đúng một cờ là `true`
- `"low"` nếu cả ba cờ đều là `false`

Ví dụ 1:

Đầu vào:
```text
unboundedCache = true
noEviction = true
mutableValues = false
```

Đầu ra:
```text
"high"
```

Giải thích:
Có hai dấu hiệu mạnh cho thấy cache có thể gây memory pressure hoặc dữ liệu khó kiểm soát.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"low"`, `"medium"`, hoặc `"high"`
- Không cần mô phỏng cache thật

## Links

- [[001-profiling-tools]]
- [[002-memory-profiling]]
- [[003-CPU-profiling]]
- [[004-benchmarking-with-JMH]]
- [N+1 query problem](https://vladmihalcea.com/n-plus-1-query-problem/)
- [Spring Cache Abstraction](https://docs.spring.io/spring-framework/reference/integration/cache.html)
- [Java SE 21, `StringBuilder` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuilder.html)

