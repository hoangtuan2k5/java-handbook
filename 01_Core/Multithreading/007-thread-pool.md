# ThreadPool

## What is it

Thread pool là nhóm worker thread được quản lý để chạy nhiều task.

Thay vì tạo thread mới cho từng việc, task được nộp vào executor, có thể đi qua queue, rồi worker có sẵn lấy ra xử lý.

Mental model: thread pool giống một quầy có số nhân viên giới hạn. Có bao nhiêu người phục vụ, có hàng chờ dài bao nhiêu, và khi quầy quá tải thì xử lý thế nào, tất cả đều là một phần của thiết kế.

## How I used to misunderstand it

Mình từng nghĩ tăng pool size gần như luôn làm app nhanh hơn.

Sai ở hai chỗ lớn.

Thứ nhất, nhiều thread hơn không giúp CPU-bound task chạy nhanh mãi. Sau một điểm nào đó, bạn chỉ đang trả thêm giá context switching và memory stack.

Thứ hai, với I/O-bound workload, pool lớn hơn có thể hợp lý hơn, nhưng vẫn phải nhìn downstream limit như database, HTTP API, hoặc connection pool. Pool lớn quá có thể chỉ làm bạn overload chỗ nghẽn nhanh hơn.

## How it actually works

Một thread pool thực tế thường có mấy phần quan trọng:

- worker count,
- task queue,
- rejection policy,
- shutdown lifecycle.

Nếu worker bận hết, task mới thường phải chờ trong queue hoặc bị reject, tuỳ config.

### Pool flow đơn giản

```text
submit(task)
    -> worker rảnh? yes -> chạy ngay
    -> worker rảnh? no  -> vào queue
    -> queue đầy? yes   -> rejection policy quyết định chuyện gì xảy ra tiếp
```

### Sizing mindset

| Workload shape | Pool size intuition | Vì sao |
|---|---|---|
| CPU-bound | Gần số core, rồi đo | Thread dư chỉ làm tăng context switching |
| I/O-bound | Có thể nhiều hơn số core | Worker thường ngồi chờ I/O |
| Downstream-limited | Bị giới hạn bởi DB, API, queue ngoài | Pool lớn quá chỉ đẩy nghẽn sang nơi khác |
| Unknown | Đo trước, đừng đoán | Không có con số thần kỳ |

### `Executors` tiện, nhưng chưa đủ cho mọi bài toán

| Câu hỏi | Cần quan tâm gì |
|---|---|
| Queue có giới hạn không | Queue vô hạn có thể che overload cho tới khi memory tăng |
| Rejection policy là gì | Khi quá tải thì block, fail, hay chạy ở caller |
| Ai shutdown pool | Custom pool cần lifecycle rõ |
| Downstream chịu được bao nhiêu concurrency | Pool app không thể lớn hơn reality của hệ thống |

## Code example

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(4);

        pool.submit(() -> System.out.println("task on " + Thread.currentThread().getName()));

        // custom pool cần lifecycle rõ, không để worker sống mãi
        pool.shutdown();
    }
}
```

## When to use / when NOT to use

Dùng thread pool khi có nhiều task ngắn hoặc lặp lại, cần giới hạn concurrency, cần reuse worker, hoặc cần tách producer khỏi worker execution.

Không dùng mindset “pool càng to càng tốt”. Đó thường là bước đầu tiên dẫn tới queue backlog, downstream overload, và latency khó đoán.

Không dùng unbounded queue mà không hiểu traffic shape. Queue vô hạn có thể biến overload thành memory problem âm thầm.

## How this connects to real Java projects

Spring MVC, `@Async`, scheduler, application task executor, servlet container, database connection pool, tất cả đều có khái niệm pool ở đâu đó.

Một request chậm có thể không phải vì CPU yếu, mà vì nó đang chờ worker thread, chờ connection, hoặc đứng trong queue.

Vì vậy cấu hình thread pool không thể nhìn riêng lẻ. Nó phải đi cùng khả năng chịu tải của database, HTTP downstream, và các pool khác trong cùng request path.

## Gotchas

- Queue không giới hạn có thể che giấu overload cho tới khi memory tăng mạnh.
- Pool quá lớn có thể làm downstream chết nhanh hơn chứ không cứu throughput.
- Quên shutdown custom executor làm app hoặc test treo.
- Dùng chung một pool cho workload rất khác nhau có thể làm task quan trọng bị starvation.

## Handbook rule

- Pool size phải có giới hạn; pool unbounded biến overload thành memory problem.
- Queue có cap rõ; chọn rejection policy phù hợp business (caller-runs, abort, drop).
- Workload khác biệt không dùng chung một pool; cô lập để tránh starvation.
- Custom executor phải `shutdown()`/`awaitTermination()` trong vòng đời rõ.
- Đo throughput/latency trước khi tăng pool; “to hơn” không tự nhanh hơn.

## Check yourself

- Vì sao pool size tốt cho CPU-bound thường khác I/O-bound?
- Nếu app gọi database chậm, tăng thread pool có thể làm tình hình xấu hơn ở điểm nào?
- Queue vô hạn nguy hiểm ở chỗ nào?
- Rejection policy giúp trả lời câu hỏi gì khi hệ thống quá tải?
- Vì sao phải nhìn thread pool cùng với downstream limit, không chỉ nhìn riêng app?

## Exercises

### Bài 1: Detect Queue Backlog
Độ khó: Dễ

Đề bài:
Cho queued task count và worker count, trả về `true` nếu số queued task lớn hơn số worker.

Ví dụ 1:
Đầu vào:
```text
queuedTasks = 10, workers = 4
```

Đầu ra:
```text
true
```

Giải thích:
Số task đang chờ nhiều hơn số worker sẵn có.

Ràng buộc:
- `queuedTasks >= 0`
- `workers > 0`
- Trả về một boolean

### Bài 2: Assess Thread Pool Sizing Risk
Độ khó: Trung bình

Đề bài:
Cho biết workload có phải CPU-bound hay không, có block trên I/O hay không, và có gọi một downstream service bị giới hạn hay không, trả về sizing concern quan trọng nhất. Trả về `"check-downstream-limit"` nếu downstream bị giới hạn, `"keep-near-cores"` nếu CPU-bound, `"can-use-more-workers"` nếu block trên I/O, ngược lại trả về `"measure-first"`.

Ví dụ 1:
Đầu vào:
```text
cpuBound = false, blockingIo = true, downstreamLimited = true
```

Đầu ra:
```text
"check-downstream-limit"
```

Giải thích:
Downstream capacity là mối quan tâm đầu tiên vì pool lớn hơn có thể làm nó bị quá tải.

Ràng buộc:
- Input là boolean
- Downstream limit có độ ưu tiên cao nhất
- Chỉ trả về một trong các label đã liệt kê

### Bài 3: Should Reject Task
Độ khó: Trung bình

Đề bài:
Cho queue size và queue capacity, trả về `true` khi queue đã đầy.

Ví dụ 1:
Đầu vào:
```text
queueSize = 5, capacity = 5
```

Đầu ra:
```text
true
```

Giải thích:
Không nên nhận thêm task khi queue đạt đến capacity.

Ràng buộc:
- `queueSize >= 0`
- `capacity >= 0`
- Trả về `true` khi `queueSize >= capacity`

## Links

- [[001-thread-vs-runnable-vs-executor-service]]
- [[008-completable-future]]
- [[../../02_Advanced/Concurrency-Advanced/007-fork-join-pool]]
- `ThreadPoolExecutor` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html
- `Executors` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Executors.html
