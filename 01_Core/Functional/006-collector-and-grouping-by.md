# collector and groupingBy

## What is it

`Collector` là cơ chế gom kết quả stream về một hình dạng cuối như `List`, `Set`, `Map`, summary, hoặc custom aggregation.

`groupingBy` là collector rất hay dùng để nhóm phần tử theo key.

Mental model nên nhớ:

- stream lo pipeline đi qua từng item
- collector lo kết quả cuối cùng sẽ trông như thế nào
- `groupingBy` lo câu hỏi “gom theo key nào”

## How I used to misunderstand it

Mình từng xem `groupingBy` như một phép màu gom nhóm tự động.

Kết quả là viết xong rồi mới ngạc nhiên vì output là `Map<K, List<T>>`, hoặc không hiểu khi nào cần `counting()`, `mapping()`, `summingInt()` làm downstream collector. `groupingBy` chỉ thật sự rõ khi bạn luôn hỏi ba câu:

1. group theo key nào
2. value trong mỗi group là gì
3. cuối cùng muốn aggregate mỗi group thành gì

## How it actually works

Collector là contract mà terminal operation `collect(...)` dùng để xây kết quả cuối.

Một vài collector phổ biến:

- `toList()`
- `toSet()`
- `joining()`
- `counting()`
- `mapping()`
- `groupingBy()`

### `groupingBy` mặc định trả gì

| Cú pháp | Kết quả |
|---|---|
| `groupingBy(classifier)` | `Map<K, List<T>>` |
| `groupingBy(classifier, counting())` | `Map<K, Long>` |
| `groupingBy(classifier, mapping(..., toList()))` | `Map<K, List<R>>` |
| `groupingBy(classifier, summingInt(...))` | `Map<K, Integer>` |

### Mental flow

```text
stream elements
    -> classify each element by key
    -> put into the right group
    -> optionally aggregate inside each group
    -> return final map
```

Đây là chỗ mental model của `Stream API` và `Collector` gặp nhau rất đẹp. Stream mô tả đường đi. Collector mô tả đích gom.

## Code example

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        Map<Integer, Long> countByLength = List.of("an", "linh", "binh").stream()
                .collect(Collectors.groupingBy(String::length, Collectors.counting()));

        System.out.println(countByLength); // {2=1, 4=1, 5=1}
    }
}
```

## When to use / when NOT to use

Dùng collector khi bài toán kết thúc bằng một aggregation rõ như list, set, map, grouped map, count, summary.

Dùng `groupingBy` khi grouping key là trung tâm của bài toán.

Không nên cố nhét mọi aggregation vào một collector lồng nhau nếu một loop đơn giản sẽ dễ hiểu hơn nhiều. Khi pipeline có nhiều tầng `groupingBy` và downstream collector sâu, hãy cân nhắc tách bước hoặc quay về loop.

## How this connects to real Java projects

Trong Spring Boot, collector và `groupingBy` hay dùng khi aggregate dữ liệu nhỏ trong memory như response DTO, report nhỏ, hay config objects.

Nếu grouping/summing xảy ra trên tập dữ liệu lớn từ database, thường nên cân nhắc để database làm aggregation thay vì kéo hết dữ liệu lên app rồi mới group.

## Gotchas

- `groupingBy` mặc định trả `Map<K, List<T>>`, không phải count hay set.
- Kết quả map mặc định thường là `HashMap`.
- Collector pipeline quá lồng nhau rất khó debug.
- Grouping trong memory không thay thế query aggregation khi dữ liệu lớn.

## Handbook rule

- Mỗi pipeline kết bằng đúng một collector phù hợp: list, set, map, count, joining.
- `groupingBy` mặc định trả `Map<K, List<T>>`; specify downstream khi cần count/sum/sorted/etc.
- Map mặc định là `HashMap`; cần order/sorted phải truyền `LinkedHashMap`/`TreeMap` factory.
- Collector lồng nhau quá sâu là tín hiệu nên tách step hoặc quay về loop.
- Grouping in-memory không thay thế query aggregation khi dữ liệu lớn.

## Check yourself

- Stream và collector khác nhau ở vai trò nào?
- Vì sao `groupingBy(classifier)` mặc định cho `Map<K, List<T>>`?
- Khi nào cần downstream collector như `counting()`?
- Nếu muốn count theo nhóm thay vì giữ list gốc, bạn đổi phần nào của collector?
- Khi nào một loop thủ công lại rõ hơn một collector lồng nhau?

## Exercises

### Bài 1: Group Names By First Letter
Độ khó: Dễ

Đề bài:
Cho một list các tên, trả về một map nhóm chúng theo chữ cái đầu tiên.

Ví dụ 1:
Đầu vào:
```text
names = ["An", "Binh", "Bao"]
```

Đầu ra:
```text
{A=[An], B=[Binh, Bao]}
```

Giải thích:
Các tên được nhóm theo ký tự đầu tiên làm key.

Ràng buộc:
- 0 <= names.length <= 100000
- names[i] là non-null và non-empty
- Giữ nguyên encounter order trong từng group

### Bài 2: Count Orders By Status
Độ khó: Trung bình

Đề bài:
Cho một list các order status, trả về một map từ status sang số lần xuất hiện.

Ví dụ 1:
Đầu vào:
```text
statuses = ["PAID", "CREATED", "PAID"]
```

Đầu ra:
```text
{PAID=2, CREATED=1}
```

Giải thích:
Mỗi group được aggregate bằng cách đếm số phần tử của nó.

Ràng buộc:
- 0 <= statuses.length <= 100000
- statuses[i] là non-null
- Không yêu cầu thứ tự của output

### Bài 3: Group Users By Age Band
Độ khó: Trung bình

Đề bài:
Cho một list các age của user, nhóm chúng vào các band `minor`, `adult`, và `senior`, rồi trả về grouped map.

Ví dụ 1:
Đầu vào:
```text
ages = [15, 21, 67, 18]
```

Đầu ra:
```text
{minor=[15], adult=[21, 18], senior=[67]}
```

Giải thích:
Mỗi age được phân loại vào đúng một grouping key.

Ràng buộc:
- 0 <= ages.length <= 100000
- age >= 0
- Giữ nguyên encounter order bên trong từng group

## Links

- [[003-stream-api]]
- [[004-optional]]
- [[../Collections/003-hash-map]]
- `Collector` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collector.html
- `Collectors.groupingBy` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html
