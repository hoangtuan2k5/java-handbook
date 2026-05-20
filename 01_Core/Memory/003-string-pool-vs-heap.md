# String pool vs heap

## What is it

`String pool` là vùng JVM dùng để canonicalize và reuse một số `String`, đặc biệt là string literal và string được `intern()`.

`Heap` là nơi object Java nói chung được cấp phát.

Mental model dễ nhớ:

- string literal như `"java"` thường trỏ tới object canonical trong pool
- `new String("java")` vẫn tạo thêm object mới trên heap
- cùng text chưa chắc cùng reference

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là thấy hai string có cùng chữ thì nghĩ chúng là cùng object. Sai.

- `equals()` so sánh nội dung
- `==` so sánh reference identity

Pool chỉ làm một số string trỏ chung tới một object canonical. Nó không biến mọi string có cùng text thành cùng object trong mọi tình huống.

Hiểu nhầm thứ hai là xem `intern()` như tối ưu mặc định. Thực tế nó là công cụ khá đặc biệt, chỉ nên nghĩ tới khi có lý do đo đạc rõ ràng.

## How it actually works

Khi class được load, string literal xuất hiện trong class có thể được đưa vào string pool. Nếu nhiều literal có cùng nội dung, JVM có thể cho chúng dùng cùng một canonical instance.

Ngược lại, `new String("java")` tạo object mới. Nội dung giống nhau, nhưng reference khác.

`intern()` nói với JVM: “hãy trả cho tôi canonical pooled instance cho nội dung này”. Nếu pool đã có, bạn nhận reference đang có. Nếu chưa có, JVM có thể thêm canonical entry mới.

### So sánh nhanh

| Cách tạo | Object mới trên heap? | Có thể dùng pooled canonical object? | Dùng `==` có đáng tin để so sánh nội dung? |
|---|---|---|---|
| `String a = "java";` | Không tạo mới mỗi lần theo nghĩa thông thường trong source | Có | Không |
| `new String("java")` | Có | Literal bên trong vẫn có thể ở pool, nhưng object mới thì không | Không |
| `someString.intern()` | Không đảm bảo tạo object mới | Có, mục tiêu chính là lấy canonical pooled reference | Chỉ có ý nghĩa nếu bạn chủ đích so identity sau khi canonicalize |

### Tiny mental diagram

```text
String a = "java";            ---> pooled "java"
String b = "java";            ---> same pooled "java"
String c = new String("java") ---> separate heap object

a == b -> true
a == c -> false
a.equals(c) -> true
```

Điểm quan trọng là pool ảnh hưởng tới `identity`, còn `equals()` mới là contract so sánh nội dung bạn dùng hằng ngày.

## Code example

```java
String a = "java";
String b = "java";
String c = new String("java");
String d = c.intern();

// true, same pooled literal
System.out.println(a == b);
// false, c is a different heap object
System.out.println(a == c);
// true, same text content
System.out.println(a.equals(c));
// true, d is the canonical pooled reference
System.out.println(a == d);
```

## When to use / when NOT to use

Rule mặc định rất đơn giản:

- dùng `equals()` để so sánh nội dung string
- dùng `==` chỉ khi bạn thật sự muốn so identity

Chỉ nghĩ đến `intern()` khi:

- có lượng lớn string lặp lại
- bạn đã đo memory hoặc performance
- bạn hiểu rõ trade-off của canonicalization

Không dùng `==` cho string đến từ request, database, file, JSON, hoặc object runtime khác.

## How this connects to real Java projects

Trong Spring app, string đi khắp nơi: config key, HTTP header, path variable, JSON field, query param, message code.

Bug dùng `==` thay vì `equals()` rất dễ lọt qua test nhỏ vì test hay dùng literal. Đến production, string đến từ network hoặc deserialization thì identity thay đổi, logic bắt đầu sai ngẫu nhiên.

## Gotchas

- `==` có thể vô tình đúng với literal rồi sai với runtime input.
- `new String("x")` hầu như là mùi code không cần thiết.
- `intern()` không phải tối ưu mặc định cho mọi app.
- Pool giúp reuse, nhưng không thay đổi việc `String` vẫn là object trên JVM.

## Handbook rule

- So sánh nội dung dùng `equals()`; `==` chỉ dùng khi muốn so identity reference.
- `new String("x")` hầu như là code smell; tránh tạo String thừa.
- `intern()` chỉ dùng khi đo được lợi ích về memory cho lượng lớn string lặp.
- Pool reuse không đổi semantics: `String` vẫn là object trên JVM.
- Không dựa vào pool identity cho dữ liệu runtime đến từ request/DB/file.

## Check yourself

- Vì sao hai string cùng text vẫn có thể là hai object khác nhau?
- `equals()` và `==` trả lời hai câu hỏi khác nhau thế nào?
- Trong ví dụ, vì sao `a == c` là `false` nhưng `a.equals(c)` là `true`?
- `intern()` thật ra đang giúp chuẩn hóa cái gì?
- Vì sao bug so string bằng `==` hay chỉ lộ ra khi nhận input từ runtime?

## Exercises

### Bài 1: Compare String Content
Độ khó: Dễ

Đề bài:
Cho hai string, trả về `true` nếu content của chúng bằng nhau.

Ví dụ 1:
Đầu vào:
```text
a = "java", b = "java"
```

Đầu ra:
```text
true
```

Giải thích:
Text content là giống nhau.

Ràng buộc:
- a and b are non-null
- 0 <= a.length, b.length <= 10000
- So sánh content, không so sánh reference identity

### Bài 2: Detect New String Smell
Độ khó: Trung bình

Đề bài:
Cho một code snippet, trả về `true` nếu nó chứa `new String(`.

Ví dụ 1:
Đầu vào:
```text
code = "String s = new String(\"x\");"
```

Đầu ra:
```text
true
```

Giải thích:
Việc tạo một string mới từ literal thường là không cần thiết.

Ràng buộc:
- code là non-null
- code length <= 100000
- Chỉ cần substring detection đơn giản là đủ

### Bài 3: Choose String Comparison Operator
Độ khó: Dễ

Đề bài:
Cho mục đích so sánh, trả về `"equals"` cho `"content"` và `"=="` cho `"identity"`.

Ví dụ 1:
Đầu vào:
```text
purpose = "content"
```

Đầu ra:
```text
"equals"
```

Giải thích:
So sánh content nên dùng `equals`.

Ràng buộc:
- purpose chỉ có thể là `"content"` hoặc `"identity"`
- Chỉ trả về `"equals"` hoặc `"=="`
- Matching là case-sensitive

## Links

- [[001-gc]]
- [[004-shallow-copy-vs-deep-copy]]
- [[../Object-Methods/002-equals-and-hash-code-contract]]
- [[../../00_Mental-Models/007-immutability]]
- [[../../00_Mental-Models/011-value-type-vs-reference-type]]
- [Java SE 21, `String` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html)
- [Java SE 21, `String#intern()` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html#intern())
- [JLS §3.10.5, String Literals](https://docs.oracle.com/javase/specs/jls/se21/html/jls-3.html#jls-3.10.5)
