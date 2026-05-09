# Value Type vs Reference Type

## What is it

`Value type` nghĩa là bạn làm việc với chính giá trị. Copy nó thì có một bản giá trị độc lập. `Reference type` nghĩa là biến giữ một `reference value` dùng để truy cập object. Copy biến thì bạn copy `reference value`, không copy whole object.

Trong Java hiện tại, primitive như `int`, `boolean`, `double` cho value behavior rõ nhất. Còn object như `User`, `List`, `String` được truy cập qua reference.

Nghĩ đơn giản: value giống tờ tiền bạn đưa bản thật cho người khác. Reference giống tờ giấy ghi cách đi tới cùng một căn nhà.

## How I used to misunderstand it

Mình từng nhầm `reference type` với `pass-by-reference`. Sai. Java vẫn là `pass-by-value`, chỉ là value được copy đôi khi là `reference value`.

Mình cũng từng nghĩ wrapper như `Integer` giống hệt `int`. Thực tế `Integer` là object, có thể `null`, có identity, và có `autoboxing` hoặc `unboxing` làm mọi thứ trông mượt hơn.

Một nhầm lẫn khác là immutable object như `String` hoặc `Money` “chính là value type”. Đúng hơn, chúng là reference type được thiết kế để có value-like semantics ở mức API.

## How it actually works

Ở note này, “value type vs reference type” là mental model để học Java hiện tại. Mình không nói tới future JVM value classes hay Project Valhalla. Cách dạy thực dụng và đúng cho Java ngày nay là:

- primitive có value behavior
- object được truy cập qua `reference value`
- một số object có value-like semantics vì immutable và vì `.equals()` được định nghĩa theo content

Với primitive value, assignment tạo ra bản copy độc lập. Đổi biến này không làm biến kia đổi.

Với reference type, assignment copy `reference value`. Hai biến có thể cùng truy cập một object. Đây là nguồn của `aliasing`: nhiều tên khác nhau cùng đi tới một object, nên mutate qua tên này thì tên kia cũng thấy.

Reference type còn có khái niệm identity. Hai object có thể có cùng content nhưng vẫn là hai object khác nhau. Vì vậy `==` thường hỏi “hai reference có truy cập đúng cùng object không?”, còn `.equals()` thường được thiết kế để hỏi “hai object có cùng logical value không?”.

Một bảng ngắn để chốt model:

| Case                       | Copy cái gì              | Đổi qua biến B có ảnh hưởng A không             | So sánh hay dùng                                      |
| -------------------------- | ------------------------ | ----------------------------------------------- | ----------------------------------------------------- |
| Primitive assignment       | actual value             | Không                                           | `==` so value                                         |
| Reference assignment       | `reference value`        | Có thể có, nếu mutate object chung              | `==` so identity, `.equals()` thường so logical value |
| Immutable reference object | vẫn là `reference value` | Không mutate được state bên trong qua API chuẩn | `.equals()` thường quan trọng hơn `==`                |

## Code example

```java
class User {
    String name;

    User(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        int a = 10;
        // copy primitive value
        int b = a;
        b = 20;

        User first = new User("Linh");
        // copy reference value
        User second = first;
        // mutate shared object
        second.name = "An";

        String x = new String("java");
        String y = new String("java");

        System.out.println(a); // 10
        System.out.println(b); // 20
        System.out.println(first.name); // An
        // false, khác object identity
        System.out.println(x == y);
        // true, cùng logical content
        System.out.println(x.equals(y));
    }
}
```

```text
a ----10
b ----20

first ----+
          |--> User{name="An"}
second ---+
```

Ví dụ này gom ba mental model lại: primitive copy độc lập, mutable reference gây aliasing, immutable reference object có thể behave giống value ở level dùng API nhưng vẫn không biến thành primitive.

## When to use / when NOT to use

Dùng mental model này khi debug aliasing, `==` vs `.equals()`, mutable DTO hoặc entity, collection side effect, hoặc khi thiết kế domain object nên mutable hay immutable.

Ví dụ, nếu hai service cùng nhận một `List<Order>` và một service sort trực tiếp list đó, service còn lại sẽ thấy thứ tự bị đổi vì cả hai đang giữ `reference value` tới cùng object.

Không cần đào quá sâu khi chỉ làm arithmetic primitive đơn giản. Lúc đó value behavior đã đủ trực quan.

## How this connects to Spring

Trong Spring Boot, DTO, entity, request object, bean đều là reference type, nên khi bạn truyền chúng qua controller, service, repository, các layer có thể đang nhìn cùng một object.

Entity trong persistence context còn đặc biệt hơn. Cùng một row database có thể được quản lý như một object identity trong session, và mutation trên entity có thể được flush ra database. Vì vậy hiểu reference semantics giúp bạn tránh side effect ngầm trong service layer.

## Gotchas

- `==` với object thường so identity, không phải logical value. Dùng `.equals()` khi muốn so content.
- Copy reference không copy object, nên mutation qua alias là nguồn bug rất phổ biến.
- Immutable object giảm rủi ro mutation nhưng vẫn là reference type. `null`, identity, allocation, và reference semantics vẫn tồn tại.
- Đừng dạy Java hiện tại như thể nó đã có “value class” phổ biến trong everyday code. Với người học Java hôm nay, model hữu ích nhất vẫn là primitive values, reference values, và value-like semantics.

## Check yourself

- `User second = first;` copy object hay copy `reference value`?
- Vì sao `x == y` có thể là `false` nhưng `x.equals(y)` lại là `true`?
- `String` có phải primitive value không?
- Nếu một object immutable, nó có thôi là reference type không?

## Links

[[002-everything-is-object]]
[[004-pass-by-value]]
[[007-immutability]]
- JLS Chapter 4, Types, Values, and Variables: https://docs.oracle.com/javase/specs/jls/se21/html/jls-4.html
- JLS 15.21.3, Reference Equality Operators: https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.21.3
- String Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html
