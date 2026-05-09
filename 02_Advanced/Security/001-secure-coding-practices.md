# Secure Coding Practices

## What is it

`Secure coding practices` là tập các thói quen viết code giúp giảm khả năng tạo ra lỗ hổng ngay từ lúc mình đang code, thay vì đợi tới lúc scan bảo mật hoặc incident mới sửa. Trọng tâm của nó không phải là một library cụ thể, mà là cách mình xử lý input không tin cậy, quyền truy cập, logging, error handling, và các boundary nơi dữ liệu đi từ bên ngoài vào code Java của mình.

Nói ngắn gọn, đây là lớp phòng thủ ở cấp source code. Infra mạnh vẫn chưa đủ nếu code tự mở cửa cho injection, lộ dữ liệu nhạy cảm, hoặc cho phép một action nguy hiểm chạy chỉ vì thiếu một check nhỏ.

## How I used to misunderstand it

Mình từng nghĩ security chủ yếu là việc của DevOps, WAF, HTTPS, hoặc Spring Security config. Nếu app đã có login và chạy sau reverse proxy thì phần Java code bên trong chắc cũng ổn.

Mình cũng từng nghĩ validate input chỉ là kiểm tra `null` hoặc viết một regex đơn giản. Thực tế secure coding còn liên quan đến deny by default, least privilege, không log secret, không tin dữ liệu nội bộ mù quáng, và luôn coi mọi input đi qua boundary là potentially hostile.

## How it actually works

Mental model hữu ích là: dữ liệu băng qua boundary thì phải bị nghi ngờ. Boundary có thể là HTTP request, message queue, environment variable, file upload, config do người vận hành nhập, hoặc dữ liệu từ service khác.

Khi dữ liệu đi vào hệ thống, mình nên hỏi ba điều:

- có hợp lệ theo format và domain rule không
- có được phép dùng ở đây không
- nếu bị log, hiển thị, hoặc gửi tiếp thì có làm lộ dữ liệu nhạy cảm không

Secure coding thường dựa vào vài nguyên tắc lặp lại:

- `Validate early`: kiểm tra format, length, allow-list, và business rule càng gần đầu vào càng tốt
- `Fail safe`: thiếu thông tin hoặc check không pass thì mặc định từ chối
- `Least privilege`: code và account chỉ nên có đúng quyền cần dùng
- `Output-aware handling`: dữ liệu an toàn ở context này chưa chắc an toàn ở context khác

Trong Java, các nguyên tắc đó biến thành quyết định rất cụ thể: dùng enum hoặc allow-list thay vì nhận mọi string tự do, ưu tiên immutable value object ở boundary, không ghép chuỗi để tạo query hoặc command, và tách dữ liệu nhạy cảm khỏi log path. Secure coding không đảm bảo không có bug, nhưng nó làm bug khó biến thành incident nghiêm trọng hơn.

### Boundary checklist

| Boundary | Câu hỏi chính | Safe default | Ví dụ Java/Spring |
|---|---|---|---|
| HTTP request | Input có đúng format và domain rule không? | Reject sớm bằng validation | `@Valid`, custom validator |
| Service-to-service data | Có thật sự được phép tin dữ liệu này không? | Re-check authz và assumptions | service layer authorization |
| File path hoặc command | Input có đi vào dangerous interpreter không? | Tách data khỏi command, giới hạn base path | `Path.normalize()`, không ghép shell command |
| Logging | Dữ liệu nào không được xuất hiện trong log? | Mask hoặc bỏ hẳn secret | filter password, token, cookie |
| DB query | User input có thành một phần của câu lệnh parse không? | Parameterized query | `PreparedStatement`, bind parameter |

### Luồng an toàn tại boundary

```text
Dữ liệu không đáng tin cậy
    -> chuẩn hóa
    -> validate format và length
    -> validate business rule
    -> authorize action
    -> thực thi bằng safe API
    -> chỉ log các field không nhạy cảm
```

## Code example

```java
String normalizedRole = rawRole == null ? "" : rawRole.trim().toUpperCase();

Set<String> allowedRoles = Set.of("USER", "ADMIN", "AUDITOR");
if (!allowedRoles.contains(normalizedRole)) {
    throw new IllegalArgumentException("Unsupported role");
}
```

Ví dụ này minh họa hai ý nhỏ nhưng quan trọng: normalize input trước khi dùng, và dùng allow-list rõ ràng thay vì chấp nhận mọi string rồi hy vọng downstream xử lý đúng.

## When to use / when NOT to use

Use secure coding practices khi:

- code nhận input từ user, external service, file, queue, hoặc config
- code chạm vào authorization, secret, session, token, hoặc dữ liệu cá nhân
- code tạo query, path, expression, hoặc bất kỳ thứ gì sẽ được interpreted ở bước sau

Không xem secure coding như một checklist hình thức chỉ để pass review. Nếu chỉ thêm vài `if` cho có mà không hiểu trust boundary thật sự nằm ở đâu, mình vẫn có thể bỏ sót root cause.

Cũng không nên nghĩ secure coding thay thế hoàn toàn cho threat modeling, dependency scanning, hoặc runtime hardening. Nó là một lớp phòng thủ, không phải toàn bộ hệ thống phòng thủ.

## How this connects to Spring

Trong Spring app, secure coding xuất hiện ở rất nhiều chỗ quen thuộc: `@Valid` và custom validator ở request DTO, `@PreAuthorize` hoặc method security cho authorization, `PasswordEncoder` cho password handling, và logging filter để mask dữ liệu nhạy cảm.

Khi mình viết controller, service, hoặc integration adapter, nguyên tắc secure coding quyết định dữ liệu nào được accept, dữ liệu nào bị reject, và reject sớm ở đâu. Spring giúp mình nhiều, nhưng framework không tự biết business rule của mình. Nếu service tự ghép query, tự log token, hoặc cấp quyền theo string không validate, thì lỗi vẫn là lỗi ứng dụng chứ không phải lỗi framework.

## Gotchas

- Validate format nhưng quên validate semantics, ví dụ email đúng regex nhưng không hợp domain rule
- Check authorization ở controller nhưng quên check lại ở service dùng chung, hậu quả là một internal call path có thể bypass protection
- Log toàn bộ request body để debug dễ làm lộ password, token, hoặc PII vào log retention lâu dài
- Dùng blacklist thay vì allow-list cho input hẹp dễ bị bypass vì mình không thể liệt kê hết mọi chuỗi xấu
- Tin dữ liệu từ internal service là an toàn tuyệt đối có thể mở đường cho chain compromise khi một service khác bị lỗi trước

## Check yourself

- Boundary nào trong app của bạn đang nhận `untrusted input` rõ nhất?
- Một input "đúng format" nhưng sai business rule có thể gây lỗi gì?
- Vì sao check authz chỉ ở controller chưa đủ cho các shared service method?
- Field nào trong request hoặc response tuyệt đối không nên log nguyên văn?

## Exercises

### Bài 1: Choose Access Decision

Độ khó: Dễ

Đề bài:
Cho ba cờ `requestValid`, `authenticated`, và `authorized`. Hãy trả về `"ALLOW"` nếu cả ba đều là `true`. Nếu một trong ba là `false`, trả về `"DENY"`.

Ví dụ 1:

Đầu vào:
```text
requestValid = true
authenticated = true
authorized = false
```

Đầu ra:
```text
"DENY"
```

Giải thích:
Security decision nên deny by default. Thiếu quyền là đủ để từ chối request.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"ALLOW"` hoặc `"DENY"`
- Không cần gọi framework security thật

### Bài 2: Count Missing Secure Checks

Độ khó: Trung bình

Đề bài:
Cho `String[] requiredChecks` và `String[] appliedChecks`. Mỗi phần tử biểu diễn tên một check bảo mật, ví dụ `"length"`, `"allow-list"`, `"mask-log"`. Hãy đếm có bao nhiêu check khác nhau trong `requiredChecks` chưa xuất hiện trong `appliedChecks`.

Ví dụ 1:

Đầu vào:
```text
requiredChecks = ["length", "allow-list", "mask-log"]
appliedChecks = ["length", "mask-log"]
```

Đầu ra:
```text
1
```

Giải thích:
Check `"allow-list"` còn thiếu.

Ràng buộc:

- `0 <= requiredChecks.length, appliedChecks.length <= 100000`
- Mỗi chuỗi có độ dài từ `1` đến `50`
- So sánh chuỗi theo đúng case

### Bài 3: Find First Sensitive Log Field

Độ khó: Trung bình

Đề bài:
Cho `String[] fieldNames`. Hãy trả về tên field đầu tiên có lowercase chứa một trong các từ khóa nhạy cảm: `password`, `token`, `secret`, `authorization`, hoặc `cookie`. Nếu không có field nào nhạy cảm, trả về `"NONE"`.

Ví dụ 1:

Đầu vào:
```text
fieldNames = ["traceId", "userEmail", "refreshToken", "status"]
```

Đầu ra:
```text
"refreshToken"
```

Giải thích:
`"refreshToken"` là field đầu tiên có chứa từ khóa `token` khi đổi về lowercase.

Ràng buộc:

- `0 <= fieldNames.length <= 100000`
- Mỗi field name có độ dài từ `1` đến `100`
- Nếu không có kết quả, phải trả về đúng `"NONE"`

## Links

- [[003-common-vulnerabilities]]
- [[002-cryptography-in-Java]]
- Oracle Java Security Overview: https://docs.oracle.com/en/java/javase/21/security/java-security-overview1.html
- Spring Security Reference: https://docs.spring.io/spring-security/reference/
- OWASP Java Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Java_Security_Cheat_Sheet.html
- OWASP Secure Coding Principles Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Secure_Coding_Principles_Cheat_Sheet.html

