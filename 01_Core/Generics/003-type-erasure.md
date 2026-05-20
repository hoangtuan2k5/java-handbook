# Type Erasure

## What is it

Type erasure là cơ chế Java dùng để implement generics bằng cách bỏ phần lớn thông tin type argument cụ thể ở runtime.

Mental model quan trọng là: generics giúp compiler kiểm tra type ở compile time, nhưng object runtime thường không mang đầy đủ type argument cụ thể như người mới hay tưởng tượng. `List<String>` và `List<Integer>` đều thường là cùng runtime class, dù một số generic metadata vẫn có thể tồn tại trong class/method/field signatures để reflection hoặc framework đọc.

## How I used to misunderstand it

Mình từng nghĩ `new ArrayList<String>()` sẽ tạo ra một runtime type khác hẳn `new ArrayList<Integer>()`.

Điều đó không đúng theo nghĩa object instance. Sau erasure, cả hai thường chỉ là `ArrayList` ở runtime. Đây là lý do nhiều hạn chế của generics nhìn có vẻ kỳ lạ lúc đầu, như không thể `new T()`, không thể `instanceof List<String>`, và đôi khi phải truyền thêm `Class<T>` hoặc type token khác.

## How it actually works

Compiler dùng generics để:

- kiểm tra type safety
- chèn cast cần thiết
- biến code generic thành dạng runtime tương thích với JVM cũ hơn

### Một vài hệ quả hay gặp

| Bạn muốn làm | Vì sao bị chặn |
|---|---|
| `new T()` | runtime không biết `T` cụ thể là gì |
| `instanceof List<String>` | generic argument bị erase |
| overload khác nhau chỉ bởi generic type arg | erasure làm signature runtime trùng nhau |

### Nhìn nhanh compile time vs runtime

| Tầng | Generics giúp gì |
|---|---|
| Compile time | kiểm tra type, giảm cast tay, bắt lỗi sớm |
| Runtime | thường không giữ đủ concrete type argument trên object instance |

Bound như `<T extends Number>` vẫn ảnh hưởng code sinh ra ở mức erased upper bound, nhưng concrete type argument như `String`, `Integer` thường không còn hiện diện đầy đủ trên object runtime.

Nuance quan trọng: erasure không có nghĩa mọi dấu vết generic đều biến mất khỏi `.class`. Java vẫn có thể lưu generic signature cho superclass, interface, field, method, hoặc parameter để reflection/framework đọc trong một số trường hợp. Điều bị mất ở runtime object identity là chuyện một `new ArrayList<String>()` không trở thành runtime class riêng khác với `new ArrayList<Integer>()`.

## Code example

```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        List<Integer> values = new ArrayList<>();

        System.out.println(names.getClass() == values.getClass()); // true
    }
}
```

## When to use / when NOT to use

Bạn không “chọn dùng” type erasure. Nó là cách Java generics hoạt động.

Điều cần làm là nhớ đến nó khi thiết kế API generic, reflection, factory, serialization, hoặc integration với framework cần runtime type.

Không nên giả định runtime còn giữ đủ generic information nếu bạn chưa verify rõ cơ chế ở chỗ đó.

## How this connects to real Java projects

Trong Spring Boot, hiểu type erasure giúp đọc code reflection, JSON mapping, generic bean resolution, hoặc HTTP client code tốt hơn.

Đó là lý do bạn thường gặp các thứ như `Class<T>` hoặc `ParameterizedTypeReference<T>`. Framework không phải làm màu. Nó đang cần bù lại phần type info mà erasure làm mờ đi.

## Gotchas

- Không thể `new T()` trực tiếp trong generic code thông thường.
- `instanceof List<String>` không hợp lệ.
- Raw type warning thường là tín hiệu type safety đang bị thủng.
- Đừng nhầm compile-time generic clarity với runtime type availability.
- Đừng nói “runtime mất hết generic info” theo nghĩa tuyệt đối; metadata generic có thể còn trong signature, nhưng object instance thường không giữ concrete type argument như bạn muốn.

## Check yourself

- Type erasure lấy đi thông tin gì ở runtime?
- Vì sao `List<String>` và `List<Integer>` thường có cùng runtime class?
- Tại sao nhiều API phải nhận thêm `Class<T>`?
- Vì sao `instanceof List<String>` không hợp lệ?
- Raw type warning thường cảnh báo rủi ro gì?
- Generic signature còn tồn tại ở đâu, và khác gì với runtime class của object instance?

## Exercises

### Bài 1: Compare Erased Runtime Classes
Độ khó: Dễ

Đề bài:
Cho một `List<String>` và một `List<Integer>`, trả về việc runtime class của chúng có bằng nhau hay không.

Ví dụ 1:
Đầu vào:
```text
listA = new ArrayList<String>(), listB = new ArrayList<Integer>()
```

Đầu ra:
```text
true
```

Giải thích:
Sau erasure, cả hai list dùng cùng một runtime class.

Ràng buộc:
- listA là non-null
- listB là non-null
- Chỉ so sánh runtime class

### Bài 2: Create Instance With Class Token
Độ khó: Trung bình

Đề bài:
Cho một `Class<T>` token, tạo và trả về một instance mới bằng no-arg constructor.

Ví dụ 1:
Đầu vào:
```text
type = StringBuilder.class
```

Đầu ra:
```text
new StringBuilder()
```

Giải thích:
Class token mang runtime type information mà chỉ riêng `T` thì không có.

Ràng buộc:
- type là non-null
- Type phải có no-arg constructor
- Wrap reflection failure trong runtime exception

### Bài 3: Detect Raw List Usage Risk
Độ khó: Trung bình

Đề bài:
Cho một flag cho biết list có đang được dùng như raw type hay không, trả về `true` nếu code nên bị xem là type-unsafe.

Ví dụ 1:
Đầu vào:
```text
usesRawType = true
```

Đầu ra:
```text
true
```

Giải thích:
Raw type bỏ qua các compile-time guarantee của generics.

Ràng buộc:
- Input chỉ là một boolean flag
- Trả về risk judgment trực tiếp
- Không inspect nội dung thực tế của bất kỳ list nào

## Links

- [[001-what-is-T]]
- [[002-wildcard]]
- [[004-generic-method-vs-generic-class]]
- Java tutorial, Type Erasure: https://docs.oracle.com/javase/tutorial/java/generics/erasure.html
- `ParameterizedType` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/ParameterizedType.html
