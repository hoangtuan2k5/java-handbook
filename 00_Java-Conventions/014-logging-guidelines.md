# Logging Guidelines

## Why this file exists

Log tốt giúp debug production. Log tệ làm tăng noise, lộ dữ liệu nhạy cảm, và khiến người đọc không biết chuyện gì quan trọng.

## Rules

- Log message nên nói điều gì vừa xảy ra và tại sao nó quan trọng.
- Không log dữ liệu nhạy cảm như password, secret, token, private key.
- Dùng level hợp lý: `debug`, `info`, `warn`, `error`.
- Không spam log trong loop nóng nếu không có lý do rất rõ.
- Khi log exception, giữ context business ngắn gọn thay vì chỉ `e.getMessage()`.

## Good examples

```java
log.warn("Payment retry limit reached for orderId={}", orderId);
log.error("Failed to publish invoice event for invoiceId={}", invoiceId, exception);
```

## Bad examples

```java
log.info("here");
log.error("something went wrong");
log.info("token={}", token);
```

## Level decision matrix

| Level | Use when | Avoid when |
|---|---|---|
| `debug` | Chi tiết phục vụ investigation | Event production cần thấy mặc định |
| `info` | Business/system event bình thường | Log từng item trong loop lớn |
| `warn` | Bất thường nhưng recover được | Failure cần action ngay |
| `error` | Operation thất bại cần chú ý | Error đã được boundary log đầy đủ |

## Context checklist

Log production hữu ích thường có vài field sau:

- operation name
- entity id như `orderId`, `invoiceId`, `userId`
- external dependency như `paymentGateway`, `kafka`, `postgres`
- retry count hoặc attempt number
- request/correlation id nếu hệ thống có tracing
- outcome hoặc reason ngắn gọn

## Notes

Log nên giúp người vận hành trả lời: chuyện gì xảy ra, với entity nào, ở operation nào, và cần làm gì tiếp theo. Đừng log secret để “debug nhanh”; hậu quả bảo mật thường lớn hơn lợi ích.

## Official references

- [SLF4J manual](https://www.slf4j.org/manual.html)
- [Logback manual](https://logback.qos.ch/manual/)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)

## Related rules

[[013-exception-handling-guidelines]]

[[016-common-java-anti-patterns]]
