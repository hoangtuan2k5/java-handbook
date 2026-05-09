# Custom Annotation

## What is it

`Custom annotation` là annotation do chính mình định nghĩa bằng `@interface` để diễn đạt metadata riêng của domain hoặc framework nội bộ.

Nó hữu ích khi annotation built-in của Java không đủ diễn đạt intent mà team muốn gắn trực tiếp lên code, ví dụ audit label, security marker, mapping hint, hoặc validation contract.

## How I used to misunderstand it

Mình từng nghĩ cứ tự tạo annotation là codebase sẽ “xịn” hơn.

Thực tế, annotation chỉ có giá trị khi có consumer rõ ràng đọc nó một cách nhất quán. Nếu không có processor, interceptor, scanner, validator, hoặc framework hook nào đọc lại nó, custom annotation chỉ là metadata bị bỏ quên.

## How it actually works

Khi thiết kế custom annotation, có ba quyết định rất quan trọng:

| Quyết định | Trả lời câu hỏi gì | Nếu chọn sai thì sao |
|---|---|---|
| `@Target` | Được đặt ở đâu | Dễ bị dùng sai chỗ |
| `@Retention` | Sống tới giai đoạn nào | Consumer không thấy được annotation |
| Attributes | Annotation mang dữ liệu gì | API annotation khó hiểu hoặc khó mở rộng |

### Một mental model hữu ích

```text
Custom annotation = declarative input
Consumer          = nơi đọc metadata và tạo ra behavior thật
```

Vì vậy, custom annotation gần giống một mini language cho framework hoặc tooling của bạn. Nó không phải business logic. Nó là input cho một cơ chế khác.

### Compile-time vs runtime boundary

| Nếu consumer là... | Retention thường cần gì |
|---|---|
| Annotation processor | `SOURCE` hoặc `CLASS` có thể đủ |
| Reflection utility hoặc Spring runtime scanner | Thường cần `RUNTIME` |

Rất nhiều lỗi tới từ việc chọn retention theo thói quen thay vì theo consumer thật.

## Code example

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AuditAction {
    String value();
}
```

Annotation này mới chỉ khai báo metadata. Nó chưa log gì cả. Logging chỉ xuất hiện khi một interceptor, aspect, hoặc reflection-based utility đọc `@AuditAction` rồi phản ứng lại.

## When to use / when NOT to use

Dùng custom annotation khi nhiều nơi cần khai báo cùng một loại metadata, và metadata đó có một consumer chung, ví dụ security rule, audit label, validation hint, hay mapping rule.

Đừng tạo custom annotation nếu logic chỉ dùng ở một chỗ nhỏ, hoặc method call trực tiếp đã đủ rõ. Khi không có shared consumer, annotation chỉ làm tăng API surface mà không tăng clarity.

### Small design checklist

- Annotation này đang mô tả metadata gì?
- Ai sẽ đọc nó?
- Giai đoạn nào đọc nó, compile time hay runtime?
- Default value có rõ nghĩa không?
- Tên annotation có nói đúng intent không?

## How this connects to Spring

Trong Spring, custom annotation thường được dùng cho meta-annotation, custom stereotype, security marker, validation marker, hoặc AOP pointcut marker.

Ví dụ, team có thể tạo `@InternalApi`, `@TenantRequired`, hoặc `@AuditAction`, rồi cho Spring interceptor, aspect, hoặc scanner đọc metadata đó để gắn behavior ở runtime.

## Gotchas

- Quên `@Retention(RetentionPolicy.RUNTIME)` thì runtime scanner hoặc Spring AOP logic sẽ không thấy annotation.
- `@Target` quá rộng làm annotation bị đặt sai chỗ và mất ý nghĩa.
- Default attribute mơ hồ khiến người đọc không biết bỏ trống có nghĩa gì.
- Đổi tên attribute sau này là breaking change cho mọi nơi đang dùng annotation.
- Nếu annotation name nghe rất “mạnh” nhưng không có consumer thật, người đọc sẽ kỳ vọng sai về behavior.

## Check yourself

- Vì sao custom annotation chỉ có giá trị khi có consumer rõ ràng?
- Khi nào `RUNTIME` là cần thiết, và khi nào chỉ làm metadata “sống” quá lâu?
- `@Target` giúp bảo vệ API annotation như thế nào?
- Nếu annotation có `value()` mặc định mơ hồ, người dùng annotation sẽ dễ hiểu sai điều gì?
- Meta-annotation trong Spring khác gì với việc chỉ tạo một marker annotation trống?

## Exercises

### Bài 1: Validate Custom Annotation Meta

Độ khó: Dễ

Đề bài:
Cho một retention label và một target label, chỉ trả về `true` khi retention là `"RUNTIME"` và target là `"TYPE"` hoặc `"METHOD"`.

Ví dụ 1:

Đầu vào:
```text
retention = "RUNTIME", target = "METHOD"
```

Đầu ra:
```text
true
```

Giải thích:
Configuration này có thể được đọc ở runtime trên một Spring interception target phổ biến.

Ràng buộc:

- Cả hai input đều là non-null
- `retention` là một trong `SOURCE`, `CLASS`, `RUNTIME`
- `target` là một trong `TYPE`, `METHOD`, `FIELD`, `PARAMETER`

### Bài 2: Select Secured Paths

Độ khó: Dễ

Đề bài:
Cho các array `paths` và `secured` theo cùng một thứ tự, trả về list các path có `secured[i]` bằng `true`.

Ví dụ 1:

Đầu vào:
```text
paths = ["/health", "/admin", "/profile"]
secured = [false, true, true]
```

Đầu ra:
```text
["/admin", "/profile"]
```

Giải thích:
Chỉ các endpoint được đánh dấu bảo vệ mới được chọn.

Ràng buộc:

- Cả hai array đều là non-null
- Cả hai array có cùng độ dài
- Trả về các path theo đúng thứ tự ban đầu

### Bài 3: Count Default Role Fallbacks

Độ khó: Trung bình

Đề bài:
Cho một array `configuredRoles` và một `defaultRole`, đếm xem có bao nhiêu entry sẽ dùng giá trị mặc định. Một entry được xem là dùng default khi nó là chuỗi rỗng `""`.

Ví dụ 1:

Đầu vào:
```text
configuredRoles = ["ADMIN", "", "USER", ""], defaultRole = "USER"
```

Đầu ra:
```text
2
```

Giải thích:
Hai annotation usage đang dựa vào giá trị role mặc định.

Ràng buộc:

- `configuredRoles` là non-null
- `defaultRole` là non-null
- Độ dài array nằm trong khoảng từ 0 đến 100000

## Links

- [[002-Annotation]]
- [[004-Dynamic-proxy]]
- [[005-annotation-processor]]
- `Retention` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/annotation/Retention.html
- `Target` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/annotation/Target.html
- `Inherited` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/annotation/Inherited.html
- JLS 9.6 Annotations: https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html#jls-9.6
- Spring meta-annotations: https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html#beans-meta-annotations
