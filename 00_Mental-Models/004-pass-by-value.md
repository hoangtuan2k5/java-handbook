# Pass by value

## What is it

Java luôn là `pass-by-value`, không có ngoại lệ cho object. Điểm làm nhiều người vấp là: khi truyền object vào method, thứ được copy không phải whole object mà là `reference value` dùng để truy cập object đó.

Hình dung đơn giản: bạn photo một tờ giấy ghi “đi tới căn nhà này”. Hai người cùng cầm hai bản giấy khác nhau, nhưng cả hai đều chỉ tới cùng một căn nhà.

Nếu một người đổi nội thất trong căn nhà, người kia sẽ thấy. Nếu một người vứt tờ giấy cũ và cầm tờ giấy ghi địa chỉ nhà khác, người kia không bị ảnh hưởng.

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là: “primitive thì pass-by-value, còn object thì pass-by-reference”. Câu này nghe hợp lý vì object bị mutate trong method thì caller cũng thấy thay đổi.

Nhưng đó không phải vì Java truyền object “theo reference”. Lý do thật là caller và callee đang giữ hai bản copy của cùng một `reference value`, nên cùng truy cập một object.

Hiểu nhầm thứ hai là nghĩ gán parameter sang object mới sẽ đổi luôn biến ở ngoài. Thật ra bạn chỉ đang đổi bản copy local của reference.

## How it actually works

Khi gọi method, Java luôn copy argument sang parameter.

- Nếu argument là `int`, bản copy là một số nguyên độc lập.
- Nếu argument là `User`, bản copy là một `reference value` tới object `User` đó.

Đây là lý do có hai hành vi nhìn như mâu thuẫn nhưng thực ra rất logic:

- Nếu bạn đổi field của object qua parameter, caller nhìn thấy thay đổi vì hai bên đang đi tới cùng một object.
- Nếu bạn gán parameter sang object mới, caller không đổi gì vì chỉ biến local trong method đổi đích.

Điểm chốt cần nhớ là Java không cho callee thay trực tiếp binding của biến local bên caller. Method chỉ nhận bản copy của value mà caller gửi vào.

## Code example

```java
class User {
    String name;

    User(String name) {
        this.name = name;
    }
}

public class Main {
    static void changePrimitive(int number) {
        // only the local copy changes
        number = 99;
    }

    static void mutateUser(User user) {
        // both references access the same object
        user.name = "An";
    }

    static void reassignUser(User user) {
        // only the local parameter now refers to another object
        user = new User("Binh");
    }

    static void swapInts(int left, int right) {
        int temp = left;
        left = right;
        right = temp;
    }

    static void swapUsers(User first, User second) {
        User temp = first;
        first = second;
        second = temp;
    }

    public static void main(String[] args) {
        int age = 20;
        changePrimitive(age);
        // still 20
        System.out.println(age);

        User user = new User("Linh");
        mutateUser(user);
        System.out.println(user.name); // An

        reassignUser(user);
        // still An
        System.out.println(user.name);

        int a = 1;
        int b = 2;
        swapInts(a, b);
        // still 1, 2
        System.out.println(a + ", " + b);

        User first = new User("First");
        User second = new User("Second");
        swapUsers(first, second);
        // still First, Second
        System.out.println(first.name + ", " + second.name);
    }
}
```

`swap` là ví dụ rất tốt để tự kiểm tra hiểu biết. Nếu Java là pass-by-reference thật, `swapInts(a, b)` hoặc `swapUsers(first, second)` có thể đổi luôn biến ở caller. Nhưng thực tế không đổi được, vì method chỉ đang tráo hai bản copy local của argument.

## When to use / when NOT to use

Dùng mental model này khi debug side effect, thiết kế method API, xử lý DTO hoặc entity mutable, hoặc khi tranh luận xem một method có thể “đổi object của caller” hay không.

Ví dụ, nếu service method nhận `List<Order>` rồi sort trực tiếp, caller sẽ thấy list bị đổi vì method mutate cùng object. Ngược lại, nếu method gán `orders = new ArrayList<>()`, caller không hề bị đổi biến.

Không cần kéo full model này ra khi chỉ học syntax method cơ bản. Lúc đó chỉ cần nhớ quy tắc thực dụng: Java luôn copy argument, nhưng copied value có thể là reference value.

## How this connects to Spring

Trong Spring Boot, controller gọi service rồi truyền DTO, entity, hoặc request object xuống dưới. Các method đó vẫn chỉ nhận bản copy của `reference value`, nhưng nếu chúng mutate object được truyền vào thì thay đổi vẫn lan ngược lên các layer khác vì tất cả đang nhìn cùng object.

Đây là lý do side effect trong service layer có thể rất khó theo dõi nếu object mutable bị chuyền qua nhiều bean. Nhìn code thì tưởng chỉ sửa local variable, nhưng thực ra đang sửa shared object.

## Gotchas

- Reassign parameter bên trong method không bao giờ thay luôn biến local của caller.
- `String`, `Integer`, `LocalDate` là immutable, nên cảm giác “không pass được ra ngoài” thường đến từ tính immutable chứ không phải từ cơ chế truyền tham số khác đi.
- Truyền `List` hoặc `Map` vào method rồi mutate trực tiếp là nguồn side effect rất phổ biến trong code business.
- `swap` thất bại không phải vì Java “khó chịu”, mà vì cơ chế call của nó nhất quán: luôn copy value.

## Check yourself

- Vì sao `mutateUser(user)` đổi được `user.name`, nhưng `reassignUser(user)` không đổi được biến `user` ở caller?
- Vì sao `swapInts(a, b)` không swap được hai biến bên ngoài?
- Khi truyền `List` vào method, thứ được copy là gì?
- Một method có thể đổi binding của biến local bên caller không?

## Links

[[003-Stack-vs-Heap]]
[[011-value-type-vs-reference-type]]
- JLS Chapter 4, Types, Values, and Variables: https://docs.oracle.com/javase/specs/jls/se21/html/jls-4.html
