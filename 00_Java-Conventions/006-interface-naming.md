# Interface Naming

## Why this file exists

Java dùng interface rất nhiều, đặc biệt trong API design và Spring DI. Đặt tên interface sai sẽ dẫn tới model sai về abstraction.

## Rules

- Interface vẫn dùng PascalCase.
- Ưu tiên tên theo capability hoặc role, không cần thêm prefix `I`.
- Không đặt interface chỉ để “có vẻ enterprise”.
- Nếu chỉ có một implementation hiện tại, interface vẫn ổn nếu nó đại diện cho boundary thật.
- Tránh cách đặt tên `IUserService` vì không tự nhiên trong hệ sinh thái Java.

## Good examples

```java
PaymentGateway
DiscountPolicy
NotificationSender
CacheStore
```

## Bad examples

```java
IPaymentGateway
UserServiceInterface
AbstractableProcessor
```

## About Impl suffix

- `DefaultPaymentGateway` hoặc `JdbcUserRepository` thường tốt hơn `PaymentGatewayImpl`.
- Chỉ dùng `Impl` khi thật sự chưa có tên implementation nào mang nghĩa tốt hơn.

## Interface decision matrix

| Situation | Interface? | Why |
|---|---|---|
| Boundary với external system | Có | Caller phụ thuộc capability, không phụ thuộc adapter |
| Nhiều implementation thật hoặc sẽ có trong test | Có thể có | Abstraction có payoff rõ |
| Chỉ một class nội bộ không cần mock/substitute | Thường không cần | Interface chỉ thêm indirection |
| Domain policy/rule thay đổi theo strategy | Có | `DiscountPolicy` đọc như capability |
| Interface chỉ để Spring DI “cho enterprise” | Không | Spring không yêu cầu interface cho mọi bean |

## Official references

- [JLS: Interfaces](https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html)
- [Spring Framework docs: Beans and dependency injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)

## Related rules

[[005-class-naming]]

[[011-code-organization]]
