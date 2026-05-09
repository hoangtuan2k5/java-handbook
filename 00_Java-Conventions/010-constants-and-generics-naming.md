# Constants and Generics Naming

## Why this file exists

Constants và generic type parameters có convention khá ổn định trong Java. Bám theo convention này làm code dễ đọc hơn ngay lập tức.

## Rules

### Constants

- Dùng `UPPER_SNAKE_CASE` cho constant thật sự, thường là `static final` field có meaning ổn định.
- Không đổi mọi `final` variable thành `UPPER_SNAKE_CASE`; local `final` variable vẫn thường dùng lowerCamelCase.
- Chỉ dùng constant cho giá trị có meaning ổn định hoặc được share ở nhiều nơi.
- Không biến mọi literal thành constant nếu nó chỉ dùng một chỗ và không tăng clarity.

### Generics

- Dùng tên một chữ cái khi là convention phổ biến.
- `T` cho type tổng quát.
- `E` cho element.
- `K`, `V` cho map key/value.
- `R` cho result/return type.
- Nếu generic có meaning đặc biệt, có thể dùng tên dài hơn như `RequestT` hoặc `PayloadT`, nhưng đừng lạm dụng.

## Good examples

```java
private static final int DEFAULT_TIMEOUT_SECONDS = 30;
class Box<T> {}
interface Cache<K, V> {}
```

## Bad examples

```java
private static final int timeout = 30;
class Box<TypeParameterBecauseWhyNot> {}
interface Cache<A, B> {}
```

## Nuance

`static final` không tự động đồng nghĩa với “constant” theo nghĩa style. Một logger, một injected collaborator giả lập trong test, hoặc một object mutable được giữ trong `static final` không nên đặt như hằng số business. Casing nên phản ánh vai trò đọc hiểu, không chỉ modifier.

Với generic type parameter, tên một chữ cái tốt khi convention đã rõ. Nếu type parameter xuất hiện ở API public phức tạp, tên dài hơn như `RequestT` hoặc `ResponseT` có thể tăng clarity hơn `T` và `R` trơ trọi.

## Official references

- [Java Code Conventions: Constants](https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html)
- [JLS: Type Variables](https://docs.oracle.com/javase/specs/jls/se21/html/jls-4.html#jls-4.4)

## Related rules

[[007-enum-record-annotation-naming]]

[[009-variable-parameter-field-naming]]
