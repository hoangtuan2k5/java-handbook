# Recursion

## What is it

Recursion là kỹ thuật một method gọi lại chính nó để giải bài toán bằng phiên bản nhỏ hơn của cùng bài toán.

Một recursive solution cần ít nhất hai phần:

- base case để dừng
- recursive case để tiến gần về base case

Mental model: mỗi lời gọi là một stack frame mới giữ phần việc hiện tại. Khi gặp base case, các frame lần lượt trả kết quả ngược lại.

## How I used to misunderstand it

Mình từng nghĩ recursion là “vòng lặp kiểu khó hơn”.

Cách hiểu đó thiếu phần quan trọng nhất: recursion dựa vào call stack và cần thiết kế trạng thái truyền qua từng lời gọi.

Nếu không có base case đúng, recursion không chỉ chạy mãi mà còn làm đầy stack và ném `StackOverflowError`.

Cũng dễ quên rằng Java không đảm bảo tail-call optimization, nên recursion sâu không phải lúc nào cũng an toàn.

## How it actually works

Khi method gọi lại chính nó, JVM tạo một stack frame mới giống như gọi method bình thường.

Frame cũ tạm chờ frame mới trả kết quả.

Base case là điểm không gọi tiếp nữa.

Recursive case phải giảm kích thước bài toán, ví dụ giảm index, cắt chuỗi ngắn hơn, hoặc đi xuống node con trong tree.

### Recursion checklist

| Câu hỏi | Vì sao quan trọng |
|---|---|
| Base case là gì | Nếu không có, recursion không dừng |
| Mỗi bước có tiến gần base case không | Nếu không, bạn chỉ đang lặp vô hạn |
| Độ sâu tối đa có an toàn không | Java stack có giới hạn |
| Loop có đơn giản hơn không | Không phải bài toán nào cũng đáng dùng recursion |

### Flow scaffold

```text
solve(problem)
  if base case -> return answer
  reduce problem
  call solve(smaller problem)
  combine result
```

Recursion hợp với dữ liệu tự nhiên có cấu trúc phân cấp như tree, nested folders, expression parser, hoặc bài toán chia nhỏ rõ ràng.

Với array hoặc list dài tuyến tính, loop thường an toàn hơn trong Java vì không tiêu thụ stack theo số phần tử.

## Code example

```java
public class Main {
    static int sumDigits(int number) {
        int value = Math.abs(number);
        if (value < 10) {
            // base case
            return value;
        }
        // bài toán nhỏ hơn
        return value % 10 + sumDigits(value / 10);
    }

    public static void main(String[] args) {
        System.out.println(sumDigits(472)); // 13
    }
}
```

## When to use / when NOT to use

Dùng recursion khi bài toán tự nhiên chia thành cùng bài toán nhỏ hơn, đặc biệt với tree hoặc cấu trúc nested.

Dùng loop khi dữ liệu tuyến tính lớn và độ sâu có thể vượt stack an toàn.

Không dùng recursion chỉ để “code ngắn hơn” nếu người đọc phải tốn nhiều công hơn để hiểu state.

Không dùng recursion khi base case không rõ hoặc input có thể tạo độ sâu rất lớn mà không kiểm soát.

## How this connects to real Java projects

Trong Spring apps, recursion có thể xuất hiện khi xử lý category tree, menu hierarchy, organization chart, comment threads, hoặc folder-like data.

Khi dữ liệu đến từ database, recursion cần cẩn thận với N+1 queries và cycle trong graph.

Với API nhận nested JSON, recursion giúp validate cấu trúc con, nhưng cần giới hạn depth để tránh input độc hại làm tràn stack hoặc tốn tài nguyên.

## Gotchas

- Thiếu base case hoặc không tiến gần base case sẽ gây `StackOverflowError`.
- Java không đảm bảo tail-call optimization, nên tail recursion vẫn có thể làm sâu stack.
- Recursion trên graph có cycle cần visited set, nếu không sẽ lặp vô hạn.
- Recursion đẹp trên bảng trắng chưa chắc là lựa chọn tốt trong production nếu input depth không kiểm soát.

## Check yourself

- Base case đóng vai trò gì trong recursion?
- Vì sao “gọi lại chính mình” chưa đủ để thành recursive solution tốt?
- Khi nào loop là lựa chọn an toàn hơn recursion trong Java?
- Tail recursion có luôn an toàn trong Java không?
- Nếu data structure có cycle, recursion cần thêm cơ chế gì?

## Exercises

### Bài 1: Sum Digits Recursively
Độ khó: Dễ

Đề bài:
Cho một số nguyên không âm, trả về tổng các chữ số thập phân của nó bằng recursive thinking.

Ví dụ 1:
Đầu vào:
```text
number = 472
```

Đầu ra:
```text
13
```

Giải thích:
`4 + 7 + 2 = 13`.

Ràng buộc:
- 0 <= number <= 2147483647
- Base case phải xử lý số có một chữ số

### Bài 2: Validate Palindrome Recursive
Độ khó: Trung bình

Đề bài:
Cho một string, trả về `true` nếu đọc xuôi và đọc ngược giống nhau bằng cách check boundary đệ quy.

Ví dụ 1:
Đầu vào:
```text
text = "level"
```

Đầu ra:
```text
true
```

Giải thích:
Ký tự đầu và cuối khớp nhau, sau đó inner substring cũng khớp.

Ràng buộc:
- text là non-null
- 0 <= text.length <= 100000
- So khớp có phân biệt chữ hoa/thường

### Bài 3: Count Nested Parentheses Depth
Độ khó: Trung bình

Đề bài:
Cho một string chứa dấu ngoặc đơn và các ký tự khác, trả về độ sâu lồng nhau lớn nhất của balanced parentheses. Trả về `-1` nếu dấu ngoặc không cân bằng.

Ví dụ 1:
Đầu vào:
```text
text = "a(b(c)d)e"
```

Đầu ra:
```text
2
```

Giải thích:
Cặp lồng nhau sâu nhất là `(c)` nằm bên trong `(b(c)d)`.

Ràng buộc:
- text là non-null
- 0 <= text.length <= 100000
- Bỏ qua các ký tự không phải parenthesis
- Trả về `-1` cho input không cân bằng

## Links

- [[001-for-vs-while-vs-do-while]]
- [[003-break-vs-continue-vs-return]]
- [[../Exception/004-custom-exception]]
- Java language guide, recursive methods: https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html

