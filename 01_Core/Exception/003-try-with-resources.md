# try-with-resources

## What is it

`try-with-resources` là cú pháp để tự động đóng resource sau khi dùng xong, miễn là resource implement `AutoCloseable`.

Mental model: thay vì nhớ đóng thủ công trong `finally`, bạn khai báo resource ngay cạnh `try` và để ngôn ngữ đảm bảo cleanup nhất quán.

## How I used to misunderstand it

Mình từng xem `try-with-resources` chỉ là syntax sugar cho `finally`.

Nó đúng nhưng chưa đủ. Giá trị thật không chỉ là ngắn hơn, mà là giảm hẳn bug quên close, close sai thứ tự, hoặc exception trong cleanup che mất exception chính.

Khi code làm việc với stream, reader, connection, statement, hoặc custom handle, cú pháp này vừa an toàn vừa nói intent rất rõ.

## How it actually works

Resource được khai báo trong header của `try`, và JVM sẽ gọi `close()` tự động khi block kết thúc.

Nếu có nhiều resource, chúng được đóng theo thứ tự ngược lại với thứ tự khai báo.

Nếu cả business logic và `close()` cùng ném exception, lỗi từ `close()` thường trở thành suppressed exception thay vì che mất lỗi chính.

### Resource cleanup comparison

| Cách viết | Ưu điểm | Điểm yếu |
|---|---|---|
| `finally` đóng thủ công | Dùng được với code cũ | Dễ quên close, dễ lặp code |
| `try-with-resources` | Gọn, an toàn, xử lý suppressed exception tốt hơn | Cần `AutoCloseable` |

### Close order scaffold

```text
try (A a = ...; B b = ...) {
    use a and b
}

close order: B first, then A
```

`try-with-resources` không chỉ dành cho file IO. Bất kỳ type nào implement `AutoCloseable` đều dùng được, kể cả custom resource của bạn.

## Code example

```java
class FakeResource implements AutoCloseable {
    @Override
    public void close() {
        System.out.println("closed");
    }
}

public class Main {
    public static void main(String[] args) {
        try (FakeResource resource = new FakeResource()) {
            System.out.println("using");
        }
    }
}
```

## When to use / when NOT to use

Dùng `try-with-resources` cho file, stream, socket, JDBC resource, custom `AutoCloseable`, hoặc bất kỳ resource nào cần đóng đúng lúc.

Không cần dùng nếu object không có lifecycle close rõ hoặc không implement `AutoCloseable`.

Không quay lại `finally` thủ công trừ khi bạn đang làm việc với API legacy chưa hỗ trợ cách tốt hơn.

Nếu resource do container quản lý ở vòng đời lớn hơn method hiện tại, đừng tự đóng nó theo kiểu local resource.

## How this connects to Spring

Trong Spring Boot, nhiều resource lifecycle lớn như datasource pool, bean, hoặc HTTP client pool được container quản lý.

Nhưng ở chỗ bạn tự mở `InputStream`, `Reader`, `ZipInputStream`, hoặc native handle trong method, `try-with-resources` vẫn rất quan trọng.

Nói ngắn gọn:

- container-managed lifecycle, Spring quản lý
- method-local resource, bạn vẫn phải đóng đúng cách

## Gotchas

- Resource trong header phải implement `AutoCloseable`.
- Nhiều resource được đóng theo thứ tự ngược lại, nên dependency giữa chúng cần được hiểu đúng.
- Đừng nhầm bean lifecycle với resource local trong method. Cái sau vẫn cần đóng đúng cách.
- Suppressed exception tồn tại để giữ lỗi cleanup mà không làm mất lỗi chính. Đừng bỏ qua nó khi debug.

## Check yourself

- Vì sao `try-with-resources` tốt hơn `finally` thủ công trong đa số code mở và đóng resource?
- Nếu có hai resource trong header, resource nào đóng trước?
- Suppressed exception giải quyết bẫy nào của cleanup thủ công?
- Khi nào không nên tự đóng resource trong method vì lifecycle đã do framework quản lý?
- Nếu một custom class có lifecycle open và close rõ, điều kiện nào để nó dùng được với `try-with-resources`?

## Exercises

### Bài 1: Use AutoCloseable Safely
Độ khó: Dễ

Đề bài:
Cho một resource `AutoCloseable` và một block trả về string, hãy execute block đó và đảm bảo resource được đóng tự động.

Ví dụ 1:
Đầu vào:
```text
resource = FakeResource(), result = "ok"
```

Đầu ra:
```text
"ok"
```

Giải thích:
Result được trả về và resource vẫn được đảm bảo sẽ close sau block đó.

Ràng buộc:
- resource là non-null
- Resource must be closed exactly once
- Khi execute thành công thì trả về block result

### Bài 2: Count Suppressed Exceptions
Độ khó: Trung bình

Đề bài:
Cho một primary exception và một close failure, trả về số suppressed exception được attach vào primary exception sau một luồng kiểu try-with-resources.

Ví dụ 1:
Đầu vào:
```text
primary = new RuntimeException("main"), closeFailure = new IllegalStateException("close")
```

Đầu ra:
```text
1
```

Giải thích:
Close failure trở thành suppressed exception của primary exception.

Ràng buộc:
- Cả hai exception đều là non-null
- Primary exception vẫn là failure chính
- Chỉ trả về số lượng suppressed exception

### Bài 3: Close Multiple Resources In Reverse Order
Độ khó: Trung bình

Đề bài:
Cho hai resource có tên được khai báo trong cùng một try-with-resources header, trả về thứ tự mà chúng được đóng.

Ví dụ 1:
Đầu vào:
```text
resources = ["reader", "buffer"]
```

Đầu ra:
```text
["buffer", "reader"]
```

Giải thích:
Resource được đóng theo thứ tự ngược với thứ tự khai báo.

Ràng buộc:
- Có đúng hai resource name được cung cấp
- Name là non-null
- Giữ nguyên chính xác close order

## Links

- [[002-try-catch-finally]]
- [[004-custom-exception]]
- [[../IO/001-input-stream-vs-reader]]
- Java exception tutorial: https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html
- `AutoCloseable` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/AutoCloseable.html
