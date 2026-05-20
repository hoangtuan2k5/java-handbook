# JVM Class Loading

## What is it

JVM không thấy file `.class` là “chạy class” ngay. Một class thường đi qua các pha chính: `load`, `link`, rồi `initialize`.

Nghĩ đơn giản: `load` là mang bản thiết kế vào hệ thống, `link` là kiểm tra và chuẩn bị nó có thể dùng an toàn, còn `initialize` mới là lúc chạy logic `static` thật sự. Takeaway chính là: biết tên class chưa có nghĩa là class đã chạy code khởi tạo.

## How I used to misunderstand it

Mình từng nghĩ app start là JVM load hết mọi class một lượt. Không đúng: nhiều class chỉ được load hoặc initialize khi thực sự bị dùng tới.

Mình cũng từng dễ nhầm `import` nghĩa là class đã được load. Thật ra `import` chỉ là compile-time convenience cho source code.

Một nhầm lẫn nữa là thấy có `static` block thì tưởng nó chạy khi compile hoặc ngay lúc app start. Thực tế nó chỉ chạy lúc class được initialize theo đúng trigger của JVM.

## How it actually works

Pha `load` xảy ra khi `ClassLoader` đọc binary definition của class và tạo ra `Class` object tương ứng trong JVM.

Sau đó đến `link`. Nếu nhìn ở mức vừa đủ để debug, `link` gồm ba ý chính: `verify` kiểm tra bytecode có hợp lệ không, `prepare` cấp phát và gán default value cho static fields, còn `resolve` biến symbolic references thành các runtime-resolved target mà JVM có thể dùng trực tiếp khi cần.

Nhưng ngay cả sau `load` và `link`, logic `static` của bạn vẫn chưa chắc đã chạy. Pha `initialize` mới là lúc JVM thực thi class initialization logic như `static` field initializers và `static` blocks, theo thứ tự chúng xuất hiện trong source.

Việc tách riêng các pha này giúp JVM vừa an toàn vừa lazy: class có thể được biết tới và kiểm tra trước, nhưng chỉ trả chi phí side effect khi thật sự cần.

Đây là bảng tách phần việc cho dễ nhớ:

| Phase | JVM làm gì | Chưa làm gì |
| --- | --- | --- |
| `load` | Tìm binary definition, tạo `Class` object | Chưa chạy `static` code |
| `link` | `verify`, `prepare`, có thể `resolve` | Chưa chạy `static` block hay field initializer thực tế |
| `initialize` | Chạy `static` field initializers và `static` blocks | Không tạo instance trừ khi code của bạn tự tạo |

Một bảng khác đáng nhớ hơn khi debug là trigger nào làm `initialize` xảy ra:

| Trigger thường gặp | Có làm initialize không? | Ghi chú |
| --- | --- | --- |
| Dùng `new SomeClass()` | Có | Trước khi tạo instance |
| Đọc hoặc ghi `static` field không phải compile-time constant | Có | Ví dụ `Config.version` |
| Gọi `static` method | Có | Ví dụ `Config.load()` |
| `Class.forName("...")` mặc định | Có | Trừ khi dùng overload tắt initialize |
| Chỉ `import` class | Không | Chỉ là compile-time syntax help |
| Đọc `static final` compile-time constant | Thường không | Vì giá trị có thể đã bị inline |

Analogy “bản thiết kế vào hệ thống” chỉ hữu ích nếu chốt đúng kết luận: load không đồng nghĩa với side effect đã nổ.

## Code example

```java
class Config {
    // initialization runs only when the class is actually initialized
    static int version = loadVersion();

    static {
        // this side effect is why class initialization timing matters
        System.out.println("Config initialized");
    }

    static int loadVersion() {
        // makes the initialization moment visible instead of theoretical
        System.out.println("Loading version");
        return 1;
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        // at this point Config may still be completely untouched by the runtime
        System.out.println("Start");

        // forcing class lookup this way also triggers initialization by default
        Class<?> clazz = Class.forName("Config");

        // by now the static initialization has already happened exactly once
        System.out.println(Config.version);
    }
}
```

Nếu chạy ví dụ này, bạn sẽ thấy `Config initialized` và `Loading version` không xuất hiện ở lúc compile, cũng không phải chỉ vì class tồn tại trong project. Chúng chỉ xuất hiện khi JVM đi tới bước cần initialize class đó.

## When to use / when NOT to use

Dùng mental model này khi debug lỗi startup, `static` side effect, classpath problem, lazy loading, hoặc khi không hiểu vì sao một class “chưa gọi mà đã nổ”.

Nếu app fail ở startup chỉ vì một static initializer đọc config lỗi, bạn cần nghĩ theo class initialization timing chứ không chỉ nhìn stack trace bề mặt.

Không cần dùng model này khi chỉ code business logic CRUD hằng ngày. Lúc đó nó thường sâu hơn mức cần thiết.

## How this connects to Spring

Spring Boot sống khá sát với cơ chế này, nhưng không phải theo kiểu một-một hoàn toàn. Lúc startup, framework thường scan classpath và đọc metadata trước; còn việc một class thật sự được initialize hay một object thật sự được instantiate vẫn phụ thuộc vào lúc nào runtime và container cần tới nó.

Đây là lý do một bean có thể không được tạo ra dù class tồn tại, hoặc một lỗi trong `static` initializer có thể làm app fail rất sớm trước cả khi request đầu tiên tới. Khi hiểu JVM class loading, mình sẽ đọc startup behavior của Spring tỉnh táo hơn nhiều.

## Gotchas

- `import` không làm class được load hay initialize; nó chỉ giúp compiler hiểu tên ngắn.
- `static final` compile-time constant có thể bị inline, nên truy cập nó chưa chắc trigger class initialization như bạn nghĩ.
- `Class.forName()` mặc định có thể trigger initialization và kéo theo side effect từ `static` code.

## Check yourself

- Một class được `load` rồi thì `static` block đã chắc chắn chạy chưa?
- Sự khác nhau ngắn gọn giữa `load`, `link`, và `initialize` là gì?
- Vì sao `import` không nói gì về runtime class loading?
- Truy cập `static final` compile-time constant có thể không trigger initialize, vì sao?
- Khi app Spring fail rất sớm ở startup vì `static` code, bạn nên nghĩ tới pha nào?

## Links

- [[001-java-in-jvm-eyes]]
- [[006-compile-time-vs-runtime]]
- [[004-pass-by-value]]
- [JVMS Chapter 5, Loading, Linking, and Initializing](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-5.html)
- [JLS Chapter 12, Execution](https://docs.oracle.com/javase/specs/jls/se21/html/jls-12.html)
- [ClassLoader Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ClassLoader.html)
