# record

## What is it

`record` là một cách khai báo type ngắn gọn cho dữ liệu bất biến, chủ yếu dùng để mang data thay vì chứa mutable business state.

Nếu class thường chỉ có fields, constructor, getters, `equals`, `hashCode`, `toString`, thì `record` giúp compiler sinh sẵn phần boilerplate đó.

Mental model: `record` là “data carrier với contract rõ ràng”, không phải “POJO rút gọn cho mọi trường hợp”.

## How I used to misunderstand it

Mình từng nghĩ `record` chỉ là class viết ít hơn.

Nhưng khác biệt cốt lõi không chỉ nằm ở syntax, mà ở intent và semantics: component của record là final theo default, equality dựa trên value, và canonical constructor gắn chặt với shape của dữ liệu.

Nếu dùng record cho object có lifecycle mutable hoặc identity business riêng, code sẽ nhanh chóng khó chịu.

## How it actually works

Khi khai báo `record UserSummary(Long id, String name)`, compiler tạo private final fields, accessor methods `id()` và `name()`, cùng `equals`, `hashCode`, `toString` dựa trên tất cả components.

Bạn vẫn có thể thêm validation trong canonical constructor, thêm method phụ, hoặc implement interface.

Nhưng record không phù hợp nếu object cần setter, mutable field, hoặc kế thừa class khác.

### Advanced type comparison

| Nếu bạn cần... | `record` | `enum` | `sealed class` |
|---|---|---|---|
| Một immutable data shape rõ ràng | Rất hợp | No | Có thể nhưng thường nặng hơn |
| Value-based equality mặc định | Yes | Không phải mục tiêu chính | Tự định nghĩa |
| Tập constant hữu hạn | No | Rất hợp | Có thể nhưng không tối ưu |
| Nhiều variant với payload khác nhau | Hạn chế | Hạn chế | Rất hợp |

### Decision scaffold

```text
Need one immutable data shape? -> record
Need finite named constants? -> enum
Need a closed hierarchy of variants? -> sealed class
```

## Code example

```java
record UserSummary(long id, String name) {
    UserSummary {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("name must not be blank");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        UserSummary user = new UserSummary(1L, "Linh");
        System.out.println(user.name()); // Linh
    }
}
```

## When to use / when NOT to use

Dùng `record` cho immutable DTO, API response, query projection, event message, hoặc value object nhỏ.

Không dùng `record` cho JPA entity mutable, object cần no-arg constructor framework cũ, hoặc class có state thay đổi theo thời gian.

Nếu business identity khác với tất cả fields hiện có, cần cân nhắc kỹ vì equality của record đi theo toàn bộ components.

## How this connects to real Java projects

Trong Spring Boot, `record` rất hợp cho request hoặc response DTO, config binding hiện đại, projection, hoặc message payload.

Với REST API, record giúp object bất biến và rõ shape hơn.

Nhưng với JPA entity hoặc proxy-heavy framework behavior, record thường không phải lựa chọn chính vì nhiều framework kỳ vọng mutable entity và constructor pattern khác.

## Gotchas

- `record` không tự biến object thành immutable sâu. Nếu component là `List` mutable thì bên trong vẫn đổi được.
- Equality của record dùng toàn bộ components, nên thêm field mới có thể đổi semantics của `equals` và `hashCode`.
- Record accessor là `name()` chứ không phải bắt buộc `getName()`, nên code cũ hoặc framework cũ có thể cần chú ý.
- Record rất hợp cho shape ổn định, nhưng không hợp cho object có lifecycle nhiều bước.

## Handbook rule

- Dùng `record` cho immutable DTO, value object, event, hoặc query projection.
- Tránh `record` cho JPA entity mutable hoặc class cần no-arg constructor framework cũ.
- `record` chỉ shallow-immutable; component mutable vẫn đổi được nội bộ, phải copy/wrap khi cần.
- Equality theo toàn bộ components; thêm/bớt component đổi semantics nên cân nhắc trước.
- Accessor là `name()` không phải `getName()`; framework cũ phải biết quy ước này.

## Check yourself

- Khi nào `record` rõ hơn class thường?
- Vì sao `record` hợp với DTO nhưng thường không hợp với JPA entity mutable?
- “Immutable nông” nghĩa là gì trong bối cảnh record?
- Nếu thêm component mới vào record, equality thay đổi ra sao?
- Khi nào `sealed class` hợp hơn `record` dù cả hai đều có vẻ “gọn và hiện đại”?

## Exercises

### Bài 1: Create User Summary Record
Độ khó: Dễ

Đề bài:
Cho `id` và `name`, trả về một record `UserSummary` chứa đúng hai giá trị đó.

Ví dụ 1:
Đầu vào:
```text
id = 7, name = "An"
```

Đầu ra:
```text
UserSummary[id=7, name=An]
```

Giải thích:
Kết quả là một immutable data carrier gọn với hai component.

Ràng buộc:
- id >= 0
- name là non-null
- name không được blank

### Bài 2: Detect Duplicate User Summary
Độ khó: Trung bình

Đề bài:
Cho một list các record `UserSummary`, trả về `true` nếu có bất kỳ record trùng nào xuất hiện nhiều hơn một lần theo value.

Ví dụ 1:
Đầu vào:
```text
users = [UserSummary(1, "Linh"), UserSummary(2, "An"), UserSummary(1, "Linh")]
```

Đầu ra:
```text
true
```

Giải thích:
Hai record có cùng component values sẽ được xem là bằng nhau.

Ràng buộc:
- 0 <= users.length <= 100000
- users[i] là non-null
- Equality phải dùng record value semantics

### Bài 3: Group Users By Age Band Record
Độ khó: Trung bình

Đề bài:
Cho một list các age, trả về một list các record `AgeBand` trong đó mỗi record chứa age gốc và band text tương ứng: `"minor"`, `"adult"`, hoặc `"senior"`.

Ví dụ 1:
Đầu vào:
```text
ages = [15, 20, 66]
```

Đầu ra:
```text
`[AgeBand[age=15, band=minor], AgeBand[age=20, band=adult], AgeBand[age=66, band=senior]]`
```

Giải thích:
Mỗi item đầu ra mang theo raw data cùng với một label được suy ra.

Ràng buộc:
- 0 <= ages.length <= 100000
- age >= 0
- Giữ nguyên thứ tự đầu vào

## Links

- [[001-enum]]
- [[003-sealed-class]]
- [[004-pattern-matching]]
- [[../../00_Mental-Models/007-Immutability]]
- Java language guide, records: https://docs.oracle.com/en/java/javase/21/language/records.html
