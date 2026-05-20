# Iterator

## What is it

`Iterator` là cách Java cho bạn đi qua collection mà không cần biết bên dưới là array, linked nodes, hash table hay tree.

Nó đưa ra ba ý chính:

- còn phần tử không,
- lấy phần tử tiếp theo,
- và trong một số trường hợp, xoá phần tử hiện tại một cách an toàn.

Hãy nghĩ iterator như người dẫn đường trong kho. Bạn không cần biết kho sắp xếp thế nào. Bạn chỉ hỏi: còn món tiếp theo không.

## How I used to misunderstand it

Nhiều dev nghĩ enhanced for-loop là một cơ chế riêng. Thật ra `for (String item : items)` thường được compiler dịch thành iterator nếu object là `Iterable`.

Hiểu nhầm khác là có thể vừa iterate vừa gọi `list.remove(item)` trực tiếp. Với nhiều collection fail-fast, cách đó dễ ném `ConcurrentModificationException` vì collection bị sửa ngoài iterator.

Cũng dễ nhầm `ConcurrentModificationException` là lúc nào cũng liên quan tới nhiều thread. Không đúng. Một thread tự sửa collection sai cách cũng đủ gây lỗi.

## How it actually works

Collection như `ArrayList` giữ một modification count nội bộ. Iterator được tạo ra sẽ ghi nhớ count tại thời điểm đó.

Nếu collection bị structural modification ngoài iterator, iterator phát hiện count lệch và fail-fast bằng `ConcurrentModificationException`.

Mục tiêu không phải bảo vệ thread-safety tuyệt đối. Mục tiêu là fail sớm thay vì tiếp tục iterate trên trạng thái không còn đáng tin.

`Iterator#remove()` là ngoại lệ có kiểm soát. Nó xoá phần tử vừa được trả về bởi chính iterator, đồng thời cập nhật trạng thái iterator để tiếp tục hợp lệ.

Với collection concurrent như `ConcurrentHashMap`, iterator có behavior khác. Nó thường weakly consistent, không fail-fast như `ArrayList`.

### Fail-fast timeline

```text
create iterator -> iterator remembers modCount
modify collection outside iterator -> modCount changes
iterator continues -> may throw ConcurrentModificationException
```

## Code example

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>(List.of("Linh", "An", "Binh"));

        Iterator<String> iterator = names.iterator();
        while (iterator.hasNext()) {
            // iterator sở hữu traversal state
            String name = iterator.next();
            if (name.equals("An")) {
                // an toàn vì remove đi qua cùng iterator
                iterator.remove();
            }
        }

        System.out.println(names); // [Linh, Binh]
    }
}
```

## When to use / when NOT to use

Dùng `Iterator` trực tiếp khi bạn cần xoá có điều kiện trong lúc duyệt, khi viết custom collection, hoặc khi muốn hiểu enhanced for-loop đang thực sự làm gì bên dưới.

Dùng `removeIf()` nếu logic xoá đơn giản, vì nó ngắn hơn và diễn đạt ý tốt hơn.

Không dùng iterator như một giải pháp thread-safety. Nếu nhiều thread cùng đọc hoặc ghi, bạn cần collection concurrent hoặc synchronization rõ ràng.

## How this connects to real Java projects

Trong Spring Boot, bạn thường iterate qua list DTO, entity, validation errors, hoặc bean definitions. Nếu service vừa enhanced for-loop vừa remove trực tiếp khỏi collection, lỗi `ConcurrentModificationException` có thể xuất hiện dù chỉ có một request.

Với shared collection trong singleton bean, vấn đề còn nghiêm trọng hơn vì iterator fail-fast không thay thế được thread-safety.

## Gotchas

- `ConcurrentModificationException` không nhất thiết là bug multi-thread. Một thread sửa collection sai cách cũng gây ra.
- `Iterator#remove()` chỉ được gọi sau `next()` và thường chỉ một lần cho mỗi phần tử vừa lấy.
- Fail-fast là best-effort để phát hiện bug sớm, không phải cơ chế đồng bộ hoá đáng tin.

## Handbook rule

- Không gọi `Collection#remove(...)` bên trong enhanced for-loop; dùng `Iterator#remove()` hoặc `removeIf()`.
- `ConcurrentModificationException` là dấu hiệu code đang sai, không phải bug multi-thread.
- Fail-fast iterator là best-effort, không phải cơ chế thread-safety.
- Khi remove theo predicate đơn giản, ưu tiên `removeIf()` cho intent rõ.
- Multi-thread đụng cùng collection cần concurrent collection hoặc synchronization rõ ràng, không phải iterator.

## Check yourself

- Enhanced for-loop trên `List` thường đang dùng cơ chế nào ở bên dưới?
- Vì sao `list.remove(...)` bên trong enhanced for-loop dễ gây lỗi, dù chỉ có một thread?
- Khi nào `Iterator#remove()` là cách đúng?
- Vì sao fail-fast không đồng nghĩa với thread-safe?
- Nếu bạn chỉ muốn xoá item theo predicate đơn giản, API nào thường rõ hơn iterator thủ công?

## Exercises

### Bài 1: Remove Even Numbers
Độ khó: Dễ

Đề bài:
Cho một mutable list số nguyên, hãy remove toàn bộ số chẵn trong khi vẫn iterating một cách an toàn.

Ví dụ 1:
Đầu vào:
```text
numbers = [1, 2, 3, 4, 5]
```

Đầu ra:
```text
[1, 3, 5]
```

Giải thích:
`2` và `4` là số chẵn và phải được remove mà không gây concurrent modification error.

Ràng buộc:
- 0 <= numbers.length <= 10^5
- Modify list ngay trên chính object ban đầu
- Không tạo list thứ hai để chứa kết quả cuối

### Bài 2: Sửa việc remove không an toàn
Độ khó: Trung bình

Đề bài:
Cho đoạn code remove item khỏi một list bên trong enhanced for-loop. Hãy viết lại nó bằng cách tiếp cận an toàn.

Ví dụ 1:
Đầu vào:
```text
names = ["Linh", "An", "Binh"], removeName = "An"
```

Đầu ra:
```text
["Linh", "Binh"]
```

Giải thích:
Việc remove qua collection trong lúc enhanced for-loop đang iterate có thể fail. Hãy dùng iterator removal hoặc `removeIf()`.

Ràng buộc:
- 0 <= names.length <= 10^5
- names[i] là non-null
- Remove toàn bộ name khớp điều kiện

### Bài 3: Giải thích fail-fast behavior
Độ khó: Khó

Đề bài:
Cho một list và một iterator được tạo từ list đó, sau đó list bị structurally modified trước khi iterator kết thúc. Hãy giải thích điều gì xảy ra và vì sao.

Ví dụ 1:
Đầu vào:
```text
list = ["A", "B"], iterator = list.iterator(), then list.add("C"), then iterator.next()
```

Đầu ra:
```text
Có thể ném `ConcurrentModificationException`
```

Giải thích:
Iterator phát hiện collection modification count đã thay đổi từ bên ngoài iterator.

Ràng buộc:
- Giả sử list là `ArrayList`
- Structural modification nghĩa là thao tác add hoặc remove, không phải thay thế value đang có

## Links

- [[001-list-vs-set-vs-map]]
- [[010-concurrent-hash-map-vs-hash-map]]
- [[../Functional/003-Stream-API]]
- `Iterator` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Iterator.html
- `Iterable` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Iterable.html
