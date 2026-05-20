# HashSet vs TreeSet vs LinkedHashSet

## What is it

`HashSet`, `TreeSet`, và `LinkedHashSet` đều là `Set`, nghĩa là cùng giữ rule “không có phần tử trùng”.

Khác nhau nằm ở ordering contract và cách chúng quyết định uniqueness.

- `HashSet` tối ưu membership nhanh nhưng không hứa iteration order.
- `LinkedHashSet` giữ insertion order.
- `TreeSet` giữ sorted order theo natural ordering hoặc `Comparator`.

Nói ngắn gọn: cùng là danh sách khách mời không trùng, nhưng một cái không quan tâm thứ tự, một cái giữ thứ tự đăng ký, một cái tự sắp theo alphabet.

## How I used to misunderstand it

Nhiều dev nghĩ `Set` chỉ có một loại và thứ tự in ra nếu “nhìn ổn” thì có thể dựa vào. Đây là bẫy lớn của `HashSet`.

`TreeSet` cũng hay bị hiểu sai như `HashSet` có sort thêm. Thực ra uniqueness của `TreeSet` dựa trên comparison, không đơn thuần dựa trên `equals()` và `hashCode()`.

Nếu comparator trả `0`, `TreeSet` coi hai phần tử là cùng một item.

## How it actually works

`HashSet` thường backed bởi `HashMap`, nên nó dùng `hashCode()` và `equals()` để quyết định phần tử đã tồn tại chưa. Lookup trung bình nhanh, nhưng iteration order không được hứa hẹn.

`LinkedHashSet` thêm linked list chạy qua các entry, nên giữ được insertion order với chi phí memory cao hơn một chút. Nó hợp khi bạn vừa cần uniqueness vừa muốn output ổn định theo thứ tự input.

`TreeSet` backed bởi tree structure, nên phần tử luôn được giữ theo sorted order. Nó dùng `compareTo()` hoặc `Comparator` để quyết định cả ordering lẫn duplicate. Đổi lại, operation thường là O(log n), không phải O(1) trung bình như hash-based set.

### Comparison table

| Need | `HashSet` | `LinkedHashSet` | `TreeSet` |
|---|---|---|---|
| Fast membership, order không quan trọng | Best fit | Có thể | Thường không cần |
| Giữ first-seen order | Không | Best fit | Không |
| Luôn sorted | Không | Không | Best fit |
| Uniqueness dựa trên | `equals()` + `hashCode()` | `equals()` + `hashCode()` | `compareTo()` hoặc `Comparator` |
| Typical cost | O(1) trung bình | O(1) trung bình | O(log n) |

### Decision shortcut

```text
Need uniqueness only?              -> HashSet
Need uniqueness + insertion order? -> LinkedHashSet
Need uniqueness + sorted order?    -> TreeSet
```

## Code example

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<String> input = List.of("B", "A", "B", "C");

        // removes duplicates, no order contract
        Set<String> hashSet = new HashSet<>(input);
        // keeps first-seen order
        Set<String> linkedHashSet = new LinkedHashSet<>(input);
        // keeps sorted order
        Set<String> treeSet = new TreeSet<>(input);

        System.out.println(hashSet);
        System.out.println(linkedHashSet); // [B, A, C]
        System.out.println(treeSet); // [A, B, C]
    }
}
```

## When to use / when NOT to use

Dùng `HashSet` khi bạn chỉ cần uniqueness và fast membership check, như visited IDs hoặc permission lookup.

Dùng `LinkedHashSet` khi cần remove duplicate nhưng vẫn giữ order đầu vào, như tags từ request hoặc menu items theo thứ tự cấu hình.

Dùng `TreeSet` khi sorted order là requirement thật, như names sorted alphabetically, ranges, hoặc leaderboard theo comparator.

Không dùng `TreeSet` chỉ để sort output một lần. Khi đó sort một `List` ở boundary thường rõ hơn.

## How this connects to real Java projects

Trong Spring Boot, `LinkedHashSet` hữu ích khi bind config hoặc request list cần loại trùng nhưng giữ order người dùng nhập. `HashSet` hợp với role hoặc permission membership check. `TreeSet` hợp khi API luôn phải trả sorted unique values.

Nếu dùng `Set` trong DTO response mà không chọn implementation hoặc order rõ, JSON output có thể không ổn định và làm test snapshot hoặc client expectation bị flaky.

## Gotchas

- `HashSet` không đảm bảo order, dù output local nhìn có vẻ ổn định.
- `TreeSet` coi comparator trả `0` là duplicate, kể cả `equals()` trả `false`.
- Element mutable có thể phá cả hash-based set lẫn tree-based set nếu field dùng trong equality hoặc ordering bị đổi sau khi `add`.

## Handbook rule

- Default cho membership/uniqueness là `HashSet`.
- Cần unique nhưng giữ insertion order, dùng `LinkedHashSet`, không dùng `HashSet` rồi tự sort.
- `TreeSet` chỉ chọn khi sorted order hoặc range query là requirement thật.
- Không phụ thuộc iteration order của `HashSet` cho API/test/snapshot.
- Element mutable theo field tham gia equality/ordering không an toàn trong bất kỳ `Set` nào.

## Check yourself

- Nếu requirement là “remove duplicate nhưng giữ first-seen order”, vì sao `HashSet` chưa đủ?
- Vì sao `TreeSet` không chỉ là `HashSet` có thêm bước sort?
- Nếu comparator của `TreeSet` chỉ so sánh `age`, rủi ro dữ liệu gì có thể xảy ra?
- Trong response JSON cần deterministic order, `HashSet` có gì thiếu?
- Khi nào uniqueness nên dựa trên ordering, và khi nào nên dựa trên `equals()` và `hashCode()`?

## Exercises

### Bài 1: Unique Emails In Request Order
Độ khó: Dễ

Đề bài:
Cho các email từ một request, trả về các email unique và giữ nguyên thứ tự xuất hiện đầu tiên.

Ví dụ 1:
Đầu vào:
```text
emails = ["b@test.com", "a@test.com", "b@test.com"]
```

Đầu ra:
```text
["b@test.com", "a@test.com"]
```

Giải thích:
Duplicate bị loại bỏ, nhưng thứ tự xuất hiện đầu tiên trong input vẫn được giữ nguyên.

Ràng buộc:
- 0 <= emails.length <= 10^5
- emails[i] là non-null
- Giữ nguyên thứ tự xuất hiện đầu tiên

### Bài 2: Sorted Unique Names
Độ khó: Trung bình

Đề bài:
Cho một list các tên, trả về các tên unique đã được sắp xếp theo alphabet.

Ví dụ 1:
Đầu vào:
```text
names = ["Binh", "An", "Binh", "Linh"]
```

Đầu ra:
```text
["An", "Binh", "Linh"]
```

Giải thích:
Duplicate bị loại bỏ và các tên còn lại được sắp xếp.

Ràng buộc:
- 0 <= names.length <= 10^5
- names[i] là non-null
- Sắp xếp theo natural string ordering

### Bài 3: Choose Set For Roles
Độ khó: Khó

Đề bài:
Một Spring Security API trả về user role. Hãy chọn giữa `HashSet`, `LinkedHashSet`, và `TreeSet`, rồi giải thích API contract được ngụ ý bởi lựa chọn đó.

Ví dụ 1:
Đầu vào:
```text
roles = ["ADMIN", "USER", "ADMIN"]
```

Đầu ra:
```text
["ADMIN", "USER"] or ["ADMIN", "USER"] with an explicit ordering contract
```

Giải thích:
Set được chọn sẽ quyết định việc order là unspecified, insertion-based, hay sorted.

Ràng buộc:
- Role phải là unique
- API consumer có thể so sánh response order trong test
- Phải nêu rõ order guarantee

## Links

- [[001-list-vs-set-vs-map]]
- [[005-comparable-vs-comparator]]
- [[../Object-Methods/002-equals-and-hash-code-contract]]
- `HashSet` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashSet.html
- `LinkedHashSet` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedHashSet.html
- `TreeSet` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/TreeSet.html
