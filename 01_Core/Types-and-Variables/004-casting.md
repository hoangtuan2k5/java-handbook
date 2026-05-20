# Casting

## What is it

Casting là việc chuyển một value hoặc reference từ type này sang type khác trong Java. Nó xuất hiện ở numeric conversion như `long` sang `int`, `double` sang `int`, và reference conversion như cast từ superclass sang subclass.

Điểm quan trọng là casting không chỉ là syntax. Nó là quyết định về semantics dữ liệu. Có cast an toàn, có cast làm mất dữ liệu, và có cast compile được nhưng fail ở runtime.

## How I used to misunderstand it

Mình từng nghĩ cứ thêm `(int)` hay `(SomeType)` vào trước value là mọi thứ ổn. Compiler hết kêu thì coi như xong.

Thực tế compiler chỉ xác nhận phép cast đó hợp lệ về mặt ngôn ngữ. Nó không bảo đảm dữ liệu còn đúng nghĩa sau khi đổi type, và cũng không bảo đảm reference cast sẽ an toàn ở runtime.

## How it actually works

Với numeric types, Java có widening conversion và narrowing conversion.

* Widening như `int` sang `long` thường an toàn hơn vì miền giá trị rộng hơn.
* Narrowing như `long` sang `int` hoặc `double` sang `int` có thể mất dữ liệu, mất phần thập phân, hoặc overflow.

Ví dụ `(int) 3.9` cho ra `3`, không làm tròn. Còn cast một `long` quá lớn sang `int` có thể cho kết quả âm hoặc giá trị hoàn toàn khác vì bit bị cắt.

Với reference types, cast không đổi object thật sự là gì. Nó chỉ nói với compiler rằng mình tin object hiện tại có type phù hợp. Nếu niềm tin đó sai, runtime sẽ ném `ClassCastException`.

### Comparison table

| Loại cast | Ví dụ | Có thể mất dữ liệu không | Có thể fail ở runtime không |
|---|---|---|---|
| Numeric widening | `int` -> `long` | Thường không | Không |
| Numeric narrowing | `double` -> `int` | Có | Không, nhưng có thể ra value sai nghĩa |
| Reference upcast | `Dog` -> `Animal` | Không | Không |
| Reference downcast | `Animal` -> `Dog` | Không theo nghĩa dữ liệu, nhưng assumption có thể sai | Có |

### Tiny mental scaffold

```text
Need smaller numeric type? -> validate range first
Need subclass reference?   -> be sure object really is that subclass
```

## Code example

```java
double price = 19.99;
int truncated = (int) price;

long big = 3_000_000_000L;
int overflowed = (int) big;

System.out.println(truncated); // 19
// value bị đổi vì overflow
System.out.println(overflowed);
```

Ví dụ này cho thấy cast không hề miễn phí về nghĩa dữ liệu. Compiler cho phép, nhưng responsibility vẫn thuộc về người viết code.

## When to use / when NOT to use

Use cast khi:

* bạn hiểu rõ miền giá trị trước và sau khi chuyển
* API thật sự yêu cầu type khác
* bạn đang chủ động truncate, clamp, hoặc convert theo rule rõ ràng

Không cast chỉ để compiler im lặng nếu bạn chưa chắc dữ liệu có phù hợp không.

Không dùng narrowing cast trên dữ liệu đến từ external system mà không validation trước. Khi đó bug thường rất khó truy vết vì value vẫn tồn tại nhưng đã sai nghĩa.

Không xem `(int)` như bước “làm sạch” dữ liệu. Nó chỉ đổi representation, không tự kiểm tra business rule cho bạn.

## How this connects to real Java projects

Trong Spring app, casting xuất hiện ở data binding, JSON numeric conversion, JDBC result mapping, và khi làm việc với API trả `Object`. Nếu cast sai trong service hoặc mapper, lỗi có thể chỉ nổ ở runtime sau khi request thật chạy qua.

Một chỗ hay gặp khác là đọc config hoặc payload rồi ép sang type nhỏ hơn. Nếu business rule cần `int` nhưng input là `long`, nên validate range rõ ràng trước khi cast.

## Gotchas

* `(int) someDouble` là truncate, không phải round.
* Narrowing cast có thể overflow mà không ném exception.
* Reference cast compile được chưa có nghĩa là runtime sẽ an toàn.
* Cast có thể che đi bug ở tầng trước, ví dụ source data đã sai type từ đầu.

## Check yourself

* Vì sao `(int) 3.9` ra `3` chứ không phải `4`?
* Vì sao cast `long` lớn sang `int` có thể cho kết quả âm mà không ném exception?
* Numeric cast và reference cast khác nhau ở kiểu risk nào?
* Nếu nhận `Object value` từ framework, vì sao downcast luôn cần assumption đúng về runtime type?
* Trước khi narrowing cast dữ liệu từ request hoặc DB, bạn nên kiểm tra điều gì?

## Exercises

### Exercise 1: Clamp Long To Int

Độ khó: Dễ

Đề bài:
Cho một `long value`. Hãy trả về value đó dưới dạng `int`, nhưng nếu nó nhỏ hơn `Integer.MIN_VALUE` thì trả `Integer.MIN_VALUE`, còn nếu lớn hơn `Integer.MAX_VALUE` thì trả `Integer.MAX_VALUE`.

Ví dụ 1:

Đầu vào:
```text
value = 3000000000
```

Đầu ra:
```text
2147483647
```

Giải thích:
`3000000000` vượt quá miền của `int`, nên cần clamp về `Integer.MAX_VALUE` trước khi cast.

Ràng buộc:

* `-10^18 <= value <= 10^18`
* Phải trả về đúng miền `int`
* Không được để overflow âm thầm làm sai kết quả

### Bài 2: Count Fraction Loss

Độ khó: Trung bình

Đề bài:
Cho `double[] values`. Hãy đếm xem có bao nhiêu phần tử sẽ bị mất phần thập phân nếu cast trực tiếp sang `int`.

Ví dụ 1:

Đầu vào:
```text
values = [1.0, 2.5, -3.2, 4.0]
```

Đầu ra:
```text
2
```

Giải thích:
`2.5` và `-3.2` sẽ bị truncate phần thập phân khi cast sang `int`.

Ràng buộc:

* `0 <= values.length <= 100000`
* `-10^9 <= values[i] <= 10^9`
* Không có `NaN` hoặc infinite value

### Bài 3: Decode Ascii Codes

Độ khó: Trung bình

Đề bài:
Cho `int[] codes`, mỗi phần tử nằm trong khoảng mã ASCII in được từ `32` đến `126`. Hãy cast từng số sang `char` và trả về string được ghép theo đúng thứ tự.

Ví dụ 1:

Đầu vào:
```text
codes = [74, 97, 118, 97]
```

Đầu ra:
```text
"Java"
```

Giải thích:
`74 -> 'J'`, `97 -> 'a'`, `118 -> 'v'`, `97 -> 'a'`.

Ràng buộc:

* `0 <= codes.length <= 100000`
* `32 <= codes[i] <= 126`
* Các mã phải được giữ nguyên thứ tự khi build kết quả

## Links

* [[001-Primitive-vs-Wrapper]]
* [[003-var-type-inference]]
* [[../Data-Types-Advanced/004-pattern-matching]]
* [JLS §5, Conversions and Contexts](https://docs.oracle.com/javase/specs/jls/se21/html/jls-5.html)
* [Java Tutorials, Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)
* [Java SE 21, `ClassCastException` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ClassCastException.html)

