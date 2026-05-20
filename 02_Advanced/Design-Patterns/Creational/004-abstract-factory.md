# Abstract Factory

## What is it

`Abstract Factory` là creational pattern dùng để tạo ra một họ object liên quan hoặc tương thích với nhau mà caller không cần biết concrete class cụ thể.

Nó giải quyết bài toán “tạo nhiều object cùng family một cách nhất quán”. Ví dụ nếu đã chọn `dark` theme, button, dialog, và input nên cùng thuộc family đó.

Điểm dạy học quan trọng là pattern này không chỉ chọn một object. Nó chọn cả một family, rồi giữ cho các product trong family đó đi cùng nhau.

### Quick distinction

| Câu hỏi | `Abstract Factory` trả lời thế nào |
|---|---|
| Có nhiều product type liên quan không? | Có |
| Các product này có cần tương thích theo family không? | Có |
| Lợi ích chính | tránh trộn sai implementation giữa các family |
| Giá phải trả | thêm abstraction, khó mở rộng product type mới |
| Nhầm lẫn phổ biến | nghĩ đây chỉ là `Factory` lớn hơn |

## How I used to misunderstand it

Mình từng nghĩ `Abstract Factory` chỉ là `Factory` nhưng “to hơn”. Thực ra điểm khác biệt quan trọng là nó tạo nhiều product có liên hệ, không phải chỉ chọn một object đơn lẻ.

Nếu bài toán của bạn chỉ là chọn một `PaymentProcessor`, `Factory` thường đủ. `Abstract Factory` chỉ đáng dùng khi sự nhất quán giữa nhiều product là phần cốt lõi của thiết kế.

## How it actually works

Ta định nghĩa một factory interface với nhiều method tạo product như `createButton()`, `createDialog()`, `createInput()`. Mỗi concrete factory đại diện cho một family.

Khi caller chọn một factory cụ thể, toàn bộ product lấy ra từ factory đó sẽ cùng thuộc một family tương thích.

```text
chọn factory family một lần
        |
        +--> create product A
        +--> create product B
        +--> create product C
```

### Decision matrix

| Tình huống | `Abstract Factory` hợp không? | Vì sao |
|---|---|---|
| Nhiều product phải đi theo cùng family | Hợp | tránh mix sai implementation |
| Chỉ cần chọn một object đơn lẻ | Ít hợp | `Factory` gọn hơn |
| Family hiếm khi thay đổi và product rất ít | Cân nhắc | abstraction có thể nặng quá mức |
| Cần swap vendor hoặc environment nguyên bộ | Hợp | mỗi vendor là một family |
| Thường xuyên thêm product type mới | Cẩn thận | mọi factory đều phải cập nhật |

## Code example

```java
interface UiFactory {
    String createButton();
    String createDialog();
}

class DarkUiFactory implements UiFactory {
    public String createButton() {
        return "dark-button";
    }

    public String createDialog() {
        return "dark-dialog";
    }
}
```

Ở đây caller không tự ghép `dark-button` với `light-dialog`. Nó chỉ chọn `DarkUiFactory`, còn tính nhất quán do factory đảm bảo.

## When to use / when NOT to use

Use `Abstract Factory` khi:

- cần tạo nhiều object liên quan theo cùng family
- compatibility giữa các product là điều bắt buộc
- muốn swap cả bộ implementation theo vendor, platform, profile, hoặc environment

Do NOT use `Abstract Factory` khi:

- chỉ cần chọn một object đơn lẻ
- family chưa thật sự tồn tại trong domain
- product set nhỏ và gần như không đổi, khiến abstraction trở nên nặng quá mức

Misconception cần sửa là nghĩ pattern này luôn “chuẩn kiến trúc” hơn `Factory`. Không phải. Nếu không có nhu cầu family consistency, nó chỉ thêm ceremony.

## How this connects to real Java projects

Trong Spring, pattern này xuất hiện khi một configuration chọn cả bộ bean theo environment, profile, tenant, hoặc vendor. Ví dụ chọn một family adapter cho AWS, GCP, hoặc local mock mà các bean trong family phải đi cùng nhau.

Nó cũng xuất hiện khi một module cần thay nguyên bộ dependency theo deployment mode, không chỉ thay một bean lẻ.

## Gotchas

- Thêm product type mới làm tất cả concrete factory phải cập nhật.
- Nếu caller vẫn tự trộn product từ nhiều family, pattern mất ý nghĩa.
- Tạo quá nhiều abstraction khi thực tế chỉ có một family hiện dùng sẽ làm code nặng.
- Naming product và family không rõ khiến team khó thấy quan hệ tương thích.
- Dễ dùng quá tay khi bài toán thật ra chỉ là registry của một implementation đơn lẻ.

## Check yourself

- `Abstract Factory` khác `Factory` ở đâu nếu nhìn từ số lượng product được tạo?
- Vì sao pattern này giúp tránh bug mix giữa các family?
- Khi nào abstraction này trở nên quá nặng?
- Vì sao việc thêm product type mới lại là điểm đau của pattern?
- Nếu caller vẫn tự ghép product từ nhiều family, pattern đã hỏng ở chỗ nào?

## Exercises

### Exercise 1: Select UI Family

Độ khó: Easy

Đề bài:
Cho `theme`. Hãy trả về một trong các string chính xác sau: `"LIGHT"`, `"DARK"`, `"MOBILE"`, hoặc `"DEFAULT"`. Chỉ match chính xác với `"light"`, `"dark"`, và `"mobile"`.

Ví dụ 1:

Đầu vào:
```text
theme = "dark"
```

Đầu ra:
```text
"DARK"
```

Giải thích:
Factory family được chọn sẽ quyết định những component tương thích nào được tạo ra.

Ràng buộc:

- `theme` is non-null
- Việc so sánh có phân biệt hoa thường
- Output must be one of the four exact strings

### Exercise 2: Validate Component Family

Độ khó: Easy

Đề bài:
Cho `buttonFamily`, `dialogFamily`, và `inputFamily`. Hãy trả về `true` chỉ khi cả ba giá trị hoàn toàn bằng nhau.

Ví dụ 1:

Đầu vào:
```text
buttonFamily = "DARK", dialogFamily = "DARK", inputFamily = "DARK"
```

Đầu ra:
```text
true
```

Giải thích:
All products come from the same family, so the bundle is compatible.

Ràng buộc:

- All inputs are non-null strings
- Việc so sánh có phân biệt hoa thường
- Trả về một boolean

### Exercise 3: Build Widget Bundle

Độ khó: Medium

Đề bài:
Cho `family` và `screenName`. Hãy trả về một string theo đúng format `"<family>:<screenName>:button,dialog,input"`.

Ví dụ 1:

Đầu vào:
```text
family = "DARK", screenName = "checkout"
```

Đầu ra:
```text
"DARK:checkout:button,dialog,input"
```

Giải thích:
Bundle summary cho thấy toàn bộ widget được tạo ra như một family phối hợp thống nhất.

Ràng buộc:

- `family` là non-null
- `screenName` là non-null
- Output format must match exactly

## Links

- [[003-Factory]]
- [[002-Builder]]
- [Refactoring Guru, Abstract Factory](https://refactoring.guru/design-patterns/abstract-factory)
- [Spring Framework, Environment Abstraction](https://docs.spring.io/spring-framework/reference/core/beans/environment.html)
- [SourceMaking, Abstract Factory](https://sourcemaking.com/design_patterns/abstract_factory)
