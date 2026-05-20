# Builder

## What is it

`Builder` là creational pattern dùng để tạo object phức tạp từng bước một thay vì dồn hết tham số vào constructor dài.

Nó giải quyết bài toán object có nhiều field, nhiều field optional, hoặc có invariant cần validate trước khi object được coi là hợp lệ.

Điểm dạy học quan trọng là builder không tồn tại để “trông fluent hơn”. Nó tồn tại để tách trạng thái đang được lắp ráp ra khỏi object cuối cùng.

### Quick distinction

| Câu hỏi | `Builder` trả lời thế nào |
|---|---|
| Object có nhiều tham số dễ nhầm thứ tự không? | Có thể nên cân nhắc |
| Có nhiều field optional không? | Rất hợp |
| Có invariant cần check trước khi tạo object không? | Rất hợp |
| Lợi ích chính | readability, validation, tách state tạm khỏi object cuối |
| Giá phải trả | thêm lớp phụ, thừa nếu object quá đơn giản |

## How I used to misunderstand it

Mình từng nghĩ `Builder` chỉ là cách làm code “đẹp mắt” hơn constructor. Thực ra lợi ích chính là readability, validation, và diễn tả rõ object còn đang ở trạng thái chưa hoàn chỉnh.

Nếu object chỉ có 2 hoặc 3 tham số rõ ràng, builder có thể là overkill. Pattern này mạnh nhất khi constructor bắt đầu làm caller phải nhớ thứ tự, hoặc khi object có nhiều optional combination.

## How it actually works

Builder giữ state tạm cho các field đang được cấu hình. Caller điền dữ liệu qua các method fluent hoặc setter-like. Khi gọi `build()`, builder kiểm tra điều kiện hợp lệ rồi tạo ra object cuối cùng.

Object tạo ra thường immutable hoặc ít nhất ổn định hơn state tạm trong builder.

```text
raw values
   -> builder giữ state tạm
   -> validate required fields và invariant
   -> build object hoàn chỉnh
```

### Decision matrix

| Tình huống | `Builder` hợp không? | Vì sao |
|---|---|---|
| Nhiều field optional | Hợp | tránh constructor overload dài |
| Cần validate trước khi object hoàn chỉnh | Hợp | gom check vào `build()` |
| Chỉ 2 đến 3 tham số bắt buộc rõ ràng | Ít hợp | constructor thường đủ |
| Muốn chọn một trong nhiều implementation | Không | đó gần `Factory` hơn |
| Object nên immutable sau khi tạo | Rất hợp | builder gom state tạm ở ngoài |

## Code example

```java
class UserProfile {
    private final String id;
    private final String email;

    private UserProfile(String id, String email) {
        this.id = id;
        this.email = email;
    }

    static class Builder {
        private String id;
        private String email;

        Builder id(String id) {
            this.id = id;
            return this;
        }

        Builder email(String email) {
            this.email = email;
            return this;
        }

        UserProfile build() {
            return new UserProfile(id, email);
        }
    }
}
```

Ví dụ này mới chỉ minh họa shape cơ bản. Trong thực tế, giá trị lớn nhất thường nằm ở chỗ `build()` validate required fields và invariant trước khi trả object cuối.

## When to use / when NOT to use

Use `Builder` khi:

- object có nhiều field optional
- constructor dài dễ gây nhầm thứ tự tham số
- cần validate object ở thời điểm kết thúc lắp ráp
- muốn tạo object immutable nhưng input đến từng phần

Do NOT use `Builder` khi:

- object rất nhỏ với ít field bắt buộc
- constructor hoặc static factory đã đủ rõ
- thứ bạn cần là chọn implementation chứ không phải lắp ráp dữ liệu

Misconception thường gặp là nghĩ mọi DTO đều nên có builder. Nếu DTO nhỏ và rõ, builder chỉ thêm ceremony.

## How this connects to real Java projects

Trong Spring app, `Builder` hay xuất hiện ở DTO response, domain command object, test data setup, hoặc object dữ liệu không do container quản lý.

Ngay cả khi Spring inject bean bằng constructor, builder vẫn hữu ích cho object dữ liệu có nhiều field optional hoặc invariant phức tạp. Nó không cạnh tranh với dependency injection, vì hai thứ giải quyết hai bài toán khác nhau.

## Gotchas

- Builder không tự động đảm bảo object hợp lệ nếu `build()` không validate required field.
- Reuse cùng một builder instance nhiều lần có thể vô tình giữ state cũ.
- Fluent API dài nhưng không nói rõ field nào bắt buộc vẫn khiến caller dùng sai.
- Nếu object vẫn mutable mạnh sau khi build, lợi ích safety của builder giảm nhiều.
- Builder quá sớm cho object đơn giản sẽ làm code verbose vô ích.

## Check yourself

- `Builder` sửa nhược điểm nào của constructor nhiều tham số?
- Khi nào builder là overkill?
- Vì sao `build()` mới là nơi quan trọng nhất của pattern này?
- `Builder` khác `Factory` ở chỗ nào?
- Nếu builder có thể reuse nhiều lần, bạn cần cảnh giác vấn đề state gì?

## Exercises

### Exercise 1: Validate Builder Requirements

Độ khó: Easy

Đề bài:
Cho `hasId`, `hasName`, `hasEmail`, và `buildCalledOnce`. Hãy trả về `true` chỉ khi tất cả required field đều tồn tại và builder được dùng trong một lượt build hợp lệ duy nhất.

Ví dụ 1:

Đầu vào:
```text
hasId = true, hasName = true, hasEmail = true, buildCalledOnce = true
```

Đầu ra:
```text
true
```

Giải thích:
All required data exists and the build call is used correctly.

Ràng buộc:

- Tất cả input đều là boolean
- Trả về một boolean
- Bỏ qua builder do framework sinh ra

### Exercise 2: Count Missing Builder Fields

Độ khó: Easy

Đề bài:
Cho hai array `requiredFields` và `providedFlags` có cùng độ dài. Hãy trả về số lượng required builder field vẫn còn thiếu.

Ví dụ 1:

Đầu vào:
```text
requiredFields = ["id", "name", "email", "phone"]
providedFlags = [true, true, false, false]
```

Đầu ra:
```text
2
```

Giải thích:
Only `email` and `phone` are still missing at this point.

Ràng buộc:

- Cả hai array đều là non-null
- Both arrays have the same length
- Độ dài array nằm trong đoạn từ 0 đến 100000

### Exercise 3: Build Profile Summary

Độ khó: Medium

Đề bài:
Cho `id`, `name`, `email`, và `phone`. Hãy trả về một summary string theo đúng format `"id=<id>,name=<name>,email=<email>,phone=<phone>"`. Nếu `phone` rỗng, dùng `"N/A"` thay thế.

Ví dụ 1:

Đầu vào:
```text
id = "u-10", name = "Linh", email = "linh@example.com", phone = ""
```

Đầu ra:
```text
"id=u-10,name=Linh,email=linh@example.com,phone=N/A"
```

Giải thích:
Summary này mô hình hóa object cuối cùng được build từ các field bắt buộc và tùy chọn.

Ràng buộc:

- `id`, `name`, `email`, and `phone` are non-null
- Output format phải khớp chính xác
- `phone` may be empty

## Links

- [[003-Factory]]
- [[../../../01_Core/Data-Types-Advanced/002-record]]
- [Refactoring Guru, Builder](https://refactoring.guru/design-patterns/builder)
- [Spring Framework, Collaborating Beans](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [Effective Java Builder Pattern Overview](https://javadevcentral.com/effective-java-builder-pattern/)
