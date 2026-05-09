# Reflection

## What is it

`Reflection` là cơ chế cho phép code đọc `metadata` của class, field, method, constructor, annotation ở runtime, và trong một số trường hợp còn có thể tạo object hoặc gọi method động thông qua `Class<?>`, `Field`, `Method`, `Constructor`.

Nó giải quyết bài toán: compile time mình chưa biết chính xác type hoặc member nào sẽ được dùng, nhưng tới runtime vẫn cần inspect cấu trúc hoặc kích hoạt một hành vi nào đó.

## How I used to misunderstand it

Hiểu nhầm dễ gặp nhất là nghĩ `reflection` chỉ là cách rắc rối hơn để gọi method bình thường.

Thực ra, giá trị lớn nhất của nó nằm ở chỗ framework và infrastructure code có thể nhìn vào program metadata rồi quyết định phải làm gì. `Reflection` không sinh ra để thay toàn bộ direct call. Nó sinh ra để xử lý các tình huống động mà compile-time API không diễn đạt hết.

## How it actually works

Mỗi class đã được JVM load đều có metadata đi kèm. `Reflection` cho phép mình bước vào phần metadata đó rồi hỏi các câu như:

- class này có field nào
- constructor nào đang tồn tại
- method này có annotation gì
- generic signature cơ bản trông như thế nào

Sau đó có hai mức dùng rất khác nhau:

| Mức dùng | Làm gì | Thường an toàn tới đâu | Điểm cần nhớ |
|---|---|---|---|
| Inspect metadata | Đọc field, method, annotation, modifier | An toàn hơn | Rất phổ biến trong framework |
| Act on metadata | `newInstance`, `invoke`, đọc/ghi field | Rủi ro hơn | Dễ chậm hơn, dễ hỏng khi refactor |

### Runtime boundary cần nhớ

```text
Compile time: compiler biết type tĩnh, kiểm tra lời gọi trực tiếp
Runtime:      reflection đọc metadata thật của class đã được load
```

Điểm quan trọng là `reflection` luôn đi sau compile time. Compiler không bảo vệ được các string như tên field hoặc tên method mà bạn truyền vào reflection API.

### Mental model ngắn gọn

```text
Normal call:      caller -> target.method()
Reflection call:  caller -> metadata lookup -> access checks -> invoke target
```

Khi gọi `setAccessible(true)` hoặc `trySetAccessible()`, code đang cố nới lỏng access check. Đây là vùng mạnh nhưng nguy hiểm, vì nó tăng coupling với cấu trúc nội bộ và có thể vướng module boundary trong Java hiện đại.

## Code example

```java
import java.lang.reflect.Field;

record User(String name, int age) {
}

Class<User> type = User.class;

for (Field field : type.getDeclaredFields()) {
    System.out.println(field.getName());
}
```

Ví dụ này chỉ đọc metadata ở runtime. Nó chưa đụng tới việc sửa field hay invoke method, nên đây là nửa `inspect` của reflection, cũng là nửa thường gặp hơn trong code framework.

## When to use / when NOT to use

Dùng `reflection` khi đang viết framework code, serializer, object mapper, test utility, plugin loader, hoặc tool cần inspect class động.

Đừng dùng `reflection` cho business logic bình thường nếu direct call đã đủ rõ. Khi type và method đã biết ở compile time, gọi trực tiếp vừa nhanh hơn, vừa rõ intent hơn, vừa được IDE refactor bảo vệ tốt hơn.

### Quick decision scaffold

| Tình huống | Hợp với reflection không | Vì sao |
|---|---|---|
| Scan class để tìm annotation | Có | Đây là metadata problem |
| Bind data động vào nhiều type khác nhau | Có thể | Thường là framework or utility concern |
| Gọi `userService.save()` khi đã biết type | Không | Direct call rõ hơn nhiều |
| Chạm vào private field để né thiết kế hiện tại | Thường không | Rất dễ tạo technical debt |

## How this connects to Spring

Spring dùng `reflection` rất nhiều để scan bean, đọc annotation, resolve constructor injection, bind configuration, và gọi lifecycle callback.

Khi viết application code, mình ít khi gọi reflection bằng tay. Nhưng container vẫn đang dùng nó phía sau để hiểu class của mình mang metadata gì và cần được dựng như thế nào ở runtime.

## Gotchas

- `getDeclaredFields()` lấy cả private field nhưng không tự cấp quyền đọc giá trị của field đó.
- Thứ tự field, method, constructor trả về không nên được coi là contract ổn định cho business logic.
- Reflection call thường chậm hơn direct call và kém thân thiện với refactor tool.
- Đổi tên member có thể làm code reflection hỏng âm thầm nếu đang dùng string literal.
- Java module system có thể chặn deep reflection nếu package không được mở phù hợp.

## Check yourself

- Vì sao đọc metadata bằng reflection ít rủi ro hơn `invoke()` hoặc ghi thẳng vào field?
- Khi nào bài toán thực sự là runtime discovery, không phải compile-time direct call?
- Nếu code reflection phụ thuộc vào tên method dạng string, compiler còn bảo vệ được gì?
- `setAccessible(true)` mạnh ở chỗ nào, và nguy hiểm ở chỗ nào?
- Trong Spring, vì sao bạn không gọi reflection tay nhưng vẫn cần hiểu nó?

## Exercises

### Bài 1: Find Field Index By Name

Độ khó: Dễ

Đề bài:
Cho một array các field name và một target field name, trả về index đầu tiên có value bằng target đó. Trả về `-1` nếu target không tồn tại.

Ví dụ 1:

Đầu vào:
```text
fieldNames = ["id", "email", "createdAt"], targetName = "email"
```

Đầu ra:
```text
1
```

Giải thích:
Field tên `email` xuất hiện ở index 1.

Ràng buộc:

- `fieldNames` là non-null
- `targetName` là non-null
- `fieldNames.length` nằm trong khoảng từ 0 đến 100000

### Bài 2: Count Public No Arg Methods

Độ khó: Dễ

Đề bài:
Cho các array `isPublic` và `parameterCounts` mô tả các method theo cùng một thứ tự, trả về số method vừa là public vừa có zero parameter.

Ví dụ 1:

Đầu vào:
```text
isPublic = [true, false, true], parameterCounts = [0, 0, 2]
```

Đầu ra:
```text
1
```

Giải thích:
Chỉ method đầu tiên vừa là public vừa là no-arg.

Ràng buộc:

- Cả hai array đều là non-null
- Cả hai array có cùng độ dài
- Độ dài array nằm trong khoảng từ 0 đến 100000

### Bài 3: Select Writable Property Names

Độ khó: Trung bình

Đề bài:
Cho các array `fieldNames`, `hasSetter`, và `isFinalField` theo cùng một thứ tự, trả về list field name có thể được xem là writable property. Một field chỉ writable khi `hasSetter[i]` là `true` và `isFinalField[i]` là `false`.

Ví dụ 1:

Đầu vào:
```text
fieldNames = ["id", "name", "status"]
hasSetter = [false, true, true]
isFinalField = [true, false, false]
```

Đầu ra:
```text
["name", "status"]
```

Giải thích:
`id` bị loại vì nó là final và không có setter.

Ràng buộc:

- Tất cả array đều là non-null
- Tất cả array có cùng độ dài
- Trả về các name theo đúng thứ tự ban đầu

## Links

- [[002-Annotation]]
- [[004-Dynamic-proxy]]
- Java Reflection package summary: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/package-summary.html
- `Class` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html
- `Method` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/Method.html
- `Field` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/Field.html
- Spring classpath scanning: https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html
