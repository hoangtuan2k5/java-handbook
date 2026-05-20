# Unit Test vs Integration Test

## What is it

`Unit test` và `integration test` đều là automated test, nhưng chúng trả lời hai câu hỏi khác nhau.

`Unit test` hỏi: logic của một đơn vị nhỏ có đúng không nếu mình cô lập nó khỏi external boundary? `Integration test` hỏi: khi nhiều phần ghép lại qua boundary thật, hệ thống có còn đúng không?

Điểm quan trọng nhất không phải là test dài hay ngắn, mà là test đang đi qua boundary nào.

### Quick distinction

| Câu hỏi | `Unit test` | `Integration test` |
|---|---|---|
| Mục tiêu chính | kiểm tra logic cục bộ | kiểm tra wiring và collaboration thật |
| Dependency ngoài process | thường không có | thường có hoặc mô phỏng gần thật |
| Tốc độ | rất nhanh | chậm hơn |
| Khi fail | dễ khoanh vùng | đôi khi khó pinpoint hơn |
| Giá trị lớn nhất | feedback loop nhanh | confidence ở boundary dễ sai |

## How I used to misunderstand it

Mình từng nghĩ `unit test` là test nhỏ, còn `integration test` là test lớn hơn và chậm hơn.

Cách hiểu đó bỏ sót phần quan trọng nhất là ranh giới của test. Có những test rất ngắn nhưng vẫn là `integration test` vì nó đụng database thật, khởi động Spring context, hoặc đi qua HTTP serialization thật. Ngược lại, có `unit test` khá dài vì nó cover nhiều branch business logic, nhưng nó vẫn là unit vì mọi dependency bên ngoài đã được thay bằng fake, stub, hoặc mock.

Hiểu sai ở chỗ này dễ làm mình chọn sai mức test. Kết quả là suite hoặc quá chậm, hoặc quá giả lập nên không còn đáng tin.

## How it actually works

Mental model dễ nhớ là hỏi theo thứ tự sau:

```text
Test này đang xác nhận điều gì?
        |
        +--> Logic thuần của một class hoặc function? --> nghiêng về unit test
        |
        +--> Wiring giữa nhiều thành phần? -----------> nghiêng về integration test
        |
        +--> Có chạm boundary thật như DB, HTTP, file, Spring context? --> integration test
```

`Unit test` nên nhanh, deterministic, và ít phụ thuộc môi trường. Nó rất hợp để bắt bug ở business rule, edge case, validation, hoặc behavior nhỏ cần feedback liên tục.

`Integration test` chấp nhận tốn hơn để mua lấy độ tin cậy ở những chỗ wiring dễ sai, ví dụ query thật, transaction, JSON mapping, bean configuration, security filter, hoặc integration giữa service với repository.

Điểm đáng nhớ là hai loại này không cạnh tranh nhau.

- `unit test` giúp mình sửa logic với tốc độ cao
- `integration test` giúp mình phát hiện bug mà mock không thể nói thật

### Decision matrix

| Tình huống | Test level hợp hơn | Vì sao |
|---|---|---|
| Hàm tính giá, validate input, parse dữ liệu | `unit test` | logic thuần, không cần môi trường thật |
| Repository query, transaction, entity mapping | `integration test` | cần chứng minh behavior với DB và framework thật |
| Controller JSON request/response | `integration test` hoặc slice test | cần kiểm tra serialization, validation, status code |
| Service có nhiều branch business rule | `unit test` | cần feedback nhanh và cover edge case dày |
| Security filter chain hoặc configuration | `integration test` | wiring sai thường chỉ lộ khi chạy thật |

## Code example

```java
@Test
void applyDiscount_returnsFinalPrice() {
    PricingService service = new PricingService(new FlatTaxPolicy(0.1));

    BigDecimal result = service.applyDiscount(new BigDecimal("100"), 10);

    assertEquals(new BigDecimal("99.0"), result);
}
```

Đây là một `unit test` vì nó chỉ tập trung vào logic của `PricingService`, và dependency đi kèm cũng là object đơn giản trong memory.

Nếu cùng mục tiêu đó nhưng test phải khởi động Spring context, nạp bean thật, hoặc query database, test đã chuyển sang vùng `integration test`. Không phải vì nó dài hơn, mà vì boundary đã đổi.

## When to use / when NOT to use

Dùng `unit test` khi:

- cần feedback nhanh cho business logic
- muốn cover nhiều edge case nhỏ
- muốn refactor code an toàn mà không phụ thuộc môi trường ngoài

Dùng `integration test` khi:

- cần xác nhận wiring giữa nhiều layer
- cần kiểm tra query, transaction, serialization, security filter, hoặc configuration thật
- muốn chắc rằng ứng dụng vẫn hoạt động đúng ở boundary quan trọng

Không dùng `unit test` để giả lập quá nhiều framework behavior phức tạp. Khi test phải mock nửa hệ thống để tái hiện flow thật, đó thường là tín hiệu mình nên có `integration test` hoặc test slice.

Không dùng toàn `integration test` cho mọi branch logic nhỏ. Suite sẽ chậm, khó debug, và team sẽ ngại chạy test thường xuyên.

## How this connects to real Java projects

Trong Spring, phân biệt mức test còn quan trọng hơn vì framework cho rất nhiều điểm vào khác nhau.

- service thuần thường hợp với `unit test`
- `@WebMvcTest` hợp khi muốn kiểm tra web layer
- `@DataJpaTest` hợp khi muốn kiểm tra JPA mapping và query
- `@SpringBootTest` hợp khi thật sự cần context rộng hơn

Điểm hay là không phải lúc nào cũng cần full `@SpringBootTest`. Slice test thường là điểm cân bằng tốt giữa tốc độ và confidence.

Một mental model hữu ích là: Spring cho mình nhiều mức `integration`, không chỉ một mức duy nhất.

## Gotchas

- Gọi một test là `unit test` chỉ vì file nhỏ, trong khi nó vẫn chạm database thật, sẽ làm mình ước lượng sai tốc độ và độ ổn định của suite.
- Mock repository quá chi tiết trong service test có thể khiến test pass nhưng query thật ngoài đời vẫn sai.
- Integration test dùng shared state như cùng database record hoặc cùng port dễ bị flaky khi chạy song song.
- Một unit test quá khó viết thường là tín hiệu code đang dính quá nhiều dependency hoặc class đang làm quá nhiều việc.
- Không tách rõ mục tiêu test sẽ dẫn tới assertion mơ hồ, fail một cái là không biết lỗi nằm ở logic hay wiring.

## Check yourself

- Nếu một test chỉ có 15 dòng nhưng khởi động Spring context và gọi DB thật, nó thuộc mức nào?
- Vì sao tốc độ test không phải tiêu chí đủ để phân biệt `unit test` với `integration test`?
- Khi nào mock quá nhiều là tín hiệu boundary của test đang đặt sai?
- Vì sao `@WebMvcTest` và `@DataJpaTest` thường hữu ích hơn full `@SpringBootTest` cho nhiều trường hợp?
- Một suite chỉ có unit test sẽ bỏ sót loại bug nào?

## Exercises

### Bài 1: Choose Test Level

Độ khó: Dễ

Đề bài:
Cho ba cờ `touchesRealDatabase`, `touchesNetwork`, và `needsApplicationContext`. Trả về `"integration"` nếu bất kỳ cờ nào là `true`. Ngược lại trả về `"unit"`.

Ví dụ 1:

Đầu vào:
```text
touchesRealDatabase = false
touchesNetwork = false
needsApplicationContext = false
```

Đầu ra:
```text
"unit"
```

Giải thích:
Test này không đụng external boundary nào, nên nó là unit test.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"unit"` hoặc `"integration"`
- Không cần khởi động framework thật

### Bài 2: Count Database Touching Scenarios

Độ khó: Dễ

Đề bài:
Cho `boolean[] touchesDatabase`, trong đó mỗi phần tử biểu diễn một test scenario có đụng database thật hay không. Hãy trả về số scenario có giá trị `true`.

Ví dụ 1:

Đầu vào:
```text
touchesDatabase = [false, true, false, true, true]
```

Đầu ra:
```text
3
```

Giải thích:
Có ba scenario dùng database thật, nên đây là ba candidate cho integration test.

Ràng buộc:

- `0 <= touchesDatabase.length <= 100000`
- Mỗi phần tử là `true` hoặc `false`
- Kết quả phải vừa trong kiểu `int`

### Bài 3: Find First External Dependency

Độ khó: Trung bình

Đề bài:
Cho `String[] dependencyTypes`, trong đó mỗi phần tử là một trong các giá trị `"in-memory"`, `"database"`, `"network"`, hoặc `"filesystem"`. Hãy trả về index đầu tiên không phải `"in-memory"`. Nếu mọi dependency đều là `"in-memory"`, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
dependencyTypes = ["in-memory", "in-memory", "database", "network"]
```

Đầu ra:
```text
2
```

Giải thích:
`"database"` là dependency đầu tiên vượt ra khỏi in-memory boundary.

Ràng buộc:

- `0 <= dependencyTypes.length <= 100000`
- Mỗi chuỗi chỉ thuộc tập giá trị đã cho
- So sánh chuỗi có phân biệt hoa thường

## Links

- [[002-JUnit5]]
- [[003-Mockito]]
- [Martin Fowler, The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Spring Boot Testing](https://docs.spring.io/spring-boot/reference/testing/index.html)
- [Spring Framework Testing Reference](https://docs.spring.io/spring-framework/reference/testing.html)


