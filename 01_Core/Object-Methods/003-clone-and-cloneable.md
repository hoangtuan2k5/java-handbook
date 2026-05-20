# clone() and Cloneable

## What is it

`clone()` là method trên `Object` dùng để tạo bản sao của object.

`Cloneable` là marker interface báo rằng object cho phép `Object#clone()` hoạt động mà không ném `CloneNotSupportedException`.

Khái niệm này gắn chặt với copy semantics, nhất là phân biệt **shallow copy** và **deep copy**.

## How I used to misunderstand it

Mình từng nghĩ implement `Cloneable` là cách chuẩn nhất để copy object trong Java.

Sau đó mới thấy `clone()` khá awkward. Default behavior thường là shallow copy, method contract khó yêu, và class có mutable field lồng nhau rất dễ copy thiếu. Nhiều code hiện đại đọc dễ hơn nếu dùng constructor copy hoặc factory method rõ nghĩa.

## How it actually works

`Object#clone()` tạo field-by-field copy.

- primitive fields được copy theo value
- reference fields chỉ copy reference

Vì vậy default clone gần như là **shallow copy**.

### So sánh nhanh

| Cách copy | Điều gì được copy |
|---|---|
| Shallow copy | object ngoài cùng mới, nhưng nested mutable state có thể vẫn dùng chung |
| Deep copy | object ngoài cùng và nested mutable state đều được copy riêng |

### Mental flow

```text
super.clone()
    -> tạo object mới cùng runtime type
    -> copy field by field
    -> reference field vẫn trỏ vào cùng nested object nếu bạn không copy tiếp
```

Đây là lý do clone mặc định rất dễ gây bug chia sẻ state nếu object chứa array, list, map, hoặc object con mutable.

## Code example

```java
final class Report implements Cloneable {
    private final int[] scores;

    Report(int[] scores) {
        this.scores = scores;
    }

    @Override
    public Report clone() {
        try {
            Report copy = (Report) super.clone();
            return new Report(scores.clone());
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(e);
        }
    }
}
```

Điểm có ích ở ví dụ này là `scores.clone()` tạo array mới. Nếu không có bước đó, bản gốc và bản sao sẽ còn chia sẻ cùng array.

## When to use / when NOT to use

Dùng `clone()` khi bạn đang làm việc với API cũ, framework cũ, hoặc codebase đã thống nhất convention này rồi.

Không nên chọn `clone()` làm default cho thiết kế mới nếu constructor copy, static factory, hoặc immutable design sẽ rõ hơn.

Nếu class có nested mutable state, bạn phải rất rõ mình muốn shallow copy hay deep copy. Không làm rõ điểm này là nguồn bug rất phổ biến.

## How this connects to real Java projects

Trong Spring app, nhu cầu copy object thường xuất hiện khi tạo DTO snapshot, bảo vệ request state trước khi mutate, hoặc chuẩn bị data cho async processing.

Phần lớn code Spring hiện đại ít dựa vào `clone()`. Mapper, copy constructor, immutable value object, hoặc record thường dễ reasoning hơn.

## Gotchas

- `Cloneable` không tự tạo deep copy.
- Clone mặc định copy reference fields, nên nested mutable object vẫn có thể bị dùng chung.
- `clone()` khó kết hợp đẹp với inheritance.
- `super.clone()` và `CloneNotSupportedException` làm API này khá khó chịu so với copy constructor.

## Handbook rule

- Mặc định cho copy là copy constructor hoặc static factory; không dùng `clone()` cho thiết kế mới.
- Phải nói rõ shallow copy hay deep copy trước khi viết logic copy.
- `Cloneable` không tự tạo deep copy; nested mutable vẫn dùng chung reference.
- Tránh `clone()` khi có inheritance phức tạp; ưu tiên immutable design hoặc builder.
- Nếu phải duy trì `clone()` legacy, document rõ semantics và giới hạn.

## Check yourself

- Vì sao clone mặc định thường chỉ là shallow copy?
- `Cloneable` thật sự đảm bảo điều gì, và không đảm bảo điều gì?
- Khi nào constructor copy rõ hơn `clone()`?
- Vì sao nested mutable state là rủi ro lớn nhất của clone?
- Nếu object cần copy sâu, bạn phải làm thêm gì ngoài `super.clone()`?

## Exercises

### Bài 1: Clone Score Array

Độ khó: Dễ

Đề bài:
Cho một integer array, trả về một cloned array có cùng value theo đúng thứ tự cũ.

Ví dụ 1:

Đầu vào:
```text
scores = [10, 20, 30]
```

Đầu ra:
```text
[10, 20, 30]
```

Giải thích:
Kết quả phải chứa cùng value nhưng là một array instance khác.

Ràng buộc:

- `scores` là non-null
- `0 <= scores.length <= 100000`
- Trả về một array instance mới

### Bài 2: Detect Shallow Clone Risk

Độ khó: Trung bình

Đề bài:
Cho biết một object có mutable field hay không và field đó có được deep copy hay không, trả về `true` nếu clone vẫn có nguy cơ share mutable state.

Ví dụ 1:

Đầu vào:
```text
hasMutableField = true, fieldCopiedDeeply = false
```

Đầu ra:
```text
true
```

Giải thích:
Nếu tồn tại mutable field nhưng không được deep copy, original và clone vẫn có thể share cùng dữ liệu.

Ràng buộc:

- Input là các giá trị boolean
- Chỉ trả về `true` khi việc share mutable state vẫn còn có thể xảy ra
- Bỏ qua các immutable field

### Bài 3: Deep Clone Grid

Độ khó: Trung bình

Đề bài:
Cho một 2D integer array, trả về một deep clone trong đó cả outer array lẫn mọi inner row đều được copy.

Ví dụ 1:

Đầu vào:
```text
grid = [[1, 2], [3, 4]]
```

Đầu ra:
```text
[[1, 2], [3, 4]]
```

Giải thích:
Value vẫn giữ nguyên, nhưng mỗi row trong kết quả phải là một array mới.

Ràng buộc:

- `grid` là non-null
- Rows are non-null
- Total number of elements <= 100000

## Links

- [[001-to-string]]
- [[002-equals-and-hash-code-contract]]
- [[../Memory/004-shallow-copy-vs-deep-copy]]
- [[004-finalize-and-cleaner]]
- `Object#clone` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#clone()
- Effective guidance on why copy constructors are often preferred: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html#clone()
