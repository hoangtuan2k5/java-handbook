# ReentrantLock vs synchronized

## What is it

`synchronized` và `ReentrantLock` đều giải bài toán **mutual exclusion**, tức là không cho nhiều thread cùng sửa shared mutable state tại một thời điểm.

Điểm quan trọng là chúng không khác nhau ở mức “cái nào thread-safe hơn”. Nếu dùng đúng, cả hai đều cho bạn **locking** và **memory visibility** cần thiết. Khác biệt chính nằm ở **API**, **ergonomics**, và **trade-off giữa simplicity với control**.

Mental model ngắn:

- `synchronized` là monitor lock built sẵn trong ngôn ngữ, ngắn gọn, scope rõ, khó quên release.
- `ReentrantLock` là explicit lock trong `java.util.concurrent.locks`, dài tay hơn nhưng cho bạn nhiều nút điều khiển hơn.

```text
Need mutual exclusion
        |
        +--> Simple critical section, scope rõ, ít policy --> synchronized
        |
        +--> Cần timeout / interruptible wait / nhiều Condition --> ReentrantLock
```

## How I used to misunderstand it

Hiểu nhầm phổ biến là `ReentrantLock` luôn là bản “pro hơn”, còn `synchronized` chỉ dành cho ví dụ cơ bản.

Thực ra phần lớn critical section ngắn chỉ cần `synchronized`. Nó ít ceremony, JVM quản lý monitor enter và exit giúp bạn, và reader khác cũng nhìn code dễ hơn.

Hiểu nhầm thứ hai là so sánh chúng theo kiểu “cái nào nhanh hơn”. Với người học, câu hỏi đúng hơn thường là: **mình đang thiếu capability gì**. Nếu không cần `tryLock`, timeout, `lockInterruptibly()`, hay nhiều `Condition`, thì đổi sang `ReentrantLock` thường chỉ làm code dài hơn.

## How it actually works

Khi thread vào block `synchronized`, JVM cố lấy monitor của object mục tiêu. Nếu monitor đang bị thread khác giữ, thread này phải chờ. Khi ra khỏi block, monitor được nhả tự động, kể cả khi có exception.

`ReentrantLock` cũng là **reentrant**, nghĩa là cùng một thread có thể lock cùng một lock nhiều lần. Nhưng khác ở chỗ lifecycle của lock là trách nhiệm của bạn. Gần như luôn phải viết theo pattern `lock(); try { ... } finally { unlock(); }`.

### Hai công cụ này giống nhau ở phần nào

| Câu hỏi | `synchronized` | `ReentrantLock` |
|---|---|---|
| Bảo vệ critical section | Có | Có |
| Cung cấp happens-before khi unlock rồi lock lại | Có | Có |
| Reentrant | Có | Có |
| Tự release khi ra khỏi scope | Có | Không |

### Khác nhau ở phần nào

| Nhu cầu | `synchronized` | `ReentrantLock` |
|---|---|---|
| Code ngắn, ít chỗ sai | Rất hợp | Kém hơn |
| `tryLock()` | Không có | Có |
| Timeout khi chờ lock | Không có | Có |
| `lockInterruptibly()` | Không có | Có |
| Nhiều wait-set riêng | Chỉ có một monitor wait-set | Có nhiều `Condition` |
| Fairness option | Không cấu hình trực tiếp | Có |

### Visibility, locking, throughput, đừng trộn ba ý này

- **Visibility** là thread khác có thấy state mới không.
- **Locking** là chỉ một thread được vào critical section tại một thời điểm.
- **Throughput** là hệ thống xử lý được bao nhiêu việc trong một đơn vị thời gian.

Cả `synchronized` và `ReentrantLock` đều giải quyết visibility và locking. Không cái nào tự đảm bảo throughput tốt hơn trong mọi workload. Bật fairness cho `ReentrantLock` còn có thể làm throughput giảm vì giảm cơ hội barging.

## Code example

```java
import java.util.concurrent.locks.ReentrantLock;

class Counter {
    private final ReentrantLock lock = new ReentrantLock();
    private int value;

    void increment() {
        lock.lock();
        try {
            value++;
        } finally {
            lock.unlock();
        }
    }
}
```

Ví dụ này cho thấy sức mạnh và cái giá của `ReentrantLock`. Bạn kiểm soát lock rõ ràng hơn, nhưng cũng phải tự chịu trách nhiệm release đúng đường.

## When to use / when NOT to use

Dùng `synchronized` khi:

- critical section ngắn,
- rule đơn giản,
- không cần timeout, interruptible wait, hoặc nhiều `Condition`,
- bạn muốn code dễ đọc và khó dùng sai hơn.

Dùng `ReentrantLock` khi:

- cần `tryLock()` để tránh chờ vô hạn,
- cần `lockInterruptibly()` để hủy thread đang chờ lock,
- cần nhiều `Condition` tách biệt cho các nhóm waiter,
- cần fairness có chủ đích và chấp nhận trade-off throughput.

Không nên chọn `ReentrantLock` chỉ vì nghĩ nó “nhanh hơn”. Cũng không nên nhét `synchronized` vào chỗ mà requirement thật sự là timeout hoặc cancel khi chờ lock.

## How this connects to real Java projects

Trong Spring, chỗ cần hỏi đầu tiên không phải “dùng lock gì”, mà là “vì sao bean này lại có shared mutable state”. Với singleton bean, state mutable mới là nguồn rủi ro chính.

Nếu state đó là đơn giản và bắt buộc phải giữ trong memory, `synchronized` thường là điểm bắt đầu tốt. `ReentrantLock` chỉ nên vào cuộc khi bạn có nhu cầu cụ thể như timed lock attempt, coordination phức tạp hơn, hoặc chẩn đoán contention rõ ràng.

## Gotchas

- Quên `unlock()` trong `finally` là bug kinh điển với `ReentrantLock`.
- Fair lock nghe hấp dẫn, nhưng fairness cao hơn thường đổi lấy throughput thấp hơn.
- Reentrant không có nghĩa nested locking trở nên dễ reason. Nó chỉ tránh self-deadlock cho cùng một thread trên cùng một lock.
- Đừng dùng lock để che thiết kế state quá rối. Nhiều lúc immutable snapshot hoặc tách state mới là fix đúng.

## Handbook rule

- Default vẫn là `synchronized` cho critical section ngắn, không cần feature đặc biệt.
- Chọn `ReentrantLock` chỉ khi cần `tryLock`, `lockInterruptibly`, hoặc nhiều `Condition`.
- Mọi `lock()` phải có `unlock()` trong `finally`; thiếu là leak chắc chắn.
- Fairness đổi throughput; bật chỉ khi đo được starvation thật.
- Reentrant tránh self-deadlock cùng thread, không làm nested locking thành an toàn.

## Check yourself

- `synchronized` và `ReentrantLock` khác nhau chủ yếu ở visibility hay ở API control?
- Khi nào `lockInterruptibly()` quan trọng hơn code ngắn gọn?
- Vì sao fairness có thể làm throughput giảm?
- `Semaphore(1)` và `ReentrantLock` có cùng semantic ownership không?
- Nếu critical section rất ngắn và không có requirement đặc biệt, vì sao `synchronized` thường là lựa chọn tốt hơn?

## Exercises

### Exercise 1: Choose Lock Primitive

Độ khó: Easy

Đề bài:
Cho ba cờ `needsTimedAttempt`, `needsInterruptibleWait`, `needsMultipleConditions`. Trả về `"reentrant-lock"` nếu có ít nhất một nhu cầu cần API nâng cao, ngược lại trả về `"synchronized"`.

Ví dụ 1:

Đầu vào:
```text
needsTimedAttempt = false
needsInterruptibleWait = true
needsMultipleConditions = false
```

Đầu ra:
```text
"reentrant-lock"
```

Giải thích:
`lockInterruptibly()` là khả năng mà `synchronized` không có.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"reentrant-lock"` hoặc `"synchronized"`
- Không xét benchmark hoặc micro-optimization

### Exercise 2: Max Reentrant Depth

Độ khó: Medium

Đề bài:
Cho mảng `events` chỉ gồm `"enter"` và `"exit"`, mô phỏng một thread re-enter cùng một lock. Trả về độ sâu lớn nhất từng đạt được.

Ví dụ 1:

Đầu vào:
```text
events = ["enter", "enter", "exit", "enter", "exit", "exit"]
```

Đầu ra:
```text
2
```

Giải thích:
Độ sâu tăng lên `2` ở lần `enter` thứ hai, rồi không vượt qua mức đó nữa.

Ràng buộc:

- `1 <= events.length <= 100000`
- Mỗi phần tử chỉ là `"enter"` hoặc `"exit"`
- Không có thời điểm nào depth hợp lệ được nhỏ hơn `0`

### Exercise 3: Detect Missing Unlock Path

Độ khó: Medium

Đề bài:
Cho mảng `actions` chỉ gồm `"lock"` và `"unlock"`. Trả về index đầu tiên làm trạng thái lock không hợp lệ. Nếu mọi action hợp lệ nhưng cuối cùng vẫn còn lock chưa được nhả, trả về `actions.length`. Nếu hoàn toàn cân bằng, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
actions = ["lock", "unlock", "unlock"]
```

Đầu ra:
```text
2
```

Giải thích:
Action ở index `2` cố `unlock` khi không còn lock nào đang được giữ.

Ràng buộc:

- `1 <= actions.length <= 100000`
- Mỗi phần tử chỉ là `"lock"` hoặc `"unlock"`
- Không dùng real thread hoặc real lock

## Links

- [[../../01_Core/Multithreading/002-race-condition]]
- [[../../01_Core/Multithreading/003-synchronized]]
- [[../../01_Core/Multithreading/004-volatile]]
- [[../../01_Core/Multithreading/006-deadlock]]
- [[002-read-write-lock]]
- [[005-semaphore]]
- [JDK 21 `ReentrantLock`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/ReentrantLock.html)
- [JDK 21 `Condition`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/Condition.html)
- [JLS §17, Threads and Locks](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html)
