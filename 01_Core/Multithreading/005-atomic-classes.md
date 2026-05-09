# Atomic Classes

## What is it

Atomic classes như `AtomicInteger`, `AtomicLong`, `AtomicBoolean`, `AtomicReference` cung cấp operation thread-safe cho một giá trị đơn lẻ mà không cần bọc mọi thứ trong `synchronized`.

Mental model: atomic class biến một số mẫu read-modify-write phổ biến thành một operation có guarantee rõ ở mức API.

## How I used to misunderstand it

Mình từng nghĩ atomic class là bản nâng cấp luôn tốt hơn lock.

Không đúng. Atomic class rất mạnh cho counter, compare-and-set, sequence number, hoặc swap một reference. Nhưng nó không tự động bảo vệ invariant liên quan nhiều field hoặc nhiều bước business logic.

Hiểu nhầm nữa là thấy code không có `synchronized` thì tưởng không có contention. Atomic vẫn có contention, chỉ là contention được xử lý bằng CAS retry thay vì monitor lock.

## How it actually works

Nhiều atomic classes hoạt động dựa trên compare-and-set, thường gọi tắt là CAS.

Ý tưởng là:

1. đọc giá trị hiện tại,
2. thử update nếu giá trị vẫn đúng như mình kỳ vọng,
3. nếu thread khác đã đổi trước đó, thử lại.

Vì vậy `incrementAndGet()` an toàn hơn `value++` khi nhiều thread cùng tăng counter.

### Bảng nhanh, dùng loại nào cho trường hợp nào

| Type | Trường hợp sử dụng điển hình | Vì sao hợp |
|---|---|---|
| `AtomicBoolean` | one-time flag, state flip nhỏ | swap trạng thái đơn |
| `AtomicInteger` | counter, retry count, sequence nhỏ | increment và add atomic |
| `AtomicLong` | counter lớn, metrics | giống `AtomicInteger` nhưng rộng hơn |
| `AtomicReference<T>` | publish hoặc swap một object reference | compare-and-set trên reference |

### Atomic vs volatile vs synchronized

| Nếu bạn cần... | `volatile` | Atomic class | `synchronized` |
|---|---|---|---|
| Chỉ thấy giá trị mới nhất | Tốt | Tốt | Tốt |
| Increment không mất update | Không | Tốt | Tốt |
| CAS, set nếu đúng expected | Không | Tốt | Làm được nhưng nặng hơn |
| Nhiều field phải nhất quán cùng nhau | Không | Thường không | Tốt hơn |

### CAS mindset

```text
read current
    -> if current == expected, update succeeds
    -> if not, another thread won first, retry or handle failure
```

## Code example

```java
import java.util.concurrent.atomic.AtomicInteger;

public class Main {
    private final AtomicInteger counter = new AtomicInteger();

    int nextValue() {
        // atomic increment, không mất update như value++
        return counter.incrementAndGet();
    }
}
```

## When to use / when NOT to use

Dùng atomic class cho counter, flag, single-reference update, compare-and-set, hoặc state machine rất nhỏ.

Nó hợp khi correctness xoay quanh một giá trị đơn lẻ và operation đã có sẵn trên API.

Không dùng atomic class như một cây búa cho mọi bài toán concurrency. Nếu bạn phải giữ quan hệ nhất quán giữa nhiều field, hoặc code CAS loop bắt đầu khó đọc, lock thường rõ hơn.

## How this connects to Spring

Trong Spring singleton bean, atomic class đôi khi hợp cho metrics nhẹ, sequence number, hoặc in-memory marker đơn giản.

Nhưng business state quan trọng thường không nên được bảo vệ chỉ bằng vài atomic field rời rạc trong app instance. Nếu dữ liệu phải nhất quán qua nhiều bước hoặc nhiều node, bài toán đã vượt xa một `AtomicInteger`.

## Gotchas

- Atomic cho một field không làm cả object thread-safe.
- `get()` rồi `set()` riêng biệt không giống một operation atomic có điều kiện.
- CAS loop dưới contention cao có thể retry nhiều lần.
- Code quá khéo với atomic đôi khi khó review hơn `synchronized` ngắn gọn.

## Check yourself

- Atomic class mạnh hơn `volatile` ở điểm nào?
- Vì sao atomic class chưa chắc thay thế được `synchronized`?
- Khi nào `AtomicReference` hợp hơn `AtomicInteger`?
- Tại sao `get()` rồi `set()` riêng biệt vẫn có thể race?
- Nếu một invariant liên quan hai field phải đổi cùng lúc, atomic một field có đủ không?

## Exercises

### Bài 1: Choose Atomic For Counter
Độ khó: Dễ

Đề bài:
Cho một operation label, trả về `true` nếu atomic class là lựa chọn phù hợp. Chỉ `"counter-increment"` và `"single-flag-swap"` là phù hợp.

Ví dụ 1:
Đầu vào:
```text
operation = "counter-increment"
```

Đầu ra:
```text
true
```

Giải thích:
Atomic integer là lựa chọn phù hợp cho counter đơn giản.

Ràng buộc:
- `operation` là non-null
- Việc matching có phân biệt chữ hoa chữ thường
- Trả về một boolean

### Bài 2: Apply Atomic Increment Result
Độ khó: Dễ

Đề bài:
Cho giá trị current và số lần increment, trả về giá trị cuối cùng sau các lần increment theo kiểu atomic.

Ví dụ 1:
Đầu vào:
```text
current = 5, increments = 3
```

Đầu ra:
```text
8
```

Giải thích:
Ba lần increment làm giá trị tăng thêm ba.

Ràng buộc:
- `current >= 0`
- `increments >= 0`
- Kết quả nằm trong phạm vi của `int`

### Bài 3: Simulate Compare And Set
Độ khó: Trung bình

Đề bài:
Cho giá trị current, expected, và update, trả về giá trị update khi current bằng expected; ngược lại trả về current.

Ví dụ 1:
Đầu vào:
```text
current = 10, expected = 10, update = 20
```

Đầu ra:
```text
20
```

Giải thích:
Update chỉ thành công khi current khớp với expected.

Ràng buộc:
- Các value là integer
- Chỉ dùng phép equality
- Trả về giá trị kết quả

## Links

- [[002-race-condition]]
- [[003-synchronized]]
- [[004-volatile]]
- Package summary, `java.util.concurrent.atomic`: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/package-summary.html
- `AtomicInteger` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html
- `AtomicReference` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/AtomicReference.html

