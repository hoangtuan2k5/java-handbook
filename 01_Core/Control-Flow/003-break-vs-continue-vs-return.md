# break vs continue vs return

## What is it

`break`, `continue`, và `return` đều làm thay đổi luồng chạy bình thường, nhưng phạm vi tác động khác nhau.

- `break` thoát khỏi loop hoặc `switch` gần nhất
- `continue` bỏ qua phần còn lại của vòng lặp hiện tại và chuyển sang lần lặp tiếp theo
- `return` thoát khỏi method hiện tại và trả kết quả nếu method có return type

Mental model:

- `continue` là “bỏ item này”
- `break` là “dừng vòng này”
- `return` là “xong method này”

## How I used to misunderstand it

Mình từng dùng `break` và `return` khá lẫn lộn trong loop.

Khi chỉ muốn dừng scan và trả kết quả, `return` thường rõ hơn vì method kết thúc ngay.

Nhưng khi method còn cleanup hoặc còn logic phía sau loop, `break` mới đúng.

Cũng có lúc dùng `continue` quá nhiều khiến branch chính bị giấu ở cuối loop, trong khi đảo điều kiện hoặc return sớm có thể dễ đọc hơn.

## How it actually works

`continue` chỉ có ý nghĩa bên trong loop. Nó không thoát loop, mà nhảy tới lần kiểm tra điều kiện hoặc update tiếp theo.

`break` kết thúc loop gần nhất, nên với nested loop, nó không tự thoát tất cả loop bên ngoài.

`return` thoát toàn bộ method, bất kể đang ở sâu trong loop hay branch nào.

### Comparison table

| Keyword | Tác động | Hợp khi nào |
|---|---|---|
| `continue` | Bỏ iteration hiện tại | Item hiện tại không đáng xử lý nhưng loop vẫn tiếp tục |
| `break` | Thoát loop gần nhất | Đã đạt mục tiêu hoặc gặp sentinel |
| `return` | Thoát method | Đã có kết quả cuối cùng hoặc lỗi đủ để kết thúc method |

### Decision scaffold

```text
Skip this item only? -> continue
Stop the loop, but method still has work? -> break
Method is done now? -> return
```

## Code example

```java
import java.util.List;

public class Main {
    static String firstInvalid(List<String> tokens) {
        for (String token : tokens) {
            if (token.isBlank()) {
                // token rỗng bị bỏ qua
                continue;
            }
            if (token.equals("STOP")) {
                // dừng scan nhưng vẫn return default bên dưới
                break;
            }
            if (token.length() > 10) {
                // method kết thúc
                return "too long";
            }
        }
        return "ok";
    }
}
```

## When to use / when NOT to use

Dùng `continue` khi một iteration không đáng xử lý nhưng loop vẫn nên chạy tiếp.

Dùng `break` khi loop đã đạt mục tiêu hoặc gặp sentinel nhưng method vẫn còn việc sau loop.

Dùng `return` khi đã có kết quả cuối cùng hoặc lỗi đủ để kết thúc method.

Không lạm dụng nhiều `continue` hoặc `break` trong cùng một loop nếu nó làm người đọc khó theo dõi.

Không dùng `return` sớm nếu cần đảm bảo một đoạn logic phía sau luôn chạy, trừ khi logic đó nằm trong `finally` hoặc được tách rõ.

## How this connects to real Java projects

Trong Spring service, `return` sớm thường giúp validation rõ hơn: request invalid thì trả lỗi hoặc throw exception ngay, không để business logic chạy tiếp.

`continue` hữu ích khi xử lý batch và muốn skip record không đủ điều kiện.

`break` thường xuất hiện khi scan rule list hoặc fallback chain và dừng ở rule đầu tiên match.

Chọn đúng keyword giúp service method tránh nested `if` quá sâu.

## Gotchas

- `break` chỉ thoát loop gần nhất, không thoát toàn bộ nested loops.
- `continue` trong loop có counter thủ công dễ làm quên update nếu update nằm sau nó.
- `return` trong `try` vẫn chạy `finally`, nên side effect trong `finally` có thể gây bất ngờ.
- Nhiều `continue` liên tiếp có thể là dấu hiệu loop đang ôm quá nhiều trách nhiệm.

## Handbook rule

- Skip iteration không cần xử lý dùng `continue`; loop đã đạt mục tiêu dùng `break`; method đã có kết quả dùng `return`.
- `break` chỉ thoát loop gần nhất; nested loop muốn thoát hết phải tách method hoặc dùng labeled break có chủ ý.
- `return` trong `try` vẫn chạy `finally`; tính cả side effect ở đó trước khi quyết định.
- Nhiều `continue`/`break` lồng nhau là dấu hiệu loop đang ôm nhiều trách nhiệm; tách method.
- Không dùng `return` sớm khi logic phía sau bắt buộc chạy, trừ khi đã đặt trong `finally` hoặc helper rõ.

## Check yourself

- Khi nào `return` rõ hơn `break`?
- Khi nào `break` đúng hơn `return` dù cả hai đều có thể “dừng sớm”?
- `continue` khác gì với việc đặt toàn bộ logic còn lại trong một `if` lớn?
- Trong nested loop, `break` dừng được đến mức nào?
- Nếu có nhiều `continue` khiến loop khó đọc, bạn nên nghi ngờ điều gì về thiết kế hiện tại?

## Exercises

### Bài 1: Filter Valid Scores
Độ khó: Dễ

Đề bài:
Cho một list các score, chỉ trả về những score nằm trong đoạn từ `0` đến `100` inclusive, đồng thời giữ nguyên encounter order.

Ví dụ 1:
Đầu vào:
```text
scores = [80, -1, 100, 120, 60]
```

Đầu ra:
```text
[80, 100, 60]
```

Giải thích:
`-1` và `120` bị bỏ qua vì chúng nằm ngoài valid score range.

Ràng buộc:
- 0 <= scores.length <= 100000
- scores[i] là non-null
- Giữ nguyên thứ tự của các valid score

### Bài 2: Find First Invalid Token
Độ khó: Trung bình

Đề bài:
Cho một list token, trả về token đầu tiên có chứa dấu cách. Dừng việc scan nếu sentinel token `"END"` xuất hiện trước. Trả về empty string nếu không tìm thấy invalid token nào trước sentinel.

Ví dụ 1:
Đầu vào:
```text
tokens = ["user", "role admin", "END"]
```

Đầu ra:
```text
"role admin"
```

Giải thích:
Token thứ hai chứa dấu cách trước khi sentinel xuất hiện.

Ràng buộc:
- 0 <= tokens.length <= 100000
- tokens[i] là non-null
- So khớp sentinel có phân biệt chữ hoa/thường

### Bài 3: Return Early Validation Message
Độ khó: Trung bình

Đề bài:
Cho `email`, `password`, và `age`, trả về validation error message đầu tiên. Trả về `"ok"` khi tất cả giá trị đều hợp lệ. Email phải chứa `@`, password phải có độ dài ít nhất `8`, và age phải ít nhất là `18`.

Ví dụ 1:
Đầu vào:
```text
email = "a@test.com", password = "short", age = 20
```

Đầu ra:
```text
"password too short"
```

Giải thích:
Email hợp lệ, nhưng password fail trước khi tới bước check age.

Ràng buộc:
- email là non-null
- password là non-null
- age >= 0
- Chỉ trả về lỗi đầu tiên

## Links

- [[001-for-vs-while-vs-do-while]]
- [[002-switch-expression]]
- [[004-recursion]]
- Java language guide, branch statements: https://docs.oracle.com/javase/tutorial/java/nutsandbolts/branch.html

