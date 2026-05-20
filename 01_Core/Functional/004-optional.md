# optional

## What is it

`Optional<T>` là wrapper nói rằng một giá trị có thể có hoặc không có.

Mental model nên nhớ là: `Optional` là **contract ở boundary**, không phải container để quấn mọi field mọi nơi. Nó hợp nhất khi API muốn caller thấy rõ rằng kết quả có thể vắng mặt.

## How I used to misunderstand it

Mình từng nghĩ `Optional` là cách sạch để loại bỏ `null` hoàn toàn.

Kết quả là quấn `Optional` vào field, parameter, entity, serializer, chỗ nào cũng thấy. Cách đó thường chỉ thêm ceremony mà không thêm clarity. `Optional` hữu ích nhất khi return value có thể không có kết quả, như lookup, parse, `findFirst`, hoặc derive value có thể fail nhẹ.

## How it actually works

`Optional` cung cấp các method như `map`, `flatMap`, `filter`, `orElse`, `orElseGet`, `orElseThrow` để bạn xử lý case có giá trị và không có giá trị một cách có chủ đích.

### Nhìn nhanh theo intent

| Nhu cầu | Method hay dùng |
|---|---|
| biến đổi giá trị nếu có | `map(...)` |
| biến đổi sang một `Optional` khác | `flatMap(...)` |
| giữ lại nếu thỏa điều kiện | `filter(...)` |
| lấy fallback có sẵn | `orElse(...)` |
| lấy fallback tạo lười | `orElseGet(...)` |
| fail rõ ràng khi thiếu | `orElseThrow(...)` |

### Một chi tiết rất hay bị quên

`orElse(...)` luôn evaluate đối số trước.

`orElseGet(...)` chỉ chạy supplier khi optional rỗng.

Nếu fallback đắt hoặc có side effect, khác biệt này rất đáng quan tâm.

## Code example

```java
import java.util.Optional;

public class Main {
    static Optional<String> findNickName(boolean exists) {
        return exists ? Optional.of("linh") : Optional.empty();
    }

    public static void main(String[] args) {
        String result = findNickName(false).orElse("anonymous");
        System.out.println(result); // anonymous
    }
}
```

## When to use / when NOT to use

Dùng `Optional` cho return value có thể vắng, như lookup, parse, search, hoặc derive result từ một chain ngắn.

Không nên dùng `Optional` cho field entity, method parameter public API, hoặc collection item, trừ khi team có lý do rất cụ thể và đã cân nhắc integration cost.

Nếu absence là một domain state có meaning riêng, đôi khi custom result type hoặc sealed hierarchy sẽ rõ hơn `Optional`.

## How this connects to real Java projects

Trong Spring Boot, `Optional` hợp ở repository/service lookup, helper parse function, hoặc mapping chain ngắn.

Nhưng với request DTO, JPA entity, config binding, và serialization boundary, `Optional` thường không phải default tốt nhất. Dùng đúng chỗ thì intent rõ. Dùng sai chỗ thì framework integration và data model trở nên khó chịu hơn.

## Gotchas

- `orElse(...)` eager, `orElseGet(...)` lazy.
- `Optional.get()` mà không chắc có giá trị thì gần như quay lại bug kiểu cũ.
- `Optional` không thay thế việc modeling domain cho nghiêm túc.
- Đừng dùng `Optional` chỉ để nhìn code có vẻ hiện đại hơn.

## Handbook rule

- `Optional` chỉ dùng cho return value có thể vắng; không dùng cho field/parameter/collection item.
- Tránh `optional.get()` không guard; ưu tiên `orElse`, `orElseGet`, hoặc `ifPresent`.
- `orElse` eager, `orElseGet` lazy; chọn theo cost của default value.
- Đừng dùng `Optional` chỉ để thay `null` ở mọi chỗ; absence có domain meaning thì cân nhắc sealed/result type.
- Không serialize `Optional` qua API/JPA; chuyển về plain type trước khi qua boundary.

## Check yourself

- Vì sao nói `Optional` hợp ở return type hơn là ở field?
- Khi nào `orElseGet(...)` tốt hơn `orElse(...)`?
- `map(...)` và `flatMap(...)` khác nhau ở điểm nào?
- Nếu dữ liệu thiếu là một trạng thái domain thật sự, vì sao `Optional` chưa chắc là lựa chọn tốt nhất?
- Vì sao `Optional.get()` thường là mùi code xấu?

## Exercises

### Bài 1: Return User Nickname Or Default
Độ khó: Dễ

Đề bài:
Cho một optional nickname và một default text, trả về nickname nếu nó đang present; nếu không thì trả về default text.

Ví dụ 1:
Đầu vào:
```text
nickname = Optional.empty(), defaultText = "guest"
```

Đầu ra:
```text
"guest"
```

Giải thích:
Optional đang rỗng, nên fallback value được dùng.

Ràng buộc:
- defaultText là non-null
- nickname là non-null
- Trả về plain string result

### Bài 2: Parse Positive Integer Safely
Độ khó: Trung bình

Đề bài:
Cho một text, trả về `Optional<Integer>` chỉ khi text là numeric và value parse ra là số dương.

Ví dụ 1:
Đầu vào:
```text
text = "15"
```

Đầu ra:
```text
Optional[15]
```

Giải thích:
Việc parse thành công và value cũng vượt qua positivity filter.

Ràng buộc:
- text là non-null
- Giá trị không hợp lệ hoặc không dương trả về empty optional
- Không ném exception với invalid numeric text

### Bài 3: Map Optional Email To Domain Part
Độ khó: Trung bình

Đề bài:
Cho một optional email address, trả về optional chỉ chứa domain part phía sau `@` khi email tồn tại và hợp lệ.

Ví dụ 1:
Đầu vào:
```text
email = Optional["a@test.com"]
```

Đầu ra:
```text
Optional["test.com"]
```

Giải thích:
Optional được map sang một derived value.

Ràng buộc:
- email optional là non-null
- Email format không hợp lệ thì trả về empty optional
- Chỉ trả về optional result

## Links

- [[003-stream-api]]
- [[005-method-reference]]
- [[../../04_Lessons-from-bugs/001-null-pointer-exception]]
- `Optional` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html
- Oracle article, Optional in a Nutshell: https://www.oracle.com/technical-resources/articles/java/java8-optional.html
