# Deadlock

## What is it

Deadlock là tình huống nhiều thread chờ nhau mãi, vì mỗi thread đang giữ một resource và đợi resource mà thread khác giữ.

Mental model rất thực dụng: mỗi người đang cầm một chìa khóa, nhưng ai cũng cần chìa khóa của người kia trước khi đi tiếp. Kết quả là cả hai đứng im mãi.

## How I used to misunderstand it

Mình từng nghĩ deadlock là bug hiếm, chỉ xuất hiện ở hệ thống concurrency rất phức tạp.

Thực tế chỉ cần hai lock và lock ordering không nhất quán là đủ. Thậm chí code trông vô hại như “service A gọi service B trong lúc giữ lock” cũng có thể tạo vòng chờ.

Hiểu nhầm khác là chỉ nghĩ tới Java lock. Trên thực tế deadlock còn có thể xảy ra với database row lock, transaction, distributed lock, hoặc callback ngoài tầm kiểm soát.

## How it actually works

Deadlock cổ điển thường gắn với bốn điều kiện, còn gọi là Coffman conditions.

### Coffman conditions

| Điều kiện | Ý nghĩa ngắn | Vì sao quan trọng |
|---|---|---|
| Mutual exclusion | resource không chia sẻ cùng lúc được | lock chỉ cho một thread giữ |
| Hold and wait | đang giữ cái này, vẫn đòi thêm cái khác | thread không chịu nhả lock đang có |
| No preemption | không ai ép lấy lock lại được | phải tự release |
| Circular wait | A chờ B, B chờ C, rồi quay lại chờ A | tạo vòng kẹt |

Chỉ cần phá một điều kiện, deadlock có thể tránh được.

### Vòng chờ hay gặp nhất

```text
Thread A holds Lock 1, waits for Lock 2
Thread B holds Lock 2, waits for Lock 1

=> neither side can move
```

### Cách nghĩ thực tế hơn lý thuyết

| Tình huống | Nguy cơ | Thói quen an toàn hơn |
|---|---|---|
| Acquire nhiều lock | Circular wait | Lock ordering nhất quán |
| Giữ lock khi gọi external code | Callback có thể lấy lock khác | Release sớm, gọi ngoài lock |
| Transaction dài | DB lock giữ lâu | Thu nhỏ critical section |
| Nested service call đang giữ monitor | Vòng chờ khó thấy | Rõ ownership và call graph |

## Code example

```java
class TransferService {
    void move(Object firstLock, Object secondLock) {
        synchronized (firstLock) {
            synchronized (secondLock) {
                System.out.println("always acquire locks in one fixed order");
            }
        }
    }
}
```

Đoạn code trên chưa chứng minh deadlock, nhưng nó nhấn mạnh nguyên tắc quan trọng nhất, nếu đã cần hai lock thì hãy khóa theo cùng một thứ tự ở mọi nơi.

## When to use / when NOT to use

Khi cần nhiều lock, hãy định nghĩa lock ordering cố định và bám chặt vào nó.

Giữ critical section ngắn, tránh gọi code bên ngoài trong lúc đang giữ lock, và ưu tiên thiết kế giảm lock nesting.

Không dựa vào hi vọng rằng “đoạn này chắc không bị gọi cùng lúc đâu”. Deadlock thường không ồn ào. Nó chỉ làm app đứng yên và buộc bạn đọc thread dump lúc xấu nhất.

## How this connects to real Java projects

Trong Spring, deadlock có thể xuất hiện ở nhiều lớp khác nhau:

- Java monitor hoặc `Lock`,
- database transaction và row lock,
- scheduled job tranh cùng resource,
- singleton service giữ lock rồi gọi bean khác,
- cache hoặc listener callback chạm lại cùng state.

Vì vậy nếu một request, job, hoặc batch bỗng treo mãi không exception, deadlock luôn là một nghi phạm nghiêm túc.

## Gotchas

- Deadlock thường không ném exception, app chỉ treo.
- Lock ordering không nhất quán là nguyên nhân phổ biến nhất.
- Giữ lock trong lúc gọi code ngoài tầm kiểm soát làm nguy cơ tăng mạnh.
- Test đơn giản có thể không lộ deadlock, production load mới lộ.

## Check yourself

- Trong bốn Coffman conditions, điều kiện nào bạn có thể chủ động phá dễ nhất trong code app?
- Vì sao lock ordering nhất quán giúp tránh deadlock?
- Tại sao giữ lock rồi gọi bean khác hoặc external API là đáng lo?
- Deadlock khác starvation ở điểm nào nếu nhìn từ triệu chứng “thread chờ mãi”?
- Khi app treo mà CPU không cao, bạn sẽ nghĩ tới loại bug nào?

## Exercises

### Bài 1: Detect Opposite Lock Order
Độ khó: Trung bình

Đề bài:
Cho hai lock order string như `"A>B"`, trả về `true` nếu chúng biểu diễn thứ tự ngược nhau cho cùng hai lock.

Ví dụ 1:
Đầu vào:
```text
firstOrder = "A>B", secondOrder = "B>A"
```

Đầu ra:
```text
true
```

Giải thích:
Hai path acquire cùng các lock theo thứ tự ngược nhau.

Ràng buộc:
- `firstOrder` và `secondOrder` là non-null
- Mỗi order chứa đúng một ký tự `>`
- Lock name không chứa `>`

### Bài 2: Has Circular Wait
Độ khó: Trung bình

Đề bài:
Cho các wait pair ở dạng `"A->B"`, trả về `true` nếu tồn tại two-thread circular wait.

Ví dụ 1:
Đầu vào:
```text
waits = ["A->B", "B->A"]
```

Đầu ra:
```text
true
```

Giải thích:
Thread A chờ B trong khi B chờ A.

Ràng buộc:
- `waits` là non-null
- `0 <= waits.length <= 100000`
- Chỉ detect two-node cycle

### Bài 3: Detect External Call While Locked
Độ khó: Dễ

Đề bài:
Cho biết code có đang giữ lock hay không và có gọi external code trước khi release lock đó hay không, trả về `true` khi deadlock risk nên bị gắn cờ.

Ví dụ 1:
Đầu vào:
```text
holdsLock = true, callsExternalCode = true
```

Đầu ra:
```text
true
```

Giải thích:
Gọi code mà bạn không kiểm soát trong khi đang giữ lock có thể tạo circular-wait risk.

Ràng buộc:
- Input là boolean
- Chỉ trả về `true` khi cả hai input đều là `true`
- Không phân tích lock ordering trong bài này

## Links

- [[002-race-condition]]
- [[003-synchronized]]
- [[../../02_Advanced/Concurrency-Advanced/001-reentrant-lock-vs-synchronized]]
- `ThreadMXBean` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/ThreadMXBean.html
- Java Tutorials, Deadlock: https://docs.oracle.com/javase/tutorial/essential/concurrency/deadlock.html
