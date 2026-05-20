# toString()

## What is it

`toString()` là method của `Object` dùng để biến object thành chuỗi dễ đọc cho con người.

Mục đích chính của nó là hỗ trợ log, debug, inspect object, và làm test failure message dễ đọc hơn. Nó không phải business format chính thức.

Mental model tốt là: `toString()` là **developer-facing summary**, không phải serialization contract.

## How I used to misunderstand it

Mình từng nghĩ `toString()` chỉ là thứ IDE generate cho đẹp.

Sau đó mới thấy nếu object không override `toString()`, log thường chỉ ra kiểu `User@4a54c0de`, gần như vô dụng khi debug issue thật. Ở chiều ngược lại, mình cũng từng thấy `toString()` bị nhét quá nhiều field, kể cả secret, làm log vừa rối vừa nguy hiểm.

## How it actually works

`Object#toString()` mặc định trả về chuỗi gồm:

- tên class
- ký tự `@`
- `hashCode()` ở dạng hexadecimal

Khi class override `toString()`, Java sẽ gọi version mới đó ở nhiều chỗ như:

- `System.out.println(object)`
- string concatenation
- logger argument
- debugger watch

### So sánh nhanh

| Kiểu output | Dùng để làm gì |
|---|---|
| default `ClassName@hex` | chỉ cho biết object thuộc class nào |
| custom `toString()` | cho developer biết state quan trọng của object |
| JSON / CSV / API payload | machine-facing data contract |

Điểm quan trọng là `toString()` nên mô tả **đủ state để hiểu object**, nhưng không nên lộ secret và cũng không nên bị parse như format chính thức.

## Code example

```java
final class User {
    private final String username;
    private final boolean active;

    User(String username, boolean active) {
        this.username = username;
        this.active = active;
    }

    @Override
    public String toString() {
        return "User{username='" + username + "', active=" + active + "}";
    }
}

User user = new User("hoa", true);
System.out.println(user);
// User{username='hoa', active=true}
```

## When to use / when NOT to use

Override `toString()` khi object thường xuyên đi vào log, debug output, test diagnostics, hoặc console output và default string không đủ hữu ích.

Không dùng `toString()` làm API contract giữa services, không lưu nó như format dữ liệu chính, và không dựa vào nó cho parser ở chỗ khác.

Nếu object chứa password, token, secret key, hoặc PII nhạy cảm, cần cực kỳ cẩn thận với những gì xuất hiện trong `toString()`.

## How this connects to real Java projects

Trong Spring app, `toString()` thường xuất hiện khi log request DTO, response DTO, config object, entity, hoặc object đi qua exception message.

Một `toString()` tốt giúp đọc log nhanh hơn rất nhiều. Một `toString()` tệ có thể làm lộ secret vào log pipeline mà không ai để ý trong thời gian dài.

## Gotchas

- `System.out.println(object)` gọi `toString()` tự động.
- Không nên đưa secret hoặc PII nhạy cảm vào `toString()`.
- Không nên parse `toString()` ở code khác.
- `toString()` nên ưu tiên readability hơn là đầy đủ tuyệt đối.

## Check yourself

- Vì sao `toString()` là human-facing summary chứ không phải machine contract?
- Khi nào default `ClassName@hex` là không đủ?
- Vì sao nhét quá nhiều field vào `toString()` cũng là vấn đề?
- Loại dữ liệu nào tuyệt đối nên cân nhắc che hoặc bỏ khỏi `toString()`?
- Nếu hệ thống khác cần dữ liệu ổn định để parse, vì sao `toString()` không phải nơi phù hợp?

## Exercises

### Bài 1: Format User Summary Text

Độ khó: Dễ

Đề bài:
Cho `username`, `age`, và `active` flag, trả về một string đúng theo format `User{username='NAME', age=AGE, active=FLAG}`.

Ví dụ 1:

Đầu vào:
```text
username = "hoa", age = 28, active = true
```

Đầu ra:
```text
User{username='hoa', age=28, active=true}
```

Giải thích:
Output phải theo đúng một readable representation cố định, tương tự một custom `toString()` implementation.

Ràng buộc:

- `username` là non-null
- `0 <= age <= 150`
- Trả về đúng format được nêu trong đề bài

### Bài 2: Detect Default toString Pattern

Độ khó: Dễ

Đề bài:
Cho một string, trả về `true` nếu nó trông giống default `Object#toString()` pattern `ClassName@hexDigits`, ngược lại trả về `false`.

Ví dụ 1:

Đầu vào:
```text
text = "User@4a54c0de"
```

Đầu ra:
```text
true
```

Giải thích:
Text chứa class-like prefix, một dấu `@`, và một hexadecimal suffix.

Ràng buộc:

- `text` là non-null
- `1 <= text.length() <= 200`
- Xử lý chữ hexadecimal theo kiểu không phân biệt hoa thường

### Bài 3: Build Order Line Text

Độ khó: Trung bình

Đề bài:
Cho `orderId`, `itemCount`, và `total amount` tính bằng cents, trả về một readable summary đúng theo format `Order{id='ID', itemCount=COUNT, totalCents=TOTAL}`.

Ví dụ 1:

Đầu vào:
```text
orderId = "ORD-7", itemCount = 3, totalCents = 2599
```

Đầu ra:
```text
Order{id='ORD-7', itemCount=3, totalCents=2599}
```

Giải thích:
Bài này luyện tập xây stable human-readable object text thay vì phụ thuộc vào default output.

Ràng buộc:

- `orderId` là non-null
- `0 <= itemCount <= 100000`
- `0 <= totalCents <= 1000000000`

## Links

- [[002-equals-and-hash-code-contract]]
- [[003-clone-and-cloneable]]
- [[../Types-and-Variables/006-equals-vs-hash-code]]
- `Object#toString` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#toString()
- Effective discussion via records, useful comparison for auto-generated text: https://docs.oracle.com/en/java/javase/21/language/records.html
