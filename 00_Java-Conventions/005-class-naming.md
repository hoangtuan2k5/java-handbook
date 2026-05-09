# Class Naming

## Why this file exists

Class là abstraction chính trong Java. Tên class ảnh hưởng trực tiếp đến readability của API và architecture.

## Rules

- Dùng PascalCase.
- Tên class nên là noun hoặc noun phrase.
- Tên phải phản ánh role thật của object.
- Tránh suffix mơ hồ như `Manager`, `Helper`, `Processor` nếu không nói rõ xử lý cái gì.
- Với Spring-style app, suffix như `Controller`, `Service`, `Repository`, `Config` chỉ dùng khi class thật sự đóng đúng role đó.

## Good examples

```java
UserController
PricingService
OrderRepository
DiscountPolicy
CsvReportWriter
```

## Bad examples

```java
UserManager
CommonHelper
DataProcessor
ThingService
```

## Class shape decision matrix

| Need | Prefer | Avoid |
|---|---|---|
| Domain value | specific noun like `Money`, `EmailAddress` | `Data`, `Info` |
| Business rule | `DiscountPolicy`, `TaxCalculator` | `RuleProcessor` mơ hồ |
| External integration | `StripePaymentGateway`, `SmtpNotificationSender` | `PaymentManager` |
| Spring boundary | `OrderController`, `OrderRepository` khi đúng role | suffix framework cho class không đúng vai |
| Stateless utility | domain-specific class hoặc method gần owner | global `CommonUtil` |

## Notes

Tên class tốt thường trả lời được câu hỏi: “Object này đại diện cho cái gì?” chứ không chỉ “nó làm một ít việc gì đó”.

## Official references

- [Java Code Conventions: Naming Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html)
- [JLS: Classes](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.1)

## Related rules

[[006-interface-naming]]

[[007-enum-record-annotation-naming]]
