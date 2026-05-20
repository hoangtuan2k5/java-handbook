# JVM Flags

## What is it

`JVM flags` là các tham số cấu hình truyền cho Java process lúc start. Chúng điều khiển nhiều lớp behavior khác nhau của runtime: heap sizing, collector selection, logging, diagnostics, module access, system properties, heap dump behavior, và nhiều thứ khác.

Điểm quan trọng là không phải mọi flag cùng một loại. Nhìn startup command mà không tách đúng nhóm flag rất dễ hiểu sai hệ thống đang chạy gì.

## How I used to misunderstand it

Mình từng nghĩ mọi flag gần như ngang hàng, nên có thể copy một block dài từ blog hay Stack Overflow vào production.

Thực tế có flag ổn định, flag deprecated, flag experimental, flag diagnostic, và flag chỉ hợp cho điều tra ngắn hạn. Ngoài ra, nhiều flag tương tác với nhau. Một command line “có vẻ hợp lý” vẫn có thể tự mâu thuẫn hoặc bị option xuất hiện sau ghi đè.

## How it actually works

Một cách phân loại rất hữu ích là tách theo vai trò:

| Nhóm | Ví dụ | Dùng để làm gì |
| --- | --- | --- |
| Heap sizing | `-Xms`, `-Xmx` | Chỉnh heap ban đầu và tối đa |
| GC selection / tuning | `-XX:+UseG1GC`, `-XX:MaxGCPauseMillis=200` | Chọn collector và tuning goal |
| Diagnostics / logging | `-Xlog:gc*`, `-XX:+HeapDumpOnOutOfMemoryError` | Quan sát và điều tra runtime |
| System properties | `-Dspring.profiles.active=prod` | Đưa config vào app/framework |

Một flow đọc startup command cho đỡ rối:

```text
1. Tách heap flags
2. Tách GC flags
3. Tách diagnostics flags
4. Tách -D system properties
5. Kiểm tra flag lặp hoặc mâu thuẫn
6. Đối chiếu với JDK version thực tế
```

Contrast cần nhớ:

| Nhìn giống nhau trong command line | Nhưng bản chất khác nhau |
| --- | --- |
| `-Xmx2g` và `-Dfile.encoding=UTF-8` | Một cái tune JVM heap, một cái truyền property |
| `-Xlog:gc*` và `-XX:+HeapDumpOnOutOfMemoryError` | Một cái phục vụ logging, một cái bật artifact khi OOM |
| `-XX:+UseG1GC` và `-XX:MaxGCPauseMillis=200` | Một cái chọn collector, một cái đặt goal cho collector |

Thứ tự cũng quan trọng. Với nhiều option dạng key-value hoặc on/off, option xuất hiện sau thường thắng. Vì vậy script khởi động dài có thể làm người đọc tưởng app dùng cấu hình A, nhưng runtime thật lại dùng cấu hình B.

## Code example

```text
java \
  -Xms512m \
  -Xmx1024m \
  -XX:+UseG1GC \
  -Xlog:gc*:file=gc.log \
  -Dspring.profiles.active=prod \
  -jar app.jar
```

Command này đang làm bốn việc khác nhau:

- định heap,
- chọn collector,
- bật GC logging,
- truyền Spring profile vào application.

Nếu không tách được bốn lớp nghĩa này, bạn rất dễ tune sai hoặc debug sai.

## When to use / when NOT to use

Hãy dùng note này khi:

- cần review startup command,
- cần cấu hình heap hoặc GC theo môi trường,
- cần bật diagnostics để điều tra issue runtime,
- cần phân biệt flag nào thuộc JVM và flag nào là config cho app.

Không nên copy nguyên block flags từ hệ thống khác chỉ vì “họ đang chạy ổn”. Flag chỉ đúng trong bối cảnh JDK version, workload, container limits, và mục tiêu vận hành cụ thể.

## How this connects to real Java projects

Spring Boot thường đọc nhiều cấu hình từ environment và `system properties`, nên `-D...` flags ảnh hưởng trực tiếp tới profile, port, log config, SSL config, hoặc feature toggle.

Đồng thời, Spring apps hay chạy trong Docker hoặc Kubernetes, nên `-Xmx`, GC flags, heap-dump path, và logging flags phải ăn khớp với resource limit, volume mount, và cách thu thập artifact của môi trường chạy.

## Gotchas

- Flag lặp lại có thể bị option phía sau ghi đè.
- Một số flag đổi tên, deprecated, hoặc biến mất giữa các JDK version.
- Diagnostic logging quá mạnh có thể tạo overhead và log volume lớn.
- `-D` property nằm chung startup command nhưng không phải JVM tuning flag.

## Handbook rule

- Flag chỉ đúng trong bối cảnh JDK/version/workload/container; không copy block flag từ hệ thống khác.
- Không lặp/đè flag mơ hồ; biết flag nào ghi đè flag nào.
- Flag deprecated/đổi tên giữa version; review khi upgrade JDK.
- Diagnostic logging mạnh có chi phí; bật khi điều tra, tắt khi xong.
- `-D` property là app config, không phải JVM tuning flag; phân biệt rõ trong startup command.

## Check yourself

- Vì sao cần tách `-X`, `-XX`, và `-D` theo vai trò thay vì đọc cả command như một khối?
- Khi nào một flag “đúng” ở môi trường này nhưng sai ở môi trường khác?
- Vì sao flag lặp lại trong script có thể gây hiểu nhầm nguy hiểm?
- `-Xlog` khác gì với `-Dspring.*` về bản chất?
- Trước khi mang một diagnostic flag sang production, bạn nên hỏi điều gì?

## Exercises

### Exercise 1: Classify JVM Flag

Độ khó: Easy

Đề bài:
Cho `String flag`. Hãy phân loại nó thành `"heap"` nếu bắt đầu bằng `"-Xms"` hoặc `"-Xmx"`, `"gc"` nếu bắt đầu bằng `"-XX:+Use"` hoặc `"-XX:MaxGCPauseMillis"`, `"system-property"` nếu bắt đầu bằng `"-D"`, `"diagnostic"` nếu bắt đầu bằng `"-XX:+Print"` hoặc `"-Xlog:"`, ngược lại trả về `"unknown"`.

Ví dụ 1:

Đầu vào:
```text
flag = "-Dspring.profiles.active=prod"
```

Đầu ra:
```text
"system-property"
```

Giải thích:
Đây là system property cho application hoặc framework đọc lại.

Ràng buộc:

- `flag` là chuỗi không rỗng
- So sánh theo prefix
- Nếu không khớp quy tắc nào, trả về `"unknown"`

### Exercise 2: Count Boolean Flag Overrides

Độ khó: Medium

Đề bài:
Cho `String[] flags`. Một boolean GC flag override xảy ra khi cùng một flag base name xuất hiện nhiều lần với cả dạng `-XX:+Name` hoặc `-XX:-Name`, và mỗi lần xuất hiện sau sẽ ghi đè lần trước. Hãy đếm tổng số lần override xảy ra.

Ví dụ 1:

Đầu vào:
```text
flags = ["-XX:+UseG1GC", "-XX:-UseG1GC", "-XX:+UseStringDeduplication"]
```

Đầu ra:
```text
1
```

Giải thích:
`UseG1GC` bị set lại lần thứ hai, nên có một override.

Ràng buộc:

- `0 <= flags.length <= 100000`
- Chỉ cần xét flags bắt đầu bằng `-XX:+` hoặc `-XX:-`
- So sánh base name phân biệt hoa thường

### Exercise 3: Find First Risky Runtime Flag

Độ khó: Medium

Đề bài:
Cho `String[] flags`. Hãy trả về index đầu tiên của flag bị xem là risky theo danh sách sau: `"-XX:+HeapDumpOnOutOfMemoryError"` không risky, nhưng `"-Xverify:none"`, `"-XX:+UnlockDiagnosticVMOptions"`, và bất kỳ flag nào bắt đầu bằng `"-XX:+Print"` đều risky trong runtime production chung. Nếu không có, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
flags = ["-Xms512m", "-XX:+PrintCompilation", "-Dspring.profiles.active=prod"]
```

Đầu ra:
```text
1
```

Giải thích:
`-XX:+PrintCompilation` là diagnostic output flag, nên được xem là risky theo quy tắc bài toán.

Ràng buộc:

- `0 <= flags.length <= 100000`
- So sánh chuỗi chính xác hoặc theo prefix đã nêu
- Nếu không có flag risky, trả về `-1`

## Links

- [[004-GC-algorithms]]
- [[005-GC-tuning]]
- [[008-heap-dump-and-analysis]]
- [`java` Command Reference](https://docs.oracle.com/en/java/javase/21/docs/specs/man/java.html)
- [Extra Options for the `java` Command](https://docs.oracle.com/en/java/javase/21/docs/specs/man/java.html#extra-options-for-the-java-command)
- [Unified JVM Logging](https://docs.oracle.com/en/java/javase/21/docs/specs/man/java.html#enable-logging-with-the-jvm-unified-logging-framework)
