# volatile

## What is it

`volatile` đảm bảo visibility cho một field giữa các thread.

Khi một thread ghi vào volatile field, thread khác đọc field đó sẽ thấy update theo rule của Java Memory Model rõ hơn so với field thường.

Mental model: `volatile` là biển báo nói rằng biến này cần publish và observe đúng hơn giữa các thread. Nó không phải là lock, cũng không biến mọi logic thành thread-safe.

## How I used to misunderstand it

Mình từng nghĩ `volatile` là phiên bản nhẹ của `synchronized`, kiểu đủ nhanh mà vẫn thread-safe cho hầu hết trường hợp.

Sai ở chỗ `volatile` chỉ giải quyết visibility của từng lần đọc và ghi field đó. Nó không làm compound operation như `count++` trở thành atomic.

Hiểu nhầm khác là nghĩ “thấy giá trị mới nhất” đồng nghĩa “logic đúng”. Thấy đúng giá trị mới nhất vẫn không cứu được business invariant nếu update gồm nhiều bước hoặc nhiều biến.

## How it actually works

Ghi vào một volatile field happens-before lần đọc volatile field đó ở thread khác về sau.

Điều này rất hợp cho các state đơn giản như stop flag hoặc reference được publish sau khi đã khởi tạo xong.

Nhưng `volatile` không làm được ba chuyện sau:

- không serialize critical section,
- không giữ invariant qua nhiều field,
- không làm read-modify-write trở thành atomic.

### So sánh ngắn, visibility khác atomicity và locking ở đâu

| Câu hỏi | `volatile` | Atomic class | `synchronized` |
|---|---|---|---|
| Thread khác có thấy giá trị mới không | Có | Có | Có |
| `count++` có an toàn không | Không | Có | Có |
| Nhiều bước phải chạy như một unit không | Không | Chỉ với operation atomic phù hợp | Có |
| Bảo vệ nhiều field cùng lúc không | Không | Thường không | Có |

### Volatile works well ở shape nào

```text
Writer thread: running = false
                      |
                      | happens-before
                      v
// eventually thấy false và dừng
Reader thread: while (running) { ... }
```

### Decision matrix nhỏ

| Nếu bạn cần... | Chọn gì trước |
|---|---|
| Stop flag hoặc published reference đơn giản | `volatile` |
| Counter hoặc compare-and-set | Atomic class |
| Critical section nhiều bước | `synchronized` hoặc lock |

## Code example

```java
class WorkerFlag {
    private volatile boolean running = true;

    void work() {
        while (running) {
            // loop sẽ thấy update từ thread khác mà không cần lock riêng cho flag này
        }
    }

    void stop() {
        // publish tín hiệu dừng
        running = false;
    }
}
```

## When to use / when NOT to use

Dùng `volatile` cho flag hoặc reference state đơn giản khi operation thực tế chỉ là đọc hoặc ghi độc lập.

Nó đặc biệt hợp khi có một thread publish trạng thái, còn thread khác chỉ cần nhìn thấy trạng thái đó đúng lúc.

Không dùng `volatile` cho increment, check-then-act, composite update, hoặc invariant nhiều field. Khi đó bạn đang cần atomicity hoặc locking, không chỉ visibility.

## How this connects to real Java projects

Trong Spring, `volatile` có thể xuất hiện ở background worker, shutdown flag, health flag, hoặc lightweight cached reference.

Nhưng phần lớn business service nên stateless. Nếu service đã có shared mutable state phức tạp, chỉ thêm `volatile` thường là dấu hiệu đang dùng sai công cụ.

## Gotchas

- `volatile` không làm `++` atomic.
- Volatile reference không làm object bên trong reference đó thread-safe.
- Nếu correctness phụ thuộc nhiều field phải đổi cùng nhau, `volatile` không đủ.
- Đọc đúng giá trị mới nhất chưa chắc đã giữ đúng business rule.

## Check yourself

- `volatile` giải quyết visibility hay atomicity?
- Vì sao `volatile int count; count++;` vẫn có thể mất update?
- Khi nào stop flag là trường hợp sử dụng đẹp cho `volatile`?
- Nếu một object mutable được giữ trong volatile reference, phần object bên trong có tự thread-safe không?
- Khi nào bạn nên chuyển từ `volatile` sang atomic class hoặc `synchronized`?

## Exercises

### Bài 1: Choose Volatile Use Case
Độ khó: Dễ

Đề bài:
Cho một operation label, trả về `true` nếu `volatile` là đủ. Chỉ có `"stop-flag"` là đủ; `"increment"` và `"check-then-act"` thì không.

Ví dụ 1:
Đầu vào:
```text
operation = "stop-flag"
```

Đầu ra:
```text
true
```

Giải thích:
Stop flag đơn giản là một trường hợp sử dụng phổ biến của `volatile`.

Ràng buộc:
- `operation` là một trong các label đã liệt kê
- Việc matching có phân biệt chữ hoa chữ thường
- Trả về một boolean

### Bài 2: Detect Non Atomic Increment
Độ khó: Dễ

Đề bài:
Cho một code snippet, trả về `true` nếu nó chứa `++` trên một volatile counter label.

Ví dụ 1:
Đầu vào:
```text
code = "volatileCount++"
```

Đầu ra:
```text
true
```

Giải thích:
Increment không phải atomic ngay cả khi variable là volatile.

Ràng buộc:
- `code` là non-null
- `code.length <= 100000`
- Simple substring detection cho `++` là đủ

### Bài 3: Classify Shared Flag State
Độ khó: Trung bình

Đề bài:
Cho hai boolean `writtenByOneThread` và `readByManyThreads`, chỉ trả về `"volatile-candidate"` khi cả hai đều là `true`; ngược lại trả về `"not-enough-info"`.

Ví dụ 1:
Đầu vào:
```text
writtenByOneThread = true, readByManyThreads = true
```

Đầu ra:
```text
"volatile-candidate"
```

Giải thích:
Pattern này khớp với một visibility flag đơn giản.

Ràng buộc:
- Input là boolean
- Chỉ trả về hai label đã liệt kê
- Không quyết định correctness cho compound operation

## Links

- [[002-race-condition]]
- [[003-synchronized]]
- [[005-atomic-classes]]
- [[../../05_Open-Questions/001-volatile]]
- JLS §17.4, Memory Model: https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.4
- JLS §17.4.5, Happens-before Order: https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.4.5

