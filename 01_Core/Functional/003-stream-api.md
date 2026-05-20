# stream api

## What is it

`Stream API` là cách mô tả một pipeline xử lý dữ liệu theo từng bước như `filter`, `map`, `sorted`, `reduce`, `collect`.

Mental model quan trọng nhất là: stream **không phải data structure chứa dữ liệu**. Nó là một **kế hoạch xử lý** đi qua nguồn dữ liệu.

Bạn đang nói rõ:

- lấy dữ liệu từ đâu
- lọc gì
- biến đổi gì
- gom kết quả ra sao

## How I used to misunderstand it

Mình từng nghĩ stream tự động nhanh hơn loop vì nó “functional” hơn.

Không đúng. Stream giúp code declarative hơn, nhưng không có nghĩa luôn nhanh hơn, luôn dễ debug hơn, hay luôn ít bug hơn. Một hiểu nhầm khác là thấy viết `filter().map()` rồi tưởng code đã chạy. Thực ra nhiều stage là lazy. Không có terminal operation thì pipeline có thể chưa làm gì cả.

## How it actually works

Một stream pipeline thường có ba phần:

1. **source** như `list.stream()`
2. **intermediate operations** như `filter`, `map`, `sorted`
3. **terminal operation** như `toList`, `collect`, `count`, `forEach`

Intermediate operations thường lazy. Terminal operation mới là thứ kéo dữ liệu chạy qua pipeline.

### Pipeline mental model

```text
source -> filter -> map -> collect
```

Hoặc nghĩ chi tiết hơn:

```text
collection
    -> stream view
    -> transform step by step
    -> terminal operation asks for result
    -> elements flow through the pipeline
```

### Stream và Collector khác nhau chỗ nào

| Phần | Vai trò |
|---|---|
| `Stream` | mô tả pipeline xử lý từng phần tử |
| Lambda / method reference | mô tả từng rule nhỏ trong pipeline |
| `Collector` | mô tả cách gom kết quả cuối |

Đây là mental model rất đáng nhớ cho cả folder Functional. Stream lo **đường đi**, collector lo **đích đến**.

## Code example

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> result = List.of("linh", "an", "binh").stream()
                .filter(name -> name.length() >= 3)
                .map(String::toUpperCase)
                .toList();

        System.out.println(result); // [LINH, BINH]
    }
}
```

## When to use / when NOT to use

Dùng stream khi bài toán là một pipeline rõ ràng kiểu filter, map, group, reduce, collect và side effect ít.

Không nên dùng stream chỉ để thay mọi loop. Nếu logic cần mutation nhiều state ngoài, break sớm phức tạp, hoặc debug từng bước là chuyện quan trọng, loop imperative thường vẫn dễ đọc hơn.

Stream tốt khi giúp bạn nói rõ ý định. Nếu pipeline dài tới mức người đọc phải giải mã từng stage, lợi thế đó mất đi rất nhanh.

## How this connects to real Java projects

Trong Spring Boot, Stream API hay dùng để map entity sang DTO, filter response list, aggregate dữ liệu nhỏ trong memory, hoặc transform config/result set.

Điểm cần tỉnh táo là stream Java không thay thế query optimization. Nếu dữ liệu đến từ database và tập dữ liệu lớn, thường nên đẩy filter/group/sort xuống database trước thay vì load hết rồi mới stream trong app.

## Gotchas

- Không có terminal operation thì nhiều pipeline chưa chạy gì cả.
- Side effect trong `map` hoặc `filter` làm stream khó reasoning hơn nhiều.
- Stream chỉ dùng một lần.
- Parallel stream không phải nút “làm nhanh hơn” mặc định.

## Handbook rule

- Dùng stream cho pipeline rõ filter/map/group/reduce; tránh dùng cho logic mutation phức tạp.
- Mọi pipeline phải có terminal operation; không có thì không chạy gì.
- Tránh side effect trong `map`/`filter`; reasoning sẽ vỡ rất nhanh.
- Stream dùng một lần; muốn replay phải tạo lại từ source.
- `parallelStream()` không mặc định nhanh hơn; chỉ dùng khi đo được lợi ích thật.

## Check yourself

- Vì sao nói stream là kế hoạch xử lý chứ không phải container dữ liệu?
- Intermediate operation và terminal operation khác nhau thế nào?
- Trong mental model của folder này, collector khác stream ở điểm nào?
- Khi nào loop thường dễ đọc hơn stream?
- Vì sao side effect trong stream hay gây ngạc nhiên?

## Exercises

### Bài 1: Filter Active Users
Độ khó: Dễ

Đề bài:
Cho một list các user status, chỉ trả về những value bằng `"ACTIVE"`.

Ví dụ 1:
Đầu vào:
```text
statuses = ["ACTIVE", "DISABLED", "ACTIVE"]
```

Đầu ra:
```text
["ACTIVE", "ACTIVE"]
```

Giải thích:
Chỉ các value khớp filter condition mới được giữ lại.

Ràng buộc:
- 0 <= statuses.length <= 100000
- statuses[i] là non-null
- Giữ nguyên encounter order

### Bài 2: Map Orders To Total Text
Độ khó: Trung bình

Đề bài:
Cho một list các order total, trả về một list string theo format `"total=<value>"`.

Ví dụ 1:
Đầu vào:
```text
totals = [10, 20]
```

Đầu ra:
```text
["total=10", "total=20"]
```

Giải thích:
Mỗi số được ánh xạ sang một text representation.

Ràng buộc:
- 0 <= totals.length <= 100000
- totals[i] là non-null
- Kích thước output phải khớp với kích thước input

### Bài 3: Sum Valid Scores With Pipeline
Độ khó: Trung bình

Đề bài:
Cho một list các score, chỉ cộng những score nằm trong đoạn từ `0` đến `100` inclusive.

Ví dụ 1:
Đầu vào:
```text
scores = [80, -1, 50, 200]
```

Đầu ra:
```text
130
```

Giải thích:
Chỉ những valid score mới được filter vào trước bước tính tổng.

Ràng buộc:
- 0 <= scores.length <= 100000
- scores[i] là non-null
- Bỏ qua giá trị không hợp lệ

## Links

- [[001-lambda]]
- [[004-optional]]
- [[006-collector-and-grouping-by]]
- `Stream` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html
- `Collectors` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html
