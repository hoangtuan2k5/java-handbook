# Proxy

## What is it

`Proxy` là structural pattern trong đó một object trung gian đứng trước object thật và kiểm soát việc truy cập tới object đó.

Proxy có cùng interface với target nên caller dùng gần như giống nhau, nhưng call path đã đi qua một lớp bọc có thể chặn, kiểm tra, trì hoãn, cache, hoặc forward call theo rule riêng.

Điểm đáng học nhất là `Proxy` không tồn tại để thêm business feature mới. Nó tồn tại để kiểm soát access hoặc lifecycle của object thật.

### Quick distinction

| Câu hỏi | `Proxy` trả lời thế nào |
|---|---|
| Wrapper này đứng trước target để kiểm soát truy cập không? | Thường có |
| Caller có dùng cùng contract với target không? | Có |
| Lợi ích chính | chặn, cache, lazy-load, remote access, cross-cutting gatekeeping |
| Giá phải trả | call path ẩn hơn, debug khó hơn |
| Pattern dễ nhầm | `Decorator` |

## How I used to misunderstand it

Mình từng nghĩ proxy chỉ là một tên khác của decorator. Hai pattern đều bọc object, nhưng ý định khác nhau.

Proxy thường tập trung vào kiểm soát truy cập hoặc quản lý lifecycle. `Decorator` thì tập trung vào cộng thêm behavior theo kiểu stackable. Nếu không phân biệt intent, mọi wrapper sẽ nhìn như nhau và pattern mất tác dụng dạy học.

## How it actually works

Proxy implement cùng contract với target. Khi caller gọi method, proxy có thể:

- kiểm tra permission
- lấy dữ liệu từ cache
- lazy-create object thật
- đại diện cho remote resource
- rồi mới forward sang target

```text
caller
  -> proxy
       -> allow / deny / cache / create target
       -> target
```

### Proxy vs close alternatives

| Nếu mục tiêu là... | Pattern gần hơn |
|---|---|
| Kiểm soát truy cập hoặc lifecycle | `Proxy` |
| Cộng nhiều behavior theo kiểu xếp chồng | `Decorator` |
| Đổi interface cũ sang interface mới | `Adapter` |

## Code example

```java
interface ReportService {
    String loadReport(String id);
}

class SecurityProxy implements ReportService {
    private final ReportService target;

    SecurityProxy(ReportService target) {
        this.target = target;
    }

    public String loadReport(String id) {
        if (id.startsWith("admin-")) {
            throw new IllegalArgumentException("Forbidden");
        }
        return target.loadReport(id);
    }
}
```

Proxy ở đây không đổi business capability cốt lõi của `ReportService`. Nó chỉ đứng trước để quyết định call có được đi tiếp hay không.

## When to use / when NOT to use

Use `Proxy` khi:

- cần access control, caching, lazy loading, remote access, hoặc rate limiting
- muốn caller vẫn dùng cùng interface với target
- concern nằm ở chuyện call có nên đi tiếp, hoặc đi tiếp bằng cách nào

Do NOT use `Proxy` khi:

- mục tiêu là cộng thêm nhiều behavior tùy chọn theo kiểu tổ hợp
- wrapper bắt đầu biến đổi business result quá nhiều
- bạn đang cố dịch một interface cũ sang contract mới

Misconception thường gặp là cứ thấy wrapper là gọi `Proxy`. Nếu wrapper chủ yếu cộng thêm behavior chồng lớp, `Decorator` thường đúng hơn.

## How this connects to real Java projects

Spring AOP, `@Transactional`, `@Cacheable`, và nhiều security advice thường dựa trên proxy. Bean được inject có thể là proxy đứng trước object thật để thêm transaction boundary, cache lookup, hoặc authorization check.

Đây là ví dụ thực tế rất mạnh vì caller vẫn thấy cùng contract, nhưng runtime behavior đã khác.

## Gotchas

- Self-invocation trong cùng bean có thể bỏ qua Spring proxy.
- Proxy ẩn đi call path thật nên stack trace dài và khó đọc hơn.
- Equality, serialization, hoặc type check có thể gây bất ngờ nếu code giả định object là target thật.
- Proxy cache stale data mà không có invalidation rule rõ sẽ tạo bug khó tái hiện.
- Dùng proxy để nhét business logic chính sẽ làm intent bị lệch.

## Handbook rule

- Proxy cho cross-cutting concern (security/log/cache/transaction); không nhét business logic chính.
- Self-invocation trong bean bypass Spring proxy; gọi qua bean injected khi cần.
- Proxy ẩn call path; debug/log phải thêm context để trace.
- Cache trong proxy phải có invalidation rule rõ ràng.
- Đừng giả định proxy là target object; equality/serialization/type check có thể bất ngờ.

## Check yourself

- `Proxy` khác `Decorator` ở intent chính như thế nào?
- Khi nào lazy loading là use case tự nhiên của proxy?
- Vì sao Spring AOP thường được giải thích bằng proxy?
- Nếu wrapper thay đổi business output quá nhiều, có còn đúng tinh thần `Proxy` không?
- Self-invocation trong Spring liên quan gì đến proxy boundary?

## Exercises

### Exercise 1: Should Use Protective Proxy

Độ khó: Easy

Đề bài:
Cho `needsAccessControl`, `heavyObject`, và `remoteCall`. Hãy trả về `true` khi proxy là hợp lý vì tồn tại ít nhất một trong các concern đó.

Ví dụ 1:

Đầu vào:
```text
needsAccessControl = false, heavyObject = true, remoteCall = false
```

Đầu ra:
```text
true
```

Giải thích:
Việc đặt lớp bảo vệ lười phía trước một heavy object đã là một use case hợp lệ của proxy.

Ràng buộc:

- Tất cả input đều là boolean
- Trả về một boolean
- Không cần xét các type đặc thù của framework

### Exercise 2: Count Cached Proxy Responses

Độ khó: Easy

Đề bài:
Cho hai array `cacheHits` và `authorized` có cùng độ dài. Hãy đếm xem có bao nhiêu request có thể được phục vụ trực tiếp từ proxy cache. Một request chỉ được phục vụ từ cache khi cả `cacheHits[i]` và `authorized[i]` đều là `true`.

Ví dụ 1:

Đầu vào:
```text
cacheHits = [true, false, true, true]
authorized = [true, true, false, true]
```

Đầu ra:
```text
2
```

Giải thích:
Chỉ request thứ nhất và thứ tư vừa được authorize vừa có trong cache.

Ràng buộc:

- Cả hai array đều là non-null
- Cả hai array có cùng độ dài
- Độ dài array nằm trong đoạn từ 0 đến 100000

### Exercise 3: Build Proxy Audit Line

Độ khó: Medium

Đề bài:
Cho `resourceId`, `userRole`, `allowed`, và `cacheHit`. Hãy trả về một string theo đúng format `"<resourceId>|<userRole>|<decision>|<source>"`, trong đó `decision` là `ALLOW` hoặc `DENY`, còn `source` là `CACHE` hoặc `TARGET`.

Ví dụ 1:

Đầu vào:
```text
resourceId = "report-7", userRole = "ADMIN", allowed = true, cacheHit = false
```

Đầu ra:
```text
"report-7|ADMIN|ALLOW|TARGET"
```

Giải thích:
Request này được cho phép và phải đi tới target thật vì proxy cache bị miss.

Ràng buộc:

- `resourceId` là non-null
- `userRole` là non-null
- Output format phải khớp chính xác

## Links

- [[002-Decorator]]
- [[../../../01_Core/Reflection-and-Annotation/004-Dynamic-proxy]]
- [Spring Framework, Proxying Mechanisms](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)
- [Refactoring Guru, Proxy](https://refactoring.guru/design-patterns/proxy)
- [Oracle, Dynamic Proxies](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html)
