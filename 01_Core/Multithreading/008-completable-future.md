# CompletableFuture

## What is it

`CompletableFuture` là API để mô tả async computation có thể hoàn thành sau, rồi chain bước tiếp theo mà không cần block ngay từ đầu.

Bạn có thể transform kết quả, compose bước async tiếp theo, combine nhiều kết quả độc lập, và gắn error handling vào cùng flow.

Mental model: thay vì cầm kết quả ngay bây giờ, bạn cầm một “lời hứa” về kết quả và mô tả việc sẽ làm khi kết quả tới.

## How I used to misunderstand it

Mình từng nghĩ cứ đổi sang `CompletableFuture` là code sẽ nhanh hơn.

Không đúng. Nếu task bên trong vẫn là blocking I/O, vẫn chạy trên pool sai, hoặc bạn `join()` quá sớm, bạn chỉ đang bọc blocking code trong một lớp async-looking syntax.

Hiểu nhầm nữa là xem `CompletableFuture` như chuyện “chạy nền”. Giá trị thật của nó thường nằm ở async composition, chứ không chỉ ở việc chạy task ở thread khác.

## How it actually works

`supplyAsync()` hoặc `runAsync()` bắt đầu một async stage.

Sau đó bạn có thể nối các stage khác nhau tuỳ nhu cầu.

### Composition table

| Method | Dùng khi nào | Kết quả |
|---|---|---|
| `thenApply` | Bước sau chỉ transform giá trị hiện có | `T -> U` |
| `thenCompose` | Bước sau cũng trả về future | flatten `Future<Future<U>>` thành `Future<U>` |
| `thenCombine` | Có hai future độc lập, cần ghép kết quả | combine hai giá trị |
| `allOf` | Cần chờ nhiều future cùng xong | sync point cho nhiều task |
| `exceptionally` | Cần fallback khi fail | phục hồi lỗi |

### Flow dễ nhầm nhất

```text
Future<User>
    // sync transform
    -> thenApply(profile -> format(profile))
    // bước sau trả future khác
    -> thenCompose(id -> fetchOrdersAsync(id))
    // error path
    -> exceptionally(error -> fallbackValue)
```

Điểm cần nhớ là `thenApply` và `thenCompose` khác nhau ở chỗ bước kế tiếp có trả future hay không.

### Decision matrix nhỏ

| Nếu bạn cần... | Chọn gì |
|---|---|
| Đổi `String` thành `Integer` trong cùng flow | `thenApply` |
| Gọi tiếp một async API khác | `thenCompose` |
| Chạy song song 2 lời gọi độc lập rồi ghép lại | `thenCombine` hoặc `allOf` |
| Fallback khi lỗi | `exceptionally` hoặc `handle` |

## Code example

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(4);

        CompletableFuture<String> future = CompletableFuture
                .supplyAsync(() -> "java", executor)
                .thenApply(String::toUpperCase)
                .exceptionally(error -> "FALLBACK");

        // join ở rìa flow, không block sớm ở giữa chain
        System.out.println(future.join());
        executor.shutdown();
    }
}
```

## When to use / when NOT to use

Dùng `CompletableFuture` khi cần compose nhiều async task, chạy song song các I/O độc lập, hoặc mô tả success path và error path trong cùng pipeline.

Không dùng nó chỉ để làm code “trông async” rồi `join()` ngay sau `supplyAsync()`. Khi đó bạn vừa thêm complexity vừa không thu được lợi ích đáng kể.

Nếu workload là blocking I/O, hãy nghĩ cả về executor. Common pool không phải lúc nào cũng là nơi phù hợp cho tác vụ blocking.

## How this connects to real Java projects

Trong Spring, `CompletableFuture` thường đi cùng `@Async`, async service method, HTTP client, messaging, hoặc background orchestration.

Điểm quan trọng là executor phải rõ. Nếu service blocking database hoặc HTTP nhưng lại vô tình chạy trên common pool, bạn có thể tạo contention khó thấy.

Ngoài ra, nếu request thread `join()` quá sớm, phần async của bạn có thể chỉ là hình thức. Request vẫn block như thường.

## Gotchas

- Dùng `thenApply` thay vì `thenCompose` dễ tạo nested future khó dùng.
- Exception trong async chain cần handler rõ, nếu không lỗi rất khó lần theo.
- `join()` quá sớm làm mất lợi ích async.
- Async composition không tự sửa thread pool hoặc downstream bottleneck.

## Check yourself

- `thenApply` và `thenCompose` khác nhau ở điểm nào?
- Khi nào `CompletableFuture` giúp thật, khi nào chỉ làm code phức tạp hơn?
- Vì sao blocking task trong common pool có thể là vấn đề?
- Nếu cần gọi song song hai API độc lập rồi ghép kết quả, bạn sẽ nghĩ tới method nào?
- Tại sao `join()` nên ở rìa flow hơn là ở giữa chain?

## Exercises

### Bài 1: Choose Future Composition Method
Độ khó: Dễ

Đề bài:
Cho biết bước tiếp theo có trả về future hay không, trả về `"thenCompose"` nếu có và `"thenApply"` nếu không.

Ví dụ 1:
Đầu vào:
```text
nextReturnsFuture = true
```

Đầu ra:
```text
"thenCompose"
```

Giải thích:
`thenCompose` flatten một bước tiếp theo có trả về future.

Ràng buộc:
- Input là boolean
- Chỉ trả về `"thenCompose"` hoặc `"thenApply"`
- Không model lựa chọn executor

### Bài 2: Combine Async Results
Độ khó: Dễ

Đề bài:
Cho hai result string, trả về chuỗi được nối bằng `":"`.

Ví dụ 1:
Đầu vào:
```text
left = "user", right = "orders"
```

Đầu ra:
```text
"user:orders"
```

Giải thích:
Điều này mô hình hóa việc combine hai async result độc lập.

Ràng buộc:
- `left` và `right` là non-null
- Preserve cả hai string một cách chính xác
- Dùng `":"` làm separator

### Bài 3: Select Async Fallback
Độ khó: Trung bình

Đề bài:
Cho success value, fallback value, và failure flag, trả về fallback khi failure là `true`; ngược lại trả về success value.

Ví dụ 1:
Đầu vào:
```text
success = "data", fallback = "cached", failed = true
```

Đầu ra:
```text
"cached"
```

Giải thích:
Fallback này biểu diễn một exception handler path.

Ràng buộc:
- `success` và `fallback` là non-null
- `failed` là boolean
- Trả về chính xác một trong hai input string

## Links

- [[001-thread-vs-runnable-vs-executor-service]]
- [[007-thread-pool]]
- [[../../02_Advanced/Concurrency-Advanced/008-reactive-vs-imperative]]
- `CompletableFuture` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html
- `CompletionStage` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletionStage.html
