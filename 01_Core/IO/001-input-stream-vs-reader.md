# InputStream vs Reader

## What is it

`InputStream` đọc dữ liệu theo byte.

`Reader` đọc dữ liệu theo character.

Mental model quan trọng:

- `InputStream` hợp với dữ liệu binary hoặc dữ liệu chưa biết encoding
- `Reader` hợp với text sau khi bạn đã biết cách decode byte thành ký tự

## How I used to misunderstand it

Mình từng nghĩ `Reader` chỉ là version “tiện hơn” của `InputStream`.

Thực ra khác biệt nằm ở tầng abstraction. Máy tính lưu mọi thứ dưới dạng byte. Text chỉ xuất hiện sau bước decode bằng charset như UTF-8.

Nếu đọc file ảnh, file zip, upload binary, hoặc request body dạng bytes, dùng `InputStream`.

Nếu đọc text UTF-8, CSV, JSON, hoặc log file, dùng `Reader` hoặc API cao hơn như `BufferedReader` hay `Files.readString`.

## How it actually works

`InputStream` không quan tâm charset. Nó chỉ trả byte.

`Reader` nằm sau bước decode và trả character data.

Vì vậy khi chuyển từ `InputStream` sang `Reader`, bạn phải chọn charset rõ ràng, thường bằng `InputStreamReader`.

### Byte vs char table

| Câu hỏi | `InputStream` | `Reader` |
|---|---|---|
| Đơn vị đọc | byte | character |
| Hợp với binary | Yes | No |
| Hợp với text | Chỉ khi bạn tự decode tiếp | Yes |
| Có cần charset | Không ở chính abstraction này | Yes, trực tiếp hoặc gián tiếp |
| Ví dụ | image, zip, pdf, raw upload | csv, json, txt, log |

### Decision scaffold

```text
Is the source binary? -> InputStream
Is the source text?   -> Reader
Need to decode bytes to text? -> InputStreamReader + explicit charset
```

## Code example

```java
import java.io.ByteArrayInputStream;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;

byte[] bytes = "java".getBytes(StandardCharsets.UTF_8);

try (InputStreamReader reader = new InputStreamReader(
        new ByteArrayInputStream(bytes), StandardCharsets.UTF_8)) {
    System.out.println((char) reader.read()); // j
}
```

## When to use / when NOT to use

Dùng `InputStream` khi dữ liệu là binary hoặc bạn chưa muốn quyết định encoding.

Dùng `Reader` khi dữ liệu là text và charset đã rõ.

Không dùng `Reader` cho binary file vì decode sai sẽ làm hỏng dữ liệu.

Không dựa vào default charset ngầm định nếu dữ liệu đến từ file, request, hoặc system bên ngoài.

Nếu bạn cần đọc toàn bộ text nhỏ, API cao hơn như `Files.readString` thường rõ hơn việc tự dựng chuỗi wrapper thủ công.

## How this connects to real Java projects

Trong Spring, request body, upload file, resource file, hoặc response stream đều có thể đi qua `InputStream`.

Khi đọc text config hoặc template, Spring thường chuyển về abstraction cao hơn, nhưng lỗi charset vẫn có thể xuất hiện nếu bạn tự xử lý stream trong controller hoặc service.

Một dấu hiệu tốt là: dữ liệu vào hệ thống của bạn là bytes, nhưng business logic nên làm việc với text đã decode đúng charset.

## Gotchas

- Dùng default charset có thể chạy đúng trên máy mình nhưng sai trên server khác.
- `read()` có thể trả từng phần dữ liệu, không đảm bảo đọc hết trong một lần.
- Quên close stream hoặc reader có thể giữ file handle hoặc network connection lâu hơn cần thiết.
- Chuyển binary sang `Reader` chỉ vì “muốn đọc cho tiện” là bẫy phá dữ liệu rất phổ biến.

## Handbook rule

- Binary dùng `InputStream`; text dùng `Reader` với charset rõ ràng.
- Không dùng default charset ngầm; data từ file/HTTP/external phải set explicit charset.
- `read()` có thể trả phần dữ liệu; vòng đọc phải tôn trọng bytes-read trả về.
- Mọi stream/reader đóng bằng `try-with-resources` để tránh giữ file/connection handle.
- Convert binary sang text bừa làm hỏng dữ liệu; quyết định encoding trước khi đọc.

## Check yourself

- Vì sao `InputStream` là abstraction tự nhiên cho ảnh hoặc zip, còn `Reader` thì không?
- Khi nào bạn cần `InputStreamReader` thay vì dùng `Reader` trực tiếp?
- Vì sao text luôn quay về câu chuyện charset?
- Nếu dữ liệu là JSON UTF-8, tại sao business code thường nên làm việc với `Reader` hoặc `String` thay vì byte thô?
- Default charset gây lỗi kiểu gì khi deploy sang môi trường khác?

## Exercises

### Bài 1: Count Bytes In Payload
Độ khó: Dễ

Đề bài:
Cho một byte array payload, trả về độ dài của nó tính theo byte.

Ví dụ 1:
Đầu vào:
```text
payload = [65, 66, 67]
```

Đầu ra:
```text
3
```

Giải thích:
Payload này có ba raw byte.

Ràng buộc:
- payload là non-null
- 0 <= payload.length <= 100000
- Không decode byte thành text

### Bài 2: Decode Utf8 Text
Độ khó: Trung bình

Đề bài:
Cho các byte đã được encode theo UTF-8, trả về string sau khi decode.

Ví dụ 1:
Đầu vào:
```text
bytes = [106, 97, 118, 97]
```

Đầu ra:
```text
"java"
```

Giải thích:
Các byte này biểu diễn UTF-8 text `java`.

Ràng buộc:
- bytes là non-null
- 0 <= bytes.length <= 100000
- Luôn decode theo UTF-8

### Bài 3: Choose Stream Type
Độ khó: Dễ

Đề bài:
Cho một content type label, trả về `"reader"` cho text-like data và `"input-stream"` cho binary data. Hãy coi label bắt đầu bằng `text/` hoặc bằng `application/json` là text-like.

Ví dụ 1:
Đầu vào:
```text
contentType = "application/json"
```

Đầu ra:
```text
"reader"
```

Giải thích:
JSON là text và nên được decode bằng một charset đã biết.

Ràng buộc:
- contentType là non-null và không được blank
- Matching là case-sensitive
- Chỉ trả về `"reader"` hoặc `"input-stream"`

## Links

- [[002-buffered-reader]]
- [[003-nio-vs-io]]
- [[005-path-and-files]]
- Java I/O tutorial: https://docs.oracle.com/javase/tutorial/essential/io/
- `InputStream` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/InputStream.html
- `Reader` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/Reader.html
- `InputStreamReader` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/InputStreamReader.html
