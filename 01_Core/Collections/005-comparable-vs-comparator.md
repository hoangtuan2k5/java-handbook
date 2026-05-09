# Comparable vs Comparator

## What is it

`Comparable` và `Comparator` đều dùng để sắp xếp object, nhưng chúng trả lời hai câu hỏi khác nhau.

- `Comparable` nói: object này có natural ordering là gì.
- `Comparator` nói: trong tình huống này, tôi muốn sort theo rule nào.

Ví dụ, `User` có thể có natural ordering theo `id`, nhưng màn hình admin lại cần sort theo `lastLogin` hoặc `name`. Đó là lúc `Comparator` hợp hơn.

## How I used to misunderstand it

Hiểu nhầm phổ biến là cứ muốn sort thì implement `Comparable`.

Điều đó dễ làm domain object bị nhồi một ordering không thật sự “tự nhiên”. Một `User` có thể sort theo tên, tuổi, ngày tạo, hoặc điểm. Chọn một cái làm natural ordering đôi khi chỉ là quyết định UI tạm thời.

Cũng nhiều dev quên rằng ordering phải consistent. Nếu `compareTo()` hoặc comparator trả kết quả mâu thuẫn với `equals()` trong `TreeSet` hoặc `TreeMap`, phần tử có thể bị coi là trùng theo cách bạn không mong muốn.

## How it actually works

`Comparable<T>` đặt ordering vào chính class qua `compareTo(T other)`. Vì vậy nó hợp với value object có một thứ tự tự nhiên rõ ràng như `Integer`, `String`, `LocalDate`, hoặc một domain type mà cả team đều đồng ý về thứ tự mặc định.

`Comparator<T>` tách ordering ra ngoài object, nên bạn có thể tạo nhiều rule sort khác nhau mà không sửa class.

Các collection có ordering như `TreeSet` và `TreeMap` dùng comparator hoặc natural ordering để quyết định vị trí, và cả uniqueness theo ordering. Nếu comparator trả `0`, `TreeSet` sẽ coi hai object là duplicate dù `equals()` có thể trả `false`.

### Comparison table

| Question | `Comparable` | `Comparator` |
|---|---|---|
| Ordering nằm ở đâu | Trong class | Bên ngoài class |
| Có mấy rule sort | Thường một natural ordering | Có thể nhiều rule |
| Có sửa domain class không | Có | Không |
| Hợp khi nào | Domain có thứ tự tự nhiên rõ | Ordering phụ thuộc trường hợp sử dụng |
| Ảnh hưởng `TreeSet` và `TreeMap` | Có | Có |

### Decision shortcut

```text
There is one obvious natural ordering? -> Comparable
Need screen-specific or query-specific sort? -> Comparator
```

## Code example

```java
import java.util.*;

class User implements Comparable<User> {
    final long id;
    final String name;

    User(long id, String name) {
        this.id = id;
        this.name = name;
    }

    public int compareTo(User other) {
        // natural ordering chỉ nên ở đây nếu id thật sự là default order
        return Long.compare(id, other.id);
    }
}

public class Main {
    public static void main(String[] args) {
        List<User> users = new ArrayList<>(List.of(new User(2, "Linh"), new User(1, "An")));

        // dùng Comparable
        Collections.sort(users);
        // thứ tự sắp xếp bên ngoài cho trường hợp sử dụng khác
        users.sort(Comparator.comparing(user -> user.name));
    }
}
```

## When to use / when NOT to use

Dùng `Comparable` khi class có một natural ordering ổn định và hiển nhiên trong domain, như `String`, `LocalDate`, hoặc value object mà cả team đều hiểu thứ tự mặc định là gì.

Dùng `Comparator` khi ordering phụ thuộc trường hợp sử dụng: sort product theo price, rating, name, created time, hoặc nhiều tiêu chí khác nhau theo từng màn hình.

Không implement `Comparable` chỉ để phục vụ một màn hình hoặc một query cụ thể. Điều đó làm domain class mang một assumption dễ lỗi thời.

## How this connects to Spring

Trong Spring Boot, sorting thường xuất hiện ở repository query, API response, DTO list, hoặc in-memory post-processing. Nếu sort là concern của endpoint hoặc query, `Comparator` thường rõ hơn vì rule nằm gần trường hợp sử dụng.

Với JPA entities, cẩn thận khi dùng `Comparable` dựa trên mutable field hoặc lazy-loaded association vì sort có thể trigger load ngoài ý muốn hoặc tạo behavior khó đoán.

## Gotchas

- Comparator trả `0` trong `TreeSet` nghĩa là “coi như cùng phần tử” theo ordering, không chỉ là “đứng cạnh nhau”.
- Đừng viết comparator bằng phép trừ như `a - b`. Có thể overflow. Hãy dùng `Integer.compare()` hoặc `Long.compare()`.
- Natural ordering nên ổn định. Dùng mutable field để compare có thể phá sorted collection sau khi object đã được thêm vào.

## Check yourself

- Khi nào `Comparable` là một quyết định domain hợp lý, chứ không chỉ là tiện tay?
- Nếu một class có ba cách sort hợp lý khác nhau, vì sao `Comparator` thường tốt hơn?
- Trong `TreeSet`, vì sao comparator trả `0` có thể làm mất dữ liệu bạn tưởng là khác nhau?
- Vì sao dùng field mutable trong ordering là nguy hiểm?
- Nếu requirement chỉ là sort cho một endpoint, rule sort nên sống ở đâu?

## Exercises

### Bài 1: Sort Users By Name
Độ khó: Dễ

Đề bài:
Cho một list các user, trả về danh sách user được sort theo `name` tăng dần bằng external ordering rule.

Ví dụ 1:
Đầu vào:
```text
users = [{id: 2, name: "Linh"}, {id: 1, name: "An"}]
```

Đầu ra:
```text
[{id: 1, name: "An"}, {id: 2, name: "Linh"}]
```

Giải thích:
Kết quả được sắp xếp theo alphabet của `name`, không phải theo `id`.

Ràng buộc:
- 0 <= users.length <= 10^5
- user.name là non-null
- Do not modify the `User` class

### Bài 2: Sort By Age Then Name
Độ khó: Trung bình

Đề bài:
Cho user có `age` và `name`, sắp xếp theo age giảm dần. Nếu age bằng nhau, sắp xếp theo name tăng dần.

Ví dụ 1:
Đầu vào:
```text
users = [{name: "B", age: 20}, {name: "A", age: 20}, {name: "C", age: 30}]
```

Đầu ra:
```text
[{name: "C", age: 30}, {name: "A", age: 20}, {name: "B", age: 20}]
```

Giải thích:
Age là tiêu chí sắp xếp chính, còn name dùng để break tie.

Ràng buộc:
- 0 <= users.length <= 10^5
- 0 <= age <= 150
- user.name là non-null

### Bài 3: TreeSet Comparator Collision
Độ khó: Khó

Đề bài:
Bạn lưu user trong một `TreeSet` bằng comparator chỉ so sánh `age`. Hãy giải thích vì sao hai user khác nhau nhưng cùng age có thể bị collapse thành một entry.

Ví dụ 1:
Đầu vào:
```text
users = [{name: "A", age: 20}, {name: "B", age: 20}]
```

Đầu ra:
```text
TreeSet may contain only one user
```

Giải thích:
Comparator trả về `0` cho cả hai user, nên `TreeSet` xem chúng là cùng một element.

Ràng buộc:
- Comparator determines uniqueness inside `TreeSet`
- Different objects can be considered equal by ordering

## Links

- [[006-hash-set-vs-tree-set-vs-linked-hash-set]]
- [[007-hash-map-vs-linked-hash-map-vs-tree-map]]
- [[../Object-Methods/002-equals-and-hash-code-contract]]
- `Comparable` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Comparable.html
- `Comparator` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Comparator.html
