# checked vs unchecked

## What is it

Checked exception là exception mà compiler buộc bạn phải xử lý hoặc khai báo bằng `throws`.

Unchecked exception là subclass của `RuntimeException`, compiler không ép bắt hoặc khai báo.

Mental model ngắn gọn:

- checked exception nói: caller nên biết failure này là một phần bình thường của operation
- unchecked exception nói: đây là contract violation, invalid state, hoặc bug mà caller không nên bị ép ceremony quá sớm

## How I used to misunderstand it

Mình từng nghĩ checked exception luôn “an toàn hơn”, còn unchecked exception là “lười xử lý lỗi”.

Cách nghĩ đó dễ làm API nặng mà không làm code tốt hơn. Có những lỗi như input sai, state sai, hoặc null sai contract, nơi caller không thật sự recover được ngoài việc sửa code hoặc sửa request. Ép checked exception ở đây chỉ tăng noise.

Chiều ngược lại cũng sai. Với file, network, parse external input, hoặc resource ngoài process, failure thường là chuyện bình thường. Nếu API giấu hết thành unchecked exception, caller dễ quên chuẩn bị fallback, retry, hoặc message phù hợp.

## How it actually works

Checked exception thường đại diện cho failure có thể dự đoán từ môi trường hoặc resource bên ngoài, như `IOException`.

Unchecked exception thường dùng cho programming error hoặc invariant violation như `IllegalArgumentException`, `IllegalStateException`, `NullPointerException`.

Điểm quan trọng nhất không phải “cái nào nghiêm trọng hơn”. Câu hỏi đúng là: caller có action hợp lý nào nếu failure này xảy ra không?

### Decision table

| Tình huống | Checked hợp hơn | Unchecked hợp hơn | Vì sao |
|---|---|---|---|
| Đọc file cấu hình | Yes | Có thể wrap ở boundary | Failure là một phần bình thường của IO |
| Gọi HTTP tới service khác | Thường yes ở low level | Có thể wrap thành business/runtime ở app layer | Caller có thể retry hoặc fallback |
| Input method bị âm khi contract nói phải dương | No | Yes | Đây là contract violation |
| Entity đang ở state bất hợp lệ | No | Yes | Caller thường không recover ngay tại chỗ |
| Parse dữ liệu ngoài hệ thống | Thường yes hoặc translate rõ | Có thể nếu boundary đã chọn cách khác | Failure là expected từ input ngoài |

### Tiny mental model

```text
External, expected, caller can react? -> lean checked
Programming mistake or invalid state? -> lean unchecked
```

Nếu bạn thiết kế API, chọn checked hay unchecked chính là chọn friction cho caller.

Checked exception lan rất nhanh qua method signature nếu dùng sai chỗ.

Unchecked exception ít ceremony hơn, nhưng đổi lại bạn phải có message rõ, type rõ, và không được dùng nó để che một failure recoverable thật sự.

## Code example

```java
import java.io.IOException;

public class Main {
    static String loadConfig() throws IOException {
        throw new IOException("file not found");
    }

    static int divide(int a, int b) {
        if (b == 0) {
            throw new IllegalArgumentException("b must not be zero");
        }
        return a / b;
    }
}
```

## When to use / when NOT to use

Dùng checked exception khi failure là recoverable, expected, và caller thật sự có action hợp lý như retry, fallback, chọn resource khác, hoặc báo lỗi nghiệp vụ đúng chỗ.

Dùng unchecked exception khi input vi phạm contract, object ở state sai, hoặc caller không thể recover một cách meaningful ngoài việc sửa code hoặc sửa request.

Không biến mọi lỗi thành checked exception vì sẽ làm API nặng, caller phải chuyền `throws` xuyên nhiều layer mà vẫn không xử lý gì tốt hơn.

Cũng không ném unchecked exception bừa cho IO hoặc network failure nếu caller đáng ra cần biết để xử lý.

## How this connects to Spring

Trong Spring Boot, unchecked exception phổ biến ở service layer vì framework đã có `@ControllerAdvice`, transaction rollback, validation, và logging tập trung.

Nhưng ở integration boundary như file, S3, HTTP client, JDBC, hoặc message broker, bạn vẫn cần nghĩ rõ chiến lược translate exception:

- giữ checked exception ở low level hay không
- wrap thành custom runtime exception ở app boundary hay không
- có preserve `cause` hay không

Spring không tự quyết định thay bạn đâu là recoverable failure của domain.

## Gotchas

- Wrap checked exception thành unchecked mà mất root cause sẽ làm debug production rất khó.
- Checked exception đi xuyên qua nhiều layer có thể tạo API rất ồn nếu dùng sai chỗ.
- Unchecked exception không có nghĩa là khỏi cần message rõ và contract rõ.
- Đừng dùng checked vs unchecked như thước đo “mức độ nghiêm trọng”. Nó là thước đo về caller responsibility.

## Check yourself

- Nếu một method đọc file từ disk, vì sao checked exception thường hợp lý hơn `IllegalArgumentException`?
- Nếu caller không thể làm gì ngoài sửa bug, vì sao unchecked exception thường hợp hơn checked?
- Khi nào nên giữ checked exception ở low level nhưng translate nó ở service boundary?
- Vì sao wrap exception mà mất `cause` là một decision tệ?
- Nếu một API public bắt caller `throws` qua 5 layer nhưng không ai xử lý, dấu hiệu thiết kế sai là gì?

## Exercises

### Bài 1: Classify Exception Type
Độ khó: Dễ

Đề bài:
Cho một exception object, trả về `"checked"` nếu nó là checked exception và `"unchecked"` nếu nó là `RuntimeException`.

Ví dụ 1:
Đầu vào:
```text
exception = new IllegalArgumentException("bad input")
```

Đầu ra:
```text
"unchecked"
```

Giải thích:
`IllegalArgumentException` là một runtime exception.

Ràng buộc:
- exception là non-null
- Chỉ phân loại theo checked vs unchecked
- Bỏ qua business meaning của message

### Bài 2: Count Recoverable Failures
Độ khó: Trung bình

Đề bài:
Cho một list các exception class, đếm xem có bao nhiêu class nên được xem là recoverable checked exception. Hãy coi `IOException` và `ParseException` là recoverable; coi runtime exception là non-recoverable trong bài này.

Ví dụ 1:
Đầu vào:
```text
exceptions = [IOException, IllegalArgumentException, ParseException]
```

Đầu ra:
```text
2
```

Giải thích:
Chỉ có `IOException` và `ParseException` được tính là recoverable failure trong bài này.

Ràng buộc:
- 0 <= exceptions.length <= 100000
- entries are non-null
- Dùng đúng rule được nêu trong đề bài, không dùng custom heuristic

### Bài 3: Wrap Checked Failure With Cause
Độ khó: Trung bình

Đề bài:
Cho một checked exception và một business message, trả về một runtime exception vẫn giữ nguyên cause ban đầu.

Ví dụ 1:
Đầu vào:
```text
cause = new IOException("disk error"), message = "cannot load user profile"
```

Đầu ra:
```text
RuntimeException(message, cause)
```

Giải thích:
Wrapper giữ lại checked cause để phục vụ debugging.

Ràng buộc:
- cause là non-null
- message là non-null và không được blank
- Giữ nguyên original cause

## Links

- [[002-try-catch-finally]]
- [[003-try-with-resources]]
- [[004-custom-exception]]
- Java exception tutorial: https://docs.oracle.com/javase/tutorial/essential/exceptions/
- `Throwable` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html
- `RuntimeException` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/RuntimeException.html
