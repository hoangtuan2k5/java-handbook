# ForkJoinPool

## What is it

`ForkJoinPool` là executor được thiết kế cho **divide-and-conquer compute task**. Một bài toán lớn được tách thành subtask nhỏ hơn, rồi `join` kết quả lại.

Mental model cốt lõi là **work-stealing**:

- mỗi worker có queue riêng,
- worker nào hết việc có thể lấy task từ worker khác,
- mục tiêu là giữ CPU luôn có việc làm với workload chia nhỏ tự nhiên.

```text
big task
  -> split thành task A + task B
  -> worker 1 xử lý A
  -> worker 2 xử lý B
  -> worker rảnh có thể steal task còn lại
  -> join kết quả
```

## How I used to misunderstand it

Mình từng nghĩ `ForkJoinPool` là thread pool “nhanh hơn” cho mọi thứ.

Sai ở chỗ nó không sinh ra cho mọi loại task. Nó mạnh nhất khi:

- task là CPU-bound,
- có thể chia đệ quy tương đối cân bằng,
- mỗi subtask đủ nhỏ để song song hóa có ích nhưng không quá nhỏ tới mức overhead lấn át.

Nếu task chủ yếu là blocking I/O hoặc business flow tuần tự bình thường, `ForkJoinPool` thường không phải lựa chọn sáng nhất.

## How it actually works

Task trong `ForkJoinPool` thường là `RecursiveTask` hoặc `RecursiveAction`. Khi bài toán còn lớn hơn threshold, task tự tách tiếp, `fork` subtask, rồi `join` kết quả.

Điểm mạnh nằm ở việc giảm chi phí điều phối cho workload đệ quy, đồng thời nhờ work-stealing để tránh worker rảnh ngồi chờ.

### Workload nào hợp, workload nào kém hợp

| Câu hỏi | Hợp với `ForkJoinPool` | Kém hợp với `ForkJoinPool` |
|---|---|---|
| CPU-bound | Có | Không phải vấn đề chính |
| Blocking I/O | Không | Có rủi ro làm worker kẹt |
| Chia nhỏ đệ quy tự nhiên | Có | Không |
| Task độc lập đơn giản, không cần split | Không cần thiết | Dùng pool thường dễ hơn |

### Throughput phụ thuộc vào đâu

| Yếu tố | Tác động |
|---|---|
| Threshold split hợp lý | Giúp đủ song song mà không quá nhiều overhead |
| Task quá nhỏ | Overhead `fork/join` tăng mạnh |
| Blocking call trong worker | Làm pool lệch khỏi thiết kế compute-oriented |
| Shared mutable state | Giảm lợi ích song song, tăng bug risk |

### Decision matrix nhỏ

| Nếu bạn cần... | Nghĩ tới trước |
|---|---|
| Xử lý cây, mảng lớn, thuật toán chia để trị | `ForkJoinPool` |
| Gọi HTTP, DB, file I/O | Thread pool thường hoặc async model |
| Một ít task độc lập | `ExecutorService` |
| Nhiều blocking task chờ I/O | Virtual threads |

## Code example

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

class SumTask extends RecursiveTask<Integer> {
    private final int[] values;
    private final int from;
    private final int to;
    private static final int THRESHOLD = 2;

    SumTask(int[] values, int from, int to) {
        this.values = values;
        this.from = from;
        this.to = to;
    }

    @Override
    protected Integer compute() {
        if (to - from <= THRESHOLD) {
            int sum = 0;
            for (int i = from; i < to; i++) {
                sum += values[i];
            }
            return sum;
        }

        int mid = (from + to) / 2;
        SumTask left = new SumTask(values, from, mid);
        SumTask right = new SumTask(values, mid, to);

        left.fork();
        int rightResult = right.compute();
        int leftResult = left.join();
        return leftResult + rightResult;
    }
}

int[] values = {1, 2, 3, 4};
int result = ForkJoinPool.commonPool().invoke(new SumTask(values, 0, values.length));
```

Ví dụ này cho thấy đúng shape `split -> fork -> join -> combine`, thay vì chỉ chạy một task đơn lẻ không hề chia nhỏ.

## When to use / when NOT to use

Dùng `ForkJoinPool` khi:

- workload là CPU-bound,
- có thể chia đệ quy thành các phần tương đối cân bằng,
- không có nhiều blocking I/O bên trong compute path.

Không dùng khi:

- task chủ yếu chờ DB, HTTP, hoặc file,
- không có lợi ích rõ từ recursive splitting,
- shared mutable state làm parallelism mất ý nghĩa,
- code chỉ cần một executor đơn giản để chạy task thường.

## How this connects to real Java projects

Trong Spring app, `ForkJoinPool` hợp hơn cho job tính toán trong memory, xử lý dữ liệu lớn, hoặc utility CPU-heavy. Nó ít hợp cho service business thông thường vì tầng đó hay dính I/O.

Nếu workload là nhiều task blocking độc lập, virtual threads thường cho mental model dễ hiểu hơn. Nếu chỉ cần vài background task, thread pool thường đơn giản hơn nhiều.

## Gotchas

- Split quá nhỏ làm overhead `fork/join` lấn át phần compute thật.
- Nhét blocking I/O vào worker làm mất lợi thế work-stealing.
- Dùng `commonPool()` bừa bãi có thể tạo coupling khó thấy giữa nhiều chỗ trong app.
- Shared mutable state giữa subtask làm kết quả song song khó tin hơn nhiều.

## Handbook rule

- `ForkJoinPool` cho CPU-bound, recursive divide-and-conquer; không cho blocking I/O.
- Split granularity phải đủ lớn; chia quá nhỏ overhead lấn át.
- Tránh dùng `commonPool()` cho task quan trọng; cô lập pool theo workload.
- Shared mutable state giữa subtask phá ý nghĩa parallelism; ưu tiên pure compute.
- Blocking call trong worker dùng `ManagedBlocker`, hoặc tốt hơn là chuyển sang executor khác.

## Check yourself

- Vì sao `ForkJoinPool` không phải “pool nhanh hơn” cho mọi workload?
- Work-stealing giúp gì cho CPU utilization?
- Nếu task chủ yếu chờ I/O, vì sao `ForkJoinPool` thường không phải chọn đúng?
- Threshold split ảnh hưởng throughput như thế nào?
- Khi nào virtual threads diễn tả bài toán tốt hơn `ForkJoinPool`?

## Exercises

### Exercise 1: Count Leaf Tasks

Độ khó: Medium

Đề bài:
Cho `workloadSize` và `threshold`. Mỗi lần `workloadSize > threshold`, task sẽ tách thành hai task con với kích thước `floor(size / 2)` và `size - floor(size / 2)`. Trả về số leaf task cuối cùng.

Ví dụ 1:

Đầu vào:
```text
workloadSize = 8
threshold = 2
```

Đầu ra:
```text
4
```

Giải thích:
`8` tách thành `4 + 4`, rồi mỗi `4` tách thành `2 + 2`, tạo tổng cộng `4` leaf task.

Ràng buộc:

- `1 <= threshold <= workloadSize <= 1000000`
- Luôn tách thành đúng hai phần như mô tả
- Không dùng thread thật

### Exercise 2: Max Split Depth

Độ khó: Medium

Đề bài:
Với cùng rule tách ở bài trên, trả về độ sâu split lớn nhất, trong đó task ban đầu có depth `0`.

Ví dụ 1:

Đầu vào:
```text
workloadSize = 8
threshold = 2
```

Đầu ra:
```text
2
```

Giải thích:
`8 -> 4 -> 2`, nên độ sâu tối đa là `2`.

Ràng buộc:

- `1 <= threshold <= workloadSize <= 1000000`
- Nếu `workloadSize <= threshold`, kết quả là `0`
- Chỉ xét depth lớn nhất của cây chia task

### Exercise 3: Choose ForkJoin Workload

Độ khó: Easy

Đề bài:
Cho ba cờ `recursivelyDivisible`, `blockingIo`, `sharedMutableState`. Trả về `"fork-join-pool"` nếu workload có thể chia đệ quy, không blocking I/O, và không phụ thuộc shared mutable state. Ngược lại trả về `"thread-pool"`.

Ví dụ 1:

Đầu vào:
```text
recursivelyDivisible = true
blockingIo = false
sharedMutableState = false
```

Đầu ra:
```text
"fork-join-pool"
```

Giải thích:
Đây là case điển hình cho divide-and-conquer compute task.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"fork-join-pool"` hoặc `"thread-pool"`
- Không xét custom pool sizing

## Links

- [[../../01_Core/Multithreading/007-thread-pool]]
- [[../../01_Core/Multithreading/008-completable-future]]
- [[009-virtual-threads]]
- [JDK 21 `ForkJoinPool`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinPool.html)
- [JDK 21 `ForkJoinTask`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinTask.html)
