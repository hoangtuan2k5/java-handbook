# Singleton

## What is it

`Singleton` là creational pattern đảm bảo chỉ có đúng một instance của một class trong một phạm vi nhất định, thường là một JVM process hoặc một application context.

Nó giải quyết bài toán chia sẻ một điểm truy cập chung cho resource hoặc coordinator dùng chung. Ý chính không phải là “global variable đẹp hơn”, mà là kiểm soát số lượng instance và cách truy cập instance đó.

Vì pattern này bị lạm dụng rất nhiều, điều quan trọng nhất khi học không phải là cách viết `getInstance()`. Điều quan trọng là hiểu khi nào “chỉ một instance” thật sự là business need, và khi nào nó chỉ là thói quen xấu.

### Quick distinction

| Câu hỏi | `Singleton` trả lời thế nào |
|---|---|
| Có cần đúng một instance dùng chung không? | Có |
| Vì sao cần một instance? | để điều phối resource hoặc giữ shared access point |
| Lợi ích chính | nhất quán, tránh tạo dư thừa |
| Giá phải trả | hidden dependency, khó test, dễ lỗi shared state |
| Nhầm lẫn phổ biến | cứ dùng nhiều nơi là nên singleton |

## How I used to misunderstand it

Mình từng nghĩ `Singleton` là mặc định tốt cho mọi service dùng nhiều nơi. Sau này mới thấy đó là hiểu nhầm nguy hiểm.

Một object được share rộng rãi không có nghĩa nó nên trở thành singleton. Nếu object giữ mutable state theo request, pattern này có thể biến thành nơi rò state, race condition, hoặc test pollution.

## How it actually works

Class singleton thường ẩn constructor rồi chỉ expose một access point như `getInstance()`. Instance có thể được tạo eager khi class load, hoặc lazy khi lần đầu được gọi.

Nếu lazy trong môi trường multi-thread, implementation phải thread-safe để tránh tạo ra nhiều instance ngoài ý muốn.

```text
need one coordinated instance?
        |
        +--> yes -> xem tiếp state có shared an toàn không
        |
        +--> no  -> thường không cần singleton
```

### Decision matrix

| Tình huống | `Singleton` hợp không? | Vì sao |
|---|---|---|
| App-wide registry hoặc config access | Có thể hợp | một điểm điều phối dùng chung |
| Object giữ request-specific state | Không | shared state sẽ nguy hiểm |
| Chỉ muốn framework quản lý lifecycle | Thường không cần | Spring singleton bean đã đủ |
| Object rất nặng, cần reuse và thật sự chỉ một instance | Có thể hợp | tiết kiệm init cost |
| Có thể truyền dependency rõ qua constructor | Thường nên làm vậy | tránh hidden global access |

## Code example

```java
public final class AppConfigRegistry {
    private static final AppConfigRegistry INSTANCE = new AppConfigRegistry();

    private AppConfigRegistry() {
    }

    public static AppConfigRegistry getInstance() {
        return INSTANCE;
    }
}
```

Ví dụ này dùng eager initialization, cách đơn giản và an toàn hơn nhiều so với tự viết lazy initialization phức tạp khi chưa thật sự cần.

## When to use / when NOT to use

Use `Singleton` khi:

- object thật sự đại diện cho một resource hoặc coordinator duy nhất
- reuse một instance mang lại giá trị rõ ràng
- object không giữ mutable state theo user hoặc theo request

Do NOT use `Singleton` khi:

- object có state theo request hoặc theo session
- chỉ muốn truy cập tiện ở mọi nơi mà không muốn truyền dependency
- đang ở trong Spring và container scope mặc định đã giải quyết lifecycle cho bạn

Misconception cần sửa là: dùng nhiều nơi không đồng nghĩa với singleton. Nhiều khi thứ bạn thật sự cần chỉ là dependency injection bình thường.

## How this connects to real Java projects

Trong Spring, scope mặc định của bean là singleton trong `ApplicationContext`. Điều đó có nghĩa rất nhiều service đã là singleton theo container.

Trước khi tự viết pattern `Singleton`, cần phân biệt hai chuyện khác nhau:

- singleton do container quản lý
- singleton tự quản lý bằng `static` field

Loại đầu thường testable và linh hoạt hơn nhiều.

## Gotchas

- Mutable state trong singleton bean có thể bị nhiều request cùng sửa.
- Singleton khó test hơn vì state global dễ rò giữa test case.
- Lazy initialization tự viết rất dễ sai khi có concurrency.
- Pattern này hay bị lạm dụng để che hidden dependency.
- Trong môi trường có nhiều classloader hoặc nhiều context, khái niệm “chỉ một instance” còn phải nói rõ là trong phạm vi nào.

## Check yourself

- Vì sao “dùng nhiều nơi” chưa đủ để biện minh cho `Singleton`?
- Một Spring bean mặc định đã là singleton thì có cần tự viết singleton pattern nữa không?
- Khi nào shared mutable state biến singleton thành rủi ro?
- Nếu mục tiêu chỉ là quản lý lifecycle, giải pháp nào thường tốt hơn `static getInstance()`?
- “Một instance” là trong phạm vi JVM, classloader, hay application context, tại sao câu hỏi này quan trọng?

## Exercises

### Exercise 1: Should Use Singleton

Độ khó: Easy

Đề bài:
Cho `sharedResource`, `requestScopedState`, và `expensiveInitialization`. Hãy trả về `true` chỉ khi object đại diện cho một shared resource, không mang request-scoped state, và có lợi khi tái sử dụng expensive initialization.

Ví dụ 1:

Đầu vào:
```text
sharedResource = true, requestScopedState = false, expensiveInitialization = true
```

Đầu ra:
```text
true
```

Giải thích:
Scenario này hưởng lợi từ một shared instance duy nhất.

Ràng buộc:

- Tất cả input đều là boolean
- Trả về một boolean
- Không cần xét các type đặc thù của framework

### Exercise 2: Detect Singleton State Risk

Độ khó: Easy

Đề bài:
Cho `mutableState`, `concurrentAccess`, và `requestSpecificData`. Hãy trả về `true` khi singleton design là risky vì instance sẽ phải chia sẻ mutable state hoặc request-specific state giữa các caller đồng thời.

Ví dụ 1:

Đầu vào:
```text
mutableState = true, concurrentAccess = true, requestSpecificData = false
```

Đầu ra:
```text
true
```

Giải thích:
Shared mutable state under concurrency is already enough to make the design risky.

Ràng buộc:

- Tất cả input đều là boolean
- Trả về một boolean
- Ignore JVM tuning details

### Exercise 3: Choose Singleton Initialization

Độ khó: Medium

Đề bài:
Cho `startupCostHigh`, `instanceAlwaysNeeded`, và `multiThreaded`. Hãy trả về một trong các string chính xác sau: `"EAGER"`, `"LAZY_THREAD_SAFE"`, hoặc `"LAZY_SIMPLE"`. Dùng `EAGER` khi instance luôn cần thiết, `LAZY_THREAD_SAFE` khi instance không phải lúc nào cũng cần và code là multi-threaded, ngược lại dùng `LAZY_SIMPLE`.

Ví dụ 1:

Đầu vào:
```text
startupCostHigh = true, instanceAlwaysNeeded = false, multiThreaded = true
```

Đầu ra:
```text
"LAZY_THREAD_SAFE"
```

Giải thích:
Instance này không phải lúc nào cũng cần, nên lazy creation là hợp lý, nhưng concurrency yêu cầu một cách tiếp cận lazy có thread-safety.

Ràng buộc:

- Tất cả input đều là boolean
- Output must be one of the three exact strings
- `startupCostHigh` does not override the `instanceAlwaysNeeded` rule

## Links

- [[002-Builder]]
- [[../../../01_Core/Multithreading/003-synchronized]]
- [Spring Framework, Bean Scopes](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)
- [Effective Java, Consider enum types instead of readResolve](https://dev.to/kylec32/effective-java-tuesday-singletons-3adg)
- [Refactoring Guru, Singleton](https://refactoring.guru/design-patterns/singleton)
