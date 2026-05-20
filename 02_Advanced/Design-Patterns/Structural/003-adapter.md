# Adapter

## What is it

`Adapter` là structural pattern dùng để chuyển một interface sẵn có sang interface mà caller hiện cần.

Nó giải quyết bài toán “hai bên nói gần giống nhau nhưng không khớp contract”. Thay vì sửa caller ở nhiều nơi hoặc sửa legacy class không nên đụng vào, adapter đứng giữa để translate.

Điểm đáng học nhất là `Adapter` không thêm business capability mới. Nó làm hai contract tương thích nhau.

### Quick distinction

| Câu hỏi | `Adapter` trả lời thế nào |
|---|---|
| Caller đang cần một contract khác contract cũ? | Có |
| Có nên sửa legacy code hoặc third-party code trực tiếp không? | Thường không |
| Lợi ích chính | cô lập translation ở một chỗ |
| Giá phải trả | thêm lớp chuyển đổi, có nguy cơ mapping sai |
| Pattern dễ nhầm | `Facade` |

## How I used to misunderstand it

Mình từng nghĩ adapter chỉ là rename method. Thực tế adapter có thể map field, đổi format dữ liệu, normalize status, hoặc che giấu khác biệt giữa API cũ và API mới.

Nếu chỉ nhìn nó như rename method, ta sẽ bỏ sót cost thật của pattern: translation logic phải đúng, đầy đủ, và có thể cần test rất kỹ ở boundary.

## How it actually works

Adapter implement target interface mà caller mong muốn, nhưng bên trong nó gọi adaptee cũ rồi chuyển đổi input hoặc output sang shape mới.

Trong Java, composition thường an toàn và rõ hơn inheritance adapter.

```text
caller dùng target interface
       -> adapter
            -> adaptee cũ
            -> translate kết quả về contract mới
```

### Adapter vs close alternatives

| Nếu mục tiêu là... | Pattern gần hơn |
|---|---|
| Chuyển contract cũ sang contract mới | `Adapter` |
| Đơn giản hóa nhiều subsystem dưới một API mới | `Facade` |
| Bọc object để gatekeep hoặc cache | `Proxy` |

## Code example

```java
class LegacyUserRecord {
    String statusCode() {
        return "A";
    }
}

class UserStatusAdapter {
    static String adapt(LegacyUserRecord record) {
        return switch (record.statusCode()) {
            case "A" -> "ACTIVE";
            case "D" -> "DISABLED";
            default -> "UNKNOWN";
        };
    }
}
```

Ví dụ này cho thấy caller không cần biết code cũ dùng `A` hay `D`. Nó chỉ thấy vocabulary mới như `ACTIVE` và `DISABLED`.

## When to use / when NOT to use

Use `Adapter` khi:

- contract cũ và mới không khớp nhưng vẫn cần tái sử dụng code cũ hoặc API external
- muốn cô lập mapping và normalization ở một boundary rõ ràng
- không kiểm soát hoặc không muốn sửa trực tiếp legacy code

Do NOT use `Adapter` khi:

- bạn kiểm soát cả hai phía và có thể sửa thẳng contract cho đúng
- mục tiêu là cộng thêm business policy mới
- mục tiêu là đơn giản hóa nhiều subsystem cùng lúc chứ không phải translate một contract

Misconception hay gặp là dùng adapter như nơi nhét business rule. Khi đó nó không còn là translator nữa.

## How this connects to real Java projects

Trong Spring app, adapter xuất hiện khi map DTO external sang domain model, bọc third-party client vào interface nội bộ, hoặc chuẩn hóa response và status từ hệ thống cũ trước khi service layer dùng tiếp.

`Converter` và các lớp mapping ở integration boundary thường là chỗ pattern này xuất hiện rõ nhất.

## Gotchas

- Adapter dễ trở thành nơi nhồi business rule nếu không giữ ranh giới translation rõ ràng.
- Mapping thiếu field hoặc normalize sai enum và status sẽ gây bug dữ liệu âm thầm.
- Một adapter cho quá nhiều shape dữ liệu khác nhau thường là dấu hiệu abstraction chưa đúng.
- Nếu caller vẫn phải biết chi tiết contract cũ, adapter chưa che được complexity thật sự.
- Naming không rõ giữa target model và legacy model dễ làm mapping khó hiểu.

## Check yourself

- `Adapter` khác `Facade` ở mục tiêu chính như thế nào?
- Vì sao adapter thường nên nằm ở integration boundary?
- Nếu adapter chứa nhiều business rule, dấu hiệu đó nói gì về thiết kế?
- Khi nào sửa thẳng contract sẽ tốt hơn thêm adapter?
- Một mapping bug trong adapter nguy hiểm ở chỗ nào?

## Exercises

### Exercise 1: Should Use Adapter

Độ khó: Easy

Đề bài:
Cho `interfaceMismatch`, `canModifyLegacyCode`, và `newApiRequired`. Hãy trả về `true` chỉ khi có interface mismatch, cần một API mới, và legacy code không nên bị sửa trực tiếp.

Ví dụ 1:

Đầu vào:
```text
interfaceMismatch = true, canModifyLegacyCode = false, newApiRequired = true
```

Đầu ra:
```text
true
```

Giải thích:
Đây là trường hợp kinh điển để đưa vào một adapter.

Ràng buộc:

- Tất cả input đều là boolean
- Trả về một boolean
- Bỏ qua các converter do framework sinh ra

### Exercise 2: Map Legacy Status

Độ khó: Easy

Đề bài:
Cho `legacyCode`. Hãy trả về `"ACTIVE"` cho `"A"`, `"DISABLED"` cho `"D"`, `"PENDING"` cho `"P"`, và `"UNKNOWN"` cho mọi giá trị khác.

Ví dụ 1:

Đầu vào:
```text
legacyCode = "D"
```

Đầu ra:
```text
"DISABLED"
```

Giải thích:
Adapter này chuyển old status code sang vocabulary của hệ thống mới.

Ràng buộc:

- `legacyCode` là non-null
- Việc so sánh có phân biệt hoa thường
- Output phải là một trong bốn string chính xác đã nêu

### Exercise 3: Build Adapted User Line

Độ khó: Medium

Đề bài:
Cho `legacyId`, `legacyName`, và `legacyCode`. Hãy trả về một string theo đúng format `"id=<legacyId>,name=<legacyName>,status=<mappedStatus>"`, trong đó `mappedStatus` dùng các mapping rule của Exercise 2.

Ví dụ 1:

Đầu vào:
```text
legacyId = "u-9", legacyName = "An", legacyCode = "A"
```

Đầu ra:
```text
"id=u-9,name=An,status=ACTIVE"
```

Giải thích:
Output này mô phỏng một adapted record khớp với interface mới mà caller mong đợi.

Ràng buộc:

- `legacyId`, `legacyName`, và `legacyCode` đều là non-null
- Output format phải khớp chính xác
- Mapped status phải tuân theo các rule của Exercise 2

## Links

- [[004-Facade]]
- [[001-Proxy]]
- [Refactoring Guru, Adapter](https://refactoring.guru/design-patterns/adapter)
- [Spring Framework, Type Conversion](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/typeconversion.html)
- [Martin Fowler, Gateway](https://martinfowler.com/articles/gateway-pattern.html)
