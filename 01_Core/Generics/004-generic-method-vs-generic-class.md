# Generic Method vs Generic Class

## What is it

Generic class có type parameter ở level class, như `Box<T>`.

Generic method có type parameter chỉ ở level method, như `<T> T first(List<T> items)`.

Mental model gọn nhất:

- generic class dùng khi type là **một phần identity lâu dài của object**
- generic method dùng khi type chỉ cần cho **một operation cụ thể**

## How I used to misunderstand it

Mình từng thấy một method cần generic là tiện tay biến cả class thành generic luôn.

Cách đó thường làm abstraction rộng quá mức. Ở chiều ngược lại, có những class rõ ràng giữ typed state lâu dài, nhưng lại cố nhét generic vào từng method riêng lẻ. Câu hỏi đúng không phải là “có dùng generic không”, mà là “type parameter này nên sống ở scope nào”.

## How it actually works

Nếu object giữ state typed như `Repository<User>` hay `Box<String>`, generic class là hợp lý vì type là một phần identity của instance.

Nếu class bản thân không cần giữ state typed, nhưng một operation cần type flexibility như `first`, `identity`, `swap`, generic method sẽ nhẹ hơn và đúng scope hơn.

### So sánh nhanh

| Câu hỏi | Generic class | Generic method |
|---|---|---|
| Type parameter sống bao lâu | suốt lifecycle của object | chỉ trong một lần gọi method |
| Khi nào hợp | object giữ state typed | operation cần linh hoạt type |
| Ví dụ | `Box<T>` | `<T> T first(...)` |

### Decision shortcut

```text
Does the object itself own typed state? -> generic class
Does only one operation need type flexibility? -> generic method
```

Chọn sai level thường làm API hoặc quá cồng kềnh, hoặc quá rối để đọc.

## Code example

```java
import java.util.List;

class Box<T> {
    private final T value;

    Box(T value) {
        this.value = value;
    }

    T getValue() {
        return value;
    }
}

public class Main {
    static <T> T first(List<T> items) {
        return items.get(0);
    }
}
```

Ở đây `Box<T>` cần generic vì instance giữ `value` có type ổn định. `first(...)` chỉ cần generic ở một operation, nên generic method là đủ.

## When to use / when NOT to use

Dùng generic class khi instance thật sự mang typed state ổn định trong suốt lifecycle của nó.

Dùng generic method khi chỉ một operation cần type flexibility.

Không nên generic hóa cả class chỉ vì một helper method cần generic. Ngược lại, nếu nhiều method trong class cùng phụ thuộc vào một typed state, generic class sẽ tự nhiên hơn nhiều method generic rời rạc.

## How this connects to real Java projects

Trong Spring Boot, generic class xuất hiện ở repository, wrapper, handler abstraction, typed adapter. Generic method hay xuất hiện ở utility class, mapper helper, validation helper, common response builder.

Hiểu khác biệt này giúp bạn thiết kế abstraction nhẹ hơn, và cũng giúp đọc framework API nhanh hơn vì bạn biết type parameter đang sống ở mức nào.

## Gotchas

- Generic class quá sớm dễ làm abstraction lan rộng không cần thiết.
- Generic method quá nhiều trong class giữ state typed có thể làm API rối.
- Khi cả class và method đều generic, đặt tên type parameter phải đủ rõ để người đọc không lẫn scope.
- Đừng nhầm “generic hơn” với “thiết kế tốt hơn”.

## Handbook rule

- Generic class khi state instance phụ thuộc một type ổn định suốt lifecycle.
- Generic method khi chỉ một operation cần linh hoạt type, không cần generic cả class.
- Khi cả class và method đều generic, đặt tên type parameter rõ để không lẫn scope.
- Không generic hóa class quá sớm; chờ thấy ít nhất hai client thật sự cần.
- Bound type parameter dựa trên API tối thiểu cần dùng; tránh bound dư.

## Check yourself

- Type parameter của generic class và generic method khác nhau ở scope nào?
- Khi nào biến cả class thành generic là overkill?
- Vì sao `Box<T>` hợp là generic class còn `first(List<T>)` hợp là generic method?
- Nếu object không giữ typed state, vì sao generic method thường là lựa chọn nhẹ hơn?
- Khi class và method đều generic, điều gì làm người đọc dễ rối nhất?

## Exercises

### Bài 1: Build Generic Identity Method
Độ khó: Dễ

Đề bài:
Cho một giá trị kiểu `T`, trả về chính giá trị đó bằng một generic method.

Ví dụ 1:
Đầu vào:
```text
value = "java"
```

Đầu ra:
```text
"java"
```

Giải thích:
Method giữ nguyên cùng type và cùng giá trị.

Ràng buộc:
- value là non-null
- Chỉ dùng method-level generic parameter
- Trả về đúng chính giá trị đó

### Bài 2: Store Value In Generic Class
Độ khó: Trung bình

Đề bài:
Cho một giá trị kiểu `T`, trả về một generic holder object lưu trữ và sau đó expose đúng giá trị đó.

Ví dụ 1:
Đầu vào:
```text
value = 10
```

Đầu ra:
```text
Holder<Integer>(10)
```

Giải thích:
Type parameter thuộc về chính holder instance.

Ràng buộc:
- value là non-null
- Không dùng raw type holder
- Giữ nguyên original type

### Bài 3: Choose Generic Scope For Operation
Độ khó: Trung bình

Đề bài:
Cho một flag `stateful`, trả về `"class"` nếu type parameter nên sống ở cấp class, và `"method"` nếu nó chỉ nên sống ở một operation.

Ví dụ 1:
Đầu vào:
```text
stateful = false
```

Đầu ra:
```text
"method"
```

Giải thích:
Khi type information chỉ cần cho một operation, method scope là đủ.

Ràng buộc:
- Input chỉ là một boolean flag
- Chỉ trả về `"class"` hoặc `"method"`
- Dùng đúng rule đã nêu trong đề bài

## Links

- [[001-what-is-T]]
- [[002-wildcard]]
- [[003-type-erasure]]
- Java tutorial, Generic Methods: https://docs.oracle.com/javase/tutorial/java/generics/methods.html
- `Collections` utility class, useful example of generic methods: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html
