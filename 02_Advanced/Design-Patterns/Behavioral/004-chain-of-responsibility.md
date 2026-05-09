# Chain of Responsibility

## What is it

`Chain of Responsibility` là behavioral pattern trong đó request đi qua một chuỗi handler, mỗi handler có quyền xử lý, xử lý một phần rồi chuyển tiếp, hoặc bỏ qua để handler tiếp theo thử.

Nó giải quyết bài toán routing request qua nhiều rule theo thứ tự mà caller không cần biết handler cụ thể nào sẽ xử lý cuối cùng.

Pattern này hữu ích khi logic kiểm tra hoặc xử lý có thể xếp thành nhiều tầng như authentication, validation, rate limit, approval level, hoặc fallback support tiers.

### Quick distinction

| Câu hỏi | `Chain of Responsibility` trả lời thế nào |
|---|---|
| Request đi qua nhiều bước hay fan-out ra nhiều listener? | Đi qua nhiều bước |
| Mỗi bước có thể dừng chain không? | Có |
| Caller có biết handler cuối cùng không? | Không cần biết |
| Lợi ích chính | Tách pipeline khỏi caller |
| Giá phải trả | Order và trace quan trọng hơn nhiều |

## How I used to misunderstand it

Mình từng nhầm pattern này với `Observer` vì đều có nhiều object tham gia sau một request.

Khác biệt cốt lõi là `Observer` fan-out cho nhiều subscriber cùng phản ứng, còn `Chain of Responsibility` dẫn request đi qua một tuyến xử lý. Thường chỉ một handler quyết định kết thúc, hoặc từng handler xử lý một phần theo thứ tự rõ ràng.

## How it actually works

Mỗi handler biết handler kế tiếp. Khi nhận request, nó tự quyết định một trong ba hướng:

- xử lý hoàn toàn và dừng chain
- xử lý một phần rồi chuyển tiếp
- bỏ qua và chuyển tiếp ngay

Caller chỉ gửi request vào entry point đầu chuỗi.

```text
request
  -> handler A
       -> handler B
            -> handler C
                 -> HANDLED hoặc UNHANDLED
```

### Decision matrix

| Tình huống | `Chain of Responsibility` hợp không? | Vì sao |
|---|---|---|
| Request cần đi qua nhiều rule theo thứ tự | Hợp | chain diễn tả pipeline tốt |
| Nhiều listener cùng phản ứng độc lập | Không | `Observer` gần hơn |
| Caller luôn biết đúng target handler | Ít hợp | gọi trực tiếp rõ hơn |
| Muốn thêm hoặc bỏ từng tầng dễ dàng | Hợp | handler tách rời nhau |
| Order mơ hồ hoặc thay đổi lung tung | Cẩn thận | chain rất nhạy với thứ tự |

## Code example

```java
abstract class Handler {
    private Handler next;

    Handler linkWith(Handler next) {
        this.next = next;
        return next;
    }

    protected String next(String request) {
        return next == null ? "UNHANDLED" : next.handle(request);
    }

    abstract String handle(String request);
}
```

Điểm đáng chú ý là mỗi handler chỉ cần biết mắt xích kế tiếp, không cần biết toàn bộ pipeline.

## When to use / when NOT to use

Use `Chain of Responsibility` khi:

- request cần được xét qua nhiều tầng rule theo thứ tự
- muốn thêm, bỏ, hoặc sắp xếp lại từng tầng xử lý tương đối độc lập
- caller không nên gắn cứng với handler cụ thể nào

Do NOT use `Chain of Responsibility` khi:

- caller luôn biết đích đến chính xác
- nhiều handler phải cùng nhận một event độc lập
- pipeline quá đơn giản, chỉ một hoặc hai bước cố định gọi trực tiếp là đủ

Misconception hay gặp là thấy có nhiều class nối nhau rồi gọi là chain. Nếu nhiều bên cùng phản ứng sau một sự kiện, đó thường là `Observer`, không phải chain.

## How this connects to Spring

Spring Security filter chain là ví dụ rất thực tế. Một HTTP request đi qua nhiều filter theo thứ tự. Filter nào cũng có thể kiểm tra, enrich context, reject, hoặc cho request đi tiếp.

`HandlerInterceptor` trong Spring MVC cũng mang tinh thần tương tự, dù cơ chế chi tiết khác nhau.

## Gotchas

- Chain order sai có thể làm request bị reject quá sớm hoặc bỏ lọt bước bảo mật quan trọng.
- Request không được handler nào nhận cần có fallback rõ ràng.
- Handler làm quá nhiều việc sẽ phá tính composable của chain.
- Logging thiếu context sẽ làm rất khó biết request dừng ở mắt xích nào.
- Chain quá dài mà không có naming rõ ràng sẽ rất khó đọc.

## Check yourself

- `Chain of Responsibility` khác `Observer` ở điểm fan-out và stop condition như thế nào?
- Vì sao thứ tự handler là một phần của business behavior chứ không chỉ là chi tiết kỹ thuật?
- Khi nào caller gọi trực tiếp sẽ tốt hơn dựng một chain?
- Một request đi hết chain mà không ai xử lý thì hệ thống nên phản ứng ra sao?
- Nếu một handler vừa validate vừa gọi nhiều service phụ không liên quan, có dấu hiệu gì xấu?

## Exercises

### Exercise 1: Find Handling Stage

Độ khó: Easy

Đề bài:
Cho hai array `handlerNames` và `minScores` có cùng độ dài, cùng với một integer `requestScore`. Hãy trả về tên của handler đầu tiên có minimum score nhỏ hơn hoặc bằng `requestScore`. Nếu không có handler nào xử lý được request, trả về `"UNHANDLED"`.

Ví dụ 1:

Đầu vào:
```text
handlerNames = ["tier1", "tier2", "tier3"]
minScores = [90, 70, 50]
requestScore = 65
```

Đầu ra:
```text
"tier3"
```

Giải thích:
Request này không đủ mạnh cho `tier1` hoặc `tier2`, nên nó rơi xuống `tier3`.

Ràng buộc:

- Cả hai array đều là non-null
- Cả hai array có cùng độ dài
- Độ dài array nằm trong đoạn từ 0 đến 100000

### Exercise 2: Count Resolved Requests

Độ khó: Medium

Đề bài:
Cho hai array `requestScores` và `cutoffScores`, trong đó mỗi `requestScores[i]` tương ứng với `cutoffScores[i]`. Hãy đếm xem có bao nhiêu request được handler hiện tại xử lý thành công. Một request được xem là resolved khi `requestScores[i] >= cutoffScores[i]`.

Ví dụ 1:

Đầu vào:
```text
requestScores = [85, 40, 91, 60]
cutoffScores = [80, 50, 90, 60]
```

Đầu ra:
```text
3
```

Giải thích:
Chỉ request thứ hai không đạt cutoff của handler tương ứng.

Ràng buộc:

- Cả hai array đều là non-null
- Cả hai array có cùng độ dài
- Độ dài array nằm trong đoạn từ 0 đến 100000

### Exercise 3: Build Escalation Trace

Độ khó: Medium

Đề bài:
Cho `handlerNames` và `canHandle` có cùng độ dài. Hãy build một trace string liệt kê từng handler theo thứ tự cho tới entry `true` đầu tiên. Dùng đúng format `"name1->name2->...->HANDLED"`. Nếu không handler nào xử lý được request, trả về `"name1->name2->...->UNHANDLED"`.

Ví dụ 1:

Đầu vào:
```text
handlerNames = ["auth", "quota", "service"]
canHandle = [false, false, true]
```

Đầu ra:
```text
"auth->quota->service->HANDLED"
```

Giải thích:
Request này đi qua hai handler trước khi được service handler chấp nhận.

Ràng buộc:

- Cả hai array đều là non-null
- Cả hai array có cùng độ dài
- Tên handler là các string non-null

## Links

- [[001-Observer]]
- [[../../Testing/001-unit-test-vs-integration-test]]
- [Spring Security, Servlet Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [Refactoring Guru, Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility)
- [Spring Framework, HandlerInterceptor](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/handlermapping-interceptor.html)
