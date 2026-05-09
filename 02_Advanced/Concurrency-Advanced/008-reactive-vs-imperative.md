# Reactive vs Imperative

## What is it

Imperative style diễn tả từng bước xử lý theo flow tuần tự, rất hợp khi logic ngắn, blocking style dễ reason, và team muốn nhìn thấy câu chuyện control flow ngay trên code.

Reactive style mô hình hóa dữ liệu như stream sự kiện. Bạn khai báo pipeline bằng publisher, operator, subscriber, và để runtime điều phối việc thực thi async, non-blocking, cùng backpressure.

Điểm khác cốt lõi không nằm ở cú pháp. Nó nằm ở cách mình reason về:

- khi nào code thật sự chạy,
- ai giữ thread trong lúc chờ I/O,
- hệ thống phản ứng thế nào khi producer nhanh hơn consumer.

## How I used to misunderstand it

Mình từng nghĩ reactive chỉ là imperative code viết bằng API khó đọc hơn.

Sai ở chỗ reactive sinh ra để giải những workload mà request-per-thread model gặp áp lực, nhất là async I/O, streaming, high concurrency, và backpressure.

Hiểu nhầm ngược lại cũng nguy hiểm: tưởng reactive luôn hiện đại hơn nên nên áp dụng cho mọi endpoint CRUD. Nếu downstream vẫn blocking và team chưa quen async debugging, reactive có thể làm complexity tăng nhanh hơn lợi ích.

## How it actually works

Imperative flow thường là gọi hàm, chờ kết quả, rồi đi tiếp. Reactive flow thì mô tả pipeline trước, và nhiều khi **chưa làm gì cả** cho tới khi có subscription.

Khi pipeline chạy, runtime cố tránh giữ một thread ngồi chờ I/O vô ích. Thay vào đó, nó phối hợp non-blocking I/O, callback, scheduler, hoặc continuation để xử lý nhiều việc hơn với ít thread hơn.

### Mental model so sánh

| Câu hỏi | Imperative | Reactive |
|---|---|---|
| Cách đọc code | Giống từng bước thực thi | Giống mô tả pipeline |
| Khi nào logic chạy | Thường chạy ngay lúc gọi | Thường lazy, chạy khi subscribe |
| Quan hệ với blocking I/O | Tự nhiên hơn | Cần tránh hoặc cô lập |
| Backpressure | Không phải đặc tính mặc định | Là phần rất quan trọng |
| Debugging | Thường dễ theo stack hơn | Khó hơn nếu chưa quen async chain |

### Khi nào reactive đáng giá

| Tín hiệu | Ý nghĩa |
|---|---|
| Nhiều concurrent request chờ async I/O | Reactive có thể tận dụng thread tốt hơn |
| Cần streaming hoặc SSE | Reactive diễn tả tự nhiên hơn |
| Cần backpressure rõ ràng | Reactive mạnh hơn |
| Downstream toàn blocking | Lợi ích reactive giảm mạnh |

### Decision matrix nhỏ

| Nếu bạn cần... | Chọn gì trước |
|---|---|
| CRUD đơn giản, flow rõ, blocking stack | Imperative |
| Streaming, async pipeline, backpressure | Reactive |
| Nhiều blocking task độc lập nhưng muốn code dễ đọc | Xem thêm virtual threads |

## Code example

```java
import reactor.core.publisher.Flux;

Flux<String> names = Flux.just("java", "spring")
        .map(String::toUpperCase)
        .filter(value -> value.startsWith("J"));
```

Đoạn này mô tả pipeline. Nó chưa đại diện cho “đã xử lý xong dữ liệu”, mà mới là mô tả cách xử lý khi có subscriber.

## When to use / when NOT to use

Dùng reactive khi:

- có nhiều async I/O,
- cần streaming,
- cần backpressure giữa producer và consumer,
- team sẵn sàng debug async chain.

Không dùng chỉ để theo trend, nhất là khi:

- downstream phần lớn là blocking,
- app chủ yếu là CRUD đơn giản,
- team chưa quen mental model lazy execution và scheduler.

## How this connects to Spring

Trong Spring, đây là vùng của WebFlux, Reactor `Mono` và `Flux`, SSE, RSocket, và nhiều integration non-blocking khác.

Nhưng nếu app vẫn dùng JDBC blocking, gọi HTTP client blocking, hoặc phụ thuộc thư viện không non-blocking, ép reactive nửa vời thường chỉ đổi cú pháp chứ không đổi bottleneck. Lúc đó imperative hoặc virtual threads đôi khi thực tế hơn nhiều.

## Gotchas

- Trộn blocking call vào reactive pipeline mà không cô lập đúng scheduler sẽ phá lợi ích non-blocking.
- Reactive chain có thể khó debug hơn vì call stack không kể câu chuyện tuyến tính như imperative.
- Lazy execution làm nhiều người đọc code xong tưởng đã chạy rồi, trong khi thực tế chưa có subscription.
- Backpressure chỉ mạnh khi cả pipeline thật sự tôn trọng nó.

## Check yourself

- Reactive khác imperative chủ yếu ở cú pháp hay ở execution model?
- Vì sao lazy execution là một mental shift lớn?
- Khi downstream chủ yếu blocking, lợi ích reactive giảm ở đâu?
- Backpressure giải bài toán gì mà imperative style mặc định không cho sẵn?
- Khi nào virtual threads là lựa chọn dễ bảo trì hơn reactive?

## Exercises

### Exercise 1: Choose Processing Style

Độ khó: Easy

Đề bài:
Cho ba cờ `manyConcurrentRequests`, `mostlyBlockingCalls`, `needsBackpressure`. Trả về `"reactive"` nếu cần xử lý nhiều request đồng thời, không chủ yếu blocking, và cần backpressure. Ngược lại trả về `"imperative"`.

Ví dụ 1:

Đầu vào:
```text
manyConcurrentRequests = true
mostlyBlockingCalls = false
needsBackpressure = true
```

Đầu ra:
```text
"reactive"
```

Giải thích:
Đây là dấu hiệu của workload phù hợp với reactive pipeline.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"reactive"` hoặc `"imperative"`
- Không xét framework cụ thể

### Exercise 2: Count Backpressure Overflow

Độ khó: Medium

Đề bài:
Cho `producedPerTick`, `consumedPerTick`, và `bufferCapacity`. Mỗi tick, producer đẩy thêm phần tử vào buffer, consumer lấy bớt ra, và mọi phần tử vượt quá `bufferCapacity` sẽ bị drop ngay. Trả về tổng số phần tử bị drop.

Ví dụ 1:

Đầu vào:
```text
producedPerTick = [5, 2]
consumedPerTick = [1, 3]
bufferCapacity = 4
```

Đầu ra:
```text
2
```

Giải thích:
Tick đầu tạo overflow `1`, tick sau tiếp tục overflow `1`, tổng cộng `2` phần tử bị drop.

Ràng buộc:

- `1 <= producedPerTick.length == consumedPerTick.length <= 100000`
- `0 <= producedPerTick[i], consumedPerTick[i] <= 1000000`
- `0 <= bufferCapacity <= 1000000`

### Exercise 3: Detect Blocking Stage Index

Độ khó: Easy

Đề bài:
Cho mảng `stageKinds`, mỗi phần tử là `"non-blocking"` hoặc `"blocking"`. Trả về index đầu tiên của stage blocking. Nếu không có, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
stageKinds = ["non-blocking", "non-blocking", "blocking"]
```

Đầu ra:
```text
2
```

Giải thích:
Stage thứ ba là điểm đầu tiên làm reactive pipeline mất giả định non-blocking.

Ràng buộc:

- `1 <= stageKinds.length <= 100000`
- Mỗi phần tử chỉ là `"non-blocking"` hoặc `"blocking"`
- Không cần mô phỏng scheduler

## Links

- [[009-virtual-threads]]
- [[../../01_Core/Multithreading/008-completable-future]]
- [Spring Framework Reference, WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux.html)
- [Project Reactor Reference Guide](https://projectreactor.io/docs/core/release/reference/)
- [Reactive Streams Specification](https://www.reactive-streams.org/)
