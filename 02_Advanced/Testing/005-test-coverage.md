# Test Coverage

## What is it

`Test coverage` là cách đo xem code của mình đã được test chạm tới ở mức nào. Các loại phổ biến gồm `line coverage`, `branch coverage`, và khi muốn đào sâu hơn thì có `mutation coverage`.

Coverage rất hữu ích để nhìn ra vùng mù của test suite, nhưng nó chỉ là tín hiệu phụ. Nó không tự chứng minh assertion đủ tốt hay behavior quan trọng đã được kiểm tra đúng.

### Coverage types at a glance

| Loại coverage | Trả lời câu hỏi gì | Dễ bị hiểu lầm ở đâu |
|---|---|---|
| `line coverage` | dòng nào đã được chạy qua | chạy qua chưa chắc đã assert đúng |
| `branch coverage` | các nhánh quyết định đã được đi qua đủ chưa | vẫn không đảm bảo trường hợp business quan trọng đã được kiểm tra |
| `mutation coverage` | test có thật sự bắt được thay đổi sai không | mạnh hơn, nhưng tốn hơn và không phải lúc nào cũng cần |

## How I used to misunderstand it

Mình từng xem phần trăm coverage như điểm số chất lượng gần như tuyệt đối. Nếu số này cao thì mình thấy yên tâm, nếu thấp thì mình nghĩ code đang nguy hiểm.

Về sau mới thấy coverage rất dễ đánh lừa. Một test có thể chạy qua code nhưng assert rất yếu. Hoặc một nhánh lỗi hiếm nhưng cực kỳ quan trọng lại chưa có test, dù `line coverage` tổng thể vẫn đẹp.

Hiểu nhầm nữa là nghĩ tăng coverage luôn đồng nghĩa tăng confidence. Thực tế có những dòng rất rẻ để cover nhưng gần như không tăng giá trị, trong khi có những branch rủi ro cao chỉ cần thiếu một test là hậu quả lớn.

## How it actually works

`Line coverage` đo xem bao nhiêu dòng được thực thi khi chạy test. `Branch coverage` đo xem các nhánh điều kiện như `if`, `switch`, hoặc ternary đã được đi qua đủ cả hướng chưa.

Hai số này cho góc nhìn khác nhau, nên nhìn chung `branch coverage` thường nói thật hơn về các decision point.

Mental model hữu ích là xem coverage như bản đồ nhiệt. Nó chỉ ra chỗ nào mình đã đi qua, chứ không nói mình có hiểu chỗ đó hay không.

```text
Coverage report
      |
      +--> chỗ nào chưa từng chạy?
      |
      +--> branch nào mới chạy một nửa?
      |
      +--> path nào critical mà vẫn chưa có test?
      |
      v
decide test tiếp theo dựa trên rủi ro, không chỉ dựa trên số phần trăm
```

### Hiểu nhầm matrix

| Câu nói | Đúng hay sai | Vì sao |
|---|---|---|
| `95% coverage nghĩa là code rất an toàn` | sai | coverage không đo chất lượng assertion |
| `branch coverage thường hữu ích hơn line coverage cho logic rẽ nhánh` | thường đúng | nó nói rõ các decision path |
| `coverage giảm sau refactor luôn là tín hiệu xấu` | sai | có khi code ít branch hơn, đơn giản hơn |
| `coverage thấp ở path ít rủi ro đôi khi chấp nhận được` | đúng trong context phù hợp | nên ưu tiên path critical trước |

Coverage target cố định cũng phải dùng tỉnh táo. Nếu team chỉ chạy theo một ngưỡng số học, rất dễ sinh ra test hình thức để kéo số lên thay vì bảo vệ hành vi quan trọng.

## Code example

```java
public String classify(int score) {
    if (score < 0) {
        throw new IllegalArgumentException("score must be non-negative");
    }
    return score >= 50 ? "pass" : "fail";
}
```

Nếu test chỉ cover `score = 70`, `line coverage` có thể trông ổn vì method đã được chạy qua. Nhưng `branch coverage` vẫn thiếu nhánh `score < 0` và nhánh `fail`.

Đó là lý do một con số coverage duy nhất không kể hết câu chuyện. Điều mình cần hỏi tiếp là: nhánh nào quan trọng mà vẫn chưa được chứng minh?

## When to use / when NOT to use

Dùng coverage khi:

- muốn tìm vùng code chưa từng được test chạy tới
- review PR lớn và cần tín hiệu nhanh về nhánh mới thêm
- ưu tiên hóa nơi nên viết test tiếp theo

Không dùng coverage như mục tiêu duy nhất của chất lượng test. Một suite `95%` coverage vẫn có thể bỏ sót bug business rất đau nếu assertion sai chỗ.

Không bỏ qua context của code. Một helper thuần có thể đáng test ít hơn một nhánh authorization hiếm nhưng rủi ro cao.

Không biến coverage threshold thành game tối ưu số. Khi đó team rất dễ sinh ra test nông, nhiều assertion yếu, nhưng phần trăm lại đẹp.

## How this connects to real Java projects

Trong Spring app, coverage report giúp phát hiện controller path, service branch, exception handler, hoặc security rule nào chưa được đụng tới. Nó đặc biệt hữu ích sau khi thêm feature mới hoặc refactor logic điều kiện.

Tuy nhiên, Spring integration test thường tốn hơn unit test, nên không nên cố kéo coverage bằng mọi giá ở các path ít giá trị. Tốt hơn là ưu tiên flow critical như security, payment, transaction, validation, hoặc data mapping quan trọng.

Với `JaCoCo`, điều quan trọng không chỉ là mở report, mà là đọc report cùng context kiến trúc. Một controller có coverage đẹp nhưng service validation chính lại thiếu branch test thì confidence tổng thể vẫn chưa cao.

## Gotchas

- `Line coverage` cao có thể che việc nhánh lỗi chưa bao giờ được test.
- Getter, setter, hoặc code rất dễ có thể làm coverage đẹp nhưng không tăng confidence đáng kể.
- Chạy theo threshold mù quáng dễ tạo test nông, nhiều assertion yếu.
- Coverage giảm sau refactor không phải lúc nào cũng xấu, đôi khi nó chỉ phản ánh code ít branch hơn.
- Bỏ qua path exception hoặc fallback thường khiến production bug lọt qua dù số coverage nhìn ổn.
- Một dòng được chạy qua trong test integration lớn chưa chắc đã có assertion đủ gần để bắt bug đúng chỗ.

## Handbook rule

- Coverage là tín hiệu, không phải mục tiêu; cao chưa đảm bảo chất lượng test.
- Dùng coverage để tìm vùng chưa test, không để chốt PR.
- Branch coverage có giá trị hơn line coverage cho logic phức tạp.
- Đừng viết test ép coverage số đẹp; assertion phải có ý nghĩa business.
- Theo dõi xu hướng coverage; sụt mạnh là tín hiệu suite bị giảm chất lượng.

## Check yourself

- Vì sao coverage nên được xem như bản đồ nhiệt, không phải điểm số chất lượng tuyệt đối?
- `Line coverage` và `branch coverage` khác nhau ở insight nào?
- Một suite coverage cao nhưng assertion yếu sẽ đánh lừa mình ra sao?
- Khi nào coverage giảm sau refactor vẫn có thể là tín hiệu tốt?
- Vì sao nên dùng `JaCoCo` để đặt câu hỏi tiếp theo, không phải để kết thúc cuộc điều tra?

## Exercises

### Bài 1: Calculate Line Coverage

Độ khó: Dễ

Đề bài:
Cho hai số nguyên `coveredLines` và `totalLines`. Hãy trả về phần trăm coverage theo công thức `floor(coveredLines * 100 / totalLines)`. Nếu `totalLines` bằng `0`, trả về `100`.

Ví dụ 1:

Đầu vào:
```text
coveredLines = 45
totalLines = 60
```

Đầu ra:
```text
75
```

Giải thích:
`45 * 100 / 60 = 75`, nên line coverage là `75` phần trăm.

Ràng buộc:

- `0 <= coveredLines <= totalLines <= 1000000`
- Kết quả là số nguyên từ `0` đến `100`
- Không dùng floating point

### Bài 2: Count Uncovered Critical Paths

Độ khó: Trung bình

Đề bài:
Cho `boolean[] covered` và `boolean[] critical`, trong đó mỗi index biểu diễn một code path. Hãy đếm số path mà `critical[i]` là `true` nhưng `covered[i]` là `false`.

Ví dụ 1:

Đầu vào:
```text
covered = [true, false, true, false]
critical = [false, true, true, true]
```

Đầu ra:
```text
2
```

Giải thích:
Hai path ở index `1` và `3` là critical nhưng chưa được cover.

Ràng buộc:

- `covered.length == critical.length`
- `0 <= covered.length <= 100000`
- Mỗi phần tử là boolean

### Bài 3: Classify Coverage Status

Độ khó: Trung bình

Đề bài:
Cho `lineCoveragePercent` và `branchCoveragePercent`. Trả về `"strong"` nếu cả hai đều từ `80` trở lên. Trả về `"weak"` nếu một trong hai nhỏ hơn `50`. Trong các trường hợp còn lại, trả về `"partial"`.

Ví dụ 1:

Đầu vào:
```text
lineCoveragePercent = 92
branchCoveragePercent = 61
```

Đầu ra:
```text
"partial"
```

Giải thích:
Line coverage cao nhưng branch coverage chưa tới mức `80`, nên trạng thái chỉ là `partial`.

Ràng buộc:

- `0 <= lineCoveragePercent <= 100`
- `0 <= branchCoveragePercent <= 100`
- Chỉ trả về `"strong"`, `"partial"`, hoặc `"weak"`

## Links

- [[004-TDD]]
- [[001-unit-test-vs-integration-test]]
- [JaCoCo documentation](https://www.jacoco.org/jacoco/trunk/doc/)
- [JaCoCo counters](https://www.jacoco.org/jacoco/trunk/doc/counters.html)
- [Spring Boot Testing](https://docs.spring.io/spring-boot/reference/testing/index.html)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)

