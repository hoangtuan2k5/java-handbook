# Mockito

## What is it

`Mockito` là thư viện giúp tạo `mock`, `stub`, và `verify interaction` trong test. Nó hữu ích khi mình muốn cô lập một class khỏi dependency có side effect, chậm, nondeterministic, hoặc quá nặng để mang vào unit test.

Điểm cốt lõi là `Mockito` không tồn tại để giả vờ cho cả hệ thống. Nó tồn tại để giữ cho test của một unit vừa nhanh, vừa tập trung vào behavior của unit đó.

### Test double cheat sheet

| Loại | Mục đích chính | Có thường dùng với Mockito không? |
|---|---|---|
| `stub` | trả dữ liệu cố định cho dependency | rất thường |
| `mock` | quan sát interaction và verify call | rất thường |
| `fake` | bản cài đặt đơn giản trong memory | thường viết tay |
| `spy` | bọc object thật nhưng vẫn theo dõi call | có, nhưng nên dùng cẩn thận |

## How I used to misunderstand it

Mình từng nghĩ viết unit test chuyên nghiệp đồng nghĩa với việc mock tất cả dependency có thể mock.

Kết quả là test rất giòn. Chỉ cần đổi nhẹ cách implementation gọi dependency là test hỏng, dù behavior bên ngoài vẫn đúng. Về sau mình mới hiểu `Mockito` nên được dùng có chọn lọc, chủ yếu ở chỗ dependency thật khó tạo, chậm, nondeterministic, hoặc có side effect.

Hiểu nhầm khác là nghĩ `verify(...)` càng nhiều càng tốt. Thực tế verify quá nhiều thường khóa chặt implementation detail thay vì bảo vệ behavior.

## How it actually works

`Mockito` tạo test double để mình định nghĩa trước phản hồi và kiểm tra interaction.

- `when(...).thenReturn(...)` thường dùng cho stubbing
- `verify(...)` dùng để xác nhận lời gọi có ý nghĩa đã xảy ra
- matcher như `any()` hoặc `eq(...)` giúp viết expectation linh hoạt hơn

Mental model quan trọng là thế này: mock không chứng minh hệ thống thật chạy đúng với dependency thật. Nó chỉ chứng minh object đang test phản ứng đúng với contract giả định của dependency đó.

```text
Production class under test
        |
        +--> collaborator thật quá chậm / quá khó kiểm soát
                    |
                    v
               thay bằng mock
                    |
        test xác nhận 2 thứ:
        1. output hoặc state của class under test
        2. interaction thật sự có ý nghĩa
```

### Khi nào dùng object thật, fake, hay mock

| Dependency | Chọn gì trước | Vì sao |
|---|---|---|
| value object đơn giản | object thật | dễ đọc nhất |
| collection hoặc mapper thuần | object thật hoặc fake nhỏ | ít ceremony |
| repository hoặc external client | mock hoặc fake chuyên dụng | tránh side effect và chậm |
| gateway thanh toán, mail, queue | mock | cần quan sát interaction |
| logic có thể viết bản in-memory đơn giản | fake | thường bền hơn mock |

Một test dùng `Mockito` tốt thường chỉ verify những interaction có ý nghĩa business hoặc side effect quan trọng. Nếu verify từng call nhỏ vô nghĩa, test sẽ bắt đầu “canh implementation” thay vì kiểm tra behavior.

## Code example

```java
PaymentGateway gateway = mock(PaymentGateway.class);
when(gateway.charge("u-1", 500)).thenReturn(true);

BillingService service = new BillingService(gateway);

boolean charged = service.chargeUser("u-1", 500);

assertTrue(charged);
verify(gateway).charge("u-1", 500);
```

Ở đây `Mockito` giúp test tập trung vào việc `BillingService` có gọi gateway đúng hay không, thay vì cần một cổng thanh toán thật.

Giá trị của test này nằm ở chỗ `charge` là side effect quan trọng. Nếu mình verify thêm hàng loạt interaction phụ không liên quan, test sẽ kém bền hơn mà không tăng confidence tương xứng.

## When to use / when NOT to use

Dùng `Mockito` khi:

- dependency gọi network, database, queue, hoặc service ngoài
- dependency có side effect khó kiểm soát trong unit test
- mình cần quan sát interaction để xác nhận behavior quan trọng

Không mock value object đơn giản hoặc collection bình thường. Tạo object thật thường dễ hiểu hơn nhiều.

Không verify mọi lời gọi một cách máy móc. Nếu assertion business result đã đủ chứng minh behavior, verify thêm có thể chỉ làm test dễ vỡ hơn.

Không xem mock như bản thay thế hoàn chỉnh cho integration test. Nó chỉ giúp unit test nhanh và tập trung hơn.

## How this connects to real Java projects

Trong Spring, `Mockito` xuất hiện nhiều ở service unit test hoặc test slice khi mình muốn cắt bớt dependency như external client, mail sender, message publisher, hoặc payment gateway.

Điểm cần nhớ là nếu một service chỉ pass-through qua quá nhiều mock mới test được, đó thường là tín hiệu class đó chưa có logic rõ ràng hoặc boundary đang đặt chưa tốt.

Với Spring Boot, tài liệu chính thức hiện nay cũng nhấn mạnh chọn mức test phù hợp, thay vì mặc định mock mọi thứ. `MockMvc` hay test slice có thể đáng tin hơn một unit test giả lập quá nhiều framework behavior.

## Gotchas

- Stub method nhưng test không bao giờ đi qua branch đó sẽ tạo `unused stubbing` và làm test khó đọc.
- Verify quá chi tiết số lần gọi hoặc thứ tự gọi dễ khóa cứng implementation.
- Mock repository ở mọi nơi có thể che mất bug về query hoặc transaction mà chỉ integration test mới lộ ra.
- Dùng matcher không đồng nhất, ví dụ trộn raw value với `any()`, dễ tạo lỗi khó hiểu.
- `spy` nghe tiện nhưng dễ kéo test vào vùng nửa thật nửa giả, khó reason hơn mock hoặc object thật rõ ràng.
- Mock static hoặc constructor chỉ vì tiện có thể là mùi thiết kế, không phải thành công của test.

## Handbook rule

- Mock dependency I/O/external; không mock value object/collection thường.
- Verify ít, assertion business nhiều; verify chi tiết khóa implementation.
- Mock không thay thế integration test; query/transaction/serialization vẫn cần test thật.
- Tránh `spy` và mock static/constructor; thường là mùi thiết kế.
- Stub đầy đủ branch test sẽ chạy; tránh `unused stubbing` tích tụ.

## Check yourself

- Vì sao `mock` không thể thay thế `integration test`?
- Trong nhiều case, vì sao `fake` hoặc object thật lại bền hơn `mock`?
- Interaction nào đáng `verify`, và interaction nào chỉ làm test giòn hơn?
- Nếu service phải mock quá nhiều dependency mới test nổi, đó có thể là tín hiệu gì về design?
- `spy` khác `mock` ở chỗ nào về mental model?

## Exercises

### Bài 1: Choose Mock Strategy

Độ khó: Dễ

Đề bài:
Cho ba cờ `collaboratorSlow`, `collaboratorNondeterministic`, và `simpleValueObject`. Trả về `"real"` nếu `simpleValueObject` là `true`. Trả về `"mock"` nếu `simpleValueObject` là `false` và một trong hai cờ còn lại là `true`. Trong các trường hợp còn lại, trả về `"real"`.

Ví dụ 1:

Đầu vào:
```text
collaboratorSlow = true
collaboratorNondeterministic = false
simpleValueObject = false
```

Đầu ra:
```text
"mock"
```

Giải thích:
Dependency này chậm và không phải value object đơn giản, nên mock là lựa chọn hợp lý cho unit test.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"mock"` hoặc `"real"`
- Không cần dùng thư viện mocking thật

### Bài 2: Find Unused Stub

Độ khó: Trung bình

Đề bài:
Cho `String[] stubbedMethods` và `String[] invokedMethods`. Hãy trả về index đầu tiên trong `stubbedMethods` mà method tương ứng không xuất hiện trong `invokedMethods`. Nếu mọi stub đều được dùng, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
stubbedMethods = ["findById", "save", "publish"]
invokedMethods = ["findById", "publish"]
```

Đầu ra:
```text
1
```

Giải thích:
Stub cho `"save"` chưa bao giờ được gọi.

Ràng buộc:

- `0 <= stubbedMethods.length, invokedMethods.length <= 100000`
- Mỗi tên method có độ dài từ `1` đến `100`
- So sánh chuỗi có phân biệt hoa thường

### Bài 3: Count Verification Mismatches

Độ khó: Trung bình

Đề bài:
Cho `String[] expectedCalls` và `String[] actualCalls`. Mỗi phần tử biểu diễn một lời gọi method đã normalize về cùng format. Hãy đếm xem có bao nhiêu phần tử trong `expectedCalls` không xuất hiện trong `actualCalls`.

Ví dụ 1:

Đầu vào:
```text
expectedCalls = ["load:user-1", "save:user-1", "publish:user-1"]
actualCalls = ["load:user-1", "publish:user-1"]
```

Đầu ra:
```text
1
```

Giải thích:
Lời gọi `"save:user-1"` được kỳ vọng nhưng không có trong actual calls.

Ràng buộc:

- `0 <= expectedCalls.length, actualCalls.length <= 100000`
- Mỗi chuỗi có độ dài từ `1` đến `200`
- Mỗi mismatch chỉ tính theo từng phần tử của `expectedCalls`

## Links

- [[001-unit-test-vs-integration-test]]
- [[002-JUnit5]]
- [Mockito site](https://site.mockito.org/)
- [Mockito Javadoc](https://javadoc.io/doc/org.mockito/mockito-core/latest/org.mockito/org/mockito/Mockito.html)
- [Mockito GitHub documentation](https://github.com/mockito/mockito)
- [Spring Boot Testing](https://docs.spring.io/spring-boot/reference/testing/index.html)
