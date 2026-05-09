# Enum, Record, and Annotation Naming

## Why this file exists

`enum`, `record`, và `annotation` là những type rất Java-specific. Chúng có naming expectation riêng và nên được xử lý rõ ràng.

## Rules

### Enum

- Tên enum dùng PascalCase.
- Enum constant dùng `UPPER_SNAKE_CASE`.
- Tên enum nên là category, không phải một giá trị cụ thể.

### Record

- Record name dùng PascalCase như class.
- Tên nên phản ánh immutable data carrier.
- Tránh dùng record cho object có lifecycle/phức tạp behavior-heavy.

### Annotation

- Annotation type dùng PascalCase.
- Tên annotation thường là adjective hoặc marker rõ nghĩa.
- Nếu annotation dùng để gắn behavior, tên nên gợi đúng contract.

## Good examples

```java
enum OrderStatus { PENDING, PAID, FAILED }
record UserSummary(String id, String name) {}
@interface Audited {}
@interface Retryable {}
```

## Bad examples

```java
enum PendingOrPaid {}
record DataHolder(String value) {}
@interface DoStuff {}
```

## Nuance

Enum constant dùng `UPPER_SNAKE_CASE` vì chúng là named constants. Record component thì ngược lại: component name nên dùng lowerCamelCase như field/accessor bình thường, ví dụ `record UserSummary(String userId, String displayName) {}`.

Annotation name nên đọc như marker hoặc contract. `@Audited` nói object/method này được audit; `@Retryable` nói operation có retry behavior. Tên kiểu `@DoStuff` mơ hồ vì không nói contract nào được gắn vào target.

## Official references

- [JLS: Enum Classes](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9)
- [JLS: Record Classes](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.10)
- [JLS: Annotation Interfaces](https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html#jls-9.6)

## Related rules

[[005-class-naming]]

[[010-constants-and-generics-naming]]
