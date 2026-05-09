# Decorator

## What is it

`Decorator` là structural pattern cho phép bọc một object bằng nhiều lớp wrapper cùng interface để cộng thêm behavior từng lớp một.

Nó giải quyết bài toán mở rộng hành vi theo kiểu tổ hợp mà không cần tạo quá nhiều subclass. Mỗi decorator thêm một capability nhỏ nhưng vẫn giữ cùng contract với component gốc.

Điểm dạy học quan trọng là `Decorator` nhấn vào composition. Thứ tự bọc có thể làm kết quả khác nhau, đây là điều inheritance diễn tả kém hơn.

### Quick distinction

| Câu hỏi | `Decorator` trả lời thế nào |
|---|---|
| Có muốn cộng thêm behavior từng lớp một không? | Có |
| Các lớp bọc có cùng contract với component gốc không? | Có |
| Lợi ích chính | behavior stackable, tránh nổ subclass |
| Giá phải trả | call stack sâu hơn, order quan trọng |
| Pattern dễ nhầm | `Proxy` |

## How I used to misunderstand it

Mình từng nghĩ `Decorator` chỉ là inheritance trá hình. Thực ra pattern này cố tình dùng composition để cộng behavior linh hoạt hơn subclass.

Điểm dễ bỏ sót là thứ tự bọc ảnh hưởng kết quả. `compress -> encrypt` khác `encrypt -> compress`. Điều này làm pattern rất mạnh, nhưng cũng khiến nó dễ bị lạm dụng nếu behavior không thực sự độc lập.

## How it actually works

Component gốc định nghĩa interface chung. Concrete component làm hành vi cơ bản. Mỗi decorator cũng implement interface đó, giữ reference tới component bên trong, rồi thêm logic trước hoặc sau khi forward call.

Vì decorator và component cùng contract, ta có thể stack nhiều decorator.

```text
caller
  -> decorator C
       -> decorator B
            -> decorator A
                 -> base component
```

### Decision matrix

| Tình huống | `Decorator` hợp không? | Vì sao |
|---|---|---|
| Cần cộng nhiều behavior độc lập theo tổ hợp | Hợp | stackable tự nhiên |
| Chỉ cần gatekeep hoặc lazy-load | Ít hợp | `Proxy` đúng intent hơn |
| Chỉ cần translate interface | Không | `Adapter` gần hơn |
| Thứ tự cộng behavior ảnh hưởng output | Hợp | decorator diễn tả tốt |
| Wrapper biết quá nhiều về concrete type bên trong | Cẩn thận | abstraction đang rò rỉ |

## Code example

```java
interface Notifier {
    String send(String message);
}

class BasicNotifier implements Notifier {
    public String send(String message) {
        return message;
    }
}

class LoggingDecorator implements Notifier {
    private final Notifier target;

    LoggingDecorator(Notifier target) {
        this.target = target;
    }

    public String send(String message) {
        return "LOG->" + target.send(message);
    }
}
```

Ví dụ này cho thấy behavior mới được cộng quanh component cũ mà không sửa component gốc.

## When to use / when NOT to use

Use `Decorator` khi:

- muốn cộng thêm behavior theo tổ hợp như formatting, compression, encryption, tracing
- muốn tránh tạo rất nhiều subclass cho mọi combination feature
- mỗi behavior có thể đứng riêng thành một lớp nhỏ

Do NOT use `Decorator` khi:

- mục tiêu chỉ là kiểm soát truy cập hoặc lazy-load
- behavior mới không độc lập mà phụ thuộc quá mạnh vào concrete implementation bên trong
- flow đơn giản đến mức một wrapper duy nhất hoặc một method đã đủ rõ

Misconception thường gặp là thấy có wrapper rồi gọi luôn là `Decorator`. Nếu wrapper chủ yếu làm gatekeeper, `Proxy` thường đúng hơn.

## How this connects to Spring

Trong Spring app, decorator-style composition hay xuất hiện ở service wrappers, response formatting layers, hoặc các bean bọc bean khác để cộng thêm logging, resilience, hoặc tracing.

Điểm cần nhớ là không phải mọi wrapper bean đều là `Decorator`, nhưng rất nhiều wrapper giữ cùng contract và cộng behavior theo composition đúng tinh thần pattern này.

## Gotchas

- Thứ tự decorator quan trọng.
- Quá nhiều decorator nhỏ làm stack trace dài và khó lần logic.
- Nếu decorator bắt đầu biết quá nhiều về concrete type bên trong, abstraction đang rò rỉ.
- Mutable shared state trong decorator chain có thể tạo side effect khó đoán.
- Decorator quá hạt nhỏ có thể làm code đọc khó hơn lợi ích nó mang lại.

## Check yourself

- `Decorator` khác `Proxy` ở mục tiêu chính như thế nào?
- Vì sao thứ tự bọc có thể thay đổi kết quả cuối?
- Khi nào subclass explosion là tín hiệu nên nghĩ đến `Decorator`?
- Nếu wrapper phải cast về concrete type bên trong, pattern đang gặp vấn đề gì?
- Một chain decorator quá dài đang đổi lấy cost gì?

## Exercises

### Exercise 1: Count Enabled Decorators

Độ khó: Easy

Đề bài:
Cho `loggingEnabled`, `compressionEnabled`, và `encryptionEnabled`. Hãy trả về số lượng decorator đang active.

Ví dụ 1:

Đầu vào:
```text
loggingEnabled = true, compressionEnabled = false, encryptionEnabled = true
```

Đầu ra:
```text
2
```

Giải thích:
Sẽ có hai wrapper được stack quanh base component.

Ràng buộc:

- Tất cả input đều là boolean
- Trả về một integer
- Bỏ qua runtime reflection

### Exercise 2: Calculate Decorated Price

Độ khó: Medium

Đề bài:
Cho `basePrice`, `giftWrap`, `priorityPack`, và `insurance`. Hãy trả về giá cuối cùng sau khi cộng thêm 5 cho gift wrap, 15 cho priority pack, và 20 cho insurance.

Ví dụ 1:

Đầu vào:
```text
basePrice = 100, giftWrap = true, priorityPack = false, insurance = true
```

Đầu ra:
```text
125
```

Giải thích:
Các decorator cộng thêm chi phí độc lập lên trên base component.

Ràng buộc:

- `basePrice` nằm trong đoạn từ 0 đến 100000
- Tất cả flag đều là boolean
- Trả về một integer total

### Exercise 3: Build Decorated Label

Độ khó: Medium

Đề bài:
Cho `baseLabel`, `loggingEnabled`, `compressionEnabled`, và `encryptionEnabled`. Hãy trả về decorated label theo đúng thứ tự `LOG->ZIP->ENC-><baseLabel>`, nhưng chỉ bao gồm các decorator đang được bật. Nếu không có decorator nào được bật, trả về `baseLabel` nguyên vẹn.

Ví dụ 1:

Đầu vào:
```text
baseLabel = "payload", loggingEnabled = true, compressionEnabled = true, encryptionEnabled = false
```

Đầu ra:
```text
"LOG->ZIP->payload"
```

Giải thích:
Label này cho thấy thứ tự wrapper được áp dụng quanh base component.

Ràng buộc:

- `baseLabel` là non-null
- Output order luôn phải là logging, rồi compression, rồi encryption
- Output luôn là non-null

## Links

- [[001-Proxy]]
- [[003-Adapter]]
- [Refactoring Guru, Decorator](https://refactoring.guru/design-patterns/decorator)
- [Spring Framework, Customizing Beans with BeanPostProcessor](https://docs.spring.io/spring-framework/reference/core/beans/factory-extension.html)
- [SourceMaking, Decorator](https://sourcemaking.com/design_patterns/decorator)
