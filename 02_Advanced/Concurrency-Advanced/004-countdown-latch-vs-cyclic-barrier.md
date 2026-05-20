# CountDownLatch vs CyclicBarrier

## What is it

`CountDownLatch` và `CyclicBarrier` đều là **coordination primitive**. Chúng giúp thread chờ nhau theo một rule nào đó.

Điểm rất quan trọng là chúng **không phải lock bảo vệ shared mutable state**. Nói cách khác, chúng giải bài toán **phối hợp tiến độ**, không giải bài toán **mutual exclusion**.

Mental model ngắn:

- `CountDownLatch` là **one-shot gate**.
- `CyclicBarrier` là **meeting point lặp lại theo round**.

```text
CountDownLatch
workers finish --> countDown() --> gate mở --> waiter đi tiếp

CyclicBarrier
worker A arrive
worker B arrive
worker C arrive --> đủ parties --> cả nhóm cùng đi tiếp --> barrier reset
```

## How I used to misunderstand it

Hiểu nhầm phổ biến là gom cả hai vào nhóm “đồng bộ thread”, rồi nghĩ dùng cái nào cũng được miễn code chờ đúng lúc.

Sai ở chỗ shape của bài toán khác hẳn nhau.

- Với `CountDownLatch`, một bên thường phát tín hiệu hoàn tất cho bên chờ.
- Với `CyclicBarrier`, chính các participant phải cùng đến checkpoint rồi cùng đi tiếp.

Hiểu nhầm thứ hai là tưởng coordination primitive nào cũng giúp tránh race condition. Không đúng. Bạn vẫn có thể phối hợp tiến độ rất đẹp nhưng shared state bên dưới vẫn race nếu không có locking hay thiết kế state an toàn.

## How it actually works

Với `CountDownLatch`, bạn khởi tạo count ban đầu. Mỗi lần một công việc xong, ai đó gọi `countDown()`. Thread chờ ở `await()` sẽ chỉ đi tiếp khi count về `0`. Sau khi mở cổng rồi, latch không quay lại trạng thái ban đầu.

Với `CyclicBarrier`, mỗi participant tự gọi `await()` khi tới checkpoint. Khi đủ số parties, barrier mở cho cả nhóm. Sau đó barrier có thể dùng lại cho generation tiếp theo.

### Decision matrix nhanh

| Câu hỏi | `CountDownLatch` | `CyclicBarrier` |
|---|---|---|
| One-shot hay nhiều round | One-shot | Nhiều round |
| Một bên chờ bên khác hoàn tất | Rất hợp | Kém hợp |
| Cùng một nhóm phải gặp nhau theo phase | Không tự nhiên | Rất hợp |
| Reset để dùng lại | Không | Có |

### Coordination khác mutual exclusion ở đâu

| Bạn cần gì | Công cụ gần đúng |
|---|---|
| Chờ N việc hoàn tất rồi đi tiếp | `CountDownLatch` |
| N worker phải đồng bộ ở mỗi phase | `CyclicBarrier` |
| Chỉ một thread được sửa state tại một lúc | lock, không phải latch/barrier |
| Giới hạn số thread vào cùng lúc | `Semaphore`, không phải latch/barrier |

## Code example

```java
import java.util.concurrent.CountDownLatch;

CountDownLatch latch = new CountDownLatch(3);

latch.countDown();
latch.countDown();
latch.countDown();

latch.await();
```

Ở đây `await()` chỉ là chờ count về `0`. Nó không bảo vệ shared state nào cả.

## When to use / when NOT to use

Dùng `CountDownLatch` khi:

- chờ một batch task hoàn thành,
- chờ startup warm-up,
- chờ nhiều signal one-shot trước khi publish trạng thái sẵn sàng.

Dùng `CyclicBarrier` khi:

- cùng một nhóm worker phải đi qua nhiều phase,
- mỗi phase chỉ được qua khi mọi participant đã tới checkpoint,
- bạn cần barrier tái sử dụng theo generation.

Không dùng `CountDownLatch` nếu bài toán cần reset. Không dùng `CyclicBarrier` nếu thực ra chỉ có một bên chờ còn bên kia chỉ cần gửi tín hiệu hoàn tất.

## How this connects to real Java projects

Trong Spring, `CountDownLatch` hay gặp trong integration test, startup orchestration nhỏ, hoặc demo race condition. `CyclicBarrier` hiếm hơn trong business flow thường ngày, nhưng có thể hữu ích trong batch nhiều phase hoặc test đồng bộ worker.

Với production workflow phức tạp, nhiều bài toán coordination nên được đẩy lên message queue, scheduler, hoặc workflow engine thay vì giữ ở tầng thread primitive.

## Gotchas

- `CountDownLatch` không reset. Dùng xong round này là xong.
- `CyclicBarrier` đòi đủ parties. Thiếu một thread là cả nhóm chờ.
- `await()` là blocking call. Gọi sai chỗ có thể làm request thread hoặc pool nhỏ bị nghẽn.
- Chúng không thay cho lock. Coordination đúng không đồng nghĩa shared state bên dưới an toàn.

## Check yourself

- Vì sao `CountDownLatch` và `CyclicBarrier` là coordination primitive chứ không phải mutual exclusion primitive?
- Trong bài toán nhiều phase lặp lại, vì sao `CountDownLatch` thường không hợp?
- Nếu một worker không bao giờ tới barrier, chuyện gì xảy ra với các worker còn lại?
- Khi nào `Semaphore` phù hợp hơn latch hoặc barrier?
- Một flow dùng latch đúng có còn race condition không nếu shared state không được bảo vệ?

## Exercises

### Exercise 1: Choose Coordination Primitive

Độ khó: Easy

Đề bài:
Cho ba cờ `oneShot`, `waitsForAllParties`, `reusedAcrossRounds`. Trả về `"count-down-latch"` nếu đây là coordination one-shot. Nếu cần reuse qua nhiều round và mọi participant phải gặp nhau, trả về `"cyclic-barrier"`.

Ví dụ 1:

Đầu vào:
```text
oneShot = false
waitsForAllParties = true
reusedAcrossRounds = true
```

Đầu ra:
```text
"cyclic-barrier"
```

Giải thích:
Barrier phù hợp cho checkpoint lặp lại qua nhiều vòng.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"count-down-latch"` hoặc `"cyclic-barrier"`
- Không cần mô phỏng thread thật

### Exercise 2: Remaining Latch Count

Độ khó: Easy

Đề bài:
Cho `initialCount` và `completedSignals`. Trả về count còn lại của một latch, nhưng không nhỏ hơn `0`.

Ví dụ 1:

Đầu vào:
```text
initialCount = 5
completedSignals = 3
```

Đầu ra:
```text
2
```

Giải thích:
Ba lần `countDown()` đã giảm latch từ `5` xuống `2`.

Ràng buộc:

- `0 <= initialCount <= 1000000`
- `0 <= completedSignals <= 1000000`
- Kết quả tối thiểu là `0`

### Exercise 3: Count Barrier Generations

Độ khó: Medium

Đề bài:
Cho `parties` và `arrivals`, giả sử mỗi lần đủ `parties` lượt đến thì barrier hoàn thành một generation. Trả về số generation hoàn chỉnh.

Ví dụ 1:

Đầu vào:
```text
parties = 3
arrivals = 8
```

Đầu ra:
```text
2
```

Giải thích:
Có `2` lượt barrier đầy đủ, còn `2` lượt đến chưa đủ cho generation tiếp theo.

Ràng buộc:

- `1 <= parties <= 1000000`
- `0 <= arrivals <= 1000000`
- Chỉ tính generation hoàn chỉnh

## Links

- [[005-semaphore]]
- [[007-fork-join-pool]]
- [[../../01_Core/Multithreading/007-thread-pool]]
- [JDK 21 `CountDownLatch`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CountDownLatch.html)
- [JDK 21 `CyclicBarrier`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CyclicBarrier.html)
- [JDK 21 `Phaser`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Phaser.html)
