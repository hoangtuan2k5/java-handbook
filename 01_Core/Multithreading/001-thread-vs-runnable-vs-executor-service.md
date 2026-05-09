# Thread vs Runnable vs ExecutorService

## What is it

`Thread` là worker thật sự được JVM và OS schedule.

`Runnable` là phần mô tả task cần chạy.

`ExecutorService` là abstraction nhận task, xếp hàng khi cần, reuse worker thread, và quản lý shutdown.

Mental model dễ nhớ:

- `Runnable` là công việc.
- `Thread` là một worker cụ thể.
- `ExecutorService` là đội trưởng quản lý nhiều worker.

Nếu trộn ba vai trò này với nhau, code rất nhanh rơi vào kiểu `new Thread(...)` khắp nơi, khó theo dõi lifecycle và khó kiểm soát overload.

## How I used to misunderstand it

Mình từng nghĩ học multithreading đồng nghĩa với việc tự tạo `new Thread(...)` mỗi khi muốn chạy song song.

Cách đó ổn để hiểu khái niệm ban đầu, nhưng nó không phải default tốt cho code thật. Mỗi lần tự tạo thread, bạn tự ôm luôn chuyện naming, reuse, queue, shutdown, exception, và giới hạn số worker.

Hiểu nhầm thứ hai là xem `Runnable` như “thread nhẹ”. Không đúng. `Runnable` không phải worker, nó chỉ là task description.

## How it actually works

Một `Thread` có lifecycle riêng, còn `Runnable` chỉ là code để thread chạy.

Khi bạn viết `new Thread(task).start()`, bạn đang gắn một task với một worker mới tạo ra. Worker đó chạy xong thì kết thúc.

Khi bạn `submit()` task vào `ExecutorService`, bạn tách hai chuyện ra:

- task nào cần chạy,
- worker nào sẽ chạy task đó.

Đó là lý do executor thường là lựa chọn thực tế hơn. Nó cho phép reuse thread thay vì tạo mới liên tục.

### Quick comparison table

| Concern | `Thread` | `Runnable` | `ExecutorService` |
|---|---|---|---|
| Đại diện cho gì | Một execution thread thật | Một task không trả kết quả | Bộ quản lý worker và queue task |
| Tự tạo worker mới không | Có | Không | Thường reuse worker sẵn có |
| Có giữ lifecycle không | Có | Không | Có, ở mức pool |
| Có phù hợp cho task lặp lại nhiều lần không | Thường không | Có, nếu nộp cho executor | Có |
| Có trả kết quả trực tiếp không | Không tiện | Không | Qua `Future`, thường với `Callable` |
| Default tốt cho production app | Hiếm khi | Chỉ là task description | Thường là có |

### Thread lifecycle, nhìn ở mức đủ dùng

| State | Ý nghĩa ngắn gọn | Khi hay gặp |
|---|---|---|
| `NEW` | Thread đã tạo nhưng chưa `start()` | `new Thread(...)` xong chưa chạy |
| `RUNNABLE` | Sẵn sàng chạy hoặc đang chạy | Sau `start()` |
| `BLOCKED` / `WAITING` / `TIMED_WAITING` | Đang chờ lock, chờ tín hiệu, hoặc chờ timeout | `synchronized`, `join()`, `sleep()` |
| `TERMINATED` | Chạy xong | `run()` kết thúc |

### Flow thực tế với executor

```text
submit(task)
    -> executor nhận task
    -> nếu có worker rảnh, worker chạy ngay
    -> nếu chưa có, task vào queue
    -> worker lấy task ra chạy
    -> executor tiếp tục reuse worker cho task khác
    -> shutdown khi không cần nữa
```

## Code example

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class Main {
    public static void main(String[] args) throws Exception {
        Runnable task = () -> System.out.println("run on " + Thread.currentThread().getName());

        ExecutorService executor = Executors.newFixedThreadPool(2);
        // task được giao cho worker trong pool, không phải thread mới mỗi lần
        Future<?> future = executor.submit(task);

        // chờ để thấy exception nếu task fail
        future.get();
        // custom executor nên có lifecycle rõ
        executor.shutdown();
    }
}
```

## When to use / when NOT to use

Dùng `Runnable` khi bạn chỉ cần mô tả một task không trả kết quả.

Dùng `Thread` khi đang học lifecycle, debug thread cụ thể, hoặc thật sự cần kiểm soát worker ở mức rất thấp.

Dùng `ExecutorService` khi app cần chạy nhiều task, cần giới hạn concurrency, cần queue, hoặc cần shutdown có kiểm soát.

Không tự tạo thread mới cho từng request, từng event, hoặc từng job nhỏ lặp lại liên tục. Cách đó vừa tốn memory stack, vừa tăng context switching, vừa khó chẩn đoán khi app chậm.

## How this connects to Spring

Trong Spring, bạn hiếm khi tự quản lý thread bằng tay ở controller hoặc service. Thực tế hơn là gặp `TaskExecutor`, `@Async`, scheduler, servlet container thread pool, hoặc executor của HTTP client.

Nếu hiểu rõ ba tầng `task`, `worker`, `pool`, bạn sẽ đọc được vì sao request bị block, vì sao async method không chạy như mong đợi, hoặc vì sao app tạo quá nhiều worker rồi chậm đi.

## Gotchas

- `Runnable` không trả kết quả. Nếu cần result, thường bạn sẽ đi qua `Callable` và `Future`.
- Task ném exception trong executor có thể bị chìm nếu bạn không quan sát `Future` hoặc không log đúng chỗ.
- Quên `shutdown()` custom executor có thể làm app hoặc test không thoát.
- Tạo quá nhiều thread không làm app tự động nhanh hơn. Nó có thể làm app chậm hơn.

## Check yourself

- Vì sao `Runnable` không phải là worker thread?
- Khi nào `new Thread(task).start()` hợp lý hơn `executor.submit(task)`?
- Trong ba khái niệm này, cái nào chịu trách nhiệm queue và shutdown?
- Vì sao tạo thread mới cho từng task nhỏ thường là thiết kế kém?
- Nếu task cần trả kết quả, vì sao chỉ nói `Runnable` thôi vẫn chưa đủ?

## Exercises

### Bài 1: Choose Execution Abstraction
Độ khó: Dễ

Đề bài:
Cho một requirement label, trả về `"runnable"` cho `"task-only"`, `"thread"` cho `"manual-worker"`, và `"executor-service"` cho `"managed-pool"`.

Ví dụ 1:
Đầu vào:
```text
requirement = "managed-pool"
```

Đầu ra:
```text
"executor-service"
```

Giải thích:
Một managed pool được biểu diễn bằng `ExecutorService`.

Ràng buộc:
- `requirement` là một trong các label đã liệt kê
- Việc matching có phân biệt chữ hoa chữ thường
- Chỉ trả về các output đã liệt kê

### Bài 2: Detect Missing Executor Shutdown
Độ khó: Dễ

Đề bài:
Cho biết code có tạo custom executor hay không và có gọi shutdown hay không, trả về `true` khi tồn tại executor lifecycle risk.

Ví dụ 1:
Đầu vào:
```text
customExecutorCreated = true, shutdownCalled = false
```

Đầu ra:
```text
true
```

Giải thích:
Một custom executor không bao giờ được shut down có thể giữ worker thread tiếp tục sống.

Ràng buộc:
- Input là boolean
- Chỉ trả về `true` khi có tạo custom executor và thiếu shutdown
- Không model các framework-managed executor

### Bài 3: Estimate Minimum Pool Size
Độ khó: Trung bình

Đề bài:
Cho task count và số task tối đa trên mỗi worker, trả về số worker tối thiểu cần thiết.

Ví dụ 1:
Đầu vào:
```text
taskCount = 10, maxTasksPerWorker = 3
```

Đầu ra:
```text
4
```

Giải thích:
Bốn worker có thể xử lý mười task khi mỗi worker xử lý nhiều nhất ba task.

Ràng buộc:
- `taskCount >= 0`
- `maxTasksPerWorker > 0`
- Kết quả nằm trong phạm vi của `int`

## Links

- [[007-thread-pool]]
- [[008-completable-future]]
- `Thread` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.html
- `Runnable` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Runnable.html
- `ExecutorService` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html
