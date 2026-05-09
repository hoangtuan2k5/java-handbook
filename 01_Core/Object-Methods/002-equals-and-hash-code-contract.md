# equals() and hashCode() contract

## What is it

`equals()` quyết định hai object có được xem là bằng nhau về mặt logical equality hay không.

`hashCode()` tạo ra số nguyên để hash-based collections như `HashMap` và `HashSet` chọn bucket trước khi so sánh sâu hơn.

Contract quan trọng nhất là:

> nếu hai object `equals()` nhau thì chúng bắt buộc phải có cùng `hashCode()`

## How I used to misunderstand it

Mình từng nghĩ chỉ cần override `equals()` là đủ.

Rồi tới lúc dùng `HashSet` hoặc `HashMap`, mình gặp cảnh object nhìn có vẻ giống nhau nhưng vẫn thêm trùng hoặc lookup thất bại. Lý do là `hashCode()` không khớp với logic của `equals()`.

Một hiểu nhầm khác là xem `hashCode()` như unique id. Nó không cần unique tuyệt đối. Nó chỉ cần đủ nhất quán với equality contract.

## How it actually works

`equals()` nên thỏa các tính chất cơ bản:

- reflexive
- symmetric
- transitive
- consistent
- `x.equals(null)` phải là `false`

`hashCode()` không cần unique, nhưng phải ổn định miễn là state dùng cho equality chưa đổi.

### Hash collection mental model

```text
put / contains / get
    -> dùng hashCode() để chọn bucket
    -> trong bucket đó mới dùng equals() để so phần tử thật sự
```

### Hệ quả thực tế

| Tình huống | Điều gì xảy ra |
|---|---|
| `equals()` đúng, `hashCode()` sai | lookup hoặc dedup có thể hỏng |
| field dùng cho equality bị mutate sau khi put vào `HashMap` | object có thể gần như biến mất khỏi bucket đúng |
| `==` là `false`, `equals()` là `true` | khác reference nhưng bằng nhau về logic |

## Code example

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

record UserKey(String tenantId, String email) {
    @Override
    public int hashCode() {
        return Objects.hash(tenantId, email);
    }
}

Set<UserKey> keys = new HashSet<>();
keys.add(new UserKey("acme", "a@x.com"));

System.out.println(keys.contains(new UserKey("acme", "a@x.com"))); // true
```

## When to use / when NOT to use

Override `equals()` và `hashCode()` khi object có identity logic riêng, nhất là khi object sẽ đi vào `HashMap`, `HashSet`, cache key, deduplication logic, hoặc test comparison.

Không override nếu class thật sự nên dùng reference identity mặc định, hoặc khi bạn chưa xác định rõ field nào tạo nên equality.

Trong nhiều trường hợp, `record` là lựa chọn rõ ràng hơn vì Java tự sinh equality theo component fields.

## How this connects to Spring

Trong Spring app, contract này ảnh hưởng trực tiếp tới entity key wrapper, cache key, DTO deduplication, set membership, bean maps, và nhiều assertion trong test.

Một object có equality sai có thể gây cache miss khó hiểu, duplicate data trong `Set`, hoặc bug chỉ xuất hiện khi dữ liệu đi qua collection hash-based.

## Gotchas

- Override `equals()` mà quên `hashCode()` gần như chắc chắn gây bug với `HashMap` và `HashSet`.
- Dùng field mutable trong equality rất nguy hiểm nếu object làm key của hash collection.
- `==` và `equals()` trả lời hai câu hỏi khác nhau.
- `hashCode()` giống nhau không có nghĩa hai object chắc chắn `equals()` nhau.

## Check yourself

- Vì sao hash collection không thể chỉ dựa vào `equals()`?
- Contract tối quan trọng giữa `equals()` và `hashCode()` là gì?
- Vì sao mutable key làm `HashMap` khó đoán?
- `==` khác `equals()` ở điểm nào?
- Khi nào `record` giúp bạn tránh viết tay contract này?

## Exercises

### Bài 1: Are Customer Keys Equal

Độ khó: Dễ

Đề bài:
Cho hai customer key được biểu diễn bằng cặp `(id, email)`, trả về `true` nếu cả hai cặp bằng nhau theo từng field, ngược lại trả về `false`.

Ví dụ 1:

Đầu vào:
```text
id1 = "42", email1 = "a@example.com", id2 = "42", email2 = "a@example.com"
```

Đầu ra:
```text
true
```

Giải thích:
Cả hai field đều khớp, nên hai logical key này bằng nhau.

Ràng buộc:

- Tất cả input là non-null
- `id` and `email` comparisons are case-sensitive
- Chỉ trả về `true` khi cả hai field đều khớp

### Bài 2: Detect Hash Contract Violation

Độ khó: Trung bình

Đề bài:
Cho kết quả của một equality check và hai hash code, trả về `true` nếu input đang mô tả một vi phạm của contract giữa `equals()` và `hashCode()`.

Ví dụ 1:

Đầu vào:
```text
equalsResult = true, firstHash = 17, secondHash = 99
```

Đầu ra:
```text
true
```

Giải thích:
Nếu hai object bằng nhau, chúng phải tạo ra cùng một hash code.

Ràng buộc:

- `equalsResult` is a boolean
- Hash codes are valid `int` values
- Chỉ trả về `true` cho các contract violation thực sự

### Bài 3: Count Distinct Composite Keys

Độ khó: Trung bình

Đề bài:
Cho hai string array song song `ids` và `emails`, trong đó index `i` tạo thành một composite key `(ids[i], emails[i])`, trả về số lượng key distinct xuất hiện.

Ví dụ 1:

Đầu vào:
```text
ids = ["1", "1", "2"]
emails = ["a@example.com", "a@example.com", "b@example.com"]
```

Đầu ra:
```text
2
```

Giải thích:
Hai entry đầu mô tả cùng một logical key, còn entry thứ ba thì khác.

Ràng buộc:

- `ids` and `emails` are non-null
- `ids.length == emails.length`
- `0 <= ids.length <= 100000`

## Links

- [[001-to-string]]
- [[003-clone-and-cloneable]]
- [[../Types-and-Variables/006-equals-vs-hash-code]]
- [[../Data-Types-Advanced/002-record]]
- `Object#equals` and `Object#hashCode` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html
