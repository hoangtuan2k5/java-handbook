# ArrayList vs LinkedList

## What is it

`ArrayList` và `LinkedList` đều implement `List`, nhưng chúng tối ưu cho hai kiểu chi phí khác nhau.

- `ArrayList` dùng dynamic array.
- `LinkedList` dùng các node nối với nhau.

Hình dung nhanh:

- `ArrayList` giống dãy ghế đánh số trong rạp, nhảy tới ghế số 50 rất nhanh.
- `LinkedList` giống đoàn tàu, muốn tới toa số 50 phải đi qua các toa trước đó.

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là câu: “insert và delete nhiều thì dùng `LinkedList`”.

Câu này thiếu mất nửa sau: bạn chèn và xoá ở đâu, và bạn tới vị trí đó bằng cách nào. Nếu phải tìm vị trí bằng index trước, `LinkedList` vẫn phải traverse từng node. Trong code Java thực tế, workload thường là append, iterate, serialize, sort, hoặc random access. Những việc đó thường hợp với `ArrayList` hơn.

`LinkedList` cũng không hề “rẻ”. Mỗi phần tử đi kèm node object và các reference phụ, nên nó tốn memory hơn và ít thân thiện với CPU cache hơn.

## How it actually works

`ArrayList` giữ phần tử trong một array liên tục. Khi capacity đầy, nó tạo array lớn hơn rồi copy reference sang chỗ mới. Vì vậy `add()` ở cuối thường nhanh theo nghĩa amortized, còn `get(index)` là O(1).

Giá phải trả là khi chèn hoặc xoá ở giữa, các phần tử phía sau có thể phải shift.

`LinkedList` giữ mỗi phần tử trong một node riêng. `addFirst()` và `addLast()` tốt vì chỉ cần đổi vài reference ở đầu hoặc cuối. Nhưng `get(index)` là O(n) vì phải đi qua node trước đó.

Trong application code bình thường, `ArrayList` là default an toàn hơn. `LinkedList` chỉ đáng cân nhắc khi thao tác ở hai đầu là requirement thật, và ngay cả lúc đó `ArrayDeque` thường vẫn là lựa chọn thực tế hơn nếu bạn không cần `List` semantics.

### Comparison table

| Concern | `ArrayList` | `LinkedList` |
|---|---|---|
| Random access `get(index)` | Rất hợp | Kém |
| Append cuối list | Rất hợp | Hợp |
| Insert/remove ở giữa theo index | Phải shift | Phải traverse tới vị trí trước |
| Thao tác ở đầu list | Không lý tưởng | Tốt |
| Memory overhead | Thấp hơn | Cao hơn |
| Default choice cho app code | Thường đúng | Hiếm khi là default tốt |

### Decision shortcut

```text
Need a general-purpose List?      -> ArrayList
Need deque behavior at both ends? -> Think ArrayDeque first
Need LinkedList specifically?     -> Only if node-based behavior is truly the point
```

## Code example

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<String> arrayList = new ArrayList<>();
        arrayList.add("A");
        arrayList.add("B");
        arrayList.add("C");
        // nhanh vì index map trực tiếp tới array slot
        System.out.println(arrayList.get(2));

        LinkedList<String> linkedList = new LinkedList<>();
        linkedList.add("A");
        linkedList.add("B");
        // hiệu quả vì LinkedList có thể relink head node
        linkedList.addFirst("Start");

        // vẫn chạy, nhưng cần traversal trước khi tìm thấy value
        System.out.println(linkedList.get(2));
    }
}
```

## When to use / when NOT to use

Dùng `ArrayList` làm default cho danh sách cần iterate, append, sort, random access, hoặc trả kết quả query.

Dùng `LinkedList` khi bạn thật sự cần thao tác đầu hoặc cuối theo kiểu deque, nhưng trước khi chọn nó, hãy hỏi xem `ArrayDeque` có hợp hơn không.

Không chọn `LinkedList` chỉ vì nghe câu “insert hoặc delete là O(1)”. Trong rất nhiều code thật, bạn vẫn phải trả chi phí traverse để tới đúng chỗ.

## How this connects to real Java projects

Trong Spring Boot, repository results, request DTO arrays, validation errors, và response lists gần như luôn hợp với `ArrayList` vì dữ liệu thường được iterate hoặc serialize tuần tự.

`LinkedList` hiếm khi là lựa chọn tốt cho API payload hoặc service result thông thường. Nếu bạn thấy nó trong service layer, câu hỏi nên là: trường hợp sử dụng này có thật sự cần thao tác hai đầu không, hay chỉ đang dùng sai default.

## Gotchas

- `LinkedList#get(index)` không nhanh. Nó phải traverse node.
- `ArrayList` resize làm copy array, nhưng chi phí trung bình của append vẫn tốt trong hầu hết trường hợp.
- `LinkedList` tốn memory hơn vì mỗi element cần node object và references phụ.

## Handbook rule

- Default cho `List` trong app code là `ArrayList`; chỉ đổi khi có lý do cụ thể.
- `LinkedList` không phải “rẻ”; nó hơn `ArrayList` chỉ ở thao tác đầu/cuối thật sự.
- Khi cần queue hoặc stack non-concurrent, nghĩ tới `ArrayDeque` trước `LinkedList`.
- `get(index)` trên `LinkedList` là O(n); đừng coi nó như random access.
- API response và repository result mặc định là `ArrayList` để iterate và serialize tuần tự.

## Check yourself

- Nếu operation chính là `get(5000)`, implementation nào hợp hơn, và vì sao?
- Vì sao câu “xoá nhiều thì dùng `LinkedList`” dễ dẫn tới lựa chọn sai?
- Nếu bạn cần queue hoặc stack local trong code mới, vì sao `ArrayDeque` thường đáng nghĩ tới trước `LinkedList`?
- Trong API response hoặc repository result, vì sao `ArrayList` thường là default tốt hơn?
- `LinkedList` có thể mạnh ở đầu và cuối, nhưng điểm yếu lớn nhất của nó trong app code thường là gì?

## Exercises

### Bài 1: Get Item By Index
Độ khó: Dễ

Đề bài:
Cho một list các string và một index, trả về item tại vị trí đó. Hãy chọn list implementation phù hợp nhất cho random access thường xuyên.

Ví dụ 1:
Đầu vào:
```text
items = ["A", "B", "C"], index = 2
```

Đầu ra:
```text
"C"
```

Giải thích:
Index-based lookup là thao tác chính, nên array-backed access là mental model phù hợp.

Ràng buộc:
- 1 <= items.length <= 10^5
- 0 <= index < items.length
- items[i] là non-null

### Bài 2: Process Actions From Front
Độ khó: Trung bình

Đề bài:
Cho một sequence các action, liên tục thêm các urgent action mới vào đầu và xử lý từ đầu danh sách. Hãy chọn một implementation làm rõ front operation.

Ví dụ 1:
Đầu vào:
```text
initial = ["B", "C"], urgent = ["A"]
```

Đầu ra:
```text
["A", "B", "C"]
```

Giải thích:
Urgent item phải được chèn vào đầu trước khi bắt đầu xử lý.

Ràng buộc:
- 0 <= initial.length <= 10^5
- 0 <= urgent.length <= 10^4
- Giữ nguyên processing order sau các lần chèn ở đầu

### Bài 3: Review Repository Result Storage
Độ khó: Khó

Đề bài:
Một service lưu 100.000 database row trong `LinkedList` trước khi serialize chúng sang JSON. Hãy chỉ ra các vấn đề khả dĩ về performance và memory, rồi đề xuất collection phù hợp hơn.

Ví dụ 1:
Đầu vào:
```text
rows = 100000, operations = ["iterate", "serialize"]
```

Đầu ra:
```text
"Prefer ArrayList"
```

Giải thích:
Workload chủ yếu là sequential iteration và serialization, nên node overhead của `LinkedList` gần như không đem lại lợi ích.

Ràng buộc:
- rows >= 0
- Operation chủ yếu là append và sequential iteration
- Không có yêu cầu frequent front insertion/removal

## Links

- [[001-List-vs-Set-vs-Map]]
- [[008-Queue-vs-Deque-vs-Stack]]
- `ArrayList` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html
- `LinkedList` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html
