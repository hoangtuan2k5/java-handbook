# synchronized

## What is it

`synchronized` là keyword dùng intrinsic lock của object hoặc class để bảo vệ critical section.

Nếu nhiều thread cùng muốn đi vào cùng một block `synchronized` dùng cùng lock, chỉ một thread được vào tại một thời điểm.

Mental model: đây là cánh cửa có chìa khóa chung. Ai giữ chìa khóa thì được vào vùng sửa shared state, người còn lại phải chờ.

## How I used to misunderstand it

Mình từng nghĩ thêm `synchronized` vào method là xong thread safety.

Không đơn giản vậy. `synchronized` chỉ bảo vệ code nào thật sự đi qua cùng lock đó. Nếu cùng state bị truy cập ở chỗ khác mà không dùng chung lock, bug vẫn còn nguyên.

Hiểu nhầm nữa là xem `synchronized` chỉ là bài toán mutual exclusion. Thực ra nó còn liên quan visibility vì unlock và lock tạo ra happens-before relationship.

## How it actually works

Mỗi object trong Java có thể đóng vai intrinsic monitor.

Khi một thread vào `synchronized (lock) { ... }`, nó phải acquire monitor của `lock` trước. Khi block kết thúc, monitor được release, kể cả nếu có exception.

Điều bạn nhận được từ `synchronized` thường là hai thứ cùng lúc:

1. mutual exclusion, tức là không cho nhiều thread cùng vào critical section,
2. visibility, tức là update được publish đúng hơn cho thread khác dùng cùng lock.

### `synchronized` cho bạn gì

| Concern | `synchronized` có xử lý không | Ghi chú |
|---|---|---|
| Chặn hai thread cùng sửa state cùng lúc | Có | Nếu dùng cùng lock |
| Publish thay đổi cho thread khác | Có | Qua happens-before khi unlock rồi lock lại cùng monitor |
| Atomic cho nhiều bước trong critical section | Có | Miễn là mọi bước nằm trong block |
| Timeout khi chờ lock | Không | Cần API như `Lock.tryLock()` |
| Tránh deadlock tự động | Không | Bạn vẫn phải thiết kế lock ordering |

### Nhìn bằng flow ngắn

```text
Thread A acquires lock
    -> đọc và sửa shared state
    -> releases lock

Thread B acquires same lock later
    -> thấy state sau khi A đã publish qua unlock
```

### So sánh nhanh với hai công cụ hay bị nhầm

| Nếu bạn cần... | `volatile` | Atomic class | `synchronized` |
|---|---|---|---|
| Chỉ visibility cho một flag đơn giản | Tốt | Có thể | Thường quá tay |
| Counter `++` không mất update | Không | Tốt | Tốt |
| Giữ invariant qua nhiều field | Không | Thường không đủ | Tốt hơn |
| Critical section gồm nhiều dòng | Không | Chỉ khi gom vào một operation phù hợp | Tốt |

## Code example

```java
public class Counter {
    private final Object lock = new Object();
    private int value;

    public void increment() {
        synchronized (lock) {
            // read-modify-write được bảo vệ trong cùng critical section
            value++;
        }
    }

    public int getValue() {
        synchronized (lock) {
            // đọc cùng lock để giữ logic nhất quán
            return value;
        }
    }
}
```

## When to use / when NOT to use

Dùng `synchronized` khi shared mutable state nhỏ, ownership rõ, và bạn muốn correctness dễ đọc hơn lock-free trick.

Nó rất hợp cho các critical section ngắn và rõ ràng, nhất là khi invariant liên quan nhiều bước hoặc nhiều field.

Không nên giữ lock trong lúc gọi I/O, gọi database, gọi HTTP, hoặc gọi callback ngoài tầm kiểm soát. Thời gian giữ lock càng dài, contention và nguy cơ deadlock càng tăng.

## How this connects to real Java projects

Spring singleton bean được nhiều request thread gọi cùng lúc. Nếu bean giữ mutable field, bạn cần một chiến lược bảo vệ rõ ràng.

Nhiều trường hợp tốt nhất vẫn là bỏ shared state đi. Nếu bắt buộc phải có state chung trong bean, `synchronized` là cách dễ đọc hơn nhiều so với việc rải logic CAS phức tạp ở khắp nơi.

Nhưng cũng phải rất cẩn thận. Khóa cả service method rồi gọi tiếp repository hoặc API ngoài có thể biến latency nhỏ thành queue chờ lớn.

## Gotchas

- Dùng lock object khác nhau thì không bảo vệ cùng một state.
- `synchronized` trên instance method khóa `this`, đôi khi quá rộng hoặc dễ bị code ngoài vô tình dùng chung.
- Đọc ngoài lock nhưng ghi trong lock có thể làm logic vẫn sai.
- `synchronized` không tự cứu bạn khỏi deadlock.

## Check yourself

- `synchronized` giải quyết mutual exclusion thôi, hay còn giải quyết cả visibility?
- Vì sao cùng một state nhưng đọc bằng lock A, ghi bằng lock B vẫn là bug?
- Khi nào `synchronized` rõ hơn atomic class?
- Vì sao giữ lock rồi gọi API ngoài là nguy hiểm?
- Nếu chỉ cần stop flag đơn giản, vì sao `synchronized` có thể là quá nặng?

## Exercises

### Bài 1: Review Shared State Locking
Độ khó: Dễ

Đề bài:
Cho lock được dùng bởi write path và lock được dùng bởi read path trên cùng một shared state, chỉ trả về `true` khi cả hai path dùng cùng một lock khác `"none"`.

Ví dụ 1:
Đầu vào:
```text
writeLock = "counterLock", readLock = "counterLock"
```

Đầu ra:
```text
true
```

Giải thích:
Cả read access lẫn write access đều được bảo vệ bởi cùng một logical lock.

Ràng buộc:
- `writeLock` và `readLock` là non-null
- `"none"` có nghĩa là path đó không được bảo vệ
- Việc matching có phân biệt chữ hoa chữ thường

### Bài 2: Count Protected Writes
Độ khó: Trung bình

Đề bài:
Cho các operation label, đếm số write được bảo vệ bởi lock. Một protected write có label là `"LOCKED_WRITE"`.

Ví dụ 1:
Đầu vào:
```text
operations = ["LOCKED_WRITE", "READ", "UNLOCKED_WRITE"]
```

Đầu ra:
```text
1
```

Giải thích:
Chỉ có một write operation được bảo vệ.

Ràng buộc:
- `operations` là non-null
- `0 <= operations.length <= 100000`
- `operations[i]` là non-null

### Bài 3: Should Synchronize Access
Độ khó: Dễ

Đề bài:
Cho biết state có shared và mutable hay không, trả về `true` khi access cần synchronization hoặc một thread-safety strategy khác.

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
Shared mutable state cần được bảo vệ.

Ràng buộc:
- Input là boolean
- Chỉ trả về `true` khi cả hai đều là `true`
- Không chọn exact locking strategy

## Links

- [[002-race-condition]]
- [[004-volatile]]
- [[005-atomic-classes]]
- [[006-deadlock]]
- JLS §17.1, Synchronization: https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.1
- JLS §17.4.5, Happens-before Order: https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.4.5

