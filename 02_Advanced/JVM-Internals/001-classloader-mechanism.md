# Classloader Mechanism

## What is it

`ClassLoader` là cơ chế JVM dùng để tìm binary definition của class, nạp nó vào runtime, rồi tham gia vào quá trình `load`, `link`, và `initialize`.

Điểm cần nhớ là JVM không chỉ hỏi “class này tên gì?”, mà hỏi thêm “class này do loader nào định nghĩa?”. Trong thực tế, identity của class gắn với cặp:

`(fully qualified class name, defining classloader)`

Vì vậy class loading không phải chi tiết phụ. Nó quyết định vì sao một class có thể tìm thấy hoặc không, vì sao cùng tên class vẫn cast lỗi, và vì sao runtime trong IDE, test, app server, hoặc plugin container có thể khác nhau.

## How I used to misunderstand it

Mình từng nghĩ classloader chỉ là bước nền tảng mà business code không cần biết.

Nhưng rất nhiều lỗi runtime khó hiểu đều quay lại đây: `ClassNotFoundException`, `NoClassDefFoundError`, duplicate classes trong fat jar, hoặc `ClassCastException` giữa hai type nhìn như cùng một class.

Mình cũng từng nghĩ “đã thấy class trên classpath” nghĩa là JVM sẽ dùng đúng class đó. Thực tế còn phải hỏi loader nào tìm thấy trước, có `parent delegation` không, và class đó được define ở boundary nào.

## How it actually works

Trong HotSpot hiện đại, bạn thường gặp ba loader chính:

| Loader | Thường phụ trách | Ghi chú quan trọng |
| --- | --- | --- |
| `Bootstrap ClassLoader` | Core classes như `java.lang.*`, `java.util.*` | Không phải Java object thông thường, nên `getClassLoader()` thường ra `null` |
| `Platform ClassLoader` | Platform modules được export phù hợp | Thay cho phần lớn vai trò cũ của extension mechanism |
| `Application ClassLoader` | Classpath/module path của app | Thường là loader gần code của bạn nhất |

Luồng mặc định là `parent delegation`:

```text
Application loader receives request
        |
        v
Ask Platform loader
        |
        v
Ask Bootstrap loader
        |
        v
If parent cannot define it,
current loader tries to define it
```

Cơ chế này giúp tránh chuyện application vô tình thay thế core classes như `java.lang.String`. Nó cũng giúp các dependency dùng chung theo hướng nhất quán hơn trong đa số ứng dụng bình thường.

Một bảng rất đáng nhớ khi debug là bảng phân biệt các failure mode:

| Lỗi | Thường xảy ra khi nào | Câu hỏi debug đúng |
| --- | --- | --- |
| `ClassNotFoundException` | Code chủ động load theo tên, ví dụ `Class.forName(...)` | Loader hiện tại đang tìm ở đâu? tên class có đúng không? |
| `NoClassDefFoundError` | Compile time có class, nhưng runtime không define được class hoặc dependency của nó | Có thiếu jar, sai version, hoặc lỗi static init trước đó không? |
| `ClassCastException` giữa hai type cùng tên | Cùng FQCN nhưng khác defining loader | Có loader boundary hoặc child-first behavior không? |

`Thread Context ClassLoader` cũng rất quan trọng. Nhiều framework không dùng loader của chính class hiện tại, mà dùng context loader của thread để scan SPI, load driver, load resource, hoặc khởi tạo plugin. Vì vậy debug class loading trong framework phải nhìn cả `this.getClass().getClassLoader()` lẫn `Thread.currentThread().getContextClassLoader()`.

## Code example

```java
ClassLoader currentLoader = Thread.currentThread().getContextClassLoader();

System.out.println(String.class.getClassLoader());
System.out.println(java.sql.Driver.class.getClassLoader());
System.out.println(currentLoader.loadClass("com.example.MyService").getClassLoader());
```

Điểm đáng nhớ ở đây:

- `String.class.getClassLoader()` thường in ra `null`, nghĩa là class đó do `Bootstrap ClassLoader` phụ trách.
- Class của application thường ra `Application ClassLoader`.
- Nếu cùng một FQCN được define bởi loader khác, JVM xem đó là type khác.

## When to use / when NOT to use

Hãy bật mental model này khi:

- debug lỗi thiếu class ở runtime,
- đọc stack trace liên quan tới container, plugin, agent, reload,
- làm bytecode instrumentation, SPI, JDBC driver loading, hoặc custom runtime isolation.

Không nên tự viết custom classloader trong app business bình thường chỉ để “học sâu”. Đây là vùng rất dễ tạo class identity bug, memory leak sau redeploy, và behavior khó tái hiện trong production.

## How this connects to Spring

Spring dựa mạnh vào scanning, reflection, proxy, resource loading, và dynamic type discovery. Vì vậy classloader issue thường biểu hiện thành lỗi rất vòng vo ở startup.

Ví dụ:

- cùng dependency nhưng test chạy, app server không chạy,
- devtools reload sinh hai loader boundary khác nhau,
- SPI hoặc JDBC driver không được tìm thấy dù jar có mặt,
- proxy cast lỗi vì target type đi qua loader khác.

Nếu thấy lỗi của Spring mà message có vẻ “không liên quan”, rất đáng kiểm tra loader boundary trước khi kết luận framework có bug.

## Gotchas

- Cùng một tên class nhưng khác defining loader vẫn là hai type khác nhau.
- `ClassNotFoundException` và `NoClassDefFoundError` không đồng nghĩa.
- `Thread Context ClassLoader` có thể khác loader của chính class đang chạy.
- Custom loader hoặc reload mechanism dễ giữ reference tới class metadata và gây `classloader leak`.

## Check yourself

- Vì sao identity của class trong JVM không chỉ là FQCN?
- `parent delegation` giúp tránh loại bug nào?
- Khi nào `ClassNotFoundException` khác bản chất với `NoClassDefFoundError`?
- Vì sao hai class cùng tên vẫn có thể cast lỗi?
- Nếu framework load sai resource hoặc SPI, vì sao nên kiểm tra `Thread Context ClassLoader`?

## Exercises

### Exercise 1: Resolve Class Loader Path

Độ khó: Easy

Đề bài:
Cho ba cờ `inBootstrap`, `inPlatform`, `inApplication` mô tả class có tồn tại ở từng loader hay không. Hãy trả về loader nào sẽ định nghĩa class theo `parent delegation`. Kết quả hợp lệ là `"bootstrap"`, `"platform"`, `"application"`, hoặc `"not-found"`.

Ví dụ 1:

Đầu vào:
```text
inBootstrap = false
inPlatform = true
inApplication = true
```

Đầu ra:
```text
"platform"
```

Giải thích:
Parent được hỏi trước, nên `Platform ClassLoader` thắng trước dù application cũng có class đó.

Ràng buộc:

- Mỗi input là một boolean
- Phải bám đúng `parent delegation`
- Không xét custom `child-first` behavior

### Exercise 2: Count Repeated Load Requests

Độ khó: Medium

Đề bài:
Cho `String[] classNames` là chuỗi request load class theo thời gian. Hãy trả về số request bị lặp lại, nghĩa là class đó đã từng được yêu cầu trước đó trong cùng JVM session.

Ví dụ 1:

Đầu vào:
```text
classNames = ["A", "B", "A", "C", "B"]
```

Đầu ra:
```text
2
```

Giải thích:
Request thứ ba cho `"A"` và request thứ năm cho `"B"` là các request lặp.

Ràng buộc:

- `0 <= classNames.length <= 100000`
- Mỗi tên class là `String` không rỗng
- So sánh theo đúng nội dung chuỗi, phân biệt hoa thường

### Exercise 3: Detect Child First Class Loading Risk

Độ khó: Medium

Đề bài:
Cho hai mảng boolean `childFirstModules` và `sharesApiWithParent` có cùng độ dài. Mỗi index mô tả một module: nếu module dùng `child-first` và đồng thời chia sẻ API type với parent, đó là một điểm có risk class identity bug. Hãy trả về index đầu tiên có risk, hoặc `-1` nếu không có.

Ví dụ 1:

Đầu vào:
```text
childFirstModules = [false, true, true]
sharesApiWithParent = [true, false, true]
```

Đầu ra:
```text
2
```

Giải thích:
Module ở index `2` vừa `child-first` vừa chia sẻ API type với parent, nên dễ tạo duplicate type definitions.

Ràng buộc:

- `0 <= childFirstModules.length <= 100000`
- Hai mảng phải có cùng độ dài
- Nếu không có risk nào, trả về `-1`

## Links

- [[../../00_Mental-Models/005-JVM-load-class]]
- [[../../00_Mental-Models/006-Compile-time-vs-Runtime]]
- [[002-bytecode-and-class-file]]
- [JVMS Chapter 5, Loading, Linking, and Initializing](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-5.html)
- [JLS Chapter 12, Execution](https://docs.oracle.com/javase/specs/jls/se21/html/jls-12.html)
- [ClassLoader Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ClassLoader.html)
- [Class Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html)
