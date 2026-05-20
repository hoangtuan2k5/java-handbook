# StampedLock

## What is it

`StampedLock` là lock API thiên về tối ưu read-heavy workload. Nó có ba mode chính:

- `writeLock()` cho exclusive write,
- `readLock()` cho shared read,
- `tryOptimisticRead()` cho optimistic read không giữ lock thật.

Điểm khác biệt lớn nhất là optimistic read chỉ trả về một `stamp`. Bạn tự đọc dữ liệu, rồi tự `validate(stamp)` để xem trong lúc đó có writer chen vào hay không.

```text
optimistic read
    -> lấy stamp
    -> đọc dữ liệu vào local variable
    -> validate(stamp)
         |
         +--> true  -> dùng dữ liệu vừa đọc
         +--> false -> fallback sang read lock thật
```

## How I used to misunderstand it

Mình từng nghĩ `StampedLock` là bản nâng cấp đơn giản của `ReadWriteLock`, cứ thay vào là nhanh hơn.

Sai ở hai điểm.

Thứ nhất, nó khó dùng đúng hơn nhiều. Thứ hai, lợi ích của optimistic read chỉ xuất hiện khi conflict thấp và read path đủ ngắn. Nếu validation fail liên tục, code sẽ vừa phức tạp hơn vừa không được lợi bao nhiêu.

Hiểu nhầm khác là quên rằng `StampedLock` **không reentrant**. Dùng nó với intuition của `ReentrantLock` là rất dễ tự kẹt.

## How it actually works

Với optimistic read, bạn đang nói: “Tôi thử đọc trước mà không giữ lock, rồi kiểm tra lại xem lúc đó có writer làm bẩn snapshot không.” Đây là đánh đổi giữa **ít contention hơn trên read path** và **độ khó reasoning cao hơn**.

### So sánh ngắn với hai lựa chọn gần nhất

| Câu hỏi | `ReadWriteLock` | `StampedLock` |
|---|---|---|
| Reentrant | Thường có | Không |
| Shared read lock | Có | Có |
| Optimistic read | Không | Có |
| Dễ dùng đúng | Dễ hơn | Khó hơn |
| Hợp cho team chưa quen concurrency | Hợp hơn | Ít hợp hơn |

### Khi optimistic read đáng giá

| Điều kiện | Vì sao quan trọng |
|---|---|
| Read path ngắn | Càng ngắn, cửa sổ bị writer chen vào càng nhỏ |
| Conflict thấp | Validation fail ít thì mới có lợi |
| Dữ liệu đọc vào local variable nhanh | Dễ validate rồi dùng ngay |
| Team hiểu rõ fallback path | Nếu không, bug sẽ rất khó thấy |

### Mental model chốt

- `StampedLock` không thay thế tư duy locking cơ bản.
- Nó chỉ thêm một đường đọc lạc quan hơn.
- Nếu bạn không đo workload hoặc không chắc team sẽ giữ đúng pattern validate, chọn công cụ đơn giản hơn thường tốt hơn.

## Code example

```java
import java.util.concurrent.locks.StampedLock;

class Point {
    private final StampedLock lock = new StampedLock();
    private int x;

    int readX() {
        long stamp = lock.tryOptimisticRead();
        int current = x;
        if (!lock.validate(stamp)) {
            stamp = lock.readLock();
            try {
                current = x;
            } finally {
                lock.unlockRead(stamp);
            }
        }
        return current;
    }
}
```

Điểm quan trọng không nằm ở cú pháp, mà ở flow `read -> validate -> fallback`. Thiếu một mắt xích là mental model sụp ngay.

## When to use / when NOT to use

Dùng `StampedLock` khi:

- workload đọc áp đảo,
- read path ngắn và nóng,
- bạn đã có lý do đo đạc cho thấy optimistic read đáng giá,
- team hiểu rõ non-reentrant behavior và fallback path.

Không dùng khi:

- requirement chỉ là read/write separation thông thường,
- codebase ưu tiên readability và maintainability,
- team dễ quên `validate(stamp)`,
- bạn cần reentrancy rõ ràng.

Trong nhiều case, `ReadWriteLock` hoặc immutable snapshot sẽ là lựa chọn ít rủi ro hơn.

## How this connects to real Java projects

Trong Spring app, `StampedLock` chỉ nên xuất hiện ở component thực sự nóng về read path, như in-memory index, snapshot metadata, hoặc cấu trúc tra cứu đặc thù.

Nó hiếm khi là lựa chọn đầu tiên cho service business bình thường. Nếu app chủ yếu là CRUD và I/O, complexity của `StampedLock` thường không trả lại giá trị tương xứng.

## Gotchas

- Quên `validate(stamp)` thì optimistic read gần như vô nghĩa.
- `StampedLock` không reentrant, đừng mang thói quen của `ReentrantLock` sang đây.
- Write lock giữ lâu sẽ làm optimistic read fail dày, lúc đó lợi ích lý thuyết tan rất nhanh.
- Nếu dữ liệu phải đọc qua nhiều bước phức tạp mới dùng được, optimistic read thường kém hợp hơn.

## Check yourself

- Vì sao optimistic read không phải là “read lock nhanh hơn” theo nghĩa đơn giản?
- `validate(stamp)` đang bảo vệ assumption nào?
- Khi validation fail liên tục, `StampedLock` còn đáng giá không?
- Vì sao non-reentrant behavior làm công cụ này rủi ro hơn cho team chưa quen?
- Khi nào `ReadWriteLock` là lựa chọn thực tế hơn dù có thể chậm hơn một chút?

## Exercises

### Exercise 1: Select StampedLock Mode

Độ khó: Easy

Đề bài:
Cho ba cờ `needsWrite`, `optimisticReadCandidate`, `validationRequired`. Trả về `"write-lock"` nếu cần ghi. Nếu không cần ghi và hai cờ còn lại đều `true`, trả về `"optimistic-read"`. Trong các trường hợp còn lại, trả về `"read-lock"`.

Ví dụ 1:

Đầu vào:
```text
needsWrite = false
optimisticReadCandidate = true
validationRequired = true
```

Đầu ra:
```text
"optimistic-read"
```

Giải thích:
Đây là trường hợp read path ngắn và sẵn sàng validate lại `stamp`.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về một trong ba nhãn đã nêu
- Không xét benchmark hoặc fairness

### Exercise 2: Count Invalid Optimistic Reads

Độ khó: Easy

Đề bài:
Cho mảng `validations`, trong đó mỗi phần tử là kết quả `validate(stamp)` của một optimistic read. Trả về số lần validation thất bại.

Ví dụ 1:

Đầu vào:
```text
validations = [true, false, false, true]
```

Đầu ra:
```text
2
```

Giải thích:
Có hai optimistic read phải retry hoặc fallback vì `stamp` không còn hợp lệ.

Ràng buộc:

- `1 <= validations.length <= 100000`
- Mỗi phần tử là boolean
- Không dùng real lock

### Exercise 3: Should Fallback to Read Lock

Độ khó: Medium

Đề bài:
Cho mảng `validations` và số nguyên `maxFailures`. Nếu có ít nhất `maxFailures` lần validation fail liên tiếp, trả về `true`, ngược lại trả về `false`.

Ví dụ 1:

Đầu vào:
```text
validations = [true, false, false, true]
maxFailures = 2
```

Đầu ra:
```text
true
```

Giải thích:
Có hai lần fail liên tiếp, đủ để xem optimistic read không còn đáng tin và nên fallback sang read lock.

Ràng buộc:

- `1 <= validations.length <= 100000`
- `1 <= maxFailures <= validations.length`
- Chỉ xét số lần fail liên tiếp

## Links

- [[002-read-write-lock]]
- [[001-reentrant-lock-vs-synchronized]]
- [[006-concurrent-hash-map-internals]]
- [JDK 21 `StampedLock`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/StampedLock.html)
- [JDK 21 `Lock`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/locks/Lock.html)
- [JLS §17, Threads and Locks](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html)
