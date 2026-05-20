# Observer

## What is it

`Observer` là behavioral pattern dùng khi một object phát ra thay đổi, còn nhiều object khác muốn phản ứng theo thay đổi đó mà không bị hard-code trực tiếp vào nơi phát sinh event.

Nó giải quyết bài toán one-to-many notification. `Subject` chỉ biết rằng có nhiều `observer` đang đăng ký một contract chung. Nó không cần biết observer nào sẽ gửi email, ghi audit log, cập nhật metric, hay refresh cache.

Điểm dạy học quan trọng ở đây là: `Observer` không phải cách làm cho code “thông minh hơn”. Nó là cách tách nơi tạo ra thay đổi khỏi nơi tiêu thụ thay đổi.

### Quick distinction

| Câu hỏi | `Observer` trả lời thế nào |
|---|---|
| Có bao nhiêu bên phản ứng sau một thay đổi? | Có thể nhiều bên cùng phản ứng |
| Publisher có biết chi tiết subscriber không? | Không, chỉ biết contract chung |
| Flow là fan-out hay đi tuần tự qua từng tầng? | Fan-out |
| Lợi ích chính | Giảm coupling giữa publisher và reaction |
| Giá phải trả | Side effect bị phân tán, khó trace hơn |

## How I used to misunderstand it

Mình từng nghĩ `Observer` chỉ là “callback nhiều chỗ” hoặc chỉ hợp cho UI. Cách hiểu đó làm mờ mất ý quan trọng nhất: pattern này là về dependency direction.

Publisher không nên biết hết mọi hậu quả của một event. Nếu `OrderService` tự gọi email, audit, loyalty point, analytics, rồi cache invalidation, nó đang gánh quá nhiều knowledge về downstream behavior.

## How it actually works

`Subject` thường có các thao tác như subscribe, unsubscribe, và notify. Khi event xảy ra, nó duyệt qua danh sách observer đã đăng ký và gọi một method chung như `onOrderCreated(...)` hoặc `update(...)`.

Điểm mạnh là coupling chỉ còn nằm ở interface. Bạn có thể thêm một observer mới mà không phải sửa logic tạo event.

Đổi lại, flow xử lý trở nên gián tiếp hơn. Một action nhìn có vẻ đơn giản có thể kích hoạt nhiều reaction ẩn phía sau.

```text
State change xảy ra ở subject
        |
        +--> notify observer A
        +--> notify observer B
        +--> notify observer C
```

### Decision matrix

| Tình huống | `Observer` hợp không? | Vì sao |
|---|---|---|
| Một event cần nhiều reaction độc lập | Hợp | fan-out tự nhiên |
| Chỉ có đúng một downstream action ổn định | Ít hợp | gọi trực tiếp rõ hơn |
| Cần trace tuyến tính, dễ debug từng bước | Cẩn thận | side effect bị tản ra |
| Nhiều listener có thể thêm bớt theo thời gian | Hợp | mở rộng mà ít chạm publisher |
| Các bước phải chạy theo thứ tự business cố định | Thường không | `Chain of Responsibility` hoặc orchestration rõ hơn |

## Code example

```java
import java.util.ArrayList;
import java.util.List;

interface OrderObserver {
    void onOrderCreated(String orderId);
}

class OrderSubject {
    private final List<OrderObserver> observers = new ArrayList<>();

    void subscribe(OrderObserver observer) {
        observers.add(observer);
    }

    void createOrder(String orderId) {
        for (OrderObserver observer : observers) {
            observer.onOrderCreated(orderId);
        }
    }
}
```

Ví dụ này cho thấy `OrderSubject` không cần biết observer làm gì sau khi nhận event. Nó chỉ phát tín hiệu theo contract chung.

## When to use / when NOT to use

Use `Observer` khi:

- một thay đổi cần fan out tới nhiều reaction độc lập
- muốn thêm reaction mới mà không sửa code phát event
- downstream concern là notification, audit, metrics, cache invalidation, hoặc domain event handling

Do NOT use `Observer` khi:

- chỉ có một hành động downstream ổn định và đơn giản
- flow cần cực kỳ tuyến tính, dễ đọc, dễ debug
- business logic phụ thuộc mạnh vào thứ tự chạy của các handler

Misconception hay gặp là thấy nhiều object tham gia rồi gọi luôn là `Observer`. Nếu request đi qua từng tầng và thường dừng ở một tầng nào đó, đó gần hơn với `Chain of Responsibility`, không phải `Observer`.

## How this connects to real Java projects

Spring có `ApplicationEventPublisher` và `@EventListener`, đây là ví dụ rất gần của `Observer`. Service gốc publish `OrderCreatedEvent`, còn nhiều listener độc lập có thể phản ứng.

Điểm cần phân biệt là Spring event mặc định có thể chạy đồng bộ. Vì vậy nếu listener nặng, request chính cũng có thể bị chậm theo.

## Gotchas

- Observer chạy đồng bộ có thể kéo chậm publisher.
- Quá nhiều observer làm side effect phân tán, khó lần nguyên nhân khi data thay đổi.
- Event payload mutable dễ tạo bug nếu observer trước sửa state observer sau đang đọc.
- Nếu business logic phụ thuộc order notify nhưng không nói rõ order, bug sẽ rất khó tái hiện.
- Dùng event cho mọi thứ có thể biến flow đơn giản thành flow khó hiểu.

## Handbook rule

- Observer chỉ dùng khi nhiều reaction độc lập cần fan out từ cùng một thay đổi.
- Không phụ thuộc vào order notify; nếu order quan trọng phải làm rõ trong contract.
- Event payload phải immutable; tránh observer trước mutate state observer sau đọc.
- Observer chạy đồng bộ kéo chậm publisher; cân nhắc async hoặc batch.
- Đừng nhầm với Chain of Responsibility: chain dừng ở 1 handler, observer fan out tới mọi handler.

## Check yourself

- Khi nào `Observer` tốt hơn gọi trực tiếp một service phụ?
- Vì sao `Observer` giảm coupling nhưng lại tăng cost debug?
- Một event mà nhiều listener đều phụ thuộc thứ tự chạy có phải là dấu hiệu thiết kế chưa ổn không?
- `Observer` khác `Chain of Responsibility` ở điểm fan-out như thế nào?
- Nếu listener làm việc nặng, bạn cần nghĩ thêm điều gì ngoài interface design?

## Exercises

### Exercise 1: Should Notify Observer

Độ khó: Easy

Đề bài:
Cho `observerTopic`, `eventTopic`, và `active`. Hãy trả về `true` chỉ khi observer đang active và topic của nó khớp chính xác với event topic.

Ví dụ 1:

Đầu vào:
```text
observerTopic = "order.created", eventTopic = "order.created", active = true
```

Đầu ra:
```text
true
```

Giải thích:
Observer này subscribe đúng topic và hiện đang active.

Ràng buộc:

- `observerTopic` là non-null
- `eventTopic` là non-null
- Việc so sánh có phân biệt hoa thường

### Exercise 2: Count Observers For Event

Độ khó: Easy

Đề bài:
Cho hai array `observerTopics` và `activeFlags` có cùng độ dài, cùng với một `eventTopic`. Hãy trả về số lượng observer nên nhận event. Một observer chỉ nhận event khi `activeFlags[i]` là `true` và `observerTopics[i]` bằng `eventTopic`.

Ví dụ 1:

Đầu vào:
```text
observerTopics = ["order.created", "payment.failed", "order.created"]
activeFlags = [true, true, false]
eventTopic = "order.created"
```

Đầu ra:
```text
1
```

Giải thích:
Chỉ observer đầu tiên vừa active vừa subscribe đúng target topic.

Ràng buộc:

- Cả hai array đều là non-null
- Cả hai array có cùng độ dài
- Độ dài array nằm trong đoạn từ 0 đến 100000

### Exercise 3: Build Observer Dispatch Summary

Độ khó: Medium

Đề bài:
Cho `observerNames`, `activeFlags`, và `eventTopic`. Hãy trả về một string theo đúng format `"<eventTopic>:name1,name2,..."`, chỉ chứa các observer đang active theo đúng input order. Nếu không có observer nào active, trả về `"<eventTopic>:none"`.

Ví dụ 1:

Đầu vào:
```text
observerNames = ["email", "audit", "metrics"]
activeFlags = [true, false, true]
eventTopic = "order.created"
```

Đầu ra:
```text
"order.created:email,metrics"
```

Giải thích:
Các observer không active sẽ bị bỏ qua, nhưng các tên còn lại vẫn giữ nguyên thứ tự ban đầu.

Ràng buộc:

- `observerNames` và `activeFlags` đều là non-null
- Cả hai array có cùng độ dài
- Tên observer là các string non-null

## Links

- [[002-Strategy]]
- [[004-Chain-of-responsibility]]
- [[../../../01_Core/Reflection-and-Annotation/004-Dynamic-proxy]]
- [Spring Framework, Application Events and Listeners](https://docs.spring.io/spring-framework/reference/core/beans/context-introduction.html#context-functionality-events)
- [Refactoring Guru, Observer](https://refactoring.guru/design-patterns/observer)
- [Martin Fowler, Event-Driven](https://martinfowler.com/articles/201701-event-driven.html)
