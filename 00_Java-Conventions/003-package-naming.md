# Package Naming

## Why this file exists

Package name trong Java xuất hiện ở khắp nơi: import, stack trace, IDE navigation, Maven source tree. Đặt tên package tệ sẽ làm codebase trông lộn xộn rất nhanh.

## Rules

- Dùng lowercase toàn bộ.
- Không dùng dấu gạch ngang `-` trong package.
- Không dùng khoảng trắng.
- Ưu tiên tên ngắn nhưng mô tả đúng concept.
- Tên package nên phản ánh domain, feature, hoặc responsibility, không phản ánh trạng thái tạm thời.
- Nếu tên concept có nhiều từ, ghép liền lowercase thay vì snake case.
- Root package thường dùng reverse domain khi code cần định danh ổn định ngoài phạm vi project nhỏ, ví dụ domain `hoangtuan.dev` -> root package `dev.hoangtuan`.
- Package con nên ưu tiên business/domain boundary trong application code; technical package chỉ nên dùng khi nó thật sự là boundary rõ như `config`, `security`, `persistence`, hoặc `generated`.

## Good examples

```java
com.example.orders.order
com.example.orders.payment
com.example.orders.security
```

## Bad examples

```java
com.example.orders.PaymentService
com.example.orders.common_utils
com.example.orders.new-package
com.example.orders.temp2
```

## Naming heuristics

- `payment` tốt hơn `Payment`
- `security` tốt hơn `security-utils` nếu đây là package Java
- `customerprofile` tốt hơn `customer_profile` nếu cần ghép nhiều từ trong package
- `internal` chỉ tốt khi boundary thật sự là internal API, không phải nơi chứa code chưa biết đặt đâu

## Nuance

Không phải mọi package theo technical role đều sai. Trong app lớn, package toàn cục kiểu `controller`, `service`, `repository` thường phình to và làm mất domain boundary. Nhưng trong library nhỏ, generated code, framework adapter, hoặc Spring configuration, một package technical rõ nghĩa có thể hợp lý hơn ép mọi thứ thành domain package.

Rule thực dụng là: tên package phải giúp người đọc đoán được dependency direction và responsibility. Nếu package technical làm điều đó rõ hơn domain name giả tạo, nó vẫn là lựa chọn tốt.

## Notes

Java package không cần đẹp theo kiểu prose. Nó cần ổn định, predictable, và IDE-friendly.

## Official references

- [Java Code Conventions: Package Names](https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html)
- [JLS: Packages and Modules](https://docs.oracle.com/javase/specs/jls/se21/html/jls-7.html)

## Related rules

[[002-module-and-package-boundaries]]

[[004-file-naming]]
