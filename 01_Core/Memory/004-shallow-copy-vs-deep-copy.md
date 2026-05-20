# Shallow Copy vs Deep Copy

## What is it

`Shallow copy` tạo object hoặc container mới, nhưng vẫn chia sẻ reference tới object con.

`Deep copy` tạo bản sao mới cho cả những phần mutable cần độc lập.

Mental model dễ nhớ:

- shallow copy là đổi cái hộp
- deep copy là đổi cả hộp lẫn đồ bên trong, ít nhất với những món có thể bị sửa

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là tưởng `new ArrayList<>(oldList)` đã là “copy an toàn”. Thực tế constructor đó chỉ copy danh sách reference.

Nếu element là mutable object như `User`, `Address`, `StringBuilder`, hoặc nested `List`, cả collection cũ và mới vẫn nhìn vào cùng object con.

## How it actually works

Điểm mấu chốt là phân biệt `container` với `element`.

Khi shallow copy một `List<User>`:

- bạn có list mới
- nhưng từng `User` bên trong vẫn là object cũ

Khi deep copy:

- bạn có list mới
- và từng `User` mutable quan trọng cũng là object mới

### So sánh nhanh

| Câu hỏi | Shallow copy | Deep copy |
|---|---|---|
| Container mới không? | Có | Có |
| Element mutable được copy thành object mới không? | Không | Có |
| Sửa object con ở bản copy có thể ảnh hưởng bản gốc không? | Có thể có | Thường không |
| Chi phí memory và code | Thấp hơn | Cao hơn |

### Tiny aliasing diagram

```text
originalList ----> [ ref ] ----> User{name="Linh"}
copyList     ----> [ ref ] -----^

Hai list khác nhau,
nhưng vẫn cùng trỏ tới một User.
```

Với immutable element như `String`, shallow copy thường đủ vì element không đổi được. Với mutable element, shallow copy dễ sinh bug aliasing, tức sửa bên này nhưng bên kia cũng đổi theo.

## Code example

```java
import java.util.ArrayList;
import java.util.List;

List<StringBuilder> original = new ArrayList<>();
original.add(new StringBuilder("java"));

// list mới, cùng element reference
List<StringBuilder> copy = new ArrayList<>(original);
copy.get(0).append("!");

System.out.println(original.get(0)); // java!
```

Điều bị chia sẻ ở đây không phải list object, mà là `StringBuilder` bên trong.

## When to use / when NOT to use

Dùng shallow copy khi:

- element là immutable
- bạn chỉ cần copy structure bên ngoài
- shared child object là chủ đích thiết kế

Dùng deep copy khi:

- cần snapshot độc lập
- không muốn caller sửa state chung
- chuẩn bị đưa dữ liệu sang boundary khác như cache, event, hoặc async processing

Không nên deep copy mọi thứ theo phản xạ. Nhiều lúc điều đúng hơn là làm rõ ownership, hoặc dùng immutable model thay vì nhân đôi object graph.

## How this connects to real Java projects

Trong Spring app, shallow và deep copy xuất hiện ở DTO mapping, cache return value, event publishing, defensive copy trong service, và test fixture reuse.

Ví dụ điển hình là service trả về mutable object đang được cache chung. Caller sửa object đó, rồi request sau đọc lại thấy state đã bị đổi ngoài ý muốn.

## Gotchas

- Copy collection không đồng nghĩa copy element.
- Immutable object làm shallow copy an toàn hơn rất nhiều.
- Deep copy bằng serialization thường chậm, khó đoán, và nhiều edge case.
- Nếu bạn cần deep copy liên tục, đôi khi đó là tín hiệu model nên chuyển sang immutable hơn.

## Handbook rule

- Shallow copy đủ khi element immutable hoặc share là chủ đích.
- Deep copy khi cần snapshot độc lập trước khi qua boundary (cache/event/async).
- Copy collection không tự copy element; phải clone từng phần khi cần.
- Deep copy bằng serialization là last resort; chậm và nhiều edge case.
- Nếu phải deep copy liên tục, cân nhắc đổi sang immutable model.

## Check yourself

- Vì sao `new ArrayList<>(oldList)` chưa chắc tạo snapshot an toàn?
- Trong bài này, phần nào được copy, phần nào vẫn bị chia sẻ?
- Khi nào shallow copy là đủ tốt?
- Tại sao immutable element làm shallow copy bớt nguy hiểm?
- Nếu caller không được phép sửa state dùng chung, bạn nên nghĩ tới deep copy hay immutable design trước?

## Exercises

### Bài 1: Copy Array Shallow
Độ khó: Dễ

Đề bài:
Cho một integer array, trả về một array mới có cùng các giá trị.

Ví dụ 1:
Đầu vào:
```text
numbers = [1, 2, 3]
```

Đầu ra:
```text
[1, 2, 3]
```

Giải thích:
Các primitive value được copy vào một array mới.

Ràng buộc:
- numbers là non-null
- 0 <= numbers.length <= 100000
- Trả về một array instance khác

### Bài 2: Detect Shared Mutable Element Risk
Độ khó: Trung bình

Đề bài:
Cho biết element có mutable hay không và có phải chỉ tạo shallow copy hay không, trả về `true` nếu shared mutable state là rủi ro.

Ví dụ 1:
Đầu vào:
```text
mutableElements = true, shallowCopy = true
```

Đầu ra:
```text
true
```

Giải thích:
Cả hai collection sẽ share mutable element reference.

Ràng buộc:
- Input là các giá trị boolean
- Return `true` only when both inputs are `true`
- Không mô hình hóa nested immutability

### Bài 3: Clone Matrix Deeply
Độ khó: Trung bình

Đề bài:
Cho một 2D integer array, trả về một deep copy trong đó mỗi row là một array mới.

Ví dụ 1:
Đầu vào:
```text
matrix = [[1, 2], [3]]
```

Đầu ra:
```text
[[1, 2], [3]]
```

Giải thích:
Outer array và từng inner row đều phải được copy.

Ràng buộc:
- matrix là non-null
- rows are non-null
- Total number of elements <= 100000

## Links

- [[003-String-pool-vs-heap]]
- [[../../00_Mental-Models/007-Immutability]]
- [[../../00_Mental-Models/011-value-type-vs-reference-type]]
- [Java SE 21, `ArrayList` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html)
- [Java SE 21, `List.copyOf` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#copyOf(java.util.Collection))
