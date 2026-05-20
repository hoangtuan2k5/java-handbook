# Facade

## What is it

`Facade` là structural pattern cung cấp một API đơn giản hơn đứng trước nhiều subsystem phức tạp.

Nó giải quyết bài toán caller phải biết quá nhiều bước hoặc quá nhiều service để hoàn thành một tác vụ business. Facade gom orchestration lặp lại vào một entry point dễ dùng hơn.

Điểm đáng học nhất là `Facade` giảm cognitive load cho consumer. Nó không xóa subsystem cũ, chỉ đặt thêm một cổng vào phù hợp hơn với use case.

### Quick distinction

| Câu hỏi | `Facade` trả lời thế nào |
|---|---|
| Caller có đang phải biết quá nhiều subsystem và thứ tự gọi không? | Có |
| Mục tiêu là đơn giản hóa API cho use case không? | Có |
| Lợi ích chính | orchestration rõ hơn, caller mỏng hơn |
| Giá phải trả | facade có thể phình thành God service |
| Pattern dễ nhầm | `Adapter` |

## How I used to misunderstand it

Mình từng nghĩ facade là một “God service” bọc tất cả mọi thứ. Thực ra facade tốt chỉ gom workflow hợp lý cho một use case cụ thể.

Nếu một class đứng trước mọi subsystem nhưng không có boundary use case rõ ràng, nó dễ biến thành nơi chứa đủ loại logic. Khi đó bạn có một lớp to hơn, không phải một facade tốt hơn.

## How it actually works

Facade expose một entry point rõ như `checkout(order)` hoặc `generateReport(criteria)`. Bên trong, nó điều phối nhiều subsystem theo thứ tự phù hợp rồi trả về kết quả gọn hơn cho caller.

Caller không cần biết từng subsystem gọi ra sao.

```text
caller
  -> facade.useCase(...)
       -> subsystem A
       -> subsystem B
       -> subsystem C
       -> result
```

### Decision matrix

| Tình huống | `Facade` hợp không? | Vì sao |
|---|---|---|
| Caller phải nhớ nhiều bước gọi lặp lại | Hợp | facade gom workflow |
| Chỉ cần translate contract cũ sang mới | Không | `Adapter` gần hơn |
| Chỉ có một subsystem đơn giản | Ít hợp | thêm facade không có payoff lớn |
| Muốn controller hoặc UI layer mỏng hơn | Hợp | facade giữ orchestration ở một chỗ |
| Facade bắt đầu ôm mọi branch domain | Cẩn thận | nguy cơ God object |

## Code example

```java
class CheckoutFacade {
    String placeOrder(String orderId) {
        reserveInventory(orderId);
        chargePayment(orderId);
        createShipment(orderId);
        return "PLACED:" + orderId;
    }

    private void reserveInventory(String orderId) {
    }

    private void chargePayment(String orderId) {
    }

    private void createShipment(String orderId) {
    }
}
```

Điểm chính là caller chỉ thấy `placeOrder(...)`, không phải tự điều phối ba subsystem theo đúng thứ tự.

## When to use / when NOT to use

Use `Facade` khi:

- caller phải biết quá nhiều subsystem hoặc thứ tự gọi
- có một use case rõ ràng cần orchestration lặp lại
- muốn tạo API thuận tiện hơn cho layer phía trên

Do NOT use `Facade` khi:

- subsystem chỉ có một bước đơn giản
- caller thực sự cần toàn quyền điều khiển từng bước thấp hơn
- mục tiêu thật sự chỉ là dịch một contract cũ sang contract mới

Misconception thường gặp là nghĩ facade phải đứng trên toàn bộ hệ thống. Thực tế facade tốt thường khá hẹp và bám sát một nhóm use case liên quan.

## How this connects to real Java projects

Trong Spring, facade thường là application service hoặc orchestration service đứng giữa controller và nhiều domain hoặc integration service.

Nó giúp controller mỏng hơn và giữ workflow use case ở một chỗ dễ đọc hơn. Đây là một pattern rất hay để dạy ranh giới giữa web layer và application orchestration.

## Gotchas

- Facade quá rộng dễ thành God object.
- Nếu facade nuốt hết lỗi từ subsystem, caller mất context để xử lý đúng.
- Đặt quá nhiều logic domain thấp trong facade làm subsystem nghèo và khó reuse.
- Facade nên đơn giản hóa API, không nên che giấu transaction boundary hoặc side effect quá mơ hồ.
- Một facade mà chẳng hề đơn giản hóa caller thì chỉ là lớp trung gian dư thừa.

## Check yourself

- `Facade` khác `Adapter` ở mục tiêu chính như thế nào?
- Dấu hiệu nào cho thấy controller đang cần một facade?
- Khi nào facade bắt đầu trượt thành God service?
- Vì sao facade tốt thường gắn với use case chứ không bọc toàn bộ hệ thống?
- Nếu caller vẫn phải biết thứ tự gọi subsystem sau khi có facade, pattern đã thất bại ở đâu?

## Exercises

### Exercise 1: Should Introduce Facade

Độ khó: Easy

Đề bài:
Cho `subsystemCount`, `repeatedCallOrder`, và `callersNeedSimpleApi`. Hãy trả về `true` khi có ít nhất 3 subsystem, thứ tự gọi bị lặp lại, và caller cần một API đơn giản hơn.

Ví dụ 1:

Đầu vào:
```text
subsystemCount = 4, repeatedCallOrder = true, callersNeedSimpleApi = true
```

Đầu ra:
```text
true
```

Giải thích:
Nhiều subsystem với orchestration bị lặp lại là dấu hiệu tốt cho thấy facade có thể hữu ích.

Ràng buộc:

- `subsystemCount` nằm trong đoạn từ 0 đến 1000
- Các input còn lại là boolean
- Trả về một boolean

### Exercise 2: Count Facade Saved Calls

Độ khó: Easy

Đề bài:
Cho `subsystemCount` và `callerCount`, giả sử mỗi caller lẽ ra sẽ gọi trực tiếp mọi subsystem. Hãy trả về số direct subsystem call được tránh đi khi mỗi caller giờ chỉ gọi đúng một facade call.

Ví dụ 1:

Đầu vào:
```text
subsystemCount = 4, callerCount = 3
```

Đầu ra:
```text
9
```

Giải thích:
Nếu không có facade thì sẽ có 12 direct call. Với facade thì chỉ có 3 facade call, nên tránh được 9 direct subsystem call.

Ràng buộc:

- `subsystemCount` nằm trong đoạn từ 0 đến 1000
- `callerCount` nằm trong đoạn từ 0 đến 1000
- Trả về một integer không âm

### Exercise 3: Build Checkout Facade Plan

Độ khó: Medium

Đề bài:
Cho `reserveInventory`, `chargePayment`, và `createShipment`. Hãy trả về một string theo đúng format `"RESERVE=<status>,PAY=<status>,SHIP=<status>"`, trong đó mỗi status là `ON` hoặc `OFF`.

Ví dụ 1:

Đầu vào:
```text
reserveInventory = true, chargePayment = true, createShipment = false
```

Đầu ra:
```text
"RESERVE=ON,PAY=ON,SHIP=OFF"
```

Giải thích:
Plan này tóm tắt facade sẽ orchestrate những subsystem nào cho request này.

Ràng buộc:

- Tất cả input đều là boolean
- Output format phải khớp chính xác
- Thứ tự luôn là reserve, rồi pay, rồi ship

## Links

- [[003-Adapter]]
- [[../../Testing/001-unit-test-vs-integration-test]]
- [Refactoring Guru, Facade](https://refactoring.guru/design-patterns/facade)
- [Spring Framework, Web MVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html)
- [Martin Fowler, Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html)
