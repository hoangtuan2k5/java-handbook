# Factory

## What is it

`Factory` là creational pattern dùng để tách logic tạo object ra khỏi nơi sử dụng object đó.

Thay vì caller tự `new` đúng implementation, factory nhận một input như type, channel, mode, hoặc config rồi quyết định object nào cần tạo. Nó giải quyết bài toán construction branching.

Điểm đáng học nhất là factory gom creation policy vào một chỗ. Caller không phải biết từng concrete class nên ít bị buộc chặt vào chi tiết khởi tạo hơn.

### Quick distinction

| Câu hỏi | `Factory` trả lời thế nào |
|---|---|
| Có nhiều implementation cho cùng một contract không? | Thường có |
| Caller có nên tự `new` từng implementation không? | Không nên lặp ở nhiều nơi |
| Lợi ích chính | gom rule chọn implementation vào một chỗ |
| Giá phải trả | thêm lớp gián tiếp |
| Khi dễ nhầm | nhầm với mọi method trả object |

## How I used to misunderstand it

Mình từng nghĩ mọi method trả object đều là factory pattern. Không hẳn.

Điểm đáng chú ý của pattern là nó gom rule chọn implementation vào một chỗ thay vì rải ở nhiều caller. Nếu chỉ có một helper method `new Something()` mà không có creation policy nào đáng kể, đó chưa chắc là pattern đáng nói.

## How it actually works

Factory nhận một tín hiệu đầu vào, sau đó map tín hiệu đó sang implementation phù hợp. Caller chỉ biết contract chung như `NotificationSender` hoặc `PaymentProcessor`.

Từ góc nhìn caller, object được dùng mà không cần biết khởi tạo như thế nào.

```text
caller
  -> factory.create(key)
        -> chọn concrete implementation
        -> trả object theo contract chung
```

### Factory vs close alternatives

| Nếu câu hỏi của bạn là... | Pattern gần hơn |
|---|---|
| "Tôi cần chọn object đúng loại để tạo" | `Factory` |
| "Tôi cần tạo cả họ object tương thích với nhau" | `Abstract Factory` |
| "Tôi cần lắp ráp nhiều field cho một object" | `Builder` |

## Code example

```java
interface NotificationSender {
    String send(String message);
}

class SenderFactory {
    static NotificationSender create(String channel) {
        return switch (channel) {
            case "email" -> message -> "EMAIL:" + message;
            case "sms" -> message -> "SMS:" + message;
            default -> throw new IllegalArgumentException("Unsupported channel");
        };
    }
}
```

Ý chính ở đây không phải cú pháp `switch`, mà là caller không cần biết class nào tạo ra sender thật.

## When to use / when NOT to use

Use `Factory` khi:

- creation logic phụ thuộc vào type, config, hoặc feature flag
- cùng một contract có nhiều implementation
- muốn gom rule tạo object về một chỗ dễ đổi và dễ test hơn

Do NOT use `Factory` khi:

- chỉ có một implementation ổn định
- creation logic quá đơn giản, caller `new` trực tiếp vẫn rõ
- mục tiêu thật sự là tạo cả family object chứ không phải một object đơn lẻ

Misconception cần sửa là nghĩ factory luôn làm code “enterprise hơn”. Nếu không có creation branching đáng kể, pattern chỉ thêm một lớp vòng vo.

## How this connects to real Java projects

Trong Spring, bạn có thể dùng `@Bean` method, configuration class, factory bean, hoặc registry bean để chọn implementation theo property, profile, hoặc request type.

Spring cũng làm nổi bật một bài học hay: nhiều khi factory nằm ở container configuration chứ không nằm trong business code.

## Gotchas

- Factory lớn dần thành `switch` khổng lồ nếu không có registry hoặc nhóm hợp lý.
- Nếu caller vẫn phải biết quá nhiều type string, coupling chỉ chuyển chỗ.
- Trả `null` khi không tạo được object thường làm lỗi xuất hiện muộn.
- Tạo object nặng mỗi lần gọi có thể tốn chi phí không cần thiết.
- Đặt tên factory quá generic như `ManagerFactory` hoặc `CommonFactory` làm intent rất mờ.

## Check yourself

- Dấu hiệu nào cho thấy caller đang ôm creation logic quá nhiều?
- `Factory` khác `Builder` ở mục tiêu chính như thế nào?
- Khi nào `Abstract Factory` mới đáng dùng hơn `Factory`?
- Vì sao gom creation policy về một chỗ giúp giảm bug khi thêm implementation mới?
- Nếu factory trả `null` cho trường hợp unsupported, bạn đang mở ra rủi ro gì?

## Exercises

### Exercise 1: Select Notification Sender

Độ khó: Easy

Đề bài:
Cho `channel`. Hãy trả về một trong các string chính xác sau: `"EMAIL"`, `"SMS"`, `"PUSH"`, hoặc `"UNSUPPORTED"`. Chỉ match chính xác với các giá trị `"email"`, `"sms"`, và `"push"`.

Ví dụ 1:

Đầu vào:
```text
channel = "sms"
```

Đầu ra:
```text
"SMS"
```

Giải thích:
Factory sẽ chọn implementation của SMS sender cho channel `sms`.

Ràng buộc:

- `channel` is non-null
- Việc so sánh có phân biệt hoa thường
- Output must be one of the four exact strings

### Exercise 2: Count Supported Factory Requests

Độ khó: Easy

Đề bài:
Cho một array `channels`. Hãy đếm xem có bao nhiêu entry được factory hỗ trợ. Các giá trị được hỗ trợ chính xác là `"email"`, `"sms"`, và `"push"`.

Ví dụ 1:

Đầu vào:
```text
channels = ["email", "fax", "push", "sms"]
```

Đầu ra:
```text
3
```

Giải thích:
Only `fax` is unsupported.

Ràng buộc:

- `channels` is non-null
- Độ dài array nằm trong đoạn từ 0 đến 100000
- Each value is a non-null string

### Exercise 3: Normalize Factory Key

Độ khó: Medium

Đề bài:
Cho một raw string `channel`. Hãy trim khoảng trắng ở đầu và cuối, chuyển nó sang lowercase, rồi trả về normalized factory key. Nếu kết quả sau khi trim là chuỗi rỗng, trả về `"unknown"`.

Ví dụ 1:

Đầu vào:
```text
channel = "  Email  "
```

Đầu ra:
```text
"email"
```

Giải thích:
Normalizing input before factory lookup avoids duplicated branching logic across callers.

Ràng buộc:

- `channel` is non-null
- Output is always non-null
- Method này không được mutate external state

## Links

- [[004-Abstract-Factory]]
- [[../Behavioral/002-Strategy]]
- [Refactoring Guru, Factory Method](https://refactoring.guru/design-patterns/factory-method)
- [Spring Framework, Instantiating Beans](https://docs.spring.io/spring-framework/reference/core/beans/java/instantiating-container.html)
- [Martin Fowler, Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
