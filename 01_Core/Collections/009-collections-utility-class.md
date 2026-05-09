# Collections Utility Class

## What is it

`Collections` là utility class chứa các static methods để thao tác với collection: sort, reverse, shuffle, copy, tạo unmodifiable view, synchronized wrapper, empty hoặc singleton collection.

Nó không phải là `Collection` interface.

Hãy nghĩ `Collection` là loại container, còn `Collections` là hộp dụng cụ để xử lý container đó.

## How I used to misunderstand it

Nhiều dev nhầm `Collection` và `Collections` vì tên quá giống. Sai khác rất lớn:

- `Collection` là interface cha của `List` và `Set`.
- `Collections` là class tiện ích.

Một hiểu nhầm khác là `Collections.unmodifiableList(list)` tạo immutable copy. Thực tế nó thường tạo unmodifiable view bọc list gốc. Nếu list gốc bị sửa ở chỗ khác, view vẫn phản ánh thay đổi.

Cũng dễ dùng `Collections.synchronizedList()` rồi tưởng mọi iteration đã an toàn. Không đúng. Iteration vẫn cần lock thủ công quanh toàn bộ block.

## How it actually works

Các method của `Collections` thường chia thành vài nhóm:

- nhóm algorithm như `sort()`, `binarySearch()`, `reverse()`, `shuffle()`,
- nhóm factory hoặc view như `emptyList()`, `singletonList()`, `unmodifiableList()`,
- nhóm synchronized wrapper như `synchronizedList()`.

Điểm quan trọng là nhiều method không tạo deep copy.

- `unmodifiableList()` chỉ chặn mutation qua wrapper.
- `synchronizedList()` đồng bộ từng method call, không tự bảo vệ cả một multi-step iteration.
- `sort()` mutate list tại chỗ.

### Wrapper and factory matrix

| API | Returns | Source changes reflected later? | Caller can mutate through returned reference? |
|---|---|---|---|
| `Collections.emptyList()` | shared empty list | Không có source | Không |
| `Collections.singletonList(x)` | one-element immutable-style list | Không có source | Không |
| `Collections.unmodifiableList(list)` | read-only view | Có | Không |
| `Collections.synchronizedList(list)` | synchronized wrapper | Có | Có, nhưng từng method một |

## Code example

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>(List.of("Linh", "An"));
        // wrapper blocks mutation through this reference only
        List<String> readOnly = Collections.unmodifiableList(names);

        // mutating the original list still changes what the wrapper sees
        names.add("Binh");
        System.out.println(readOnly); // [Linh, An, Binh]

        List<String> sorted = new ArrayList<>(names);
        // mutates the target list
        Collections.sort(sorted);
        System.out.println(sorted);
    }
}
```

## When to use / when NOT to use

Dùng `Collections` khi cần utility nhanh, rõ intent: `emptyList()` thay vì trả `null`, `sort()` khi cần sort in-place, `unmodifiableList()` khi muốn public read-only view.

Không dùng `unmodifiableList()` nếu bạn cần immutable snapshot. Khi đó hãy tạo copy trước hoặc dùng `List.copyOf()`.

Không dùng synchronized wrappers như giải pháp concurrency mặc định cho code phức tạp. Concurrent collections hoặc thiết kế immutable thường rõ hơn.

## How this connects to Spring

Trong Spring Boot, `Collections.emptyList()` rất hữu ích để trả response không `null`. `unmodifiableList()` có thể bảo vệ config hoặc cached data khỏi bị caller sửa qua reference public, nhưng nếu source list vẫn mutable trong bean thì nó chưa thật sự immutable.

Với singleton bean, synchronized wrapper có thể tránh một số race đơn giản, nhưng không thay thế được thiết kế thread-safe rõ ràng.

## Gotchas

- `Collections.unmodifiableList()` là unmodifiable view, không nhất thiết là immutable copy.
- `Collections.sort(list)` mutate list tại chỗ. Copy trước nếu cần giữ order gốc.
- `Collections.synchronizedList()` cần manual synchronization khi iterate qua nhiều bước.

## Check yourself

- `Collection` và `Collections` khác nhau ở cấp độ khái niệm nào?
- Vì sao `unmodifiableList()` chưa chắc đã là snapshot an toàn?
- Nếu caller không được sửa kết quả nhưng source list vẫn còn bị sửa nội bộ, bạn đang trả về loại guarantee nào?
- Khi nào `Collections.emptyList()` tốt hơn trả `null`?
- Vì sao synchronized wrapper vẫn chưa đủ cho multi-step iteration?

## Exercises

### Bài 1: Return Empty Result Safely
Độ khó: Dễ

Đề bài:
Hãy refactor một method đang trả về `null` khi không có item nào, để nó trả về empty list thay thế.

Ví dụ 1:
Đầu vào:
```text
items = []
```

Đầu ra:
```text
[]
```

Giải thích:
Trả về empty collection giúp caller không phải xử lý `null` riêng biệt.

Ràng buộc:
- Return value không bao giờ được là `null`
- Caller phải iterate an toàn mà không cần branch riêng cho `null`

### Bài 2: Unmodifiable View Behavior
Độ khó: Trung bình

Đề bài:
Cho một mutable list và một unmodifiable view của nó, xác định xem view sẽ chứa gì sau khi list gốc bị modify.

Ví dụ 1:
Đầu vào:
```text
original = ["A", "B"], view = unmodifiableList(original), then original.add("C")
```

Đầu ra:
```text
["A", "B", "C"]
```

Giải thích:
Unmodifiable wrapper chặn việc mutate qua view, nhưng vẫn phản ánh thay đổi trong list gốc.

Ràng buộc:
- List gốc vẫn còn reachable
- Không tạo immutable snapshot

### Bài 3: Immutable Snapshot For Config
Độ khó: Trung bình

Đề bài:
Cho một mutable config list, trả về một read-only snapshot sẽ không thay đổi ngay cả khi list gốc bị modify về sau.

Ví dụ 1:
Đầu vào:
```text
config = ["A", "B"], snapshot created, then config.add("C")
```

Đầu ra:
```text
snapshot = ["A", "B"]
```

Giải thích:
Snapshot có nội dung riêng của nó và không phản ánh các mutation xảy ra sau này trên source.

Ràng buộc:
- Source list có thể bị modify sau khi tạo snapshot
- Snapshot phải từ chối các mutation attempt

### Bài 4: Review Synchronized List Iteration
Độ khó: Khó

Đề bài:
Một shared list được bọc bằng `Collections.synchronizedList()`, rồi bị iterate mà không synchronize thủ công toàn bộ iteration block. Hãy giải thích rủi ro và cách sửa.

Ví dụ 1:
Đầu vào:
```text
list = synchronizedList([...]), thread A iterates, thread B modifies
```

Đầu ra:
```text
Iteration có thể quan sát thấy state không nhất quán hoặc fail
```

Giải thích:
Wrapper chỉ synchronize từng method call riêng lẻ, không synchronize toàn bộ multi-step iteration.

Ràng buộc:
- Nhiều thread có thể truy cập list
- Iteration phải được bảo vệ như một critical section hoàn chỉnh

## Links

- [[001-list-vs-set-vs-map]]
- [[../../00_Mental-Models/007-Immutability]]
- [[010-concurrent-hash-map-vs-hash-map]]
- `Collections` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html
- `List.copyOf()` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#copyOf(java.util.Collection)
