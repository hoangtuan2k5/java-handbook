# ReadWriteLock

## What is it

`ReadWriteLock` tách quyền truy cập thành hai lane, `read lock` và `write lock`.

- Nhiều reader có thể vào cùng lúc nếu không có writer.
- Writer cần **exclusive access**, nên nó chặn reader khác và cũng phải chờ reader hiện tại rời đi.

Mental model đúng là **trade throughput của read-heavy workload lấy complexity cao hơn**. Nó không phải là “phiên bản tốt hơn của lock thường”.

```text
No writer active
    -> reader A vào
    -> reader B vào
    -> reader C vào

Writer wants in
    -> phải đợi toàn bộ reader hiện tại rời đi
    -> sau đó mới vào một mình
```

## How I used to misunderstand it

Hiểu nhầm phổ biến là hễ có cả đọc lẫn ghi thì nên dùng `ReadWriteLock`.

Sai ở chỗ hiệu quả của nó phụ thuộc nặng vào access pattern. Nếu write xảy ra thường xuyên, hoặc read path rất ngắn, overhead của việc quản lý hai loại lock có thể lớn hơn lợi ích cho phép nhiều reader song song.

Hiểu nhầm khác là nghĩ đọc dưới `read lock` thì đương nhiên an toàn. Không đúng. Nó chỉ an toàn nếu code bên trong **thực sự read-only** với shared state được bảo vệ.

## How it actually works

`ReadWriteLock` là interface. Implementation phổ biến là `ReentrantReadWriteLock`.

Khi không có writer, nhiều thread có thể cùng giữ `readLock()`. Khi writer muốn vào, nó phải đợi reader hiện tại rời đi. Tùy policy, implementation có thể cho reader mới chen vào hay ưu tiên writer hơn để tránh starvation.

### Bản chất trade-off

| Câu hỏi | Exclusive lock | `ReadWriteLock` |
|---|---|---|
| Reader có chặn nhau không | Có | Không, nếu không có writer |
| Writer có chặn reader không | Có | Có |
| Dễ dùng đúng không | Dễ hơn | Khó hơn |
| Hợp với read-heavy workload không | Tạm ổn | Tốt hơn |
| Hợp với write-heavy workload không | Thường hợp hơn | Thường không lời |

### Chỗ người học hay ngã

| Hiểu nhầm | Sự thật |
|---|---|
| Có read và write thì luôn nên dùng `ReadWriteLock` | Chỉ đáng cân nhắc khi read áp đảo và read path đủ đáng kể |
| `readLock()` nghĩa là code bên trong “an toàn” | Chỉ an toàn nếu bạn không mutate shared state |
| Có thể upgrade từ read sang write dễ dàng | Đây là vùng dễ tự kẹt hoặc tạo flow khó đoán |

### Lock choice ngắn gọn

| Nhu cầu | Chọn gì trước |
|---|---|
| Critical section nhỏ, đơn giản | `synchronized` hoặc `ReentrantLock` |
| Đọc áp đảo, read path đủ nặng | `ReadWriteLock` |
| Read-heavy rất nóng và sẵn sàng tối ưu khó hơn | Cân nhắc `StampedLock` |

## Code example

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class ProductCache {
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private String snapshot = "initial";

    String read() {
        lock.readLock().lock();
        try {
            return snapshot;
        } finally {
            lock.readLock().unlock();
        }
    }
}
```

Điểm đáng nhớ ở ví dụ này là reader có thể chạy song song. Nhưng giá trị đó chỉ có ý nghĩa nếu đọc thật sự là thao tác read-only và đủ thường xuyên.

## When to use / when NOT to use

Dùng `ReadWriteLock` khi:

- dữ liệu được đọc rất nhiều,
- ghi tương đối ít,
- phần đọc không quá trivial,
- team hiểu rõ shared state nào đang được bảo vệ.

Không dùng khi:

- write diễn ra dày,
- critical section rất ngắn,
- code đọc vẫn âm thầm mutate object graph,
- flow cần upgrade từ read sang write liên tục.

Trong các case đó, lock thường hoặc immutable snapshot thường dễ sống hơn.

## How this connects to real Java projects

Trong Spring app, `ReadWriteLock` có thể hợp cho local cache, config snapshot, hoặc lookup table in-memory được refresh theo đợt.

Nó ít hợp cho business service thông thường. Nếu bean sống kiểu singleton và state thay đổi liên tục theo request, dùng `ReadWriteLock` dễ làm code khó reason mà không xử lý tận gốc vấn đề thiết kế state.

## Gotchas

- Reader không chặn nhau, nhưng giữ `read lock` quá lâu vẫn làm writer chờ rất khó chịu.
- Dưới `read lock`, mutate shared state là phá luật chơi cốt lõi.
- Upgrade read sang write là vùng nhiều bug. Mẫu an toàn hơn thường là nhả read lock, lấy write lock, rồi kiểm tra lại điều kiện.
- Fairness có thể giúp giảm starvation, nhưng không miễn phí về throughput.

## Check yourself

- Vì sao `ReadWriteLock` không phải lựa chọn mặc định mỗi khi có thao tác đọc và ghi?
- Nếu code dưới `read lock` vẫn mutate object graph, assumption nào đã bị phá?
- Khi writer chờ lâu trong workload đọc dày, vấn đề là mutual exclusion hay starvation?
- So với `StampedLock`, `ReadWriteLock` đổi lấy điều gì để dễ dùng hơn?
- Nếu read path rất ngắn, vì sao lock thường đôi khi lại tốt hơn?

## Exercises

### Exercise 1: Choose ReadWrite Strategy

Độ khó: Easy

Đề bài:
Cho `readOperations` và `writeOperations`. Trả về `"read-write-lock"` nếu số lần đọc lớn hơn ít nhất `4` lần số lần ghi, ngược lại trả về `"exclusive-lock"`.

Ví dụ 1:

Đầu vào:
```text
readOperations = 100
writeOperations = 10
```

Đầu ra:
```text
"read-write-lock"
```

Giải thích:
Tỉ lệ đọc trên ghi là `10`, đủ cao để mô hình nhiều reader song song có thể đáng cân nhắc.

Ràng buộc:

- `0 <= readOperations, writeOperations <= 1000000`
- Nếu `writeOperations = 0`, xem như workload đọc áp đảo
- Chỉ trả về `"read-write-lock"` hoặc `"exclusive-lock"`

### Exercise 2: Max Concurrent Readers

Độ khó: Medium

Đề bài:
Cho hai mảng `startTimes` và `endTimes` có cùng độ dài, mỗi cặp mô tả một khoảng thời gian đọc `[start, end)`. Trả về số reader tối đa hoạt động đồng thời.

Ví dụ 1:

Đầu vào:
```text
startTimes = [1, 2, 4]
endTimes = [5, 6, 7]
```

Đầu ra:
```text
3
```

Giải thích:
Tại thời điểm `4`, cả ba reader đều đang nằm trong khoảng hoạt động.

Ràng buộc:

- `1 <= startTimes.length == endTimes.length <= 100000`
- `0 <= startTimes[i] < endTimes[i] <= 1000000`
- Không cần mô phỏng real lock, chỉ tính overlap của các read window

### Exercise 3: Detect Read-Write Upgrade Risk

Độ khó: Medium

Đề bài:
Cho mảng `steps` gồm `"read-lock"`, `"read-unlock"`, `"write-lock"`, `"write-unlock"`. Trả về index đầu tiên mà code cố lấy `write-lock` trong khi vẫn còn giữ ít nhất một `read-lock`. Nếu không có rủi ro này, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
steps = ["read-lock", "write-lock", "read-unlock", "write-unlock"]
```

Đầu ra:
```text
1
```

Giải thích:
Ở index `1`, flow đang cố upgrade từ read sang write khi read lock vẫn còn được giữ.

Ràng buộc:

- `1 <= steps.length <= 100000`
- Mỗi phần tử thuộc đúng tập giá trị đã cho
- Không dùng thread thật hoặc lock thật

## Links

- [[001-reentrant-lock-vs-synchronized]]
- [[003-stamped-lock]]
- [[../../01_Core/Multithreading/003-synchronized]]
- [[../../01_Core/Multithreading/004-volatile]]
- [JDK 21 `ReadWriteLock`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/ReadWriteLock.html)
- [JDK 21 `ReentrantReadWriteLock`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/ReentrantReadWriteLock.html)
- [JLS §17, Threads and Locks](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html)
