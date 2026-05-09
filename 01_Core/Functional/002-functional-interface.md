# functional interface

## What is it

Functional interface là interface có đúng một abstract method, nên nó có thể làm target type cho lambda hoặc method reference.

Ví dụ quen thuộc:

- `Predicate<T>`
- `Function<T, R>`
- `Consumer<T>`
- `Supplier<T>`

Mental model: functional interface là **ổ cắm contract**, còn lambda là **thứ bạn cắm vào**.

## How I used to misunderstand it

Mình từng nghĩ functional interface là một loại interface “đặc biệt” của Java 8 trở đi.

Thực ra nó vẫn là interface bình thường. Điểm khác chỉ là nó thỏa điều kiện single abstract method. `@FunctionalInterface` không bắt buộc để code chạy, nhưng rất đáng dùng vì compiler sẽ bảo vệ contract đó cho bạn nếu ai đó lỡ thêm abstract method thứ hai.

Một hiểu nhầm khác là hễ có callback thì nên tạo custom functional interface mới. Thường không cần. JDK đã có sẵn rất nhiều contract chung khá tốt rồi.

## How it actually works

Compiler nhìn vào target type và xem interface đó có đúng một abstract method hay không.

Functional interface vẫn có thể chứa:

- `default` method
- `static` method
- method từ `Object`

Những thứ đó không phá rule, vì lambda chỉ cần implement abstract method chính.

### So sánh các contract phổ biến

| Type | Nhận vào | Trả về | Dùng khi |
|---|---|---|---|
| `Predicate<T>` | `T` | `boolean` | kiểm tra điều kiện |
| `Function<T, R>` | `T` | `R` | biến đổi dữ liệu |
| `Consumer<T>` | `T` | `void` | nhận dữ liệu để xử lý side effect |
| `Supplier<T>` | không có input | `T` | cung cấp giá trị khi cần |

### Mental flow

```text
Need to pass behavior
    -> behavior must match a contract
    -> contract is usually a functional interface
    -> lambda or method reference fills that contract
```

Khi bạn thiết kế API, chọn functional interface đúng giúp caller nhìn signature là hiểu callback này nhận gì và trả gì.

## Code example

```java
@FunctionalInterface
interface PriceFormatter {
    String format(int price);
}

public class Main {
    public static void main(String[] args) {
        PriceFormatter formatter = price -> price + " VND";
        System.out.println(formatter.format(100));
    }
}
```

Ở đây, lambda chỉ hợp lệ vì `PriceFormatter` có đúng một abstract method là `format(int)`.

## When to use / when NOT to use

Dùng functional interface khi API cần nhận một behavior nhỏ, rõ input/output, như filter rule, mapping rule, action callback.

Không nên tự tạo custom functional interface nếu JDK đã có type rất khớp như `Function`, `Predicate`, `Consumer`, `Supplier`. Custom type chỉ đáng tạo khi tên domain riêng giúp intent rõ hơn, hoặc signature thật sự không khớp loại built-in.

## How this connects to Spring

Trong Spring Boot, functional interface xuất hiện ở callback style API, mapping function, strategy nhỏ, scheduler task, retry callback, hoặc functional bean registration.

Một custom functional interface đặt tên tốt có thể làm domain rule dễ đọc hơn. Nhưng nếu project có quá nhiều tiny interface chỉ bọc lại JDK types, người đọc sẽ phải nhớ thêm khái niệm mà không được lợi bao nhiêu.

## Gotchas

- `@FunctionalInterface` không bắt buộc nhưng rất nên dùng.
- Đừng tạo custom interface chỉ để bọc lại `Function` hay `Predicate` y hệt.
- Chữ “functional” không tự làm design tốt hơn, contract input/output vẫn phải rõ.
- Một method reference hay lambda chỉ hợp lệ khi signature khớp abstract method.

## Check yourself

- Vì sao lambda cần functional interface làm target type?
- `@FunctionalInterface` giúp gì, và nó không giúp gì?
- Khi nào dùng `Function<T, R>` là đủ, không cần tạo interface riêng?
- Vì sao interface có `default` method vẫn có thể là functional interface?
- Nếu callback chỉ nhận input và trả `boolean`, type built-in nào thường hợp nhất?

## Exercises

### Bài 1: Apply Discount Formatter
Độ khó: Dễ

Đề bài:
Cho một giá và một formatter functional interface, trả về formatted price string.

Ví dụ 1:
Đầu vào:
```text
price = 120, formatter = p -> p + " VND"
```

Đầu ra:
```text
"120 VND"
```

Giải thích:
Formatter quyết định output string được build như thế nào.

Ràng buộc:
- formatter là non-null
- price >= 0
- Trả về đúng formatter result

### Bài 2: Count Matching Values With Predicate
Độ khó: Trung bình

Đề bài:
Cho một list số nguyên và một predicate, trả về số lượng value thỏa predicate đó.

Ví dụ 1:
Đầu vào:
```text
values = [1, 2, 3, 4], predicate = even
```

Đầu ra:
```text
2
```

Giải thích:
Chỉ có `2` và `4` khớp predicate.

Ràng buộc:
- 0 <= values.length <= 100000
- values[i] là non-null
- predicate là non-null

### Bài 3: Build Custom Greeting Strategy
Độ khó: Trung bình

Đề bài:
Cho một name và một custom greeting functional interface, trả về greeting text được tạo ra bởi strategy đó.

Ví dụ 1:
Đầu vào:
```text
name = "Linh", greeter = n -> "Hello, " + n
```

Đầu ra:
```text
"Hello, Linh"
```

Giải thích:
Interface này trừu tượng hóa một kiểu greeting behavior.

Ràng buộc:
- name là non-null
- greeter là non-null
- Output chỉ phụ thuộc vào strategy được cung cấp

## Links

- [[001-Lambda]]
- [[005-Method-reference]]
- [[003-Stream-API]]
- `FunctionalInterface` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/FunctionalInterface.html
- `java.util.function` package summary: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html
