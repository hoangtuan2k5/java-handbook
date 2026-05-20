# Template Method

## What is it

`Template Method` là behavioral pattern định nghĩa sườn của một workflow trong class cha, còn một số bước con sẽ do subclass override.

Nó giải quyết bài toán nhiều quy trình có cùng thứ tự tổng quát nhưng khác chi tiết ở vài bước. Thứ tự lớn được giữ ổn định, còn variation được cắm vào đúng chỗ đã định.

Điểm học quan trọng là pattern này kiểm soát sequencing. Không phải chỉ có abstract method là thành `Template Method`. Cốt lõi là class cha giữ quyền quyết định các bước chạy theo thứ tự nào.

### Quick distinction

| Câu hỏi | `Template Method` trả lời thế nào |
|---|---|
| Workflow lớn có cố định thứ tự không? | Có |
| Phần nào được thay đổi? | Một số bước con |
| Variation dựa trên gì? | Inheritance và override |
| Lợi ích chính | Giữ invariant của workflow ở một chỗ |
| Giá phải trả | Coupling theo inheritance, ít linh hoạt hơn composition |

## How I used to misunderstand it

Mình từng nghĩ `Template Method` chỉ là “abstract class có vài method abstract”. Ý đó chưa sai, nhưng chưa đủ.

Nếu subclass muốn thay thứ tự workflow hoặc bỏ qua các bước quan trọng mà class cha không kiểm soát được, thì pattern đã mất giá trị. Phần cốt lõi là class cha nói rõ đâu là khung không được phá.

## How it actually works

Class cha có một method như `run()` hoặc `process()` chứa flow cố định. Bên trong nó gọi các bước con, có bước concrete, có bước abstract, có bước hook tùy chọn override.

Subclass không thay toàn bộ thuật toán. Nó chỉ điền phần được cho phép thay.

```text
template method
   -> step 1 cố định
   -> step 2 cho phép override
   -> step 3 hook tùy chọn
   -> step 4 cố định hoặc abstract
```

### Decision matrix

| Tình huống | `Template Method` hợp không? | Vì sao |
|---|---|---|
| Workflow lớn có thứ tự cố định | Hợp | giữ sequencing ở class cha |
| Chỉ có một thuật toán đơn lẻ thay đổi | Ít hợp | `Strategy` gọn hơn |
| Muốn phối hợp nhiều behavior độc lập theo composition | Ít hợp | inheritance kém linh hoạt |
| Nhiều subclass cùng chia sẻ invariant chung | Hợp | tránh copy workflow |
| Thứ tự bước thay đổi mạnh theo từng use case | Thường không | template sẽ bị kéo căng quá mức |

## Code example

```java
abstract class ImportJob {
    public final void run(String rawData) {
        String parsed = parse(rawData);
        validate(parsed);
        persist(parsed);
    }

    protected abstract String parse(String rawData);

    protected void validate(String parsed) {
    }

    protected abstract void persist(String parsed);
}
```

`run()` là template method. Nó khóa thứ tự parse rồi validate rồi persist. Subclass chỉ thay cách làm từng bước, không được đổi trật tự lớn của workflow.

## When to use / when NOT to use

Use `Template Method` khi:

- có nhiều quy trình cùng skeleton nhưng khác chi tiết ở vài bước
- muốn giữ invariant ở một chỗ thay vì copy-paste workflow
- class cha thực sự có quyền định nghĩa thứ tự đúng của business flow

Do NOT use `Template Method` khi:

- thứ tự các bước cũng cần thay đổi mạnh
- muốn phối hợp behavior bằng composition thay vì inheritance
- variation chỉ là chọn một trong nhiều thuật toán độc lập

Misconception thường gặp là dùng `Template Method` cho mọi reusable flow. Nếu reuse đến từ composition tự nhiên hơn, ép vào inheritance sẽ làm cây class cứng và khó bảo trì.

## How this connects to real Java projects

Spring có nhiều API mang tinh thần template như `JdbcTemplate`, `RestTemplate`, `TransactionTemplate`. Không phải lúc nào chúng cũng hiện thực đúng bằng inheritance pattern cổ điển, nhưng idea chung vẫn là framework giữ workflow chuẩn, còn caller chỉ cung cấp phần biến thiên.

Đây là một bài học hay: tên `template` trong Java ecosystem đôi khi chỉ ý tưởng “khung chuẩn”, không nhất thiết luôn là `Template Method` theo UML sách mẫu.

## Gotchas

- Inheritance quá sâu làm flow khó lần theo.
- Hook method mặc định rỗng có thể che việc một bước quan trọng bị bỏ qua.
- Nếu subclass phải override quá nhiều để né logic cha, base abstraction đang sai.
- Thêm một bước mới vào abstract base class có thể ảnh hưởng hàng loạt subclass.
- Dùng template khi variation thực ra độc lập sẽ làm code cứng hơn mức cần thiết.

## Check yourself

- `Template Method` khác `Strategy` ở chỗ class cha giữ quyền gì?
- Khi nào hook method là hữu ích, và khi nào nó nguy hiểm?
- Vì sao inheritance là lợi thế nhưng cũng là cost của pattern này?
- Một workflow mà thứ tự bước thay đổi theo use case có còn hợp với `Template Method` không?
- Dấu hiệu nào cho thấy base class đang ôm sai abstraction?

## Exercises

### Exercise 1: Should Use Template Method

Độ khó: Easy

Đề bài:
Cho `sharedWorkflow`, `varyingStepImplementation`, và `independentAlgorithms`. Hãy trả về `true` chỉ khi tồn tại một workflow dùng chung, các step implementation có thay đổi, và bài toán này không phù hợp hơn với cách mô hình hóa thành các algorithm hoàn toàn độc lập.

Ví dụ 1:

Đầu vào:
```text
sharedWorkflow = true, varyingStepImplementation = true, independentAlgorithms = false
```

Đầu ra:
```text
true
```

Giải thích:
Đây chính là kiểu scenario mà một fixed workflow với các bước có thể thay đổi là rất phù hợp.

Ràng buộc:

- Tất cả input đều là boolean
- Trả về một boolean
- Không cần xét các type đặc thù của framework

### Exercise 2: Find First Failed Template Step

Độ khó: Easy

Đề bài:
Cho hai array `stepNames` và `stepResults` có cùng độ dài. Hãy trả về tên của step đầu tiên có kết quả là `false`. Nếu mọi step đều thành công, trả về `"NONE"`.

Ví dụ 1:

Đầu vào:
```text
stepNames = ["parse", "validate", "persist"]
stepResults = [true, false, false]
```

Đầu ra:
```text
"validate"
```

Giải thích:
Template không còn ở trạng thái tốt ngay tại step đầu tiên bị fail.

Ràng buộc:

- Cả hai array đều là non-null
- Cả hai array có cùng độ dài
- Độ dài array nằm trong đoạn từ 0 đến 100000

### Exercise 3: Build Import Template Summary

Độ khó: Medium

Đề bài:
Cho `sourceName`, `parsed`, `validated`, và `persisted`. Hãy trả về một string theo đúng format `"<sourceName>:PARSE=<status>,VALIDATE=<status>,PERSIST=<status>"`, trong đó mỗi status là `OK` hoặc `FAIL`.

Ví dụ 1:

Đầu vào:
```text
sourceName = "orders.csv", parsed = true, validated = false, persisted = false
```

Đầu ra:
```text
"orders.csv:PARSE=OK,VALIDATE=FAIL,PERSIST=FAIL"
```

Giải thích:
Summary này phản ánh từng fixed template step theo đúng thứ tự.

Ràng buộc:

- `sourceName` là non-null
- Output format phải khớp chính xác
- Mỗi status chỉ phụ thuộc vào boolean input tương ứng của nó

## Links

- [[002-Strategy]]
- [[../Structural/004-Facade]]
- [Spring Framework, JDBC with JdbcTemplate](https://docs.spring.io/spring-framework/reference/data-access/jdbc/core.html)
- [Refactoring Guru, Template Method](https://refactoring.guru/design-patterns/template-method)
- [Design Patterns, Template Method](https://sourcemaking.com/design_patterns/template_method)
