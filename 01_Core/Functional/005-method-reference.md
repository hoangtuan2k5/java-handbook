# method reference

## What is it

Method reference là cú pháp như `String::trim`, `System.out::println`, `Integer::parseInt`, `User::new` để tham chiếu tới method hoặc constructor đã tồn tại.

Mental model đơn giản nhất là: nếu lambda chỉ đang **forward call y nguyên** tới method có sẵn, method reference thường là cách viết gọn và rõ hơn.

## How I used to misunderstand it

Mình từng xem method reference là cú pháp “cho đẹp”.

Thực ra giá trị của nó nằm ở việc giảm nhiễu. Khi lambda chỉ làm đúng một việc là gọi method đã có, method reference giúp người đọc thấy ngay ý định đó. Ngược lại, nếu lambda còn đổi thứ tự argument, thêm branch, hay thêm domain rule, ép method reference vào sẽ phản tác dụng.

## How it actually works

Method reference vẫn cần target functional interface giống lambda. Compiler sẽ map signature của functional interface vào method phù hợp.

### Bốn dạng hay gặp

| Dạng | Ví dụ | Nghĩ là |
|---|---|---|
| Static method reference | `Integer::parseInt` | gọi static method có sẵn |
| Instance method của object cụ thể | `System.out::println` | luôn gọi trên object này |
| Instance method của một object bất kỳ thuộc type đó | `String::trim` | input trở thành receiver |
| Constructor reference | `User::new` | dùng constructor làm factory |

### So sánh nhanh với lambda

| Nếu bạn đang viết | Có thể đổi sang |
|---|---|
| `name -> name.trim()` | `String::trim` |
| `x -> Integer.parseInt(x)` | `Integer::parseInt` |
| `x -> System.out.println(x)` | `System.out::println` |
| `name -> new User(name)` | `User::new` |

Nếu lambda còn làm nhiều hơn thế, thường nên giữ lambda.

## Code example

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> names = List.of(" linh ", " an ");
        names.stream()
                .map(String::trim)
                .forEach(System.out::println);
    }
}
```

## When to use / when NOT to use

Dùng method reference khi nó thay thế lambda gần như một-một và đọc tự nhiên hơn.

Không nên dùng method reference nếu lambda còn có transform nhỏ mà người đọc sẽ phải đoán ngược. Nếu method name quá generic và làm mất context business, một lambda rõ ràng hoặc method domain-specific đặt tên tốt còn dễ hiểu hơn.

## How this connects to real Java projects

Trong Spring Boot, method reference hay xuất hiện trong stream mapping, comparator, event callback, scheduler setup, repository/service adapter nhỏ, hoặc constructor mapping DTO.

Nó hợp khi logic chỉ là nối các mảnh có sẵn. Nếu mapping có domain rule, validation, hoặc branching, một method đặt tên rõ nghĩa ở mapper/service class thường tốt hơn chuỗi method reference dài.

## Gotchas

- Method reference không phải lúc nào cũng rõ hơn lambda.
- Overload hoặc generic phức tạp có thể làm type inference khó đọc.
- Constructor reference gọn, nhưng đừng để nó che đi constructor có validation hoặc side effect khó đoán.
- `String::trim` không có nghĩa là gọi trên class `String`, mà là gọi `trim()` trên từng phần tử `String` đi qua pipeline.

## Handbook rule

- Dùng method reference khi nó thay lambda gần như một-một và đọc tự nhiên hơn.
- Khi cần transform thêm hoặc context business, lambda đặt tên rõ tốt hơn method reference generic.
- Constructor reference (`Foo::new`) đẹp nhưng phải nhớ side effect/validation trong constructor.
- `Type::method` dạng instance reference áp dụng trên từng phần tử pipeline, không gọi trên class.
- Overload nặng và generic phức tạp có thể làm type inference khó đọc; ghi explicit type khi cần.

## Check yourself

- Khi nào method reference rõ hơn lambda, và khi nào không?
- `String::trim` khác `System.out::println` ở chỗ nào về mental model gọi method?
- Vì sao method reference vẫn cần functional interface?
- Nếu lambda còn thêm branch logic, vì sao không nên ép sang method reference?
- Constructor reference hợp nhất khi nào?

## Exercises

### Bài 1: Trim Names With Method Reference
Độ khó: Dễ

Đề bài:
Cho một list các tên có extra space, trả về các tên đã được trim bằng method reference style.

Ví dụ 1:
Đầu vào:
```text
names = [" a ", " b "]
```

Đầu ra:
```text
["a", "b"]
```

Giải thích:
Mỗi string được transform bằng cách tham chiếu trực tiếp tới một method đã có.

Ràng buộc:
- 0 <= names.length <= 100000
- names[i] là non-null
- Giữ nguyên encounter order

### Bài 2: Parse Text Numbers To Integers
Độ khó: Trung bình

Đề bài:
Cho một list các numeric string, trả về một list integer đã được parse bằng method reference tới parser.

Ví dụ 1:
Đầu vào:
```text
texts = ["1", "20", "3"]
```

Đầu ra:
```text
[1, 20, 3]
```

Giải thích:
Mỗi text value được convert bằng cùng một parse method có sẵn.

Ràng buộc:
- 0 <= texts.length <= 100000
- texts[i] là non-null và có dạng số
- Kích thước output phải khớp với kích thước input

### Bài 3: Build User Records With Constructor Reference
Độ khó: Trung bình

Đề bài:
Cho một list các tên, trả về một list các object `UserRecord` được tạo từ các tên đó bằng constructor reference style.

Ví dụ 1:
Đầu vào:
```text
names = ["Linh", "An"]
```

Đầu ra:
```text
`[UserRecord[name=Linh], UserRecord[name=An]]`
```

Giải thích:
Constructor được tham chiếu trực tiếp như mapping function.

Ràng buộc:
- 0 <= names.length <= 100000
- names[i] là non-null
- Giữ nguyên thứ tự đầu vào

## Links

- [[001-Lambda]]
- [[002-Functional-Interface]]
- [[003-Stream-API]]
- Java tutorial, Method References: https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html
- `BiFunction` Javadoc, useful for constructor references with two args: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/BiFunction.html
