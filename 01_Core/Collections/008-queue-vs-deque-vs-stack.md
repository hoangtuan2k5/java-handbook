# Queue vs Deque vs Stack

## What is it

`Queue`, `Deque`, và `Stack` đều mô tả cách lấy phần tử ra theo một kỷ luật nhất định.

- `Queue` thường là FIFO, vào trước ra trước.
- `Deque` là double-ended queue, thêm và xoá được ở cả hai đầu.
- `Stack` là LIFO, vào sau ra trước.

Nhưng có một chi tiết quan trọng trong Java hiện đại: `Stack` class cũ thường không nên dùng. `Deque`, đặc biệt là `ArrayDeque`, thường là lựa chọn tốt hơn cho stack behavior.

## How I used to misunderstand it

Nhiều dev thấy cần LIFO là dùng ngay `java.util.Stack`.

Vấn đề là `Stack` là class legacy, synchronized theo kiểu cũ và kế thừa `Vector`, nên API và performance không còn là default tốt cho code mới.

Cũng dễ nhầm `Queue` chỉ có một implementation, trong khi `LinkedList`, `ArrayDeque`, và `PriorityQueue` đều có thể liên quan tới queue nhưng behavior rất khác.

`PriorityQueue`, chẳng hạn, không FIFO. Nó lấy phần tử theo priority hoặc ordering.

## How it actually works

`Queue` interface có hai nhóm method cho cùng một operation:

- `add()`, `remove()`, `element()` ném exception khi fail.
- `offer()`, `poll()`, `peek()` trả về giá trị đặc biệt như `false` hoặc `null`.

Trong application code, `offer()`, `poll()`, `peek()` thường dễ xử lý hơn vì không biến empty queue thành exception flow.

`Deque` mở rộng queue bằng hai đầu: `addFirst()`, `addLast()`, `pollFirst()`, `pollLast()`.

Với `ArrayDeque`, dữ liệu được giữ trong circular array, nên thao tác ở hai đầu thường nhanh và ít overhead hơn `LinkedList`. Vì vậy Java docs thường khuyên dùng `ArrayDeque` thay cho `Stack` khi cần stack behavior.

### Comparison table

| Structure | Rule | Good mental model | Default modern implementation |
|---|---|---|---|
| `Queue` | FIFO | jobs, BFS, first-in-first-out processing | `ArrayDeque` nếu local và non-concurrent |
| `Deque` | Hai đầu | add và remove ở cả front lẫn back | `ArrayDeque` |
| `Stack` | LIFO | undo, call stack style behavior | Dùng `Deque`, không dùng legacy `Stack` |

### Decision shortcut

```text
Need FIFO?                 -> Queue
Need both ends?            -> Deque
Need LIFO in modern Java?  -> Deque with push/pop
```

## Code example

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Queue<String> queue = new ArrayDeque<>();
        queue.offer("first");
        queue.offer("second");
        System.out.println(queue.poll()); // first

        Deque<String> stack = new ArrayDeque<>();
        stack.push("first");
        stack.push("second");
        System.out.println(stack.pop()); // second

        Deque<String> deque = new ArrayDeque<>();
        deque.addFirst("front");
        deque.addLast("back");
        System.out.println(deque.pollLast()); // back
    }
}
```

## When to use / when NOT to use

Dùng `Queue` khi cần xử lý FIFO như jobs, events, hoặc breadth-first search.

Dùng `Deque` khi cần cả hai đầu hoặc stack behavior hiện đại.

Dùng `ArrayDeque` làm default cho stack hoặc queue local, non-concurrent.

Không dùng `Stack` cho code mới trừ khi đang làm việc với API legacy.

Không dùng `PriorityQueue` nếu bạn cần FIFO thật sự. Nó phục vụ priority ordering, không phải arrival order.

## How this connects to Spring

Trong Spring Boot, queue mental model xuất hiện trong xử lý events, retry jobs, batch processing, hoặc buffer tạm trong service.

Nếu collection nằm trong singleton bean và nhiều request cùng truy cập, `ArrayDeque` không đủ thread-safe. Bạn cần concurrent queue hoặc synchronization rõ ràng.

Với async processing thật, thường nên dùng message broker hoặc task executor thay vì tự giữ queue mutable trong memory.

## Gotchas

- `Stack` là legacy. Dùng `Deque` và `ArrayDeque` cho stack behavior trong code mới.
- `PriorityQueue` không giữ FIFO. Nó lấy phần tử nhỏ nhất hoặc theo comparator trước.
- `ArrayDeque` không cho phép `null`, vì `null` được dùng làm signal cho `poll()` khi rỗng.

## Check yourself

- Nếu requirement là undo, vì sao `Deque` hợp hơn legacy `Stack`?
- Khi nào `Queue` và `PriorityQueue` cho kết quả khác nhau hoàn toàn?
- Nếu cần xử lý item ở cả đầu và cuối, vì sao `Deque` diễn đạt intent rõ hơn `List`?
- Trong Spring singleton bean, vì sao `ArrayDeque` local và `ArrayDeque` shared là hai câu chuyện khác nhau?
- Khi nào nên dùng `offer()` và `poll()` thay vì `add()` và `remove()`?

## Exercises

### Bài 1: Implement Undo Stack
Độ khó: Dễ

Đề bài:
Cho một list các action, push chúng vào một stack-like structure và trả về action cuối cùng cần undo.

Ví dụ 1:
Đầu vào:
```text
actions = ["type", "delete", "paste"]
```

Đầu ra:
```text
"paste"
```

Giải thích:
Undo dùng hành vi LIFO, nên action mới nhất sẽ được trả về trước.

Ràng buộc:
- 1 <= actions.length <= 10^5
- actions[i] là non-null
- Do not use legacy `Stack`

### Bài 2: First Job In Queue
Độ khó: Dễ

Đề bài:
Cho các job được submit theo thứ tự, trả về job đầu tiên cần được xử lý theo quy tắc FIFO.

Ví dụ 1:
Đầu vào:
```text
jobs = ["job-1", "job-2", "job-3"]
```

Đầu ra:
```text
"job-1"
```

Giải thích:
FIFO processing sẽ lấy job được submit sớm nhất trước.

Ràng buộc:
- 1 <= jobs.length <= 10^5
- jobs[i] là non-null
- Giữ nguyên submission order

### Bài 3: Breadth-First Traversal
Độ khó: Trung bình

Đề bài:
Cho một graph dưới dạng adjacency list và một start node, trả về các node theo thứ tự breadth-first traversal.

Ví dụ 1:
Đầu vào:
```text
graph = {1: [2, 3], 2: [4], 3: [], 4: []}, start = 1
```

Đầu ra:
```text
[1, 2, 3, 4]
```

Giải thích:
Một queue xử lý toàn bộ node ở cùng khoảng cách hiện tại trước khi đi sâu hơn.

Ràng buộc:
- 0 <= number of nodes <= 10^5
- Graph may contain cycles
- Mỗi node chỉ được visit nhiều nhất một lần

### Bài 4: Review In-Memory Task Queue
Độ khó: Khó

Đề bài:
Một Spring singleton service lưu pending task trong `ArrayDeque`. Hãy review design này và liệt kê các rủi ro liên quan tới concurrency, memory growth, và application restart.

Ví dụ 1:
Đầu vào:
```text
serviceScope = "singleton", structure = "ArrayDeque", writers = "multiple request threads"
```

Đầu ra:
```text
"Unsafe without synchronization or a concurrent queue"
```

Giải thích:
`ArrayDeque` không thread-safe, và queue trong memory sẽ mất dữ liệu khi restart.

Ràng buộc:
- Nhiều request thread có thể cùng truy cập queue
- Task không được mất silently
- Queue growth phải được giới hạn hoặc theo dõi

## Links

- [[002-array-list-vs-linked-list]]
- [[004-iterator]]
- [[010-concurrent-hash-map-vs-hash-map]]
- `Queue` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Queue.html
- `Deque` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html
- `ArrayDeque` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html
