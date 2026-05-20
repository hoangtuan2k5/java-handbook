# Race Condition

## What is it

Race condition là bug xảy ra khi kết quả cuối cùng phụ thuộc vào timing hoặc interleaving giữa nhiều thread.

Điểm nguy hiểm là từng dòng code riêng lẻ có thể nhìn hoàn toàn hợp lý, nhưng khi nhiều thread cùng đụng vào shared mutable state, kết quả lại sai.

Mental model: vấn đề không nằm ở chỗ “có nhiều thread”, mà nằm ở chỗ “nhiều thread cùng đụng vào state chung mà không có coordination đủ tốt”.

## How I used to misunderstand it

Mình từng nghĩ race condition chỉ là chuyện hai thread chạy cùng lúc.

Thật ra concurrent execution chưa chắc là bug. Nếu hai thread chỉ đọc immutable data, hoặc mỗi thread có state riêng, không có race đáng nói.

Race thường xuất hiện khi code làm các compound operation như:

- read, rồi write,
- check, rồi act,
- read, modify, write.

## How it actually works

Ví dụ `count++` không phải một bước ma thuật. Nó là chuỗi nhỏ hơn:

1. đọc `count`,
2. cộng thêm 1,
3. ghi lại kết quả.

Nếu hai thread cùng đọc cùng một giá trị cũ trước khi ai đó ghi lại, một update có thể bị mất.

### Lost update, nhìn bằng timeline

```text
Initial count = 0

Thread A: read 0 -------- add 1 -------- write 1
Thread B:       read 0 -------- add 1 -------- write 1

Expected final count = 2
Actual final count   = 1
```

### Bảng nhanh, race hay gặp ở đâu

| Bug shape | Ví dụ | `volatile` có đủ không | Hướng xử lý thường hợp lý |
|---|---|---|---|
| Read, modify, write | `count++` | Không | `synchronized`, atomic class, lock |
| Check, then act | `if (x == null) x = ...` | Không | lock, atomic method như `computeIfAbsent()` |
| Shared list mutation | `list.add(...)` từ nhiều thread | Thường không | concurrent collection, lock, tránh shared state |
| Simple visibility flag | `running = false` | Có thể đủ | `volatile` |

### Điều quan trọng nhất

Race condition là bài toán về shared-state hazard.

Khi đã có state chung, bạn thường phải chọn một trong vài hướng rõ ràng:

- lock,
- atomic class,
- concurrent collection,
- immutable data,
- message passing,
- hoặc bỏ shared state luôn.

## Code example

```java
class UnsafeCounter {
    private int count;

    void increment() {
        // read-modify-write, nên có thể mất update khi nhiều thread cùng gọi
        count++;
    }

    int getCount() {
        return count;
    }
}
```

## When to use / when NOT to use

Ngay khi thấy shared mutable state, hãy tự hỏi thread-safety strategy là gì.

Nếu chưa trả lời được câu hỏi đó, khả năng cao code đang sống nhờ may mắn về timing.

Không sửa race bằng `Thread.sleep(...)`. Sleep chỉ làm timing thay đổi, không tạo correctness guarantee.

Đừng vì test hiện tại xanh mà kết luận không có race. Race thường flaky và chỉ lộ ra dưới load, trên máy khác, hoặc sau một lần refactor nhỏ.

## How this connects to real Java projects

Spring singleton bean là shared object trên heap, được nhiều request thread gọi cùng lúc.

Nếu bean giữ mutable field như counter, current user, temporary buffer, `Map`, hoặc `List`, bạn đã có shared mutable state. Khi đó race condition không còn là lý thuyết nữa, nó là bug production chờ được lộ ra.

Vì vậy stateless service thường là chiến thắng lớn nhất. Không có state chung, bạn bỏ được cả một lớp bug.

## Gotchas

- Race condition thường khó reproduce vì nó phụ thuộc timing.
- `volatile` không sửa được read-modify-write race.
- Một object thread-safe không tự động làm cả business flow thread-safe nếu flow đó gồm nhiều bước rời nhau.
- “Chạy ổn trên local” không nói được gì nhiều về race.

## Check yourself

- Vì sao nhiều thread cùng chạy chưa chắc đã là race condition?
- Trong `count++`, race nằm ở bước nào?
- Khi nào `volatile` đủ, khi nào không đủ?
- Vì sao stateless service giúp tránh race đơn giản hơn là vá race sau này?
- Nếu có shared `HashMap` trong singleton bean, bạn sẽ nghi ngờ loại bug nào đầu tiên?

## Exercises

### Bài 1: Detect Shared Mutable State
Độ khó: Dễ

Đề bài:
Cho hai boolean `shared` và `mutable`, trả về `true` khi tồn tại race risk.

Ví dụ 1:
Đầu vào:
```text
shared = true, mutable = true
```

Đầu ra:
```text
true
```

Giải thích:
Shared mutable state có thể race nếu không có coordination.

Ràng buộc:
- Input là boolean
- Chỉ trả về `true` khi cả hai đều là `true`
- Không chọn cách fix

### Bài 2: Count Lost Updates
Độ khó: Trung bình

Đề bài:
Cho expected count và actual count, trả về số update đã bị mất.

Ví dụ 1:
Đầu vào:
```text
expected = 100, actual = 93
```

Đầu ra:
```text
7
```

Giải thích:
Bảy lần increment đã không xuất hiện trong kết quả cuối cùng.

Ràng buộc:
- `expected >= actual`
- `actual >= 0`
- Kết quả nằm trong phạm vi của `int`

### Bài 3: Classify Operation Atomicity
Độ khó: Trung bình

Đề bài:
Cho một operation label, trả về `"compound"` cho `"increment"` hoặc `"check-then-act"`; ngược lại trả về `"single"`.

Ví dụ 1:
Đầu vào:
```text
operation = "increment"
```

Đầu ra:
```text
"compound"
```

Giải thích:
Increment là một thao tác read-modify-write.

Ràng buộc:
- `operation` là non-null
- Việc matching có phân biệt chữ hoa chữ thường
- Chỉ trả về `"compound"` hoặc `"single"`

## Links

- [[003-synchronized]]
- [[004-volatile]]
- [[005-atomic-classes]]
- [[../Collections/010-concurrent-hash-map-vs-hash-map]]
- JLS §17, Threads and Locks: https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html
- JLS §17.4, Memory Model: https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.4


