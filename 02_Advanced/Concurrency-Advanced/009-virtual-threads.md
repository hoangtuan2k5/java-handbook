# Virtual Threads

## What is it

Virtual threads là lightweight thread do JVM quản lý để bạn có thể chạy rất nhiều tác vụ theo style thread-per-task mà không phải trả chi phí nặng như platform thread truyền thống.

Mental model đúng là:

- task của bạn vẫn nhìn như blocking imperative code,
- nhưng khi block ở operation tương thích, JVM có thể park virtual thread,
- carrier thread được trả về để phục vụ task khác.

Điểm này làm model “một request, một thread” trở nên hấp dẫn lại cho nhiều bài toán I/O-heavy.

```text
virtual thread chạy
    -> gặp blocking I/O tương thích
    -> JVM park virtual thread
    -> carrier thread rảnh để chạy việc khác
    -> I/O xong thì virtual thread resume
```

## How I used to misunderstand it

Mình từng nghĩ virtual threads là thay thế hoàn toàn cho thread pool và reactive.

Thực ra chúng giải rất tốt bài toán **nhiều blocking task độc lập**, nhưng không làm CPU-bound code tự nhanh hơn, không xóa race condition, và không chữa được bottleneck ở downstream như database pool hoặc HTTP QPS limit.

Hiểu nhầm khác là tưởng đã dùng virtual threads thì không cần quan tâm lock nữa. Sai. Shared mutable state, contention, và pinning vẫn là vấn đề thật.

## How it actually works

Khi virtual thread block ở chỗ JVM có thể quản lý tốt, carrier thread có thể được giải phóng để chạy việc khác. Nhờ vậy, số lượng concurrent task bạn chịu nổi không còn bị giới hạn nặng bởi số platform thread đang tồn tại.

Nhưng lợi ích này không phải phép màu. Nếu virtual thread bị **pin**, carrier thread có thể bị giữ cứng lâu hơn mong muốn. Hai ví dụ kinh điển là:

- giữ `synchronized` rồi block,
- đi vào native hoặc foreign code path làm JVM khó unmount.

### Virtual threads khác gì platform threads

| Câu hỏi | Platform thread | Virtual thread |
|---|---|---|
| Cost per thread | Cao hơn | Thấp hơn nhiều |
| Hợp cho rất nhiều blocking task | Kém hơn | Rất hợp |
| Tự làm CPU-bound code nhanh hơn | Không | Cũng không |
| Có thể bị contention trên shared state | Có | Có |
| Có thể bị bottleneck bởi downstream pool | Có | Có |

### Chúng giải quyết gì, không giải quyết gì

| Vấn đề | Virtual threads giúp không |
|---|---|
| Quá nhiều task chờ I/O | Có |
| Code imperative khó scale vì thread quá đắt | Có |
| CPU-bound thuật toán chậm | Không |
| Shared lock nóng | Không |
| DB connection pool quá nhỏ | Không |

### Decision matrix nhỏ

| Nếu bạn cần... | Nghĩ tới trước |
|---|---|
| Nhiều blocking task độc lập, code dễ đọc | Virtual threads |
| Streaming và backpressure end-to-end | Reactive |
| CPU-bound divide-and-conquer | `ForkJoinPool` |
| Ít task nền đơn giản | Thread pool thường |

## Code example

```java
import java.util.concurrent.Executors;

try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        Thread.sleep(100);
        return null;
    });
}
```

Code vẫn nhìn rất imperative. Điểm mới là chi phí quản lý số lượng lớn task chờ I/O thấp hơn nhiều so với tạo một platform thread cho mỗi task.

## When to use / when NOT to use

Dùng virtual threads khi:

- có rất nhiều task độc lập,
- phần lớn thời gian là chờ I/O,
- bạn muốn giữ imperative style dễ đọc,
- bottleneck chính trước đây là cost của platform thread.

Không dùng như thuốc tiên khi:

- workload thuần CPU-bound,
- pinning risk cao,
- downstream resource như DB pool hoặc semaphore gate mới là giới hạn thật,
- bài toán thực ra cần backpressure pipeline hơn là nhiều blocking task.

## How this connects to real Java projects

Trong Spring, virtual threads đặc biệt hấp dẫn cho app MVC hoặc service imperative muốn scale concurrency mà không chuyển sang reactive hoàn toàn.

Tuy vậy, chúng không biến JDBC blocking với pool nhỏ thành vô hạn. Nếu database chỉ có ít connection, thêm hàng ngàn virtual thread chỉ làm hàng ngàn task chờ tới lượt vào pool đó.

Đây là chỗ mental model cần thật rõ: virtual threads giảm chi phí chờ thread, không xóa giới hạn của downstream system.

## Scope / Not covered

Note này tập trung vào mental model và fit decision. Khi áp dụng thật, vẫn phải kiểm tra JDK version, framework support, JDBC/driver behavior, pinning diagnostics, `ThreadLocal` usage, connection pool sizing, và observability. Virtual threads là concurrency model, không phải capacity plan hoàn chỉnh.

## Gotchas

- Virtual threads không làm CPU-bound code nhanh hơn.
- Pinning do `synchronized` kết hợp blocking call có thể làm carrier thread bị giữ.
- `ThreadLocal` và assumption cũ về thread-per-request cần được review lại khi số lượng concurrent task tăng mạnh.
- Concurrency cao hơn không đồng nghĩa throughput tổng thể tăng nếu bottleneck nằm ở nơi khác.
- Structured concurrency và scoped values là topic liên quan nhưng không được cover sâu trong note này.

## Check yourself

- Virtual threads đang tối ưu chi phí nào, CPU hay thread management?
- Vì sao nhiều blocking task độc lập là trường hợp sử dụng đẹp cho virtual threads?
- Pinning làm hỏng mental model ở đâu?
- Nếu DB pool vẫn nhỏ, vì sao virtual threads không tự chữa được bottleneck?
- Khi nào reactive vẫn phù hợp hơn virtual threads?

## Exercises

### Exercise 1: Choose Thread Model

Độ khó: Easy

Đề bài:
Cho ba cờ `manyBlockingTasks`, `cpuBoundOnly`, `highPinningRisk`. Trả về `"virtual-threads"` nếu workload có nhiều blocking task, không thuần CPU-bound, và pinning risk không cao. Ngược lại trả về `"platform-threads"`.

Ví dụ 1:

Đầu vào:
```text
manyBlockingTasks = true
cpuBoundOnly = false
highPinningRisk = false
```

Đầu ra:
```text
"virtual-threads"
```

Giải thích:
Đây là trường hợp rất hợp cho model nhiều virtual thread chờ I/O.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"virtual-threads"` hoặc `"platform-threads"`
- Không xét framework cụ thể

### Exercise 2: Detect Pinning Risk Index

Độ khó: Medium

Đề bài:
Cho mảng `operations`, mỗi phần tử là một trong `"cpu"`, `"blocking-io"`, `"synchronized-blocking"`, `"native-call"`. Trả về index đầu tiên có pinning risk, nghĩa là `"synchronized-blocking"` hoặc `"native-call"`. Nếu không có, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
operations = ["cpu", "blocking-io", "synchronized-blocking"]
```

Đầu ra:
```text
2
```

Giải thích:
Operation ở index `2` có nguy cơ giữ carrier thread trong lúc virtual thread đang block.

Ràng buộc:

- `1 <= operations.length <= 100000`
- Mỗi phần tử thuộc đúng tập giá trị đã cho
- Không cần mô phỏng scheduler thật

### Exercise 3: Count Pinned Task Rounds

Độ khó: Easy

Đề bài:
Cho `pinnedTasks` và `carrierThreads`. Giả sử mỗi pinned task giữ trọn một carrier thread cho tới hết round, và mỗi round chỉ chạy được tối đa `carrierThreads` pinned task. Trả về số round tối thiểu cần để chạy hết toàn bộ pinned task.

Ví dụ 1:

Đầu vào:
```text
pinnedTasks = 10
carrierThreads = 4
```

Đầu ra:
```text
3
```

Giải thích:
Hai round đầu xử lý `8` task, round cuối xử lý `2` task còn lại.

Ràng buộc:

- `0 <= pinnedTasks <= 1000000`
- `1 <= carrierThreads <= 1000000`
- Nếu `pinnedTasks = 0`, kết quả là `0`

## Links

- [[001-reentrant-lock-vs-synchronized]]
- [[007-fork-join-pool]]
- [[008-reactive-vs-imperative]]
- [JEP 444, Virtual Threads](https://openjdk.org/jeps/444)
- [Java 21 Guide, Virtual Threads](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html)
- [JDK 21 `Executors#newVirtualThreadPerTaskExecutor`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Executors.html#newVirtualThreadPerTaskExecutor())
