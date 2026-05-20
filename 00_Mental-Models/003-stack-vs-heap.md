# Stack vs Heap

## What is it

`Stack` và `heap` không phải là cách chia “kiểu dữ liệu này thuộc bên nào”. Chúng là cách JVM tổ chức runtime.

`Stack` gắn với lời gọi method đang chạy: mỗi lần gọi method, JVM tạo một `stack frame` mới để giữ local variables và trạng thái thực thi.

`Heap` là nơi các object được cấp phát để chúng có thể sống vượt qua một method call nếu vẫn còn reference trỏ tới.

Nghĩ đơn giản: `stack` là bàn làm việc tạm của từng cuộc gọi, còn `heap` là kho đồ chung của cả process. Takeaway chính là khác nhau về execution context và lifetime, không phải chỉ khác nhau về “loại biến”.

```text
stack frame of createUser()
+-------------------------+
| age = 20                |
| userRef ----------------+---------> User{name="Linh20"} on heap
+-------------------------+
```

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là “primitive ở stack, object ở heap”. Câu này tiện miệng nhưng sai nếu biến nó thành luật tuyệt đối. Điều local variable giữ trên `stack frame` có thể là primitive value hoặc object reference; còn object mà reference đó trỏ tới mới thường nằm trên `heap`.

Hiểu nhầm thứ hai là tưởng `stack vs heap` là câu chuyện về type. Bản chất của nó là execution model và lifetime: cái gì thuộc về một lần gọi method, cái gì cần tồn tại lâu hơn lần gọi đó.

## How it actually works

Flow thật là: khi `main()` chạy, JVM tạo một `stack frame` cho nó. Nếu `main()` gọi `createUser()`, JVM đẩy thêm một frame mới lên `stack`. Trong frame này có local variables như `age` hoặc `userRef`.

Nếu code gặp `new User(...)`, JVM cấp phát object trên `heap`, rồi local variable trong frame chỉ giữ reference tới object đó. Cũng nên nhắc lại một điểm chính xác: reference trong Java là abstract reference value, không nhất thiết là machine pointer literal theo cách mình hay tưởng tượng khi mới học.

Khi `createUser()` return, frame của nó bị pop khỏi `stack`, nhưng object trên `heap` chưa biến mất ngay. Nó chỉ trở thành eligible for GC khi không còn reference sống nào trỏ tới.

Đây là lý do `stack` rất nhanh: JVM chỉ cần push và pop frame theo thứ tự vào ra của method call. `Heap` chậm hơn và phức tạp hơn vì phải quản lý cấp phát động, object graph, và garbage collection.

Analogies giúp nhớ nhanh, nhưng takeaway cần chốt chính xác: `stack` tối ưu cho execution of calls, `heap` tối ưu cho objects có lifetime linh hoạt hơn một method call.

## Code example

```java
class User {
    String name;

    User(String name) {
        this.name = name;
    }
}

public class Main {
    static User createUser() {
        // this value belongs to the current stack frame because only this method execution needs it
        int age = 20;
        // the frame stores a reference, while the actual User object lives independently on the heap
        User user = new User("Linh");
        // mutating the heap object through a local reference shows stack and heap cooperate, not compete
        user.name = user.name + age;
        // returning the reference lets the object outlive this frame after the method is popped
        return user;
    }

    public static void main(String[] args) {
        // createUser's frame is gone, but the returned heap object is still reachable here
        User first = createUser();
        // both locals now point to the same heap object, which is why shared mutable state appears
        User second = first;
        // changing through one reference affects the same object, not a copy stored in another frame
        second.name = "An";
        // prints "An", proving the object is shared via references across frames
        System.out.println(first.name);
    }
}
```

## When to use / when NOT to use

Dùng mental model này khi debug `StackOverflowError`, memory leak, object sharing, recursion sâu, hoặc khi không hiểu vì sao method đã return nhưng object vẫn còn sống.

Nếu request xử lý xong mà một object lớn vẫn không được thu hồi, bạn cần nghĩ theo reachability trên `heap`, không phải theo local variable đã biến mất khỏi `stack`.

Không cần lôi model này ra khi chỉ học cú pháp Java cơ bản. Lúc đó nó dễ làm tăng cognitive load hơn là tạo giá trị ngay.

## How this connects to Spring

Trong Spring Boot, mỗi request đang được xử lý sẽ chạy trên call stack đang hoạt động của thread phục vụ nó khi đi qua controller, service, repository, nhưng các singleton bean lại là object trên `heap` được chia sẻ giữa nhiều request.

Đây là lý do local variable trong method thường an toàn hơn cho concurrency, còn mutable state đặt trong field của singleton bean thì rất dễ gây race condition. Chỗ khác nhau không nằm ở syntax, mà ở shared heap state.

## Gotchas

- Local variable `user` nằm trong `stack frame`, nhưng object `new User(...)` mà nó trỏ tới không nằm trong local variable đó.
- Method return không đồng nghĩa object bị huỷ; object chỉ được GC khi không còn reachable reference.
- `StackOverflowError` thường đến từ recursion hoặc call depth quá sâu, không liên quan trực tiếp đến việc heap còn nhiều hay ít.

## Check yourself

- Vì sao câu “primitive ở stack, object ở heap” chỉ đúng một phần?
- Trong một `stack frame`, local variable kiểu object thật ra giữ gì?
- Sau khi method return, vì sao object có thể vẫn còn sống?
- `StackOverflowError` thường liên quan tới call depth hay lượng object trên heap?
- Trong Spring, vì sao field mutable của singleton bean nguy hiểm hơn local variable?

## Links

- [[001-java-in-jvm-eyes]]
- [[008-object-lifecycle]]
- [[004-pass-by-value]]
- [[011-value-type-vs-reference-type]]
- [JVMS Chapter 2, The Structure of the Java Virtual Machine](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html)
