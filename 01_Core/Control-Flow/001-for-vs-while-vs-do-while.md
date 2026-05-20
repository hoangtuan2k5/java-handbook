# for vs while vs do-while

## What is it

`for`, `while`, và `do-while` đều là cách lặp lại một khối code, nhưng mỗi loại nói một intent khác nhau.

- `for` hợp khi bạn biết rõ vùng lặp hoặc đang duyệt index
- `while` hợp khi vòng lặp phụ thuộc vào một điều kiện thay đổi trong runtime
- `do-while` hợp khi body phải chạy ít nhất một lần trước khi kiểm tra điều kiện

Mental model ngắn gọn:

- `for` là đi qua danh sách có kế hoạch
- `while` là tiếp tục khi điều kiện còn đúng
- `do-while` là làm trước rồi mới hỏi có làm tiếp không

## How I used to misunderstand it

Mình từng nghĩ ba loại loop gần như giống nhau, chỉ khác syntax.

Vì vậy có lúc dùng `while` cho mọi thứ, khiến biến đếm nằm rải rác và dễ quên update.

Cũng dễ nghĩ `do-while` chỉ là biến thể ít dùng của `while`, nhưng khác biệt “chạy ít nhất một lần” rất quan trọng trong menu loop, retry prompt, hoặc logic cần validate sau lần xử lý đầu tiên.

## How it actually works

`for` gom initialization, condition, và update vào một chỗ, nên phù hợp với loop có ranh giới rõ như duyệt array hoặc list theo index.

Enhanced `for` hợp khi chỉ cần đọc từng element và không quan tâm index.

`while` kiểm tra điều kiện trước mỗi lần chạy, nên nếu điều kiện sai ngay từ đầu thì body không chạy lần nào.

`do-while` chạy body trước, rồi mới kiểm tra điều kiện ở cuối vòng.

### Comparison table

| Câu hỏi | `for` | `while` | `do-while` |
|---|---|---|---|
| Số lần lặp có biên rõ | Rất hợp | Dùng được nhưng kém rõ hơn | Thường không phải lựa chọn đầu tiên |
| Điều kiện dừng phụ thuộc runtime | Dùng được | Rất hợp | Chỉ khi cần chạy ít nhất một lần |
| Cần ít nhất một lần chạy body | Không nói rõ điều đó | No | Yes |
| Counter lifecycle nên ở một chỗ | Yes | Thường phân tán hơn | Có thể nhưng ít tự nhiên |

### Decision scaffold

```text
Known range or index traversal? -> for
Keep going until condition changes? -> while
Must run body once before checking? -> do-while
```

Điểm quan trọng không phải loop nào “mạnh hơn”, mà là loop nào diễn đạt đúng lifecycle của dữ liệu.

## Code example

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> names = List.of("Linh", "An", "Binh");

        for (int i = 0; i < names.size(); i++) {
            // index quan trọng
            System.out.println(i + ": " + names.get(i));
        }

        int cursor = 0;
        while (cursor < names.size() && !names.get(cursor).equals("An")) {
            // dừng khi runtime state thay đổi
            cursor++;
        }

        int attempts = 0;
        do {
            // chạy một lần trước khi check xảy ra
            attempts++;
        } while (attempts < 1);
    }
}
```

## When to use / when NOT to use

Dùng `for` khi range, index, hoặc collection traversal là trọng tâm.

Dùng enhanced `for` khi chỉ cần đọc từng element và không mutate collection structure.

Dùng `while` khi điều kiện dừng là “cho đến khi...” như đọc token, tìm sentinel, retry khi chưa thành công.

Dùng `do-while` khi bắt buộc chạy body ít nhất một lần.

Không dùng `while` với counter nếu `for` giúp gom counter lifecycle rõ hơn.

Không dùng `do-while` nếu body không nên chạy khi input ban đầu invalid.

## How this connects to real Java projects

Trong Spring service, loop thường xuất hiện khi validate danh sách request items, transform DTOs, phân trang qua nhiều page, hoặc retry một operation có giới hạn.

Chọn loop đúng giúp tránh bug production như infinite retry, skip item đầu hoặc cuối, hoặc mutate collection sai cách.

Với stream processing trong Spring apps, `for` vẫn đáng dùng khi control flow có nhiều branch, cần return sớm, hoặc cần thông báo lỗi cụ thể hơn một pipeline `Stream` dài.

## Gotchas

- Quên update biến điều kiện trong `while` có thể tạo infinite loop.
- Enhanced `for` không phù hợp nếu cần remove trực tiếp khỏi collection đang duyệt.
- `do-while` luôn chạy ít nhất một lần, kể cả khi điều kiện ban đầu đáng lẽ không cho chạy.
- Đừng chọn loop theo thói quen cá nhân. Hãy chọn loop làm người đọc đoán đúng flow ngay lần đầu.

## Check yourself

- Vì sao `for` thường rõ hơn `while` khi đang duyệt index từ `0` tới `n - 1`?
- Khi nào `while` diễn đạt bài toán tốt hơn `for`?
- `do-while` giải quyết đúng nhu cầu nào mà `while` không nói rõ bằng?
- Nếu body không nên chạy khi dữ liệu ban đầu invalid, vì sao `do-while` nguy hiểm?
- Enhanced `for` mất đi thông tin gì so với `for` theo index?

## Exercises

### Bài 1: Sum Positive Numbers
Độ khó: Dễ

Đề bài:
Cho một list các số nguyên, trả về tổng của tất cả số dương.

Ví dụ 1:
Đầu vào:
```text
numbers = [3, -1, 0, 5]
```

Đầu ra:
```text
8
```

Giải thích:
Chỉ có `3` và `5` là số dương, nên tổng là `8`.

Ràng buộc:
- 0 <= numbers.length <= 100000
- numbers[i] là non-null
- Bỏ qua số `0` và các số âm

### Bài 2: Find First Threshold Index
Độ khó: Trung bình

Đề bài:
Cho một list các giá trị reading dạng số nguyên và một ngưỡng `threshold`, trả về index đầu tiên có giá trị lớn hơn hoặc bằng ngưỡng đó. Trả về `-1` nếu không có reading nào khớp.

Ví dụ 1:
Đầu vào:
```text
readings = [4, 8, 15, 16], threshold = 10
```

Đầu ra:
```text
2
```

Giải thích:
`15` là giá trị đầu tiên lớn hơn hoặc bằng `10`, và index của nó là `2`.

Ràng buộc:
- 0 <= readings.length <= 100000
- readings[i] là non-null
- Chỉ trả về index khớp đầu tiên

### Bài 3: Count Until Stop Word
Độ khó: Trung bình

Đề bài:
Cho các token theo encounter order và một stop word, đếm xem có bao nhiêu token xuất hiện trước stop word đầu tiên. Nếu stop word không bao giờ xuất hiện, hãy đếm toàn bộ token.

Ví dụ 1:
Đầu vào:
```text
tokens = ["start", "load", "stop", "cleanup"], stopWord = "stop"
```

Đầu ra:
```text
2
```

Giải thích:
Có hai token xuất hiện trước token `"stop"` đầu tiên.

Ràng buộc:
- 0 <= tokens.length <= 100000
- tokens[i] là non-null
- stopWord là non-null
- Dừng ở lần khớp chính xác đầu tiên

## Links

- [[002-switch-expression]]
- [[003-break-vs-continue-vs-return]]
- [[004-recursion]]
- Java language guide, control flow statements: https://docs.oracle.com/javase/tutorial/java/nutsandbolts/flow.html
