# Strategy

## What is it

`Strategy` là behavioral pattern cho phép đóng gói nhiều thuật toán hoặc policy khác nhau phía sau cùng một interface, rồi chọn implementation phù hợp ở runtime.

Nó giải quyết bài toán “cùng mục tiêu, nhiều cách làm”. Thay vì nhét hết variation vào một method dài đầy `if/else`, ta biến từng cách xử lý thành một object có hành vi riêng.

Điểm đáng học nhất không phải là số lượng class. Điểm đáng học là nơi variation được đặt ở đâu. `Strategy` kéo variation ra khỏi caller để caller chỉ nói “hãy tính”, không cần biết bên trong tính kiểu nào.

### Quick distinction

| Câu hỏi | `Strategy` trả lời thế nào |
|---|---|
| Có nhiều cách thay thế nhau để làm cùng một việc không? | Có |
| Caller có nên biết chi tiết từng thuật toán không? | Không nhiều |
| Variation nằm ở đâu? | Nằm trong các strategy implementation |
| Lợi ích chính | Thay rule mà ít đụng caller |
| Giá phải trả | Nhiều class hơn, selection logic vẫn phải rõ |

## How I used to misunderstand it

Mình từng nghĩ `Strategy` chỉ là đổi `switch` thành nhiều class nhỏ. Cách hiểu đó mới thấy phần bề mặt, chưa thấy lợi ích thiết kế.

Nếu selection logic vẫn rải ở nhiều caller, thì ta chỉ mới dời code chứ chưa dời responsibility. Giá trị của `Strategy` chỉ thật sự xuất hiện khi caller không phải ôm chi tiết của từng rule nữa.

## How it actually works

Ta định nghĩa một interface chung như `calculate(...)`, `route(...)`, hoặc `authenticate(...)`. Mỗi concrete strategy implement cùng contract đó nhưng theo policy riêng.

`Context` hoặc caller giữ reference tới strategy hiện tại, rồi ủy quyền việc xử lý cho nó. Khi cần thay behavior, ta đổi object được chọn thay vì sửa thuật toán cũ.

```text
Caller
  |
  +--> chọn strategy phù hợp
            |
            +--> strategy.execute(...)
```

### Strategy vs close alternatives

| Nếu câu hỏi của bạn là... | Pattern gần hơn |
|---|---|
| "Tôi có nhiều thuật toán thay thế nhau" | `Strategy` |
| "Tôi có workflow cố định nhưng vài bước con thay đổi" | `Template Method` |
| "Tôi cần chọn đúng implementation lúc tạo object" | `Factory` |

## Code example

```java
interface ShippingStrategy {
    int fee(int distanceKm);
}

class StandardShipping implements ShippingStrategy {
    public int fee(int distanceKm) {
        return distanceKm * 5;
    }
}

class ExpressShipping implements ShippingStrategy {
    public int fee(int distanceKm) {
        return distanceKm * 10;
    }
}
```

Ở đây caller chỉ cần biết mình đang dùng `ShippingStrategy`. Nó không cần biết chi tiết công thức của `StandardShipping` hay `ExpressShipping`.

## When to use / when NOT to use

Use `Strategy` khi:

- có nhiều rule thay thế nhau như pricing policy, payment policy, sorting rule, retry policy
- muốn thêm implementation mới mà không sửa một khối `if/else` lớn
- muốn test từng thuật toán độc lập

Do NOT use `Strategy` khi:

- chỉ có một implementation ổn định
- variation quá nhỏ, vài dòng `if` đã rõ hơn cả một lớp mới
- thứ bạn đang thay đổi không phải là thuật toán thay thế, mà là vài bước trong một workflow cố định

Misconception phổ biến là áp dụng `Strategy` chỉ vì “muốn code sạch”. Nếu domain không có variation thật, pattern sẽ biến thành ceremony.

## How this connects to Spring

Trong Spring, `Strategy` xuất hiện rất tự nhiên khi inject `List<PaymentStrategy>` hoặc `Map<String, PaymentStrategy>` rồi chọn bean phù hợp theo request type, channel, hoặc feature flag.

Nó cũng xuất hiện ở các contract như `Converter`, `Validator`, `AuthenticationProvider`, hoặc `HandlerMethodArgumentResolver`. Framework thường cung cấp chỗ cắm strategy, còn ứng dụng chỉ bổ sung implementation.

## Gotchas

- Nếu selection logic vẫn là một `switch` khổng lồ ở nhiều nơi, pattern mới giải quyết nửa bài toán.
- Quá nhiều strategy gần giống nhau làm codebase khó tìm hơn thay vì rõ hơn.
- Strategy có state mutable chia sẻ giữa request dễ gây bug concurrency trong Spring singleton bean.
- Đặt tên strategy theo kỹ thuật thay vì theo business rule làm intent khó đọc.
- Có lúc thứ bạn cần là data-driven configuration, không phải thêm một class mới.

## Check yourself

- `Strategy` sửa nhầm lẫn nào của một khối `if/else` dài?
- Khi nào `Template Method` hợp hơn `Strategy`?
- Nếu strategy selection logic rất rối, có nên tách thêm factory hoặc registry không?
- Một rule chỉ có đúng một implementation ổn định có cần `Strategy` không?
- Vì sao `Strategy` giúp test business policy dễ hơn?

## Exercises

### Exercise 1: Choose Discount Strategy

Độ khó: Easy

Đề bài:
Cho `customerTier`, `flashSale`, và `bulkOrder`. Hãy trả về một trong các string chính xác sau: `"FLASH"`, `"BULK"`, `"VIP"`, hoặc `"STANDARD"`. Thứ tự ưu tiên là `FLASH` trước, rồi `BULK`, rồi `VIP`, rồi `STANDARD`.

Ví dụ 1:

Đầu vào:
```text
customerTier = "VIP", flashSale = false, bulkOrder = true
```

Đầu ra:
```text
"BULK"
```

Giải thích:
Rule về bulk có ưu tiên cao hơn rule về VIP tier.

Ràng buộc:

- `customerTier` là non-null
- Giá trị tier có thể là `"VIP"` hoặc bất kỳ string nào khác
- Output phải là một trong bốn strategy name chính xác đã nêu

### Exercise 2: Apply Pricing Strategy

Độ khó: Medium

Đề bài:
Cho `basePrice`, `quantity`, và `strategyName`. Hãy trả về tổng giá cuối cùng. Dùng các rule sau: `STANDARD` nghĩa là `basePrice * quantity`, `VIP` nghĩa là giảm 10% trên tổng, `BULK` nghĩa là giảm 15% khi `quantity >= 10` còn nếu không thì không giảm, và `FLASH` nghĩa là giảm 20% trên tổng. Trả về integer cuối cùng sau khi áp dụng discount.

Ví dụ 1:

Đầu vào:
```text
basePrice = 100, quantity = 3, strategyName = "VIP"
```

Đầu ra:
```text
270
```

Giải thích:
Tổng ban đầu là 300, sau đó strategy `VIP` áp dụng mức discount 10%.

Ràng buộc:

- `basePrice` nằm trong đoạn từ 0 đến 100000
- `quantity` nằm trong đoạn từ 0 đến 100000
- `strategyName` là một trong `STANDARD`, `VIP`, `BULK`, `FLASH`

### Exercise 3: Select Fastest Allowed Strategy

Độ khó: Medium

Đề bài:
Cho các array `strategyNames`, `estimatedMinutes`, và `costs`, cùng với `maxCost`. Hãy trả về tên của strategy nhanh nhất mà có cost nhỏ hơn hoặc bằng `maxCost`. Nếu nhiều strategy hòa nhau về thời gian, trả về strategy xuất hiện đầu tiên. Nếu không có strategy nào đạt điều kiện, trả về `"NONE"`.

Ví dụ 1:

Đầu vào:
```text
strategyNames = ["STANDARD", "EXPRESS", "PICKUP"]
estimatedMinutes = [60, 25, 15]
costs = [5, 12, 0]
maxCost = 10
```

Đầu ra:
```text
"PICKUP"
```

Giải thích:
`EXPRESS` nhanh hơn `STANDARD` nhưng vượt quá mức cost cho phép.

Ràng buộc:

- Tất cả array đều là non-null và có cùng độ dài
- Độ dài array nằm trong đoạn từ 0 đến 100000
- Strategy name là các string non-null

## Links

- [[003-Template-Method]]
- [[../Creational/003-Factory]]
- [Spring Framework, Autowiring Collaborators](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [Refactoring Guru, Strategy](https://refactoring.guru/design-patterns/strategy)
- [Martin Fowler, Replace Conditional Logic with Strategy Pattern](https://martinfowler.com/bliki/StrategyPattern.html)
