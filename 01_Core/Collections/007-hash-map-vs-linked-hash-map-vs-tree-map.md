# HashMap vs LinkedHashMap vs TreeMap

## What is it

`HashMap`, `LinkedHashMap`, và `TreeMap` đều lưu key-value, nhưng khác nhau ở ordering contract và cost model.

- `HashMap` tối ưu lookup nhanh và không hứa thứ tự iteration.
- `LinkedHashMap` giữ insertion order, hoặc access order nếu bật chế độ đó.
- `TreeMap` giữ key sorted theo natural ordering hoặc `Comparator`.

Nếu `Map` là danh bạ, thì `HashMap` là tra nhanh không quan tâm thứ tự, `LinkedHashMap` là nhớ thứ tự thêm vào, còn `TreeMap` là luôn xếp theo alphabet của key.

## How I used to misunderstand it

Hiểu nhầm phổ biến là `HashMap` in ra theo thứ tự nào đó thì có thể dựa vào. Không nên. Order đó là chi tiết implementation, không phải contract.

`TreeMap` cũng hay bị dùng chỉ vì muốn output sorted, mà quên rằng toàn bộ operation sau đó đều đi qua ordering cost và comparator rule.

Một nhầm lẫn khác là xem `LinkedHashMap` chỉ như map “có order”, trong khi access-order mode còn có thể dùng để build LRU cache nhỏ.

## How it actually works

`HashMap` dùng hash table: key hash tới bucket, rồi `equals()` xác nhận đúng key. Nó nhanh trung bình cho `put()` và `get()`, nhưng iteration order không ổn định theo contract.

`LinkedHashMap` kế thừa ý tưởng hash table nhưng thêm linked list nối entries, nên iteration đi theo insertion order mặc định. Nếu bật access order, mỗi lần `get()` entry có thể được đưa về cuối. Đó là lý do nó hay được dùng cho LRU cache nhỏ bằng `removeEldestEntry()`.

`TreeMap` dùng tree sorted theo key. Nó không cần `hashCode()`, nhưng key phải comparable hoặc có comparator. Lookup và insert thường O(log n), đổi lại bạn có sorted key view và range query như `subMap()`, `headMap()`, `tailMap()`.

### Comparison table

| Need | `HashMap` | `LinkedHashMap` | `TreeMap` |
|---|---|---|---|
| Fast default lookup | Best fit | Gần giống | Chậm hơn |
| Stable insertion order | Không | Best fit | Không |
| Sorted key order | Không | Không | Best fit |
| Range query by key | Không | Không | Best fit |
| Access-order support | Không | Có | Không |

### Decision shortcut

```text
Need fast key lookup only?        -> HashMap
Need predictable iteration order? -> LinkedHashMap
Need sorted keys or range query?  -> TreeMap
```

## Code example

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Map<String, Integer> hashMap = new HashMap<>();
        Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
        Map<String, Integer> treeMap = new TreeMap<>();

        for (String key : List.of("b", "a", "c")) {
            hashMap.put(key, key.length());
            linkedHashMap.put(key, key.length());
            treeMap.put(key, key.length());
        }

        System.out.println(hashMap.keySet());
        System.out.println(linkedHashMap.keySet()); // [b, a, c]
        System.out.println(treeMap.keySet()); // [a, b, c]
    }
}
```

## When to use / when NOT to use

Dùng `HashMap` khi lookup key là chính và order không quan trọng.

Dùng `LinkedHashMap` khi cần predictable iteration theo insertion order, như response JSON, config processing, hoặc dedupe nhưng giữ order.

Dùng `TreeMap` khi sorted key hoặc range query là requirement thật.

Không dùng `TreeMap` chỉ vì muốn “nhìn đẹp” trong log. Sort output ở boundary có thể đơn giản hơn và tránh coupling ordering vào data structure chính.

## How this connects to Spring

Trong Spring Boot, `Map<String, ...>` trong `@ConfigurationProperties` thường cần order ổn định nếu config được render lại hoặc serialize. `LinkedHashMap` khi đó có thể phù hợp hơn `HashMap`.

Cache tạm thường bắt đầu bằng `HashMap`, nhưng nếu nằm trong singleton bean và có concurrent access thì cần nghĩ tới `ConcurrentHashMap`.

`TreeMap` hữu ích cho response hoặc logic cần sorted keys hoặc ranges, nhưng comparator phải rõ để tránh behavior khác giữa môi trường.

## Gotchas

- `HashMap` không đảm bảo iteration order. Đừng để API response phụ thuộc vào nó.
- `TreeMap` key phải comparable hoặc có comparator. Key không comparable sẽ nổ runtime khi `put`.
- `LinkedHashMap` giữ order bằng linked list phụ, nên có overhead nhưng đổi lại output ổn định.

## Check yourself

- Nếu snapshot test phụ thuộc vào thứ tự key trong JSON, vì sao `HashMap` là lựa chọn yếu?
- Khi nào `LinkedHashMap` tốt hơn `HashMap`, dù cả hai đều hash-based?
- Vì sao `TreeMap` phù hợp cho range query còn `HashMap` thì không?
- Nếu sorted order chỉ cần đúng ở lúc render cuối, có nhất thiết phải dùng `TreeMap` từ đầu không?
- Access-order trong `LinkedHashMap` gợi ý use case nào?

## Exercises

### Bài 1: Preserve Config Key Order
Độ khó: Dễ

Đề bài:
Cho các cặp key-value được load từ một config file, trả về một map giữ nguyên thứ tự các key xuất hiện.

Ví dụ 1:
Đầu vào:
```text
pairs = [("b", 1), ("a", 2), ("c", 3)]
```

Đầu ra:
```text
{"b": 1, "a": 2, "c": 3}
```

Giải thích:
Kết quả giữ insertion order, không phải sorted order.

Ràng buộc:
- 0 <= pairs.length <= 10^5
- Key là unique và non-null
- Giữ nguyên insertion order của key

### Bài 2: Range Lookup By Key
Độ khó: Trung bình

Đề bài:
Cho key và value dạng số nguyên, trả về toàn bộ entry có key nằm trong đoạn `[fromKey, toKey)`.

Ví dụ 1:
Đầu vào:
```text
entries = {1: "A", 3: "C", 5: "E"}, fromKey = 2, toKey = 6
```

Đầu ra:
```text
{3: "C", 5: "E"}
```

Giải thích:
Chỉ các key lớn hơn hoặc bằng `2` và nhỏ hơn `6` mới được đưa vào kết quả.

Ràng buộc:
- 0 <= entries.length <= 10^5
- Key là unique
- Cần hỗ trợ range query hiệu quả

### Bài 3: Flaky JSON Response Order
Độ khó: Khó

Đề bài:
Một API tạo JSON response từ `HashMap`, và snapshot test đôi khi fail vì thứ tự key thay đổi. Hãy xác định nguyên nhân gốc và đề xuất một loại map làm cho order trở thành một phần của contract.

Ví dụ 1:
Đầu vào:
```text
map entries inserted as [("b", 1), ("a", 2)]
```

Đầu ra:
```text
Thứ tự có thể xuất hiện thành {"a":2,"b":1} hoặc {"b":1,"a":2} tùy vào behavior của map
```

Giải thích:
`HashMap` không hứa hẹn iteration order, nên response order không ổn định theo contract.

Ràng buộc:
- API response order phải là deterministic
- Không được phụ thuộc vào iteration order ngẫu nhiên của `HashMap`

## Links

- [[003-hash-map]]
- [[006-hash-set-vs-tree-set-vs-linked-hash-set]]
- [[010-concurrent-hash-map-vs-hash-map]]
- `LinkedHashMap` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedHashMap.html
- `TreeMap` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/TreeMap.html
