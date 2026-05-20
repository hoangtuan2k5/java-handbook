# JUnit 5

## What is it

`JUnit 5` là testing platform hiện đại cho Java. Nó cung cấp cách viết test rõ ràng hơn, lifecycle linh hoạt hơn, và extension model mạnh hơn so với JUnit cũ.

Khi nói `JUnit 5`, thực ra mình đang nói tới ba phần:

- `JUnit Platform` để discover và chạy test
- `JUnit Jupiter` để viết test mới
- `JUnit Vintage` để chạy test cũ theo kiểu JUnit 3 hoặc 4

Trong thực tế học Java và Spring Boot, phần mình dùng nhiều nhất là `Jupiter`.

### Core pieces

| Thành phần | Vai trò | Khi nào mình để ý tới nó |
|---|---|---|
| `Platform` | chạy test engine | khi IDE, Maven, Gradle discover test |
| `Jupiter` | API và engine hiện đại | gần như mọi test mới |
| `Vintage` | tương thích test cũ | chỉ khi project còn JUnit 4 |

## How I used to misunderstand it

Mình từng nghĩ `JUnit 5` chỉ là bản mới của `@Test`, khác mỗi cú pháp annotation.

Về sau mới thấy phần đáng giá hơn nằm ở test lifecycle, parameterized test, nested test, tagging, và extension model. Nếu chỉ xem nó là bản rename của JUnit 4 thì mình sẽ bỏ lỡ nhiều cách viết test sạch hơn và dễ reason hơn.

Mình cũng từng đặt quá nhiều setup vào `@BeforeEach`, khiến test đọc lên không còn thấy hành vi chính nằm ở đâu. `JUnit 5` mạnh, nhưng sức mạnh đó chỉ có ích khi mình giữ từng test đủ nhỏ và rõ.

## How it actually works

Mental model hữu ích là xem `JUnit 5` như bộ điều phối lifecycle của test class.

```text
Test engine discovers class
        |
        v
create test instance
        |
        +--> @BeforeAll     (một lần cho class)
        +--> @BeforeEach    (mỗi test)
        +--> @Test          (hoặc @ParameterizedTest)
        +--> @AfterEach     (mỗi test)
        +--> @AfterAll      (một lần cho class)
```

Khi hiểu flow này, mình sẽ biết state nào nên tạo mới cho từng test, state nào có thể chia sẻ, và vì sao test độc lập luôn dễ tin cậy hơn test phụ thuộc lẫn nhau.

### Annotation cheat sheet

| Annotation | Dùng để làm gì | Lưu ý |
|---|---|---|
| `@Test` | test case cơ bản | mặc định tốt nhất cho case đơn giản |
| `@ParameterizedTest` | chạy cùng một test với nhiều input | chỉ nên dùng khi nhiều case thật sự cùng intent |
| `@BeforeEach` | setup cho từng test | đừng giấu quá nhiều logic ở đây |
| `@AfterEach` | cleanup cho từng test | hữu ích khi có resource tạm |
| `@BeforeAll` / `@AfterAll` | setup hoặc cleanup dùng chung | nên dùng tiết kiệm |
| `@Nested` | nhóm test theo ngữ cảnh | hợp khi có state hoặc scenario rõ ràng |
| `@Tag` | nhóm test cho build hoặc CI | chỉ có giá trị nếu team thật sự dùng |
| `@DisplayName` | diễn đạt tên test dễ đọc | không thay thế method name rõ ràng |

`assertThrows` cũng là một phần của mental model tốt. Nó làm expectation về error path trở nên explicit. Điều này quan trọng vì code production thường hỏng ở exception path nhiều không kém happy path.

## Code example

```java
@DisplayName("discount is rejected when percentage is negative")
@Test
void rejectNegativeDiscount() {
    PricingService service = new PricingService();

    IllegalArgumentException exception = assertThrows(
            IllegalArgumentException.class,
            () -> service.applyDiscount(new BigDecimal("100"), -1));

    assertEquals("discount must be between 0 and 100", exception.getMessage());
}
```

Ví dụ này cho thấy `JUnit 5` không chỉ chạy method test. Nó còn giúp mình diễn đạt hành vi mong đợi rõ hơn, nhất là ở error path.

Nếu viết cùng test bằng `try-catch` thủ công, intent sẽ mờ hơn và dễ quên assert message hoặc type exception.

## When to use / when NOT to use

Dùng `JUnit 5` khi:

- viết unit test hoặc integration test trong Java hiện đại
- cần parameterized test để cover nhiều input gọn hơn
- muốn dùng extension model rõ ràng hơn cho framework integration

Không lạm dụng annotation chỉ để làm test trông fancy hơn. Nếu một test đơn giản chỉ cần `@Test`, đừng ép nó thành `@ParameterizedTest` hoặc nested structure phức tạp.

Không đặt quá nhiều logic vào lifecycle hook. Khi `@BeforeEach` làm quá nhiều việc, phần quan trọng của test bị giấu mất và reader phải nhảy qua lại mới hiểu được case đang kiểm tra gì.

## How this connects to real Java projects

Spring Boot hiện đại mặc định đi cùng `JUnit 5`. Rất nhiều test slice của Spring như `@WebMvcTest`, `@DataJpaTest`, và `@SpringBootTest` đều hoạt động tự nhiên với `Jupiter`.

`SpringExtension` cũng dựa trên extension model của `JUnit 5`. Nghĩa là khi hiểu `JUnit 5`, mình không chỉ hiểu framework test chung, mà còn hiểu cách Spring gắn test context vào lifecycle của test class.

Một lợi thế thực tế là khi mình phân biệt rõ `@BeforeEach` của JUnit và setup do Spring context cung cấp, mình sẽ reason được state nằm ở đâu, cái gì được tạo lại, và cái gì đang được cache giữa các test.

## Gotchas

- Dùng field mutable chung giữa các test mà quên reset có thể tạo state leak rất khó thấy.
- `@DisplayName` đẹp nhưng nếu method name quá mơ hồ thì khi IDE search code vẫn khó hiểu.
- `@Tag` chỉ có giá trị khi build hoặc CI thật sự dùng nó.
- `assertThrows` chỉ kiểm tra đoạn code nằm trong lambda. Đặt nhầm code bên ngoài sẽ khiến test đánh giá sai.
- Parameterized test với quá nhiều input có thể che mất intent nếu không đặt tên case rõ ràng.
- `@BeforeAll` dễ biến thành nơi khởi tạo state chia sẻ quá rộng, rồi test bắt đầu phụ thuộc nhau mà mình không để ý.

## Check yourself

- `JUnit Platform`, `Jupiter`, và `Vintage` khác nhau ở vai trò nào?
- Vì sao hiểu `@BeforeEach` và `@AfterEach` như một lifecycle loop lại quan trọng?
- Khi nào `@ParameterizedTest` giúp code gọn hơn, và khi nào nó làm intent mờ đi?
- Vì sao `assertThrows` thường tốt hơn `try-catch` thủ công trong test?
- Một test dùng `@Tag` nhưng build không filter theo tag thì annotation đó còn giá trị gì?

## Exercises

### Bài 1: Count Enabled Tests

Độ khó: Dễ

Đề bài:
Cho `boolean[] disabledFlags`, trong đó mỗi phần tử biểu diễn một test có bị disable hay không. Hãy trả về số test đang enabled.

Ví dụ 1:

Đầu vào:
```text
disabledFlags = [false, true, false, false]
```

Đầu ra:
```text
3
```

Giải thích:
Chỉ có một test bị disable, nên còn ba test được chạy.

Ràng buộc:

- `0 <= disabledFlags.length <= 100000`
- Mỗi phần tử là boolean
- Kết quả phải vừa trong kiểu `int`

### Bài 2: Find Duplicate Display Name

Độ khó: Trung bình

Đề bài:
Cho `String[] displayNames`. Hãy trả về index đầu tiên mà `displayNames[i]` đã xuất hiện trước đó. Nếu không có tên nào trùng, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
displayNames = ["creates user", "rejects duplicate email", "creates user"]
```

Đầu ra:
```text
2
```

Giải thích:
Tên `"creates user"` xuất hiện lần thứ hai ở index `2`.

Ràng buộc:

- `0 <= displayNames.length <= 100000`
- Mỗi tên có độ dài từ `1` đến `100`
- So sánh chuỗi có phân biệt hoa thường

### Bài 3: Validate Lifecycle Order

Độ khó: Trung bình

Đề bài:
Cho `String[] events`, trong đó mỗi phần tử là một trong các giá trị `"beforeAll"`, `"beforeEach"`, `"test"`, `"afterEach"`, `"afterAll"`. Trả về `true` nếu dãy event thỏa pattern sau: bắt đầu bằng đúng một `"beforeAll"`, kết thúc bằng đúng một `"afterAll"`, và ở giữa chỉ chứa một hoặc nhiều block con có thứ tự `"beforeEach"`, `"test"`, `"afterEach"`. Nếu không thỏa, trả về `false`.

Ví dụ 1:

Đầu vào:
```text
events = ["beforeAll", "beforeEach", "test", "afterEach", "beforeEach", "test", "afterEach", "afterAll"]
```

Đầu ra:
```text
true
```

Giải thích:
Dãy event đúng với một lifecycle chuẩn gồm hai test case.

Ràng buộc:

- `1 <= events.length <= 100000`
- Mỗi phần tử chỉ thuộc tập giá trị đã cho
- Không cần mô phỏng framework thật

## Links

- [[001-unit-test-vs-integration-test]]
- [[003-Mockito]]
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [JUnit 5 API Javadoc](https://junit.org/junit5/docs/current/api/)
- [Spring Framework Testing Reference](https://docs.spring.io/spring-framework/reference/testing.html)
- [Spring Boot Testing](https://docs.spring.io/spring-boot/reference/testing/index.html)
