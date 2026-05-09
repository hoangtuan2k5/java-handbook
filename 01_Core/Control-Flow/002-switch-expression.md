# switch Expression

## What is it

`switch expression` là cách viết `switch` trả về một value, thay vì chỉ chạy statement theo nhánh.

Từ Java hiện đại, `switch` có thể dùng arrow label `case X ->` và `yield` khi một branch cần block nhiều dòng.

Mental model:

- `switch statement` là “chọn nhánh để làm việc”
- `switch expression` là “chọn nhánh để tạo ra một kết quả”

## How I used to misunderstand it

Mình từng xem `switch` chỉ là phiên bản khác của nhiều `if-else`, nên hay viết statement rồi mutate một biến bên ngoài.

Cách đó dễ quên `break`, dễ để biến ở trạng thái chưa gán, và làm intent bị loãng.

Với `switch expression`, compiler ép mỗi branch đóng góp vào kết quả, nên code đọc giống mapping rule hơn là control flow rời rạc.

## How it actually works

`switch expression` dùng cú pháp `case value -> result` để trả về value cho branch đó.

Nếu branch cần nhiều statement, dùng block và `yield value`.

Vì là expression, nó phải có type thống nhất giữa các branch và phải cover đủ khả năng input, thường bằng `default` nếu selector không phải enum hoặc sealed type được cover hết.

Arrow case không fall-through như `case` cũ có dấu `:`. Đây là lợi ích lớn vì nhiều bug `switch` cổ điển đến từ việc quên `break`.

### Comparison table

| Câu hỏi | `switch statement` | `switch expression` |
|---|---|---|
| Mục tiêu chính | side effect hoặc branch control | tạo ra value |
| Dễ phải mutate biến ngoài | Thường yes | Ít hơn nhiều |
| Fall-through kiểu cũ | Có thể có | Arrow case thì không |
| Rất hợp cho mapping rule | Tạm ổn | Rất hợp |

### Decision scaffold

```text
Need to compute one result from one selector? -> switch expression
Need complex boolean predicates across different variables? -> if-else may be clearer
```

## Code example

```java
public class Main {
    public static void main(String[] args) {
        int status = 404;

        String group = switch (status / 100) {
            case 2 -> "success";
            case 4 -> "client-error";
            case 5 -> "server-error";
            default -> "other";
        };

        System.out.println(group); // client-error
    }
}
```

## When to use / when NOT to use

Dùng `switch expression` khi một input rời rạc cần map sang một output rõ ràng, như status code group, enum state, day type, hoặc pricing tier.

Dùng `if-else` khi điều kiện là range phức tạp, nhiều predicate không cùng một selector, hoặc branch cần side effect đáng kể.

Không ép `switch` vào mọi bài toán. Nếu logic là boolean predicate tuần tự, `if` thường tự nhiên hơn.

## How this connects to Spring

Trong Spring Boot, `switch expression` hữu ích khi map request enum sang behavior, map HTTP status sang message, map role sang permission level, hoặc chọn DTO response theo state.

Nó giúp service code bớt mutable local variable và giảm lỗi thiếu branch.

Với domain enum, `switch expression` cũng làm rule business dễ đọc hơn trong code review.

## Gotchas

- `switch expression` cần cover đủ branch. Thiếu `default` có thể không compile với selector chưa exhaustive.
- Arrow case không fall-through. Nếu bạn cần fall-through legacy, đó thường là dấu hiệu nên viết lại rõ hơn.
- Block branch phải dùng `yield`, không dùng `return` để trả value cho expression.
- Nếu branch chủ yếu làm side effect chứ không tạo result, có thể bạn đang ép expression vào sai bài toán.

## Check yourself

- Khi nào `switch expression` rõ hơn `if-else`?
- Vì sao nó giảm nhu cầu mutate biến bên ngoài?
- `yield` tồn tại để giải quyết tình huống nào?
- Vì sao arrow case giúp tránh một nhóm bug cũ của `switch`?
- Nếu điều kiện không cùng một selector, tại sao `if-else` thường tự nhiên hơn?

## Exercises

### Bài 1: Classify HTTP Status
Độ khó: Dễ

Đề bài:
Cho một HTTP status code, trả về group của nó: `"success"` cho 2xx, `"client-error"` cho 4xx, `"server-error"` cho 5xx, và `"other"` cho các trường hợp còn lại.

Ví dụ 1:
Đầu vào:
```text
status = 404
```

Đầu ra:
```text
"client-error"
```

Giải thích:
`404 / 100` bằng `4`, nên nó thuộc group 4xx.

Ràng buộc:
- 100 <= status <= 599
- Chỉ trả về đúng một trong các string đã được mô tả

### Bài 2: Map Day To Workload
Độ khó: Dễ

Đề bài:
Cho tên của một ngày, trả về `"workday"` cho Monday đến Friday, `"weekend"` cho Saturday hoặc Sunday, và `"unknown"` cho mọi giá trị khác.

Ví dụ 1:
Đầu vào:
```text
day = "Saturday"
```

Đầu ra:
```text
"weekend"
```

Giải thích:
Saturday là một ngày cuối tuần.

Ràng buộc:
- day là non-null
- So khớp có phân biệt chữ hoa/thường
- Giá trị không hợp lệ không được ném exception

### Bài 3: Calculate Shipping Fee
Độ khó: Trung bình

Đề bài:
Cho một shipping zone và giá trị đơn hàng, trả về shipping fee. Zone `"LOCAL"` luôn miễn phí. Zone `"DOMESTIC"` tốn `30000`, trừ khi amount ít nhất là `500000` thì được miễn phí. Zone `"INTERNATIONAL"` tốn `250000`. Zone không hợp lệ trả về `-1`.

Ví dụ 1:
Đầu vào:
```text
zone = "DOMESTIC", amount = 600000
```

Đầu ra:
```text
0
```

Giải thích:
Đơn hàng domestic có amount ít nhất `500000` sẽ được miễn phí shipping.

Ràng buộc:
- zone là non-null
- amount >= 0
- Trả về fee dưới dạng integer currency units

## Links

- [[001-for-vs-while-vs-do-while]]
- [[003-break-vs-continue-vs-return]]
- [[../Data-Types-Advanced/001-enum]]
- Java language guide, switch expressions: https://docs.oracle.com/en/java/javase/21/language/switch-expressions-and-statements.html

