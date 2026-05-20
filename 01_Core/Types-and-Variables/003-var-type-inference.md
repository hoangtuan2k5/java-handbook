# var Type Inference

## What is it

Từ Java 10, `var` cho phép compiler suy ra kiểu của local variable từ expression ở bên phải. Nó chỉ áp dụng cho local variable có initializer.

`var` không áp dụng cho:

* field
* method parameter
* method return type

Mục tiêu của `var` là giảm lặp lại type name khi kiểu đã khá rõ, đặc biệt với generic type dài như `Map<String, List<Integer>>`.

## How I used to misunderstand it

Mình từng tưởng `var` giống JavaScript, nghĩa là biến có thể đổi type về sau. Mình cũng từng muốn dùng `var` ở mọi nơi cho gọn, kể cả chỗ làm code khó đọc hơn.

Hiểu nhầm lớn nhất là xem `var` như tính năng làm Java bớt static. Thực tế type vẫn được chốt ở compile time như cũ. Chỉ có phần viết ra được rút gọn.

## How it actually works

`var` không tạo dynamic typing. Kiểu vẫn cố định ở compile time, chỉ là compiler tự suy ra thay cho mình. Ví dụ:

```java
var count = 10;
```

ở đây `count` vẫn là `int`, không phải một kiểu lỏng có thể đổi sang `String` ở dòng sau.

`var` chỉ hợp khi initializer nói đủ rõ intent. Nếu bên phải là `new HashMap<String, Integer>()`, dùng `var` thường khá dễ đọc. Nhưng nếu expression mơ hồ như `var result = compute();`, người đọc có thể phải nhảy đi tìm kiểu thật sự.

Vì `var` chỉ dùng cho local variable, nó là công cụ giảm noise chứ không thay đổi model type system của Java. Rule thực tế là ưu tiên readability trước sự ngắn gọn.

### Comparison table

| Câu hỏi | Explicit type | `var` |
|---|---|---|
| Type có vẫn là static không | Có | Có |
| Có phải compiler vẫn biết type thật không | Có | Có |
| Người đọc có thấy type ngay ở bên trái không | Có | Không |
| Hợp khi initializer quá rõ không | Có | Rất hợp |
| Hợp khi initializer mơ hồ không | Có | Thường kém |
| Dùng cho field hay parameter không | Có | Không |

### Quick decision scaffold

```text
Initializer tự nói rất rõ type và intent? -> var is fine
Initializer mơ hồ, factory-heavy, hoặc muốn nhấn mạnh abstraction? -> write the type
```

## Code example

```java
var counts = new java.util.HashMap<String, Integer>();
counts.put("java", 2);

var title = "Spring Boot";
var total = counts.getOrDefault("java", 0);

System.out.println(title + ": " + total);
```

`counts` được suy ra là `HashMap<String, Integer>`, `title` là `String`, và `total` là `int`. Không có biến nào trở thành dynamic type cả.

## When to use / when NOT to use

Dùng `var` khi:

* kiểu ở bên phải đã rất rõ
* type name dài và lặp lại không mang thêm thông tin
* local variable chỉ tồn tại trong block ngắn, dễ đọc

Không dùng `var` khi:

* initializer mơ hồ, ví dụ gọi factory method mà không rõ return type
* tên biến yếu, khiến người đọc không đoán được meaning lẫn type
* bạn cần nhấn mạnh abstraction type, ví dụ muốn người đọc thấy rõ đây là `Map` chứ không phải implementation cụ thể

Không xem `var` là default style cho mọi dòng code. Nó là công cụ đọc dễ hơn trong một số chỗ, không phải mục tiêu tự thân.

## How this connects to real Java projects

Trong Spring code, `var` thường xuất hiện ở service method, test setup, mapping logic, và stream pipeline để giảm boilerplate. Nó hữu ích khi làm việc với generic type dài như `ResponseEntity<Map<String, Object>>` hoặc `List<Map<String, String>>`.

Tuy vậy, ở public API, chỗ cần nhấn mạnh abstraction boundary, hoặc đoạn mapping khó đọc, viết explicit type vẫn thường rõ hơn. Ví dụ `Map<String, Object> attributes = ...` giao tiếp intent tốt hơn `var attributes = ...` nếu phần bên phải không đủ rõ.

## Gotchas

* `var` không dùng được nếu biến chưa có initializer.
* `var` không dùng cho field, parameter, hoặc return type trong Java thông thường.
* Dùng `var` với `null` trực tiếp là không hợp lệ vì compiler không suy ra được type.
* `var` có thể che mất abstraction quan trọng nếu initializer dùng implementation cụ thể.

## Handbook rule

- `var` chỉ áp dụng cho local variable đã có initializer rõ ràng; không dùng cho field/parameter/return.
- Khi initializer là factory hoặc method có return type mơ hồ, viết type cụ thể thay vì `var`.
- Tên biến phải đủ rõ để người đọc đoán được kiểu nếu dùng `var`; nếu không, viết type ra.
- Đừng `var` để giấu interface quan trọng như `Map`, `List`; giữ abstraction khi cần nhấn mạnh contract.
- `var = null` không hợp lệ; cần initializer có type cụ thể.

## Check yourself

* Vì sao `var` không làm Java thành dynamic language?
* Khi nào `var result = service.call();` là khó đọc hơn `ResponseEntity<UserDto> result = service.call();`?
* Nếu muốn nhấn mạnh contract là `Map`, vì sao `var` có thể che mất thông tin đó?
* Vì sao `var value = null;` không compile?
* `var` đang giúp giảm cái gì, boilerplate hay type safety?

## Exercises

### Bài 1: Count Word Frequencies

Độ khó: Dễ

Đề bài:
Cho `String[] words`. Hãy trả về một `Map<String, Integer>` chứa số lần xuất hiện của từng từ theo đúng thứ tự xuất hiện đầu tiên của key.

Ví dụ 1:

Đầu vào:
```text
words = ["java", "spring", "java", "jpa"]
```

Đầu ra:
```text
{"java"=2, "spring"=1, "jpa"=1}
```

Giải thích:
`"java"` xuất hiện 2 lần, các từ còn lại xuất hiện 1 lần.

Ràng buộc:

* `0 <= words.length <= 100000`
* `1 <= words[i].length <= 50`
* Các phần tử không phải `null`

### Bài 2: Group By Length

Độ khó: Trung bình

Đề bài:
Cho `String[] words`. Hãy trả về `Map<Integer, Integer>` trong đó key là độ dài của từ, value là số từ có độ dài đó.

Ví dụ 1:

Đầu vào:
```text
words = ["a", "go", "to", "java"]
```

Đầu ra:
```text
{1=1, 2=2, 4=1}
```

Giải thích:
Có một từ dài 1, hai từ dài 2, và một từ dài 4.

Ràng buộc:

* `0 <= words.length <= 100000`
* `1 <= words[i].length <= 100`
* Các phần tử không phải `null`

### Bài 3: Collect Unique Extensions

Độ khó: Trung bình

Đề bài:
Cho `String[] filenames`. Mỗi tên file có dạng `name.extension`. Hãy trả về mảng các extension duy nhất, theo đúng thứ tự xuất hiện đầu tiên. Nếu một file không có dấu chấm hoặc phần extension rỗng, bỏ qua file đó.

Ví dụ 1:

Đầu vào:
```text
filenames = ["app.java", "readme.md", "service.java", "Dockerfile", "notes.md"]
```

Đầu ra:
```text
["java", "md"]
```

Giải thích:
`"java"` xuất hiện đầu tiên, sau đó `"md"`. `Dockerfile` bị bỏ qua vì không có extension.

Ràng buộc:

* `0 <= filenames.length <= 100000`
* `0 <= filenames[i].length <= 200`
* Các phần tử không phải `null`

## Links

* [[001-Primitive-vs-Wrapper]]
* [[../Generics/001-what-is-T]]
* [[../Functional/003-Stream-API]]
* [Java SE 21, Local Variable Type Inference](https://docs.oracle.com/en/java/javase/21/language/local-variable-type-inference.html)
* [JEP 286, Local-Variable Type Inference](https://openjdk.org/jeps/286)
* [Style Guidelines for Local Variable Type Inference in Java](https://openjdk.org/projects/amber/guides/lvti-style-guide)


