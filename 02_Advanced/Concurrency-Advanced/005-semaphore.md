# Semaphore

## What is it

`Semaphore` là primitive dùng để **giới hạn số lượng concurrent access** vào một resource hoặc một nhóm slot.

Mental model dễ nhớ là một rổ permit:

- vào vùng giới hạn thì `acquire()` một permit,
- xong việc thì `release()` trả permit lại,
- hết permit thì người đến sau phải chờ.

Điểm cốt lõi là `Semaphore` giải bài toán **bounded parallelism**. Nó không sinh ra chủ yếu để làm ownership-based locking.

```text
permits = 2

Thread A acquire -> còn 1
Thread B acquire -> còn 0
Thread C acquire -> phải chờ
Thread A release -> còn 1 -> C mới có cơ hội vào
```

## How I used to misunderstand it

Mình từng nghĩ `Semaphore(1)` chỉ là cách viết khác của lock.

Đúng là nó có thể tạo mutual exclusion, nhưng semantic của nó khác. Lock kiểu `ReentrantLock` gắn chặt với ý tưởng một thread giữ lock rồi tự thread đó unlock theo ownership model. `Semaphore` thì nói về permit count nhiều hơn là ownership intent.

Hiểu nhầm nữa là thấy semaphore chặn được nhiều request rồi tưởng thế là đã có backpressure strategy đầy đủ. Không hẳn. Nó chỉ là một cổng giới hạn concurrency, không tự cho bạn queue policy, retry policy, timeout policy, hay overload handling hoàn chỉnh.

## How it actually works

`acquire()` cố lấy permit. Nếu còn permit, count giảm. Nếu hết permit, thread chờ cho tới khi có permit được trả lại hoặc thread bị interrupt, tùy API bạn dùng.

`release()` tăng số permit lại. Chính vì vậy, bug quên `release()` nguy hiểm ở chỗ hệ thống không vỡ ngay, mà tự bóp nghẹt dần theo thời gian.

### Semaphore khác gì so với lock và latch

| Bạn cần gì | Công cụ phù hợp hơn |
|---|---|
| Chỉ một thread được sửa state | Lock |
| Tối đa N thread vào cùng lúc | `Semaphore` |
| Chờ N signal hoàn tất rồi đi tiếp | `CountDownLatch` |
| N worker gặp nhau theo phase | `CyclicBarrier` |

### Lock choice nhỏ

| Tình huống | Chọn gì trước |
|---|---|
| Protect object invariant | `synchronized` hoặc `ReentrantLock` |
| Limit concurrent calls to downstream | `Semaphore` |
| Throttle tạm thời bằng số slot | `Semaphore` |
| Cần queue, timeout, retry, fallback phức tạp | Xem thêm resilience pattern |

### Fairness trade-off

| Mode | Ưu điểm | Đổi lại |
|---|---|---|
| Non-fair | Throughput thường tốt hơn | Có thể ít công bằng hơn |
| Fair | Giảm starvation | Overhead cao hơn, throughput thường giảm |

## Code example

```java
import java.util.concurrent.Semaphore;

Semaphore semaphore = new Semaphore(2);

semaphore.acquire();
try {
    // at most two threads enter here at once
} finally {
    semaphore.release();
}
```

Ví dụ này cho thấy semaphore diễn tả rất rõ ý “tối đa 2 việc được đi qua cùng lúc”.

## When to use / when NOT to use

Dùng `Semaphore` khi:

- cần giới hạn số request đụng tới downstream,
- cần giới hạn số task nền chạy song song,
- có một pool slot hữu hạn và muốn gate bằng permit count.

Không dùng khi:

- mục tiêu thật sự là bảo vệ invariant của shared object,
- bạn cần ownership semantics rõ ràng,
- bạn đang cần full backpressure hoặc overload strategy,
- queue hoặc executor policy mới là abstraction phù hợp hơn.

## How this connects to Spring

Trong Spring app, `Semaphore` rất hợp cho bulkhead nhỏ trong service layer, ví dụ giới hạn số call song song sang hệ thống chậm hoặc giới hạn số job nền chạm vào tài nguyên hiếm.

Nhưng nếu policy overload đã phức tạp, nên cân nhắc công cụ cao hơn như queue, rate limiter, circuit breaker, hoặc một resilience library, thay vì dồn mọi chuyện vào semaphore thủ công.

## Gotchas

- Quên `release()` làm permit rò rỉ, hệ thống sẽ nghẽn dần.
- `Semaphore(1)` có thể giống lock về hiệu ứng loại trừ, nhưng semantic không hoàn toàn giống ownership-based lock.
- Bật fairness không miễn phí. Hãy xem mình đang tối ưu starvation hay throughput.
- Semaphore không tự bảo vệ object graph khỏi race condition nếu code bên trong vẫn mutate state phức tạp.
- `acquire()` là blocking operation; nếu bạn muốn thất bại ngay khi hết permit thì mental model gần hơn là `tryAcquire()`.

## Check yourself

- `Semaphore` đang giải bài toán mutual exclusion hay bounded parallelism?
- Vì sao `Semaphore(1)` chưa chắc là lựa chọn diễn đạt tốt nhất cho object lock?
- Nếu quên `release()`, lỗi hệ thống thường biểu hiện dần như thế nào?
- Khi nào `CountDownLatch` phù hợp hơn `Semaphore`?
- Fair semaphore cải thiện điều gì, và thường đánh đổi điều gì?

## Exercises

### Exercise 1: Choose Semaphore Use Case

Độ khó: Easy

Đề bài:
Cho ba cờ `limitParallelism`, `oneShotCoordination`, `exclusiveOwnerOnly`. Trả về `"semaphore"` nếu cần giới hạn số việc chạy cùng lúc. Nếu mục tiêu là one-shot coordination, trả về `"count-down-latch"`. Nếu mục tiêu là ownership-based mutual exclusion, trả về `"reentrant-lock"`.

Ví dụ 1:

Đầu vào:
```text
limitParallelism = true
oneShotCoordination = false
exclusiveOwnerOnly = false
```

Đầu ra:
```text
"semaphore"
```

Giải thích:
Giới hạn concurrency là use case tự nhiên nhất của `Semaphore`.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về một trong ba nhãn đã nêu
- Không xét fairness hoặc timeout

### Exercise 2: Remaining Permits After Actions

Độ khó: Medium

Đề bài:
Cho `permits` ban đầu và mảng `actions` chỉ gồm `"tryAcquire"` hoặc `"release"`. Mỗi `tryAcquire` chỉ thành công nếu còn permit, nếu không thì thất bại ngay và không đổi state. Trả về số permit còn lại sau khi xử lý hết actions.

Ví dụ 1:

Đầu vào:
```text
permits = 2
actions = ["tryAcquire", "tryAcquire", "tryAcquire", "release"]
```

Đầu ra:
```text
1
```

Giải thích:
Lần `tryAcquire` thứ ba thất bại vì không còn permit, sau đó một lần `release` đưa số permit còn lại lên `1`.

Ràng buộc:

- `0 <= permits <= 1000000`
- `1 <= actions.length <= 100000`
- Mỗi phần tử chỉ là `"tryAcquire"` hoặc `"release"`

### Exercise 3: Count Rejected Acquires

Độ khó: Medium

Đề bài:
Cho `permits` ban đầu và mảng `actions` chỉ gồm `"tryAcquire"` hoặc `"release"`. Đếm xem có bao nhiêu lần `tryAcquire` thất bại ngay vì tại thời điểm đó không còn permit.

Ví dụ 1:

Đầu vào:
```text
permits = 1
actions = ["tryAcquire", "tryAcquire", "release", "tryAcquire", "tryAcquire"]
```

Đầu ra:
```text
2
```

Giải thích:
Lần `tryAcquire` thứ hai và lần cuối cùng đều đến khi không còn permit trống.

Ràng buộc:

- `0 <= permits <= 1000000`
- `1 <= actions.length <= 100000`
- Không dùng real semaphore

## Links

- [[001-reentrant-lock-vs-synchronized]]
- [[004-countdown-latch-vs-cyclic-barrier]]
- [[007-fork-join-pool]]
- [JDK 21 `Semaphore`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Semaphore.html)
- [Java Concurrency Tutorial, Synchronizers](https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html)
