# Common Vulnerabilities

## What is it

`Common vulnerabilities` là những kiểu lỗi bảo mật lặp đi lặp lại trong rất nhiều hệ thống, ví dụ injection, broken access control, path traversal, insecure deserialization, hoặc secret exposure. Điểm chung của chúng là không phải lỗi ngẫu nhiên hoàn toàn. Chúng thường sinh ra từ một mẫu sai quen thuộc.

Học nhóm này không phải để thuộc lòng tên OWASP, mà để nhìn code và nhận ra: chỗ này đang trộn untrusted input vào một context nguy hiểm, chỗ kia đang tin quyền quá sớm, chỗ khác lại deserialize dữ liệu mà không có boundary rõ ràng.

## How I used to misunderstand it

Mình từng nghĩ lỗ hổng bảo mật lớn chỉ xảy ra ở hệ thống public internet hoặc app có traffic rất cao. Với app nội bộ hoặc service nhỏ, chỉ cần viết cẩn thận là đủ.

Mình cũng từng tách từng lỗ hổng thành những trường hợp rời rạc cần nhớ máy móc. Thực tế rất nhiều vulnerability cùng quay về vài root cause giống nhau: tin input quá sớm, ghép chuỗi vào ngữ cảnh có interpreter, thiếu authorization check, hoặc diễn giải dữ liệu không tin cậy mà không giới hạn boundary.

## How it actually works

Một cách học hiệu quả là nhìn vulnerability như giao điểm giữa `untrusted data` và `dangerous interpreter`.

- SQL injection xảy ra khi input đi vào SQL parser theo cách không được tách dữ liệu khỏi câu lệnh
- XSS xảy ra khi input đi vào HTML hoặc JavaScript context mà không được encode đúng
- Path traversal xảy ra khi input được dùng để xây path truy cập tài nguyên mà không normalize và giới hạn root
- Broken access control xảy ra khi app chỉ check quyền ở chỗ dễ thấy, nhưng bỏ lọt call path khác
- Insecure deserialization xảy ra khi object data không tin cậy được diễn giải thành object graph có side effect

Không phải vulnerability nào cũng cần attacker viết payload phức tạp. Nhiều lỗi chỉ cần một chuỗi `../`, một quyền bị check nhầm tầng, hoặc một object stream đến từ nguồn không đáng tin. Vì vậy cách phòng thủ tốt nhất thường là chọn safe API mặc định, validate theo context, và đặt rule ở đúng boundary thay vì chờ sanitize ở cuối đường đi.

### Recognition matrix

| Mẫu vulnerability | Dấu hiệu nhận ra | Hướng xử lý an toàn | Sai lầm hay gặp |
|---|---|---|---|
| Injection | user input bị ghép vào query, command, expression | parameterization, safe API | escape thủ công rồi nghĩ là đủ |
| Broken access control | quyền chỉ check ở controller hoặc UI | enforce ở service và domain boundary | tin internal call luôn an toàn |
| Path traversal | input điều khiển path hoặc tên file | normalize, resolve, check base directory | chỉ chặn `..` bằng string replace |
| Insecure deserialization | parse object từ nguồn không tin cậy | avoid native Java deserialization, allow-list, serialization filter | deserialize trước rồi mới kiểm |
| Secret exposure | token, password, key xuất hiện trong log hoặc response | mask, redact, separate secret handling | log full request để debug |

### Một mental model cho nhiều bug

```text
Dữ liệu không đáng tin cậy
    -> đi vào dangerous context?
        -> SQL / HTML / file path / object stream / authorization decision
    -> nếu có, dùng context-specific safe API và boundary rule chặt chẽ
```

Khi đã quen với các mẫu lỗi, mình sẽ thấy các lỗi này không còn rời rạc nữa. Chúng là những biến thể của cùng một câu hỏi: dữ liệu này có đang được diễn giải trong một ngữ cảnh nguy hiểm không, và mình đã giữ control đủ chặt trước khi nó tới đó chưa.

## Code example

```java
String sql = "select id, email from users where email = ?";
PreparedStatement statement = connection.prepareStatement(sql);
statement.setString(1, email);
```

Ví dụ này minh họa cách tách dữ liệu khỏi câu lệnh SQL. Ý quan trọng không phải API cụ thể, mà là tránh ghép chuỗi để user input biến thành một phần của câu lệnh được parse.

## When to use / when NOT to use

Dùng kiến thức về common vulnerabilities khi:

- review code ở boundary nhận input từ bên ngoài
- thiết kế API, file access, query layer, hoặc serialization layer
- đọc security scan result và muốn hiểu root cause thay vì chỉ vá symptom

Không học danh sách vulnerability như flashcard rồi tin rằng nhớ tên là đủ. Nếu không gắn chúng với trust boundary và data flow thật trong app, mình sẽ bỏ sót bug có tên khác nhưng cùng bản chất.

Cũng không nên chỉ vá chỗ payload đã được report mà bỏ qua mẫu sai ở các call path khác. Mục tiêu là nhận ra family của bug, không chỉ một example cụ thể.

## How this connects to real Java projects

Trong Spring app, các vulnerability phổ biến thường xuất hiện quanh controller input, data access, file upload, template rendering, deserialization của request body, và authorization ở service layer. `JdbcTemplate`, JPA parameters, Bean Validation, Spring Security, và message converters đều có thể giúp mình đi theo safe path nếu dùng đúng.

Dù vậy, framework không thể tự hiểu tất cả trường hợp sử dụng nguy hiểm. Nếu mình tự ghép native query, tự map path từ user input, hoặc deserialize dữ liệu không tin cậy theo cơ chế không kiểm soát, thì nguy cơ vẫn nằm nguyên trong code ứng dụng.

## Gotchas

- Escape thủ công dấu nháy cho SQL không đáng tin bằng parameterized query và rất dễ sót edge case
- Normalize path nhưng quên kiểm tra path vẫn nằm dưới base directory có thể khiến path traversal vẫn lọt
- Authorization check chỉ làm ở UI hoặc controller dễ bị bypass qua job, event consumer, hoặc internal service path
- Deserialize dữ liệu không tin cậy thành object graph phức tạp có thể mở cửa cho gadget chain hoặc logic abuse
- Fix một payload cụ thể nhưng giữ nguyên mẫu ghép chuỗi nguy hiểm sẽ khiến bug tái xuất hiện ở input khác

## Handbook rule

- Map mọi vulnerability vào trust boundary và data flow thật, không học theo flashcard.
- Query layer dùng parameterized statement; không nối string với input.
- Deserialization dữ liệu untrusted bằng JSON/Protobuf có schema; tránh native serialization.
- File access phải resolve trong base directory cố định và check path traversal.
- SSRF/XSS/CSRF/IDOR đều xuất phát từ thiếu authorization hoặc thiếu validate boundary; sửa ở chỗ root cause, không vá triệu chứng.

## Check yourself

- Trong đoạn code bạn đang review, `untrusted input` đi vào context nguy hiểm nào?
- Vì sao `PreparedStatement` an toàn hơn việc tự escape dấu nháy trong SQL?
- Nếu app có controller check quyền, chỗ nào nữa vẫn nên check authorization lại?
- Với file path từ user, vì sao chỉ chặn chuỗi `..` là chưa đủ?

## Exercises

### Bài 1: Classify SQL Construction

Độ khó: Dễ

Đề bài:
Cho ba cờ `concatenatesUserInput`, `usesParameterizedQuery`, và `escapesQuotesManually`. Trả về `"safe"` nếu `usesParameterizedQuery` là `true` và `concatenatesUserInput` là `false`. Trả về `"vulnerable"` nếu `concatenatesUserInput` là `true`. Trong các trường hợp còn lại, trả về `"risky"`.

Ví dụ 1:

Đầu vào:
```text
concatenatesUserInput = true
usesParameterizedQuery = false
escapesQuotesManually = true
```

Đầu ra:
```text
"vulnerable"
```

Giải thích:
Việc escape thủ công không thay đổi thực tế là user input vẫn đang bị ghép trực tiếp vào câu lệnh.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"safe"`, `"risky"`, hoặc `"vulnerable"`
- Không cần kết nối database thật

### Bài 2: Find First Traversal Attempt

Độ khó: Trung bình

Đề bài:
Cho `String[] paths`. Hãy trả về index đầu tiên mà path chứa `".."` hoặc chứa chuỗi encoded `"%2e%2e"` theo so sánh không phân biệt hoa thường. Nếu không có path nào đáng ngờ, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
paths = ["images/avatar.png", "docs/%2E%2E/secrets.txt", "logs/app.log"]
```

Đầu ra:
```text
1
```

Giải thích:
Phần tử ở index `1` chứa mẫu traversal đã encode `%2E%2E`.

Ràng buộc:

- `0 <= paths.length <= 100000`
- Mỗi path có độ dài từ `1` đến `200`
- Nếu không có path đáng ngờ, phải trả về `-1`

### Bài 3: Classify Deserialization Risk

Độ khó: Trung bình

Đề bài:
Cho ba cờ `untrustedInput`, `nativeJavaDeserialization`, và `allowListEnabled`. Trả về `"block"` nếu `untrustedInput` và `nativeJavaDeserialization` đều là `true` nhưng `allowListEnabled` là `false`. Trả về `"review"` nếu đúng hai trong ba cờ là `true`. Ngược lại trả về `"safe"`.

Ví dụ 1:

Đầu vào:
```text
untrustedInput = true
nativeJavaDeserialization = true
allowListEnabled = false
```

Đầu ra:
```text
"block"
```

Giải thích:
Đây là tổ hợp rủi ro cao vì object data không tin cậy đang đi vào native Java deserialization mà không có allow-list.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"safe"`, `"review"`, hoặc `"block"`
- Không cần deserialize object thật

## Links

- [[001-secure-coding-practices]]
- [[002-cryptography-in-Java]]
- Oracle Java Security Overview: https://docs.oracle.com/en/java/javase/21/security/java-security-overview1.html
- Spring Security Reference: https://docs.spring.io/spring-security/reference/
- OWASP Top 10: https://owasp.org/Top10/
- OWASP Injection Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html
- OWASP Path Traversal: https://owasp.org/www-community/attacks/Path_Traversal
- OWASP Insecure Deserialization: https://owasp.org/www-community/vulnerabilities/Insecure_Deserialization

