# Common Java Anti-Patterns

## Why this file exists

Biết rule chưa đủ. Cần biết những mùi sai phổ biến để tránh lặp lại chúng.

## Common anti-patterns

- `Util` class phình to chứa đủ thứ không liên quan.
- Package `common`, `misc`, `helper` trở thành bãi đáp cho code không biết để đâu.
- Class tên rất chung như `Processor`, `Manager`, `Data`, `Info`.
- Catch `Exception` rồi nuốt luôn.
- Logging quá nhiều nhưng không có actionable context.
- Comment giải thích điều code vừa nói lại.
- Tạo interface chỉ vì nghĩ “enterprise code phải có interface”.
- Dùng `Impl` ở mọi nơi dù implementation có tên cụ thể tốt hơn.

## What to do instead

- Tách theo responsibility thật.
- Đặt tên theo domain language.
- Dùng boundary rõ ràng giữa package.
- Viết method nhỏ và API đọc tự nhiên.
- Review tên class/file trước khi review implementation detail.

## Nuance

Các tên như `Util`, `common`, `manager`, `helper`, hoặc `Impl` là smell khi chúng che giấu responsibility, không phải keyword bị cấm tuyệt đối. `MathUtils` trong library nhỏ, `common` trong generated code, hoặc `PaymentGatewayImpl` trong prototype ngắn hạn có thể chấp nhận được nếu context đủ rõ.

Vấn đề cần review là: tên đó có giúp người đọc hiểu boundary và responsibility không. Nếu không, đổi sang tên domain-specific như `CsvReportWriter`, `DefaultPaymentGateway`, `MoneyFormatter`, hoặc đặt helper package-private gần nơi dùng.

## Anti-pattern checklist by symptom

| Symptom | Likely anti-pattern | Better direction |
|---|---|---|
| Class khó đặt tên | responsibility mơ hồ | split or rename by domain role |
| Package `common` ngày càng lớn | boundary không rõ | move code near owner |
| Interface chỉ có một impl vô nghĩa | abstraction không có payoff | remove or rename boundary by capability |
| Log nhiều nhưng debug vẫn khó | thiếu context hoặc duplicate logs | log once with actionable fields |
| Exception bị nuốt | catch without recovery | catch specific or rethrow with context |
| Singleton bean có mutable field | shared state bug | make stateless or protect state |

## Official references

- [Java Code Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html)
- [Spring Framework docs](https://docs.spring.io/spring-framework/reference/)

## Related rules

[[003-package-naming]]

[[005-class-naming]]

[[013-exception-handling-guidelines]]

[[014-logging-guidelines]]
