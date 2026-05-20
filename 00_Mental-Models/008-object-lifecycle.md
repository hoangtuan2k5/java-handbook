# Object lifecycle

## What is it

Vòng đời của một object trong Java không chỉ là “`new` rồi sau đó bị xoá”. Một object thường đi qua các giai đoạn: được tạo ra, trở thành reachable qua một chuỗi reference, mất hết đường reference nên trở thành eligible for GC, rồi có lúc nào đó mới được garbage collector thu hồi memory.

Điểm quan trọng là lifecycle của object phụ thuộc vào `reachability`, không phụ thuộc đơn giản vào chuyện method nào vừa return hay biến local nào vừa ra khỏi scope.

```text
GC Root
  |
  v
static CACHE
  |
  v
User object

Nếu đường đi này còn tồn tại, object vẫn còn reachable.
```

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là method return thì object tạo bên trong method đó chết luôn. Sai, vì nếu method trả về reference hoặc nhét object vào chỗ khác còn sống, object vẫn tiếp tục sống.

Hiểu nhầm thứ hai là gán biến `= null` thì object sẽ bị GC ngay. Không đúng: như vậy chỉ bỏ đi một reference, còn object chỉ eligible for GC nếu không còn đường nào khác trỏ tới nó.

Cũng nhiều người nghĩ GC chạy là object chết ngay lập tức, trong khi GC chỉ quyết định thu hồi khi nó thấy cần và thấy object thực sự unreachable.

## How it actually works

Object lifecycle thường bắt đầu khi JVM cấp phát object trên `heap`, thường qua `new`. Từ đó, object sống miễn là còn reachable từ `GC roots` như local variables trong `stack frame` đang sống, static fields, active threads, hoặc reference chain đi ra từ các root đó.

Nếu một object không còn được chạm tới từ bất kỳ root nào, nó trở thành `eligible for garbage collection`. Chữ “eligible” rất quan trọng: object đã hết ý nghĩa với chương trình, nhưng memory của nó chưa chắc được thu hồi ngay ở dòng code tiếp theo.

Ở đây cũng nên giữ cách nói chính xác về reference. Java code thao tác với `reference value`, tức giá trị tham chiếu trừu tượng do JVM dùng để đi tới object. Đừng buộc bản thân nghĩ nó luôn là machine pointer literal, vì điều đó không phải phần contract mà Java hứa với bạn.

Thiết kế này giúp JVM linh hoạt hơn nhiều so với việc bắt dev tự `free` memory. Nhưng đổi lại, bug memory của Java thường không đến từ việc “quên giải phóng”, mà đến từ việc vô tình giữ reference quá lâu.

Một `List` static, một cache không eviction, hay một listener chưa được gỡ đều có thể kéo dài lifecycle của object lâu hơn bạn tưởng. Vì vậy, để hiểu object sống hay chết, câu hỏi đúng không phải là “nó được tạo ở đâu”, mà là “ai còn giữ reference tới nó”.

## Code example

```java
import java.util.ArrayList;
import java.util.List;

class User {
    String name;

    User(String name) {
        this.name = name;
    }
}

public class Main {
    static final List<User> CACHE = new ArrayList<>();

    static User createUser() {
        // the object is born on the heap, but its lifetime is not tied to this method alone
        User user = new User("Linh");
        // storing the reference elsewhere extends the object's lifetime beyond the current stack frame
        CACHE.add(user);
        // returning the same reference creates another live path to the same object
        return user;
    }

    public static void main(String[] args) {
        // the object is reachable both from a local variable and from the static cache
        User current = createUser();
        // removing one reference does not kill the object because CACHE still keeps it reachable
        current = null;

        // now the last strong reference in this example is gone, so the object becomes eligible for GC
        CACHE.clear();
    }
}
```

Điểm mấu chốt của ví dụ này là object không “đi theo” biến `current`. Biến chỉ là một đường đi tạm tới object; lifecycle thật của object phụ thuộc vào còn bao nhiêu đường đi như vậy trong object graph.

## When to use / when NOT to use

Dùng mental model này khi debug memory leak, cache lớn bất thường, listener không được gỡ, collection giữ object quá lâu, hoặc khi không hiểu vì sao request xong rồi mà memory vẫn không giảm.

Ví dụ, nếu một service thêm object vào map static để tra cứu nhanh, lifecycle của object sẽ do map đó quyết định chứ không còn do request scope nữa.

Không cần kéo model này ra khi chỉ code logic nhỏ không có shared state, cache, hoặc vòng đời dài hơn một method call.

## How this connects to Spring

Trong Spring Boot, lifecycle của object thường bị container và scope chi phối mạnh. Singleton bean có thể sống suốt vòng đời ứng dụng vì container giữ reference tới nó, còn object request-scoped chỉ sống trong phạm vi request nếu không bị reference khác giữ lại.

Đây là lý do memory issue trong app Spring nhiều khi không nằm ở business logic đơn lẻ, mà nằm ở chỗ bean, cache, scheduler, hoặc event listener vô tình giữ object lâu hơn thiết kế ban đầu.

## Gotchas

- `eligible for GC` không có nghĩa là memory được thu hồi ngay lúc đó.
- Set một local variable thành `null` chỉ có ích nếu đó thực sự là đường reference cuối cùng đáng kể.
- Static field, cache, `ThreadLocal`, hoặc listener registration là các chỗ rất hay kéo dài lifecycle của object ngoài ý muốn.

## Check yourself

- Object sống theo nơi nó được tạo ra, hay theo `reachability`?
- Vì sao `current = null` chưa chắc làm object chết?
- `eligible for GC` khác gì với “đã bị thu hồi memory”?
- Những `GC roots` nào hay gặp nhất trong ứng dụng Java?
- Trong Spring app, vì sao cache hoặc singleton bean dễ kéo dài lifecycle ngoài ý muốn?

## Links

- [[003-stack-vs-heap]]
- [[005-jvm-load-class]]
- [[004-pass-by-value]]
- [[011-value-type-vs-reference-type]]
- [JLS Chapter 12, Execution](https://docs.oracle.com/javase/specs/jls/se21/html/jls-12.html)
