# Memory Model (JMM)

## What is it

`Java Memory Model`, thường viết là `JMM`, là bộ quy tắc nói về `visibility`, `ordering`, và giới hạn reordering giữa các thread. Nó trả lời câu hỏi quan trọng hơn cả syntax concurrency:

“Thread này viết gì, thread kia được phép thấy gì, và thấy theo thứ tự nào?”

Nếu không có mental model này, concurrent code rất dễ chạy đúng do may mắn timing, không phải do có guarantee thật.

## How I used to misunderstand it

Mình từng nghĩ chỉ cần một thread ghi giá trị mới, thread khác sớm muộn gì cũng thấy.

Thực tế nếu không có `happens-before` relationship, thread khác có thể thấy giá trị cũ, thấy state chưa nhất quán, hoặc thấy reference đã publish nhưng object bên trong chưa được publish an toàn.

Mình cũng từng dễ lẫn ba khái niệm:

- `visibility`,
- `atomicity`,
- `mutual exclusion`.

Trong JMM, ba thứ này liên quan nhưng không đồng nghĩa.

## How it actually works

Trục chính của note này là `happens-before`.

Nếu một action A happens-before action B, thì thread ở B phải nhìn thấy effect của A theo rule của JMM.

Các cạnh happens-before thường gặp:

| Cơ chế | Bạn nhận được gì |
| --- | --- |
| Ghi rồi đọc cùng một `volatile` field | Visibility và ordering quanh field đó |
| Unlock rồi lock cùng monitor | Visibility giữa critical sections |
| Thread start/join rules | Một số guarantee lifecycle giữa threads |
| Publish object đúng cách qua final/volatile/lock | Giảm risk thấy state nửa khởi tạo |

Sơ đồ ngắn rất đáng nhớ:

```text
Thread A: write data
Thread A: volatile flag = true
                   |
                   | happens-before
                   v
Thread B: if (flag) read data
```

Nếu cạnh happens-before tồn tại, thread B có cơ sở để thấy state nhất quán hơn. Nếu không có cạnh này, compiler, JIT, và CPU có thể reorder hoặc giữ value ở cache/register theo cách vẫn hợp lệ với JMM nhưng phá giả định trực giác của lập trình viên.

Contrast quan trọng nhất:

| Bạn cần gì | `volatile` | `Atomic*` | `synchronized` |
| --- | --- | --- | --- |
| Visibility của một field | Có | Có | Có |
| `count++` an toàn | Không | Có, nếu dùng operation phù hợp | Có |
| Bảo vệ invariant nhiều field | Không | Thường không | Có |
| Mutual exclusion | Không | Không | Có |

`final` field cũng có vai trò riêng. Nếu object được xây xong hoàn chỉnh rồi mới publish đúng cách, thread khác có cơ hội nhìn thấy state khởi tạo đáng tin hơn. Nhưng nếu reference bị leak ngay trong constructor hoặc publish bừa, guarantee đó có thể không cứu được bạn.

## Code example

```java
class StopSignal {
    private volatile boolean stopped;

    void stop() {
        stopped = true;
    }

    boolean isStopped() {
        return stopped;
    }
}
```

Ở đây `volatile` hợp lý vì bài toán chỉ cần publish một cờ đơn giản.

Nhưng nếu logic là `if (!stopped) { count++; stopped = true; }` thì câu chuyện đã khác. Lúc đó bạn không chỉ cần visibility, mà còn cần atomicity và có thể cả locking.

## When to use / when NOT to use

Hãy dùng mental model này khi:

- code có shared mutable state,
- chọn giữa `volatile`, `Atomic*`, `synchronized`, lock, hoặc immutable design,
- debug race condition khó tái hiện,
- review singleton hoặc cache dùng chung giữa nhiều request thread.

Không nên dùng `volatile` như thuốc chữa bách bệnh. Nếu operation là read-modify-write hoặc nhiều field phải đi cùng nhau, visibility đơn thuần là chưa đủ.

## How this connects to real Java projects

Phần lớn Spring bean nên stateless. Nhưng ngay khi một singleton bean giữ mutable state dùng chung giữa các request, bạn bước vào territory của `JMM`.

Các bug hay gặp:

- in-memory counter tự viết,
- lazy init không an toàn,
- cache hoặc map mutate không đồng bộ,
- background scheduler và request thread cùng chạm một state,
- object publish cho nhiều thread nhưng không có happens-before rõ ràng.

## Gotchas

- `volatile` không làm `x++` thành atomic.
- Publish sai cách có thể làm thread khác thấy object nửa khởi tạo.
- Thêm log hoặc debug có thể làm race condition “biến mất” vì timing đổi.
- Code chạy đúng nhiều lần không chứng minh nó đúng theo JMM.

## Check yourself

- `happens-before` thực sự cho bạn guarantee gì?
- Vì sao `visibility` khác `atomicity`?
- Khi nào `volatile` là đủ đẹp, và khi nào chắc chắn chưa đủ?
- Vì sao object có thể bị thấy ở trạng thái nửa khởi tạo?
- Nếu một Spring singleton giữ mutable state dùng chung, câu hỏi JMM đầu tiên bạn nên hỏi là gì?

## Exercises

### Exercise 1: Classify JMM Requirement

Độ khó: Easy

Đề bài:
Cho `String scenario`. Hãy trả về `"visibility"` nếu scenario là `"stop-flag"`, `"atomicity"` nếu scenario là `"counter-increment"`, `"ordering"` nếu scenario là `"publish-after-init"`, ngược lại trả về `"none"`.

Ví dụ 1:

Đầu vào:
```text
scenario = "counter-increment"
```

Đầu ra:
```text
"atomicity"
```

Giải thích:
Tăng biến đếm chia sẻ cần nhiều hơn visibility đơn thuần.

Ràng buộc:

- `scenario` là chuỗi không rỗng
- So sánh chính xác theo các nhãn đã nêu
- Nếu không khớp, trả về `"none"`

### Exercise 2: Count Unsafe Publication Cases

Độ khó: Medium

Đề bài:
Cho `boolean[] publishedWithoutHappensBefore`. Mỗi phần tử `true` nghĩa là một object reference được publish mà không có `happens-before` rõ ràng. Hãy đếm số case unsafe.

Ví dụ 1:

Đầu vào:
```text
publishedWithoutHappensBefore = [false, true, true, false]
```

Đầu ra:
```text
2
```

Giải thích:
Có hai object bị publish theo cách không an toàn.

Ràng buộc:

- `0 <= publishedWithoutHappensBefore.length <= 100000`
- Mỗi phần tử là boolean
- Kết quả là số lượng `true`

### Exercise 3: Detect Race Condition

Độ khó: Medium

Đề bài:
Cho ba cờ `hasSharedMutableState`, `hasConcurrentAccess`, và `hasHappensBeforeEdge`. Hãy trả về `true` nếu có risk race condition, tức là hai cờ đầu là `true` và cờ cuối là `false`. Ngược lại trả về `false`.

Ví dụ 1:

Đầu vào:
```text
hasSharedMutableState = true
hasConcurrentAccess = true
hasHappensBeforeEdge = false
```

Đầu ra:
```text
true
```

Giải thích:
State mutable được truy cập đồng thời mà không có edge đồng bộ hóa thích hợp.

Ràng buộc:

- Mỗi input là boolean
- Chỉ cần phát hiện risk theo rule đơn giản đã nêu
- Không cần mô phỏng thread scheduler

## Links

- [[../../01_Core/Multithreading/002-race-condition]]
- [[../../01_Core/Multithreading/003-synchronized]]
- [[../../01_Core/Multithreading/004-volatile]]
- [[../../01_Core/Multithreading/005-atomic-classes]]
- [JLS Chapter 17, Threads and Locks](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html)
- [JLS §17.4, Memory Model](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.4)
- [JLS §17.4.5, Happens-before Order](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.4.5)
