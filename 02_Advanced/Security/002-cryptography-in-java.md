# Cryptography in Java

## What is it

`Cryptography in Java` là cách dùng các API như `JCA`, `JCE`, `MessageDigest`, `Cipher`, `Mac`, `Signature`, và `SecureRandom` để giải quyết các bài toán như confidentiality, integrity, authenticity, và password storage. Điểm quan trọng là crypto không phải một món duy nhất. Mỗi bài toán cần primitive khác nhau.

Trong thực tế, phần khó không nằm ở việc gọi API cho chạy được, mà nằm ở việc chọn đúng primitive, đúng mode, đúng random source, và không tự chế protocol. Java cho mình khá nhiều building blocks mạnh, nhưng cũng đủ nhiều để chọn sai nếu không có mental model rõ.

## How I used to misunderstand it

Mình từng gom mọi thứ vào một chữ `encrypt`. Muốn bảo vệ password cũng nghĩ tới encryption. Muốn kiểm tra file có bị sửa không cũng nghĩ tới encryption. Thậm chí có lúc nhìn chuỗi `Base64` rồi vô thức xem nó như dữ liệu đã được bảo mật.

Mình cũng từng nghĩ dùng `SHA-256` trực tiếp là đủ cho password storage vì nghe rất mạnh. Thực tế password storage cần `password hashing` hoặc `KDF` chậm và có salt, còn bare hash rất dễ bị brute force nhanh hơn nhiều so với mong muốn.

## How it actually works

Mental model tốt là tách nhu cầu bảo mật theo câu hỏi. Nếu mình cần phát hiện dữ liệu bị đổi, đó là bài toán integrity, thường đi với hash hoặc `HMAC`. Nếu mình cần giấu nội dung nhưng vẫn phát hiện tampering, đó là bài toán `authenticated encryption`, thường là `AEAD` như `AES/GCM`. Nếu mình cần xác thực ai đã tạo ra dữ liệu, đó là chữ ký số. Nếu mình cần lưu password, đó là password hashing hoặc key derivation, không phải reversible encryption.

Java Crypto Architecture cho phép mình chọn algorithm qua tên transformation hoặc provider. Điều này linh hoạt, nhưng cũng khiến sai lầm dễ xảy ra. Ví dụ cùng là `AES` nhưng `AES/GCM/NoPadding` và `AES/ECB/PKCS5Padding` khác nhau rất xa về độ an toàn. Tương tự, `SecureRandom` khác hẳn việc tự random bằng nguồn không phù hợp.

Một nguyên tắc cực kỳ quan trọng là đừng tự thiết kế crypto scheme riêng. Dùng primitive mạnh theo cách sai vẫn dẫn tới lỗ hổng, ví dụ reuse nonce với `GCM`, ký một payload nhưng verify payload khác sau khi canonicalization thay đổi, hoặc dùng bare digest cho password.

### Crypto decision matrix

| Mục tiêu | Primitive phù hợp | Safe choice thường gặp | Lựa chọn dễ sai |
|---|---|---|---|
| Lưu password | `Password hashing` hoặc `KDF` | Argon2, bcrypt, PBKDF2 | mã hóa reversible, bare `SHA-256` |
| Kiểm tra integrity | `Hash` hoặc `HMAC` | `HMAC-SHA-256` khi có shared secret | chỉ nhìn `Base64`, tự ghép chuỗi checksum mơ hồ |
| Giấu nội dung message | `AEAD` | `AES/GCM/NoPadding` | `AES/ECB`, tự ghép encrypt rồi hash không rõ rule |
| Xác thực người gửi | `Digital signature` | `Signature` API với key pair | chia sẻ cùng một secret cho mọi nơi khi cần non-repudiation |

### Luồng quyết định nhanh

```text
Cần lưu password?
    -> dùng password hashing hoặc KDF

Cần confidentiality kèm tamper detection?
    -> dùng AEAD như AES/GCM

Cần integrity với shared secret?
    -> dùng HMAC

Cần signer identity với public verification?
    -> dùng digital signature
```

## Code example

```java
SecureRandom secureRandom = new SecureRandom();
byte[] nonce = new byte[12];
secureRandom.nextBytes(nonce);

Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
```

Ví dụ này chỉ nhấn mạnh hai quyết định an toàn hơn: dùng `SecureRandom` cho nonce và chọn `AES/GCM/NoPadding` thay vì mode cũ như `ECB`. Nó chưa phải full encryption flow, nhưng đúng mental direction.

## When to use / when NOT to use

Use crypto trong Java khi:

- bạn cần bảo vệ dữ liệu nhạy cảm khi lưu hoặc truyền
- bạn cần verify integrity hoặc authenticity của dữ liệu
- bạn cần password storage đúng chuẩn thông qua password hashing hoặc `KDF`
- bạn cần ký hoặc verify token, document, hoặc message

Do NOT dùng crypto chỉ vì thấy dữ liệu `Base64` trông khó đọc hơn. Cũng không nên tự nghĩ ra protocol riêng kiểu ghép hash, salt, timestamp, rồi gọi đó là secure.

Nếu requirement chỉ là obfuscation nhẹ cho UX hoặc định danh nội bộ không có security value, hãy nói rõ đó không phải crypto security thật. Crypto chỉ giúp khi nó khớp đúng threat model.

## How this connects to real Java projects

Trong Spring ecosystem, crypto thường xuất hiện qua `Spring Security`, `PasswordEncoder`, token signing hoặc verification, HTTPS/TLS config, và secret management integration. Ví dụ thực tế nhất là password hashing cho user account, nơi framework giúp mình gắn primitive đúng vào authentication flow.

Điểm cần nhớ là framework không tự sửa được quyết định primitive sai. Nếu mình chọn bare `SHA-256` cho password, hoặc chấp nhận một transformation yếu, thì framework chỉ đang bọc một lựa chọn sai bằng API tiện hơn. Hiểu crypto ở mức Java core vẫn rất cần để dùng Spring an toàn.

## Gotchas

- `Base64` chỉ là encoding, không cung cấp confidentiality
- `SHA-256` trực tiếp không phù hợp cho password storage, vì quá nhanh cho attacker brute force
- `AES/ECB` làm lộ pattern và không nên xem là lựa chọn chấp nhận được cho dữ liệu mới
- Reuse nonce hoặc IV trong mode yêu cầu uniqueness như `GCM` có thể phá hỏng guarantee bảo mật
- Tự trộn nhiều primitive mà không hiểu rõ threat model dễ tạo ra scheme nhìn có vẻ phức tạp nhưng lại yếu

## Check yourself

- Bài toán của bạn là `password storage`, `integrity`, `authenticated encryption`, hay `signature`?
- Vì sao `Base64` không phải là một cơ chế bảo mật?
- Trong trường hợp nào `AES/GCM` hợp lý hơn một hash hoặc `HMAC`?
- Nếu đang lưu password trong Spring app, vì sao bare `SHA-256` là lựa chọn sai hướng?

## Exercises

### Bài 1: Choose Crypto Primitive

Độ khó: Dễ

Đề bài:
Cho `String goal`. Nếu goal là `"store-password"`, trả về `"KDF"`. Nếu goal là `"check-integrity"`, trả về `"HASH_OR_HMAC"`. Nếu goal là `"encrypt-message"`, trả về `"AEAD"`. Nếu goal là `"sign-message"`, trả về `"SIGNATURE"`. Với mọi giá trị khác, trả về `"UNKNOWN"`.

Ví dụ 1:

Đầu vào:
```text
goal = "encrypt-message"
```

Đầu ra:
```text
"AEAD"
```

Giải thích:
Mục tiêu là vừa giữ bí mật vừa phát hiện tampering, nên primitive family phù hợp là authenticated encryption.

Ràng buộc:

- `goal` là chuỗi có độ dài từ `1` đến `100`
- So sánh chuỗi theo đúng case như đề bài
- Chỉ trả về `"KDF"`, `"HASH_OR_HMAC"`, `"AEAD"`, `"SIGNATURE"`, hoặc `"UNKNOWN"`

### Bài 2: Validate Cipher Transformation

Độ khó: Trung bình

Đề bài:
Cho `String transformation`. Trả về `"safe"` nếu transformation đúng bằng `"AES/GCM/NoPadding"`. Trả về `"unsafe"` nếu transformation chứa `"ECB"`, bắt đầu bằng `"DES"`, hoặc chứa `"RC4"`. Với mọi giá trị khác, trả về `"review"`.

Ví dụ 1:

Đầu vào:
```text
transformation = "AES/ECB/PKCS5Padding"
```

Đầu ra:
```text
"unsafe"
```

Giải thích:
`ECB` là mode không nên dùng cho dữ liệu mới vì làm lộ pattern.

Ràng buộc:

- `transformation` có độ dài từ `1` đến `100`
- So sánh chuỗi theo đúng case
- Chỉ trả về `"safe"`, `"review"`, hoặc `"unsafe"`

### Bài 3: Count Weak Algorithm Selections

Độ khó: Trung bình

Đề bài:
Cho `String[] algorithms`. Hãy đếm có bao nhiêu phần tử thuộc tập lựa chọn yếu hoặc obsolete sau: `MD5`, `SHA-1`, `DES`, `3DES`, `RC4`, `AES/ECB/PKCS5Padding`.

Ví dụ 1:

Đầu vào:
```text
algorithms = ["SHA-256", "MD5", "AES/GCM/NoPadding", "RC4"]
```

Đầu ra:
```text
2
```

Giải thích:
`MD5` và `RC4` đều nằm trong danh sách lựa chọn không nên dùng.

Ràng buộc:

- `0 <= algorithms.length <= 100000`
- Mỗi chuỗi có độ dài từ `1` đến `100`
- So sánh chuỗi theo đúng case

## Links

- [[001-secure-coding-practices]]
- [[003-common-vulnerabilities]]
- Oracle JCA Reference Guide: https://docs.oracle.com/en/java/javase/21/security/java-cryptography-architecture-jca-reference-guide.html
- Oracle Security Standard Names: https://docs.oracle.com/en/java/javase/21/docs/specs/security/standard-names.html
- Spring Security Password Storage: https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html
- OWASP Cryptographic Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
- OWASP Password Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
