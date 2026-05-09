# Path and Files

## What is it

`Path` đại diện cho đường dẫn file hoặc directory theo kiểu object thay vì string thô.

`Files` là utility class chứa các operation phổ biến như kiểm tra tồn tại, đọc, ghi, copy, move, create directory.

Mental model: `Path` mô tả “ở đâu”, còn `Files` làm “hành động gì” với vị trí đó.

## How I used to misunderstand it

Mình từng nối path bằng string như `base + "/" + name` và nghĩ vậy đủ đơn giản.

Cách đó dễ sai khi gặp separator khác OS, path thừa slash, hoặc cần resolve relative path.

`Path.resolve()` và `normalize()` làm ý định rõ hơn, đặc biệt khi bạn phải ghép nhiều segment hoặc kiểm tra user input.

## How it actually works

`Path.of(...)` tạo path object.

`resolve()` nối path theo filesystem semantics.

`normalize()` loại bỏ phần dư như `.` hoặc `..` về mặt cú pháp.

`Files` thao tác với filesystem thật, nên có thể ném `IOException` hoặc chịu ảnh hưởng quyền truy cập, symbolic link, và race condition.

### Path workflow table

| Bước | API thường dùng | Ý nghĩa |
|---|---|---|
| Tạo path gốc | `Path.of(...)` | Biến string thành path object |
| Ghép thêm segment | `resolve(...)` | Nối path đúng semantics |
| Làm path gọn hơn | `normalize()` | Xử lý `.` và `..` ở mức cú pháp |
| Thao tác thật với file | `Files.*` | Đọc, ghi, copy, move, create |

### Tiny workflow scaffold

```text
build Path -> resolve child -> normalize if needed -> call Files operation
```

## Code example

```java
import java.nio.file.Path;

Path base = Path.of("data");
Path file = base.resolve("users.csv").normalize();

// data/users.csv on Unix-like systems
System.out.println(file);
```

## When to use / when NOT to use

Dùng `Path` và `Files` cho hầu hết file operation trong Java hiện đại.

Không tự nối path string nếu logic path có nhiều phần hoặc cần an toàn hơn theo OS.

Không dùng `Files.readString` cho file quá lớn.

Không giả định `Files.exists()` đảm bảo file vẫn tồn tại ở dòng tiếp theo, vì filesystem có thể thay đổi giữa check và use.

Nếu filename đến từ user, đừng resolve trực tiếp vào thư mục lưu trữ mà không kiểm tra path traversal.

## How this connects to Spring

Trong Spring Boot, file upload, generated report, temporary file, config ngoài classpath, và batch import đều cần xử lý path an toàn.

Khi nhận filename từ user, bạn cần cẩn thận với path traversal như `../`.

Một workflow thực tế thường là:

- nhận filename thô
- validate hoặc sanitize
- resolve dưới base directory cố định
- thao tác bằng `Files`

## Gotchas

- `normalize()` không kiểm tra file có tồn tại. Nó chỉ chuẩn hóa path syntax.
- `Files.exists()` có race condition nếu dùng để quyết định operation tiếp theo.
- Filename từ user có thể chứa path traversal nếu không validate.
- Relative path và absolute path khác nhau về điểm neo. Đừng assume working directory giống nhau giữa local và production.

## Check yourself

- Vì sao `Path` rõ hơn path string thô khi phải nối nhiều segment?
- `normalize()` giúp gì, và nó không giúp gì?
- Vì sao `Files.exists()` không phải là “bằng chứng chắc chắn” cho bước kế tiếp?
- Path traversal thường chui vào hệ thống qua kiểu input nào?
- Nếu file rất lớn, vì sao không nên mặc định dùng `Files.readString()`?

## Exercises

### Bài 1: Safe Resolve File Name
Độ khó: Trung bình

Đề bài:
Cho một file name, trả về `true` nếu nó an toàn để resolve dưới một base directory. Một tên an toàn không được chứa `/`, `\\`, hoặc `..`.

Ví dụ 1:
Đầu vào:
```text
fileName = "report.csv"
```

Đầu ra:
```text
true
```

Giải thích:
Tên này không cố escape khỏi base directory.

Ràng buộc:
- fileName là non-null và không được blank
- fileName length <= 255
- Chỉ validate chính name string

### Bài 2: Build Relative Path
Độ khó: Dễ

Đề bài:
Cho các directory segment và một file name, trả về relative path được ngăn cách bằng dấu `/`.

Ví dụ 1:
Đầu vào:
```text
segments = ["data", "imports"], fileName = "users.csv"
```

Đầu ra:
```text
"data/imports/users.csv"
```

Giải thích:
File name được append sau toàn bộ directory segment.

Ràng buộc:
- segments là non-null
- 0 <= segments.length <= 1000
- fileName là non-null và không được blank

### Bài 3: Detect Temporary File
Độ khó: Dễ

Đề bài:
Cho một file name, trả về `true` nếu nó kết thúc bằng `.tmp`.

Ví dụ 1:
Đầu vào:
```text
fileName = "upload.tmp"
```

Đầu ra:
```text
true
```

Giải thích:
Filename này có temporary file suffix.

Ràng buộc:
- fileName là non-null
- Matching là case-sensitive
- Trả về một boolean

## Links

- [[001-input-stream-vs-reader]]
- [[003-nio-vs-io]]
- [[../Exception/001-Checked-vs-Unchecked]]
- `Path` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/Path.html
- `Files` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/Files.html
