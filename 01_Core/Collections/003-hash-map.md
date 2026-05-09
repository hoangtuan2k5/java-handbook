# HashMap

## What is it

`HashMap` là cấu trúc key-value cho phép tìm value rất nhanh bằng key.

Thay vì scan từng phần tử như `List`, nó dùng `hashCode()` của key để tính bucket, rồi dùng `equals()` để xác nhận đúng key trong bucket đó.

Hình dung nhanh: hash chọn ngăn hồ sơ, còn `equals()` xác nhận đúng hồ sơ trong ngăn đó.

## How I used to misunderstand it

Nhiều dev mới học nghĩ `HashMap` chỉ là “mảng thông minh”. Cách nghĩ đó bỏ qua phần quan trọng nhất: correctness phụ thuộc vào contract giữa `equals()` và `hashCode()`.

Nếu hai key bằng nhau theo `equals()` nhưng hash khác nhau, map có thể không tìm lại được value theo đúng logic business.

Hiểu nhầm khác là tin iteration order của `HashMap` vì local test trông có vẻ ổn định. Order đó không phải contract. Chỉ cần đổi JDK, đổi dữ liệu, hoặc resize là kết quả có thể khác.

## How it actually works

Khi `put(key, value)`, `HashMap` lấy hash của key để chọn bucket.

- Nếu bucket trống, entry được đặt vào đó.
- Nếu bucket đã có entry, map dùng `equals()` để kiểm tra có phải cùng key không.
- Nếu cùng key, value cũ bị replace.
- Nếu khác key nhưng cùng bucket, entry mới được thêm vào bucket đó.

Khi số entry vượt threshold dựa trên load factor, `HashMap` resize backing table. Resize không chỉ là thêm chỗ trống. Nhiều entry phải được phân phối lại sang bucket mới.

Vì vậy `HashMap` nhanh trung bình O(1), nhưng không có nghĩa mọi operation luôn có chi phí cố định.

### Lookup mental picture

```text
key -> hashCode() -> bucket -> equals() check -> value
```

Nếu `equals()` nói hai key bằng nhau nhưng `hashCode()` khác nhau, cùng một key logic có thể đi vào hai bucket khác nhau. Lúc đó correctness đã vỡ.

## Code example

```java
import java.util.*;

class UserKey {
    String email;

    UserKey(String email) {
        this.email = email;
    }

    public boolean equals(Object other) {
        return other instanceof UserKey key && email.equals(key.email);
    }

    public int hashCode() {
        // key bằng nhau phải nằm trên cùng lookup path
        return email.hashCode();
    }
}

public class Main {
    public static void main(String[] args) {
        Map<UserKey, String> users = new HashMap<>();
        UserKey key = new UserKey("linh@example.com");
        users.put(key, "Linh");

        // mutate key sau put phá vỡ giả định lookup
        key.email = "changed@example.com";

        // thường là null
        System.out.println(users.get(key));
    }
}
```

## When to use / when NOT to use

Dùng `HashMap` khi lookup theo key là operation chính: cache theo ID, count frequency, group data theo code, hoặc truy cập dữ liệu theo tên.

Không dùng `HashMap` nếu bạn cần:

- sorted order,
- stable insertion order,
- concurrent writes từ nhiều thread.

Trong các trường hợp đó, hãy nghĩ tới `TreeMap`, `LinkedHashMap`, hoặc `ConcurrentHashMap`.

Cũng không nên dùng mutable object làm key nếu field tham gia `equals()` hoặc `hashCode()` có thể đổi sau khi `put`.

## How this connects to Spring

Trong Spring Boot, `HashMap` xuất hiện nhiều ở cache tạm, config binding dạng `Map<String, ...>`, grouping response, hoặc lookup trong service layer.

Nếu key là entity hoặc DTO mutable, bug lookup fail rất khó thấy vì map không báo lỗi khi key bị mutate. Với singleton bean, `HashMap` mutable dùng chung giữa nhiều request còn có thêm vấn đề thread-safety.

## Gotchas

- Key dùng trong `HashMap` phải có `equals()` và `hashCode()` nhất quán.
- Đừng mutate key sau khi `put`, nhất là field tham gia hash hoặc equality.
- `HashMap` không thread-safe. Concurrent writes có thể gây race hoặc state không đáng tin.
- `HashMap` cho phép `null` key và `null` value, nhưng điều này đôi khi làm API mơ hồ hơn.

## Check yourself

- Trong `HashMap`, vì sao `hashCode()` chưa đủ mà vẫn cần `equals()`?
- Nếu hai object bằng nhau theo `equals()` nhưng hash khác nhau, bug gì có thể xuất hiện?
- Vì sao mutate key sau khi `put` là bẫy nguy hiểm?
- Khi nào `HashMap` hợp hơn `List`, và khi nào lại không hợp bằng `ConcurrentHashMap`?
- Nếu code đang dựa vào thứ tự iteration của `HashMap`, contract nào đang bị hiểu sai?

## Exercises

### Bài 1: Word Frequency
Độ khó: Dễ

Đề bài:
Cho một list các từ, trả về một map trong đó mỗi key là một từ và mỗi value là số lần từ đó xuất hiện.

Ví dụ 1:
Đầu vào:
```text
words = ["java", "map", "java"]
```

Đầu ra:
```text
{"java": 2, "map": 1}
```

Giải thích:
`"java"` xuất hiện hai lần và `"map"` xuất hiện một lần.

Ràng buộc:
- 0 <= words.length <= 10^5
- words[i] là non-null
- Không yêu cầu thứ tự của output

### Bài 2: Find Profile By Id
Độ khó: Trung bình

Đề bài:
Cho user ID và profile name, hãy xây một map và trả về profile tương ứng với target ID.

Ví dụ 1:
Đầu vào:
```text
ids = [10, 20], names = ["Linh", "An"], targetId = 20
```

Đầu ra:
```text
"An"
```

Giải thích:
ID `20` được ánh xạ tới profile name `"An"`.

Ràng buộc:
- ids.length == names.length
- 0 <= ids.length <= 10^5
- ID là unique

### Bài 3: Mutable Key Bug
Độ khó: Khó

Đề bài:
Bạn lưu một mutable key object trong `HashMap`, rồi mutate field được dùng bởi `hashCode()`. Hãy giải thích vì sao lookup có thể fail và viết lại design để tránh bug này.

Ví dụ 1:
Đầu vào:
```text
put key.email = "a@test.com", then change key.email = "b@test.com", then get(key)
```

Đầu ra:
```text
lookup có thể trả về `null` hoặc fail ngoài dự kiến
```

Giải thích:
Key bây giờ có thể hash sang bucket khác với bucket đã được dùng khi insertion.

Ràng buộc:
- Các field của key được dùng trong `equals()` và `hashCode()` không được thay đổi khi key còn nằm trong map
- Ưu tiên dùng immutable key type

## Links

- [[001-list-vs-set-vs-map]]
- [[007-hash-map-vs-linked-hash-map-vs-tree-map]]
- [[../Object-Methods/002-equals-and-hash-code-contract]]
- `HashMap` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html
