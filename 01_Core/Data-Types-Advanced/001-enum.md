# enum

## What is it

`enum` là một kiểu dữ liệu đặc biệt để biểu diễn một tập giá trị hữu hạn, cố định, có tên rõ ràng.

Thay vì để status là `String` như `"PENDING"`, `"PAID"`, `"FAILED"`, bạn gói chúng vào một type duy nhất như `PaymentStatus`.

Mental model tốt là: `enum` biến “một vài string hợp lệ” thành “một domain nhỏ nhưng có type an toàn”.

## How I used to misunderstand it

Mình từng nghĩ `enum` chỉ là cách viết đẹp hơn cho constants.

Cách nghĩ đó bỏ lỡ điểm quan trọng nhất: `enum` là một type thực sự, có thể có field, method, constructor, interface implementation, và được compiler bảo vệ exhaustiveness tốt hơn `String`.

Khi dùng `String`, bug typo như `"SUCESS"` vẫn compile. Với `enum`, compiler chặn sớm hơn nhiều.

## How it actually works

Mỗi enum constant thực ra là một instance duy nhất của enum type đó.

Vì vậy so sánh `enum` thường dùng `==` là đúng.

`enum` có thể chứa field như label, code, fee, hoặc method như `isTerminal()`.

Ngoài ra, `switch expression` với `enum` là cặp rất mạnh vì compiler có thể cảnh báo khi bạn chưa xử lý đủ trường hợp.

### Advanced type comparison

| Nếu bạn cần... | `enum` | `record` | `sealed class` |
|---|---|---|---|
| Tập giá trị hữu hạn cố định | Rất hợp | Không phải mục tiêu chính | Có thể, nhưng thường nặng hơn |
| Mỗi variant chỉ là một constant có tên | Rất hợp | No | Có thể, nhưng dư |
| Payload khác nhau theo từng variant | Hạn chế | No | Rất hợp |
| Value object mang dữ liệu | Không phải trọng tâm | Rất hợp | Có thể |

### Tiny decision guide

```text
Finite named constants? -> enum
Need to carry data as one immutable shape? -> record
Need a closed hierarchy of different variants? -> sealed class
```

## Code example

```java
enum OrderStatus {
    CREATED,
    PAID,
    SHIPPED,
    CANCELLED;

    boolean isTerminal() {
        return this == SHIPPED || this == CANCELLED;
    }
}

public class Main {
    public static void main(String[] args) {
        OrderStatus status = OrderStatus.PAID;
        System.out.println(status.isTerminal()); // false
    }
}
```

## When to use / when NOT to use

Dùng `enum` khi tập giá trị là hữu hạn, ổn định, và có ý nghĩa domain rõ như status, role level, environment, day, payment method.

Dùng `String` khi value set thực sự mở hoặc đến từ dữ liệu bên ngoài không kiểm soát hết được.

Không dùng `enum` nếu domain thay đổi liên tục từ database hoặc config và phải thêm value mà không cần recompile app.

## How this connects to real Java projects

Trong Spring Boot, `enum` xuất hiện nhiều ở request DTO, config property, JPA entity field, business status, và `switch` trong service layer.

Nó giúp validation rõ hơn và làm API contract ít mơ hồ hơn.

Nhưng nếu expose enum ra REST API, đổi tên constant có thể là breaking change cho client, nên cần cẩn thận với backward compatibility.

## Gotchas

- Dùng `enum.name()` trực tiếp làm persisted hoặc API value dễ gây breaking change khi rename constant.
- `enum.ordinal()` không ổn để lưu database vì đổi thứ tự constant sẽ đổi meaning.
- Nếu parse từ input ngoài, cần xử lý invalid value rõ ràng thay vì để exception lan vô nghĩa.
- `enum` rất mạnh cho tập hữu hạn, nhưng rất tệ nếu domain thật ra là open-ended.

## Check yourself

- Vì sao `enum` tốt hơn `String` khi tập giá trị hợp lệ là hữu hạn và cố định?
- Khi nào `enum` không còn phù hợp vì domain quá mở?
- `ordinal()` nguy hiểm ở điểm nào?
- Nếu mỗi variant cần payload khác nhau, vì sao `sealed class` có thể hợp hơn `enum`?
- Nếu object chỉ là immutable data carrier, vì sao `record` hợp hơn `enum`?

## Exercises

### Bài 1: Parse Order Status
Độ khó: Dễ

Đề bài:
Cho một status text như `"CREATED"`, `"PAID"`, hoặc `"SHIPPED"`, trả về enum value tương ứng. Trả về `null` khi text không khớp với bất kỳ status nào đã biết.

Ví dụ 1:
Đầu vào:
```text
text = "PAID"
```

Đầu ra:
```text
PAID
```

Giải thích:
Input khớp chính xác với một enum constant đã biết.

Ràng buộc:
- text là non-null
- So khớp có phân biệt chữ hoa/thường
- Giá trị không hợp lệ nên trả về `null`

### Bài 2: Count Terminal Orders
Độ khó: Trung bình

Đề bài:
Cho một list các order status, trả về xem có bao nhiêu status là terminal. Một terminal status là `SHIPPED` hoặc `CANCELLED`.

Ví dụ 1:
Đầu vào:
```text
statuses = [CREATED, PAID, SHIPPED, CANCELLED]
```

Đầu ra:
```text
2
```

Giải thích:
Chỉ có `SHIPPED` và `CANCELLED` là terminal.

Ràng buộc:
- 0 <= statuses.length <= 100000
- statuses[i] là non-null
- Chỉ đếm các terminal status

### Bài 3: Group Fee By Membership Tier
Độ khó: Trung bình

Đề bài:
Cho một membership tier enum và order amount, trả về discount fee percentage. `BASIC` cho `0`, `SILVER` cho `5`, và `GOLD` cho `10`.

Ví dụ 1:
Đầu vào:
```text
tier = GOLD, amount = 250000
```

Đầu ra:
```text
10
```

Giải thích:
Fee percentage phụ thuộc vào enum tier, không phụ thuộc vào chính amount.

Ràng buộc:
- tier là non-null
- amount >= 0
- Chỉ trả về con số percentage

## Links

- [[002-record]]
- [[003-sealed-class]]
- [[004-pattern-matching]]
- [[../Control-Flow/002-switch-expression]]
- `Enum` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Enum.html

