# Custom Exception

## What is it

Custom exception là exception do chính domain của bạn định nghĩa để diễn đạt lỗi có meaning business hoặc technical rõ hơn exception generic của Java.

Thay vì ném `RuntimeException("bad")`, bạn có thể ném `DuplicateEmailException` hoặc `InvalidOrderStateException`.

Mental model: custom exception biến failure từ “có gì đó sai” thành “đúng loại gì đang sai”.

## How I used to misunderstand it

Mình từng nghĩ custom exception là overengineering và chỉ cần message đủ rõ là được.

Nhưng khi hệ thống lớn hơn, message text không đủ mạnh để phân loại error handling, logging, metrics, HTTP mapping, hoặc retry rule.

Cũng có lúc mình tạo quá nhiều custom exception rất mỏng mà không thêm value nào ngoài tên mới. Như vậy cũng không tốt. Nếu type mới không đổi cách code được hiểu hoặc được xử lý, nó chỉ tăng noise.

## How it actually works

Custom exception trong application hiện đại thường kế thừa `RuntimeException`, trừ khi bạn có lý do rất rõ cho checked exception.

Giá trị của nó đến từ:

- tên type nói đúng failure
- constructor hợp lý
- message có context
- `cause` chain đầy đủ
- đôi khi có field bổ sung nếu chúng thật sự giúp xử lý hoặc debug

### When a custom exception earns its keep

| Tình huống | Nên tạo custom exception | Vì sao |
|---|---|---|
| Map sang HTTP status riêng | Yes | Type giúp `@ControllerAdvice` rõ ràng |
| Retry hoặc no-retry theo loại lỗi | Yes | Caller cần phân biệt error type |
| Chỉ ném rồi mọi nơi vẫn bắt `Exception` | Thường no | Không tạo thêm value |
| Built-in exception đã diễn đạt đủ | Thường no | Tránh thừa type |
| Cần preserve low-level cause nhưng thêm domain meaning | Yes | Vừa giữ debug info vừa rõ business context |

### Tiny decision guide

```text
Need distinct handling, mapping, metrics, or debugging value? -> custom exception helps
Just renaming a generic failure with no behavior change? -> maybe noise
```

## Code example

```java
class DuplicateEmailException extends RuntimeException {
    DuplicateEmailException(String email) {
        super("email already exists: " + email);
    }
}

public class Main {
    static void register(String email, boolean exists) {
        if (exists) {
            throw new DuplicateEmailException(email);
        }
    }
}
```

## When to use / when NOT to use

Dùng custom exception khi lỗi có semantic riêng như duplicate resource, invalid transition, unauthorized business action, hoặc external integration failed theo meaning quan trọng.

Không cần tạo custom exception chỉ để bọc mọi exception low-level vô tội vạ.

Nếu loại lỗi không có handling, mapping, branching, hoặc observability riêng, một built-in exception phù hợp có thể đã đủ.

Nói cách khác, đừng tạo custom exception vì “mỗi lỗi phải có class riêng”. Hãy tạo khi type đó giúp người đọc hoặc framework làm decision tốt hơn.

## How this connects to real Java projects

Trong Spring Boot, custom exception rất hữu ích cùng `@ControllerAdvice` để map domain lỗi sang HTTP status ổn định như 400, 404, 409.

Nó cũng giúp structured logging và metrics dễ hơn. Ví dụ `DuplicateEmailException` map ra 409 tốt hơn nhiều so với parsing message text của `RuntimeException` generic.

Custom exception còn hữu ích khi bạn muốn phân biệt lỗi business với lỗi infrastructure trong service layer.

## Gotchas

- Đừng tạo custom exception mà tên rõ nhưng message hoặc cause lại nghèo nàn.
- Wrap exception cũ mà bỏ `cause` sẽ làm mất stack trace hữu ích.
- Quá nhiều custom exception siêu mảnh có thể làm domain khó đọc hơn thay vì rõ hơn.
- Nếu field bổ sung không bao giờ được dùng để debug hay xử lý, chúng chỉ tăng ceremony.

## Handbook rule

- Tạo custom exception khi loại lỗi có handling/mapping/observability riêng; không tạo chỉ để có “class riêng”.
- Phải giữ `cause` khi wrap exception khác, tránh mất stack trace.
- Đặt tên theo domain meaning, không theo low-level detail.
- Field bổ sung phải có lý do dùng cho debug hoặc handling; không thêm ceremony.
- Cân nhắc built-in exception phù hợp trước khi tự tạo type mới.

## Check yourself

- Khi nào built-in exception là đủ, và khi nào custom exception đáng tạo thêm?
- Vì sao một custom exception không có `cause` đôi khi còn tệ hơn built-in exception?
- Nếu type mới không làm thay đổi handling hoặc mapping, dấu hiệu overengineering là gì?
- Trong Spring, vì sao custom exception hợp với `@ControllerAdvice`?
- Nếu lỗi là duplicate email, điều gì làm `DuplicateEmailException` rõ hơn `RuntimeException("bad")`?

## Exercises

### Bài 1: Create Duplicate Email Exception
Độ khó: Dễ

Đề bài:
Cho một email string, trả về một instance `DuplicateEmailException` với message có chứa email đó.

Ví dụ 1:
Đầu vào:
```text
email = "a@test.com"
```

Đầu ra:
```text
DuplicateEmailException("email already exists: a@test.com")
```

Giải thích:
Exception type và message đều mô tả đúng business failure cụ thể.

Ràng buộc:
- email là non-null
- email không được blank
- Message phải chứa email text

### Bài 2: Throw Invalid State Transition
Độ khó: Trung bình

Đề bài:
Cho current order state và requested next state, ném custom `InvalidOrderTransitionException` khi transition đó không được phép.

Ví dụ 1:
Đầu vào:
```text
current = "SHIPPED", next = "PAID"
```

Đầu ra:
```text
InvalidOrderTransitionException
```

Giải thích:
Việc chuyển từ `SHIPPED` ngược về `PAID` vi phạm state flow.

Ràng buộc:
- current là non-null
- next là non-null
- Valid transition thì không được ném exception

### Bài 3: Map Domain Exception To Status Code
Độ khó: Trung bình

Đề bài:
Cho một custom exception instance, trả về HTTP status code nên đại diện cho exception đó. Map `DuplicateEmailException` sang `409`, `ResourceNotFoundException` sang `404`, và mọi exception khác sang `500`.

Ví dụ 1:
Đầu vào:
```text
exception = new DuplicateEmailException("a@test.com")
```

Đầu ra:
```text
409
```

Giải thích:
Custom exception này biểu diễn một conflict.

Ràng buộc:
- exception là non-null
- Chỉ dùng các mapping rule đã được định nghĩa trong đề bài
- Trả về integer status code

## Links

- [[001-checked-vs-unchecked]]
- [[002-try-catch-finally]]
- [[../../04_Lessons-from-bugs/001-null-pointer-exception]]
- `RuntimeException` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/RuntimeException.html
- Spring Framework reference, error responses: https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html
