# Strong vs Soft vs Weak vs Phantom Reference

## What is it

Java có nhiều loại reference để mô tả mức độ một reference giữ object sống:

- `strong`
- `soft`
- `weak`
- `phantom`

Mental model:

- strong reference là cách giữ object sống mặc định
- soft và weak cho GC nhiều quyền thu hồi hơn
- phantom không dùng để lấy lại object, mà để theo dõi giai đoạn sau khi object đã không còn usable theo cách thông thường

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là nghĩ weak hoặc soft reference là “cache miễn phí”. Thực tế behavior của chúng phụ thuộc vào GC và memory pressure, nên timing không ổn định như business cache có TTL hoặc size limit.

Hiểu nhầm thứ hai là nghĩ phantom reference là kiểu “reference yếu nhất để lấy object khi sắp chết”. Sai. `PhantomReference#get()` luôn trả `null`. Nó không dùng để truy cập object, mà dùng cho cleanup hoặc tracking rất low-level qua `ReferenceQueue`.

## How it actually works

### Bảng so sánh nhanh

| Reference type | Có tự giữ object sống không? | Khi nào object có thể bị GC? | Dùng cho gì |
|---|---|---|---|
| Strong | Có | Chỉ khi mọi strong path biến mất | Cách dùng mặc định hằng ngày |
| Soft | Không giữ chắc như strong | Thường khi JVM cần memory hơn | Một số cache mang tính best-effort |
| Weak | Không | Khi chỉ còn weak references | Metadata, canonical map, observer-like patterns |
| Phantom | Không | Sau khi object đã đến giai đoạn reclaim tracking | Cleanup low-level với `ReferenceQueue` |

### Tiny mental ladder

```text
Giữ object mạnh nhất

strong
  |
soft
  |
weak
  |
phantom

GC càng có nhiều quyền thu hồi hơn khi đi xuống dưới.
```

`Strong reference` là reference bình thường bạn dùng hằng ngày, ví dụ biến local, field, element trong collection.

`WeakReference` không ngăn GC. Nếu object chỉ còn weak references, GC có thể clear nó ở chu kỳ phù hợp.

`SoftReference` thường được giữ lâu hơn weak, nhưng không nên xem đó là một SLA. JVM không hứa “còn bao lâu nữa mới clear”.

`PhantomReference` thường đi cùng `ReferenceQueue`. Nó không cho bạn đọc object, mà chỉ giúp biết rằng object đã đi tới giai đoạn cần xử lý cleanup ngoài heap hoặc bookkeeping low-level.

## Code example

```java
import java.lang.ref.WeakReference;

Object value = new Object();
WeakReference<Object> weak = new WeakReference<>(value);

// giờ object có thể collectible nếu không còn strong reference nào khác
value = null;
Object maybeStillThere = weak.get();
```

Điều cần nhớ ở đây là `maybeStillThere` có thể còn object, hoặc có thể thành `null` sau một lần GC trong tương lai. Timing không ổn định để dựa vào cho business logic quan trọng.

## When to use / when NOT to use

Mặc định, hãy dùng strong reference.

Chỉ cân nhắc weak reference khi bạn thật sự muốn lifetime của dữ liệu phụ thuộc vào owner object, ví dụ metadata không nên kéo dài đời của key.

Soft reference chỉ nên dùng khi bạn chấp nhận retention không deterministic. Nó không thay thế cho cache policy rõ ràng.

Phantom reference rất hiếm khi cần trong application code thông thường. Nếu chưa biết chắc vì sao cần nó, gần như bạn chưa cần dùng nó.

## How this connects to real Java projects

Phần lớn Spring developer không cần tự tạo `SoftReference`, `WeakReference`, hay `PhantomReference` mỗi ngày. Nhưng hiểu chúng rất hữu ích khi đọc library internals, phân tích classloader leak, hoặc hiểu vì sao một số metadata map không giữ object sống mãi.

Nó cũng giúp tránh một sai lầm phổ biến: dùng reference types để né việc thiết kế cache hoặc lifecycle cho ra hồn.

## Gotchas

- Weak reference có thể biến mất ngay sau khi object mất strong path.
- Soft reference không phải cache policy ổn định.
- `PhantomReference#get()` luôn trả `null`.
- Reference types giúp GC linh hoạt hơn, nhưng không sửa được thiết kế ownership kém.

## Check yourself

- Vì sao strong reference là mặc định an toàn nhất?
- Weak reference khác soft reference ở trực giác sử dụng như thế nào?
- Tại sao không nên dùng soft hoặc weak reference để thay thế hoàn toàn cho eviction policy?
- Phantom reference dùng để truy cập object hay để theo dõi lifecycle low-level?
- Khi business cần dữ liệu phải tồn tại ổn định, vì sao weak reference là lựa chọn nguy hiểm?

## Exercises

### Bài 1: Classify Reference Strength
Độ khó: Dễ

Đề bài:
Cho một reference type, trả về `"keeps-alive"` cho `"strong"`; ngược lại trả về `"collectable"`.

Ví dụ 1:
Đầu vào:
```text
referenceType = "weak"
```

Đầu ra:
```text
"collectable"
```

Giải thích:
Weak reference tự nó không giữ object sống.

Ràng buộc:
- referenceType is one of `"strong"`, `"soft"`, `"weak"`, `"phantom"`
- Chỉ trả về các label đã được liệt kê
- So khớp có phân biệt chữ hoa/thường

### Bài 2: Có nên dùng weak metadata
Độ khó: Trung bình

Đề bài:
Cho biết metadata có nên biến mất cùng owner hay không và việc lookup có cần deterministic hay không, trả về `true` nếu weak metadata là phù hợp.

Ví dụ 1:
Đầu vào:
```text
disappearWithOwner = true, deterministicLookup = false
```

Đầu ra:
```text
true
```

Giải thích:
Weak metadata phù hợp khi lifetime đi theo owner và deterministic retention không phải là yêu cầu bắt buộc.

Ràng buộc:
- Input là các giá trị boolean
- Chỉ trả về `true` khi cần owner lifetime và không yêu cầu deterministic lookup
- Không mô hình hóa hành vi của `ReferenceQueue`

### Bài 3: Đếm strong reference
Độ khó: Dễ

Đề bài:
Cho các reference type label, đếm xem có bao nhiêu label là `"strong"`.

Ví dụ 1:
Đầu vào:
```text
types = ["strong", "weak", "strong"]
```

Đầu ra:
```text
2
```

Giải thích:
Hai label biểu diễn strong reference thông thường.

Ràng buộc:
- types là non-null
- 0 <= types.length <= 100000
- Mỗi type là non-null

## Links

- [[001-GC]]
- [[002-Memory-leak]]
- [[006-off-heap-memory]]
- [Java SE 21, Package `java.lang.ref`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/package-summary.html)
- [Java SE 21, `Reference` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/Reference.html)
- [Java SE 21, `ReferenceQueue` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/ReferenceQueue.html)

