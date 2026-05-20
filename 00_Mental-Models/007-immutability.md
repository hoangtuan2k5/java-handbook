# Immutability

## What is it

`Immutability` nghĩa là sau khi object được tạo ra, internal state của nó không còn bị thay đổi nữa. Nếu nhìn từ ngoài có vẻ “đổi giá trị”, thì thực ra bạn đang tạo ra object mới thay vì sửa object cũ.

Giống như ảnh chụp đã in ra giấy: bạn có thể in tấm khác, nhưng không sửa lại khung cảnh bên trong tấm đã in. Đây là lý do immutable object giúp code dễ suy luận hơn khi object đi qua nhiều hàm, nhiều thread, hay nhiều layer.

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là `final` đồng nghĩa với immutable. Không đúng. `final User user` chỉ chặn việc biến `user` trỏ sang object khác, chứ object đó vẫn có thể bị sửa field bên trong.

Hiểu nhầm thứ hai là “không có setter thì chắc immutable”. Sai tiếp, vì object vẫn có thể lộ ra mutable collection, mutable field, hoặc thay đổi state qua method khác.

Cũng nhiều người dùng `String` rồi nghĩ nó “bị đổi”, trong khi thật ra mỗi lần nối chuỗi là đang tạo instance mới.

## How it actually works

Một object immutable thường có vài đặc điểm cốt lõi:

- state được set đầy đủ lúc construction
- không có đường hợp lệ nào để đổi state sau đó
- dữ liệu mutable đi vào hoặc đi ra đều được defensive copy nếu cần

Giá trị lớn nhất của immutability không phải để “trông đẹp”, mà là cắt đứt side effect do aliasing. Nếu ba chỗ cùng giữ reference tới một object immutable, bạn không cần lo chỗ A sửa làm chỗ B hỏng ngầm.

Đây là lý do immutable object rất hợp cho `value object`, key trong map, config, và code concurrent. JVM không tự làm object thành immutable cho bạn, đây là quyết định thiết kế ở level class.

Điểm rất hay bị hiểu sai là shallow immutability. `final` chỉ cố định reference, không cố định state bên trong object mà reference đó trỏ tới.

## Code example

```java
import java.util.ArrayList;
import java.util.List;

final class Money {
    private final int amount;

    Money(int amount) {
        this.amount = amount;
    }

    Money add(int extra) {
        // return a new object instead of mutating this one
        return new Money(amount + extra);
    }

    int amount() {
        return amount;
    }
}

public class Main {
    public static void main(String[] args) {
        Money original = new Money(100);
        Money updated = original.add(50);

        System.out.println(original.amount()); // 100
        System.out.println(updated.amount()); // 150

        final List<String> names = new ArrayList<>();
        names.add("Linh");
        // allowed, because final does not make the List immutable
        names.add("An");

        final StringBuilder builder = new StringBuilder("ja");
        // also allowed, same reason
        builder.append("va");
    }
}
```

Contrast quan trọng ở đây là: với mutable object, `add()` thường sẽ sửa state tại chỗ. Với immutable object, `add()` phải trả về instance mới. API shape thường phản ánh rất rõ cách class quản lý state.

`final List<String>` và `final StringBuilder` là counterexample rất tốt. Biến không đổi reference, nhưng object bên trong vẫn đổi state bình thường. Vậy nên `final` chỉ là một mảnh nhỏ, không phải bằng chứng đủ để kết luận immutable.

## When to use / when NOT to use

Dùng immutability cho `value object`, config object, result object, domain concept kiểu `Money`, `Email`, `DateRange`, và nơi có shared state giữa nhiều thread hoặc nhiều layer.

Ví dụ, nếu một object được truyền từ controller xuống service rồi qua cache, immutable design giúp bạn không phải đoán xem ai đã sửa nó. Nó cũng hợp với key trong `Map`, vì key mutable rất dễ làm hỏng logic tra cứu.

Không cần cực đoan biến mọi thứ thành immutable nếu bạn đang làm object rất lớn, mutation-heavy, hoặc builder-style workflow mà việc tạo object mới liên tục không mang lại lợi ích rõ ràng.

## How this connects to Spring

Trong Spring Boot, immutability rất hợp với singleton bean mang config hoặc stateless service vì nó giảm nguy cơ shared mutable state giữa nhiều request.

Các object như DTO read-only, config object, hoặc result object nếu được thiết kế immutable sẽ dễ tin cậy hơn khi đi qua nhiều bean. Nhiều bug thread-safety trong Spring không đến từ framework, mà đến từ việc ta nhét mutable state vào singleton bean rồi để nhiều request cùng chạm vào.

## Gotchas

- `final` reference không làm object immutable, nó chỉ làm reference không đổi.
- Field là `List` hoặc `Map` mà trả thẳng ra ngoài thì object vẫn không thật sự immutable, dù field có là `private final`.
- Immutable facade bọc quanh mutable object bên trong vẫn có thể bị rò state nếu không defensive copy đúng chỗ.
- `String` immutable, nhưng `StringBuilder` thì không. Tên na ná nhau không có nghĩa semantics giống nhau.

## Check yourself

- Vì sao `final List<String> names` vẫn `add()` được?
- Một class không có setter đã đủ để gọi là immutable chưa?
- `String` và `StringBuilder` khác nhau ở điểm nào quan trọng nhất về state?
- Nếu constructor nhận một `List` mutable từ ngoài vào, class cần làm gì để giữ immutability thật sự?

## Links

[[004-pass-by-value]]
[[011-value-type-vs-reference-type]]
- String Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html
