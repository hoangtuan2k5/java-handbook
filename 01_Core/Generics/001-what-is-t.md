# what is T

## What is it

`T` trong Java generics chỉ là tên phổ biến cho một **type parameter**. Nó nói rằng kiểu dữ liệu thật sẽ được quyết định khi class hoặc method được dùng.

Mental model tốt nhất là: `T` là **placeholder cho một kiểu chưa biết ở compile time của người viết class**, nhưng sẽ được caller làm rõ khi dùng API.

Nó không phải một class bí ẩn tên `T` trong JVM.

## How I used to misunderstand it

Mình từng nhìn `List<T>` và tưởng `T` là một object type thật mà Java sẽ giữ nguyên đầy đủ ở runtime.

Điều đó không đúng. `T` chủ yếu giúp compiler kiểm tra type consistency ở compile time. Generics làm API chính xác hơn và giúp bỏ cast tay ở nhiều chỗ. Giá trị thực tế của nó thường đơn giản như vậy, chứ không phải một lớp trừu tượng quá cao siêu.

## How it actually works

Type parameter như `T`, `E`, `K`, `V`, `R` chỉ là tên quy ước. Điều quan trọng là **vai trò** và **scope** của chúng.

Nếu class là `Box<T>`, mọi method trong class có thể dùng cùng một type parameter đó. Khi bạn tạo `Box<String>`, compiler sẽ kiểm tra box này chỉ chứa `String`.

### Các tên hay gặp

| Tên | Thường nghĩa là | Ví dụ quen thuộc |
|---|---|---|
| `T` | type nói chung | `Box<T>` |
| `E` | element | `List<E>` |
| `K` | key | `Map<K, V>` |
| `V` | value | `Map<K, V>` |
| `R` | result | `Function<T, R>` |

Đây là convention, không phải luật ngôn ngữ. Nhưng convention tốt giúp đọc signature nhanh hơn.

### Mental model ngắn

```text
Author of API: "Tôi chưa biết type cụ thể là gì"
Caller of API:  "Tôi sẽ cung cấp type cụ thể khi dùng"
Compiler:       "Tôi sẽ kiểm tra mọi chỗ có nhất quán không"
```

## Code example

```java
class Box<T> {
    private final T value;

    Box(T value) {
        this.value = value;
    }

    T getValue() {
        return value;
    }
}

public class Main {
    public static void main(String[] args) {
        Box<String> box = new Box<>("java");
        System.out.println(box.getValue()); // java
    }
}
```

Ở đây `T` là `String` cho instance `box` này. Compiler biết `getValue()` trả về `String`, nên caller không cần cast.

## When to use / when NOT to use

Dùng generics khi API nên làm việc với nhiều type nhưng vẫn giữ type safety, như collection, wrapper, mapper, utility method.

Không cần generic hóa mọi class. Nếu class chỉ phục vụ đúng một concrete type và việc generic hóa không mang lại lợi ích đọc hoặc tái sử dụng rõ ràng, abstraction đó thường đang thừa.

## How this connects to Spring

Trong Spring Boot, generics xuất hiện ở `ResponseEntity<T>`, `JpaRepository<T, ID>`, event handler, mapper utility, wrapper response, và nhiều abstraction khác của framework.

Hiểu `T` giúp bạn đọc signature bình tĩnh hơn. Phần lớn thời gian, generics chỉ đang mô tả “framework sẽ làm việc với kiểu nào ở đây”.

## Gotchas

- `T` chỉ là tên quy ước, không có ma thuật riêng.
- Generic type giúp type safety ở compile time, không có nghĩa runtime luôn giữ đủ thông tin type cụ thể.
- Đừng đặt type parameter mơ hồ nếu một tên ngắn nhưng domain-specific sẽ rõ hơn.
- Nếu chỉ một method cần generic, chưa chắc cả class nên generic.

## Check yourself

- `T` giúp compiler việc gì cụ thể?
- Vì sao `T` không phải là một class thật trong runtime theo cách nhiều người mới tưởng tượng?
- `Box<T>` và `Box<String>` khác nhau ở tầng nào?
- Khi nào generic hóa cả class là thừa?
- `T`, `E`, `K`, `V`, `R` khác nhau ở luật ngôn ngữ hay ở convention đặt tên?

## Exercises

### Bài 1: Create Generic Box
Độ khó: Dễ

Đề bài:
Cho một giá trị kiểu `T`, trả về generic `Box<T>` lưu trữ đúng giá trị đó.

Ví dụ 1:
Đầu vào:
```text
value = "java"
```

Đầu ra:
```text
Box<String>("java")
```

Giải thích:
`Box` giữ nguyên compile-time type giống với giá trị đầu vào.

Ràng buộc:
- value là non-null
- Giữ nguyên original value
- Không dùng raw type `Box`

### Bài 2: Return First Item Safely
Độ khó: Trung bình

Đề bài:
Cho một list các giá trị `T`, trả về item đầu tiên và giữ nguyên generic type của nó. Trả về `null` khi list rỗng.

Ví dụ 1:
Đầu vào:
```text
items = ["A", "B"]
```

Đầu ra:
```text
"A"
```

Giải thích:
Method trả về một giá trị có cùng generic type với các phần tử trong list.

Ràng buộc:
- 0 <= items.length <= 100000
- items[i] là non-null
- Chỉ trả về `null` cho input rỗng

### Bài 3: Swap Pair Values
Độ khó: Trung bình

Đề bài:
Cho một generic pair gồm hai giá trị cùng type, trả về một pair mới với left và right bị hoán đổi.

Ví dụ 1:
Đầu vào:
```text
pair = Pair<Integer>(1, 2)
```

Đầu ra:
```text
Pair<Integer>(2, 1)
```

Giải thích:
Kết quả giữ nguyên generic type parameter trong khi đảo vị trí hai phần tử.

Ràng buộc:
- pair là non-null
- left và right là non-null
- Giữ nguyên generic type parameter

## Links

- [[002-wildcard]]
- [[003-type-erasure]]
- [[004-generic-method-vs-generic-class]]
- Java tutorial, Generic Types: https://docs.oracle.com/javase/tutorial/java/generics/types.html
- Java tutorial, Type Parameters Naming Conventions: https://docs.oracle.com/javase/tutorial/java/generics/types.html
