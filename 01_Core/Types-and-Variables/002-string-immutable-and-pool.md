# String immutable and pool

## What is it

`String` trong Java là immutable object. Sau khi một `String` được tạo ra, nội dung của nó không đổi. Mọi thao tác như `concat`, `replace`, `trim`, `substring`, `toUpperCase` đều trả về `String` mới thay vì sửa object cũ.

Java còn có `String pool`, một vùng canonicalization cho string literal và một số string đã được `intern()`. Cơ chế này giúp tái sử dụng object trong một số tình huống, nhưng không thay đổi contract so sánh nội dung.

## How I used to misunderstand it

Mình từng nghĩ gọi `text.toUpperCase()` là đã đổi luôn biến `text`. Mình cũng từng tưởng hai string có cùng nội dung thì `==` sẽ luôn đúng, vì ví dụ nhỏ với string literal thường vô tình cho kết quả như vậy.

Hai hiểu nhầm này hay đi chung với nhau. Một cái làm mình quên gán lại kết quả. Cái còn lại làm mình viết sai logic so sánh text.

## How it actually works

Immutability nghĩa là nội dung bên trong `String` không bị mutate. Khi viết:

```java
String name = "java";
name.toUpperCase();
```

biến `name` vẫn là `"java"` nếu không gán lại kết quả. Method tạo object mới rồi trả về.

`String pool` hoạt động chủ yếu với string literal. Hai literal `"spring"` trong source thường trỏ tới cùng pooled object, nên `==` có thể ra `true`. Nhưng nếu một string được tạo bằng `new String("spring")`, reference sẽ khác dù nội dung giống nhau.

Quy tắc thực tế cần nhớ là:

* dùng `.equals()` để so sánh nội dung
* coi `==` là so sánh reference identity
* xem pool là optimization của JVM, không phải business contract

### Quick comparison table

| Câu hỏi | Immutability | String pool |
|---|---|---|
| Nó nói về cái gì | Nội dung string có đổi được không | JVM có thể reuse object nào |
| Ảnh hưởng tới method như `replace()` không | Có | Không trực tiếp |
| Ảnh hưởng tới `==` không | Không trực tiếp | Có thể |
| Ảnh hưởng tới `.equals()` không | Không | Không |
| Có nên dựa vào để viết business logic không | Có, vì đây là contract của `String` | Không |

### Tiny mental diagram

```text
String a = "spring";             ---> pooled "spring"
String b = "spring";             ---> same pooled object
String c = new String("spring"); ---> separate object

a == b       -> true
a == c       -> false
a.equals(c)  -> true
```

## Code example

```java
String a = "spring";
String b = "spring";
String c = new String("spring");

// true, cùng pooled literal
System.out.println(a == b);
// false, khác object
System.out.println(a == c);
// true, cùng nội dung
System.out.println(a.equals(c));

String text = "java";
text.replace("j", "J");
System.out.println(text); // java
```

Ví dụ này cho thấy hai ý tách biệt: immutability làm method trả object mới, còn pool ảnh hưởng tới reference identity của một số string.

## When to use / when NOT to use

Use `String` khi:

* bạn cần value text an toàn, không bị mutate bất ngờ
* bạn truyền dữ liệu giữa các layer và muốn semantics đơn giản
* bạn cần key ổn định cho logging, config, cache, hoặc map key

Do NOT dựa vào `==` để kiểm tra nội dung text. Chỉ một thay đổi nhỏ trong cách string được tạo ra cũng có thể làm logic sai.

Do NOT nối quá nhiều string trong loop dài bằng `+` nếu performance quan trọng. Khi đó `StringBuilder` thường hợp lý hơn vì `String` là immutable.

Do NOT gọi `intern()` như một phản xạ. Đây là công cụ đặc thù, không phải bước tối ưu mặc định cho app business thông thường.

## How this connects to real Java projects

Trong Spring app, `String` đi khắp nơi: request body, header, property key, SQL fragment, log message, bean name, message code. Immutability giúp text không bị một layer sửa làm hỏng layer khác.

Pool ít khi là thứ mình chủ động dùng trong Spring business code. Điều cần nhớ hơn là luôn so sánh string bằng `.equals()` hoặc utility null-safe như `Objects.equals()` khi dữ liệu có thể thiếu.

## Gotchas

* Gọi method trên `String` mà không gán lại kết quả sẽ không thay đổi value cũ.
* `==` có thể trông như đang hoạt động với string literal, rồi fail khi dữ liệu đến từ request, database, hoặc `new String(...)`.
* Nối string lặp đi lặp lại trong loop lớn có thể tạo nhiều object trung gian.
* `intern()` có use case riêng, nhưng không nên dùng bừa chỉ để sửa bug so sánh reference.

## Handbook rule

- `String` immutable; mọi method trả về object mới, phải gán lại để giữ kết quả.
- So sánh nội dung text dùng `equals()` hoặc `equalsIgnoreCase()`, không bao giờ dùng `==`.
- Concat trong loop dài dùng `StringBuilder`, không dùng `+` nối liên tục.
- `intern()` là công cụ đặc thù; không gọi để fix bug so sánh reference.
- Treat `String` như value type ổn định cho key, log, config; tránh giả định identity.

## Check yourself

* Vì sao `text.trim();` không làm biến `text` đổi nếu không gán lại?
* `==` và `.equals()` đang trả lời hai câu hỏi khác nhau thế nào với `String`?
* Vì sao test dùng string literal có thể vô tình che bug `==`?
* Pool ảnh hưởng tới identity, nhưng vì sao không thay đổi rule so sánh nội dung?
* Khi nào bạn nên nghĩ tới `StringBuilder` thay vì nối `String` bằng `+`?

## Exercises

### Bài 1: Normalize Whitespace

Độ khó: Dễ

Đề bài:
Cho một `String text`. Hãy trả về string mới sau khi bỏ khoảng trắng ở đầu và cuối, đồng thời thay mọi chuỗi nhiều khoảng trắng liên tiếp ở giữa bằng đúng một dấu cách.

Ví dụ 1:

Đầu vào:
```text
text = "  java   spring   boot  "
```

Đầu ra:
```text
"java spring boot"
```

Giải thích:
Kết quả là một string mới, không sửa object cũ.

Ràng buộc:

* `0 <= text.length <= 100000`
* `text` chỉ chứa chữ cái English và dấu cách
* Phải trả về một `String`

### Bài 2: Count Equal Content Pairs

Độ khó: Trung bình

Đề bài:
Cho `String[] values`. Hãy đếm số cặp chỉ số `(i, j)` với `i < j` sao cho `values[i]` và `values[j]` có cùng nội dung text. Bài này kiểm tra content equality, không phải reference equality.

Ví dụ 1:

Đầu vào:
```text
values = ["ab", "ab", "cd", "ab"]
```

Đầu ra:
```text
3
```

Giải thích:
Ba cặp hợp lệ là `(0,1)`, `(0,3)`, và `(1,3)`.

Ràng buộc:

* `1 <= values.length <= 100000`
* `0 <= values[i].length <= 50`
* Các phần tử không phải `null`

### Bài 3: Join With Delimiter

Độ khó: Trung bình

Đề bài:
Cho `String[] parts` và `String delimiter`. Hãy nối các phần tử theo đúng thứ tự, chèn `delimiter` giữa hai phần tử kề nhau. Nếu mảng rỗng, trả về string rỗng.

Ví dụ 1:

Đầu vào:
```text
parts = ["core", "types", "java"]
delimiter = ":"
```

Đầu ra:
```text
"core:types:java"
```

Giải thích:
Hàm phải tạo và trả về một string mới đại diện cho toàn bộ kết quả.

Ràng buộc:

* `0 <= parts.length <= 100000`
* `0 <= parts[i].length, delimiter.length <= 100`
* Các phần tử không phải `null`

## Links

* [[005-null]]
* [[006-equals-vs-hash-code]]
* [[../Memory/003-String-pool-vs-heap]]
* [[../Functional/004-Optional]]
* [Java SE 21, `String` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html)
* [Java SE 21, `String#intern()` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html#intern())
* [JLS §3.10.5, String Literals](https://docs.oracle.com/javase/specs/jls/se21/html/jls-3.html#jls-3.10.5)
