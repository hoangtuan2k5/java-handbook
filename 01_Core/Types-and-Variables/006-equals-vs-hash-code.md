# equals vs hashCode

## What is it

`equals()` và `hashCode()` là cặp contract rất quan trọng của Java object model.

* `equals()` quyết định hai object có được xem là bằng nhau về mặt logic hay không
* `hashCode()` tạo ra số nguyên dùng bởi hash-based collection như `HashMap` và `HashSet` để chọn bucket ban đầu

Hai method này không độc lập. Nếu override `equals()`, gần như luôn phải override `hashCode()` cho nhất quán.

## How I used to misunderstand it

Mình từng nghĩ chỉ cần override `equals()` là đủ để object “so sánh đúng”. Khi dùng `HashSet` hoặc `HashMap`, mình ngạc nhiên vì object nhìn giống nhau vẫn bị xem là hai key khác nhau.

Hiểu nhầm thứ hai là nghĩ `hashCode()` giống unique id tuyệt đối. Không đúng. Nhiều object khác nhau vẫn có thể có cùng hash code. Collection hash-based luôn cần bước `equals()` để xác nhận tiếp.

## How it actually works

Contract cơ bản là: nếu `a.equals(b)` là `true` thì `a.hashCode()` phải bằng `b.hashCode()`. Chiều ngược lại không bắt buộc. Hai object có cùng hash code vẫn có thể không bằng nhau.

Hash-based collection thường làm hai bước:

1. dùng `hashCode()` để chọn bucket nhanh hơn
2. dùng `equals()` để phân biệt object trong bucket đó

Nếu override `equals()` mà quên `hashCode()`, collection sẽ hoạt động sai logic, thường dưới dạng duplicate key hoặc lookup thất bại.

### Comparison table

| Câu hỏi | `equals()` | `hashCode()` |
|---|---|---|
| Trả lời điều gì | Hai object có bằng nhau về logic không | Object đi vào bucket nào trước |
| Dùng trực tiếp bởi ai | `contains`, `remove`, equality checks | `HashMap`, `HashSet`, cache internals |
| Có thể tự quyết định uniqueness một mình không | Không, trong hash collection cần đi cùng hash | Không |
| Nếu override thì thường đi kèm cái gì | `hashCode()` | `equals()` |

### Tiny flow diagram

```text
HashMap lookup
    -> hashCode() to choose bucket
    -> equals() to find the exact matching key inside that bucket
```

## Code example

```java
String a = new String("java");
String b = new String("java");

System.out.println(a.equals(b)); // true
System.out.println(a.hashCode() == b.hashCode()); // true

var set = new java.util.HashSet<String>();
set.add(a);
set.add(b);

System.out.println(set.size()); // 1
```

Ví dụ này cho thấy collection hash-based cần cả content equality và hash code consistency để hoạt động đúng.

## When to use / when NOT to use

Use `equals()` khi:

* bạn cần so sánh logical value của object
* bạn làm việc với collection như `contains`, `remove`, `HashSet`, `HashMap`

Use `hashCode()` khi:

* object sẽ được dùng làm key trong hash-based collection
* bạn đang thiết kế value object hoặc domain object có equality rõ ràng

Do NOT tự nghĩ `hashCode()` là unique id tuyệt đối. Collision luôn có thể xảy ra.

Do NOT dùng `==` thay cho `equals()` khi điều bạn cần là logical equality của object.

Do NOT dùng mutable field trong equality logic nếu object sẽ sống trong `HashMap` hoặc `HashSet`. Sau khi field đổi, object có thể trở nên rất khó lookup lại đúng cách.

## How this connects to real Java projects

Trong Spring app, equality xuất hiện ở DTO comparison, cache key, entity collection behavior, deduplication, và các data structure trung gian như `Set`, `Map`. Nếu object dùng làm key mà contract sai, bug có thể xuất hiện ở cache miss, duplicate item, hoặc lookup không ra kết quả.

Ngay cả khi bạn chưa tự override hai method này, hiểu contract vẫn rất quan trọng vì framework và thư viện Java dựa vào nó liên tục.

## Gotchas

* Override `equals()` mà không override `hashCode()` sẽ làm `HashMap` và `HashSet` hành xử sai.
* Hash collision không có nghĩa là hai object bằng nhau.
* `==` trên object chỉ so sánh reference, không phản ánh logical equality.
* Dùng mutable field trong equality logic có thể làm key bị “mất tích” sau khi đã `put` vào hash-based collection.

## Handbook rule

- Override `equals()` thì phải override `hashCode()` cùng lúc, dùng cùng tập field.
- Field tham gia equality phải bất biến trong vòng đời object đang nằm trong hash-based collection.
- `==` chỉ so sánh reference; logical equality phải dùng `equals()` hoặc `Objects.equals(...)`.
- `hashCode()` không phải id duy nhất; collision được phép, equality vẫn dùng `equals()` để xác định.
- Đảm bảo equality contract: reflexive, symmetric, transitive, consistent, và `null`-safe.

## Check yourself

* Vì sao `equals()` và `hashCode()` phải nhất quán với nhau?
* Nếu hai object có cùng hash code, vì sao vẫn chưa thể kết luận chúng bằng nhau?
* Tại sao bug contract này hay chỉ lộ ra khi object được dùng trong `HashMap` hoặc `HashSet`?
* Vì sao `==` không phù hợp khi bạn đang hỏi “hai object này có cùng logical value không”?
* Mutable field trong key nguy hiểm ở điểm nào sau khi object đã được thêm vào collection?

## Exercises

### Bài 1: Count Distinct Values

Độ khó: Dễ

Đề bài:
Cho `String[] values`. Hãy trả về số lượng giá trị text khác nhau theo content equality.

Ví dụ 1:

Đầu vào:
```text
values = ["java", "spring", "java", "boot"]
```

Đầu ra:
```text
3
```

Giải thích:
Ba giá trị khác nhau là `"java"`, `"spring"`, và `"boot"`.

Ràng buộc:

* `0 <= values.length <= 100000`
* `0 <= values[i].length <= 100`
* Các phần tử không phải `null`

### Bài 2: Find First Duplicate Value

Độ khó: Trung bình

Đề bài:
Cho `String[] values`. Hãy trả về giá trị đầu tiên mà lần xuất hiện thứ hai của nó được gặp sớm nhất khi duyệt từ trái sang phải. Nếu không có duplicate, trả về string rỗng.

Ví dụ 1:

Đầu vào:
```text
values = ["a", "b", "c", "b", "a"]
```

Đầu ra:
```text
"b"
```

Giải thích:
`"b"` trở thành duplicate tại index `3`, sớm hơn `"a"` trở thành duplicate tại index `4`.

Ràng buộc:

* `0 <= values.length <= 100000`
* `1 <= values[i].length <= 100`
* Các phần tử không phải `null`

### Bài 3: Build Frequency Map

Độ khó: Trung bình

Đề bài:
Cho `String[] values`. Hãy trả về `Map<String, Integer>` chứa số lần xuất hiện của từng value theo đúng thứ tự key xuất hiện đầu tiên.

Ví dụ 1:

Đầu vào:
```text
values = ["api", "db", "api", "cache", "db"]
```

Đầu ra:
```text
{"api"=2, "db"=2, "cache"=1}
```

Giải thích:
Map cần phản ánh đúng tần suất của từng value theo content equality.

Ràng buộc:

* `0 <= values.length <= 100000`
* `0 <= values[i].length <= 100`
* Các phần tử không phải `null`

## Links

* [[002-string-immutable-and-pool]]
* [[../Collections/003-hash-map]]
* [[../Collections/006-hash-set-vs-tree-set-vs-linked-hash-set]]
* [[../Object-Methods/002-equals-and-hash-code-contract]]
* [Java SE 21, `Object.equals` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#equals(java.lang.Object))
* [Java SE 21, `Object.hashCode` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#hashCode())
* [Java SE 21, `Objects.hash` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Objects.html#hash(java.lang.Object...))
