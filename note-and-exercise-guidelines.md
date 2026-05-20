# Java Notes and Exercise Guidelines

File này là quy chuẩn chung để viết, sửa, review, và verify Java notes cùng Java exercise skeleton trong vault này.

Mục tiêu không chỉ là “viết cho đủ section”, mà là tạo ra tài liệu:

- dễ học lại
- dễ tra cứu
- dễ mở rộng
- đủ chính xác về mặt Java
- đủ dễ hiểu cho người học thật
- có bài tập thực hành nhất quán khi cần

Tài liệu này cũng là hướng dẫn làm việc dành cho các agent đến sau.

---

## 1. Phạm vi áp dụng

Áp dụng cho toàn bộ thư mục:

```text
01-programming-languages/java
```

Bao gồm đặc biệt:

- `00_Java-Conventions/`
- `00_Mental-Models/`
- `01_Core/`
- `02_Advanced/`
- `03_Evergreen-Notes/`
- `04_Lessons-from-bugs/`
- `05_Open-Questions/`
- `java-exercises/`

---

## 2. Mục tiêu theo từng khu vực

### `00_Java-Conventions`

Mục tiêu là ghi lại convention Java nói chung: project structure, package/class/method naming, formatting, imports, exception handling, logging, Javadoc, testing, build tool, và quality gates.

Đặc điểm:

- không viết như rule riêng cho vault tài liệu hiện tại
- ví dụ nên dùng Java application/library/service trung lập thay vì `java-exercises` hoặc chapter note hiện tại
- nếu cần nhắc tới learning note hoặc exercise skeleton, phải nói rõ đó chỉ là ví dụ phụ, không phải trọng tâm của convention
- ưu tiên rule áp dụng được cho Java codebase thật: CLI nhỏ, library, Spring service, Maven/Gradle project, hoặc backend module

### `00_Mental-Models`

Mục tiêu là giúp người học hiểu đúng model gốc của Java và JVM.

Đặc điểm:

- ưu tiên intuition và mental model
- không bắt buộc LeetCode-style exercises
- bắt buộc có phần tự kiểm tra nhanh (`## Check yourself`)
- nên có diagram, bảng, hoặc ASCII scaffold nếu concept trừu tượng

### `01_Core`

Mục tiêu là xây nền Java đủ chắc để đi tiếp sang framework, production code, và advanced topics.

Đặc điểm:

- giải thích đúng concept
- có ví dụ Java ngắn
- có self-check để tự test hiểu biết
- từ `Collections` trở đi phải có exercises

### `02_Advanced`

Mục tiêu là đi sâu hơn vào JVM, concurrency, patterns, performance, testing, security.

Đặc điểm:

- dễ over-complicate, nên phải ưu tiên clarity
- bắt buộc contrast rõ giữa khái niệm giống nhau
- rất nên có bảng, decision matrix, hoặc flow scaffold

### `03_Evergreen-Notes`

Mục tiêu là ghi lại principle, heuristic, và rule-of-thumb Java có giá trị lâu dài.

### `04_Lessons-from-bugs`

Mục tiêu là biến bug Java/JVM/Spring/build-tool thường gặp thành lesson có thể tái dùng.

### `05_Open-Questions`

Mục tiêu là giữ các câu hỏi chưa resolve, hypothesis hiện tại, evidence đã biết, và next step để research.

### `java-exercises`

Mục tiêu là cung cấp skeleton compile được, nhất quán, không có solution, để người học tự implement.

---

## 3. Ngôn ngữ và giọng viết

- Viết giải thích chính bằng tiếng Việt.
- Giữ nguyên English technical terms khi đó là thuật ngữ phổ biến và dễ nhận diện hơn.
- Không dịch gượng các thuật ngữ Java nếu bản tiếng Anh rõ hơn.
- Ưu tiên giọng viết trực tiếp, dễ hiểu, như đang giải thích cho chính mình trong tương lai.
- Tránh văn phong quá học thuật, quá dài dòng, hoặc quá giống textbook.
- Mỗi đoạn nên tập trung vào một ý chính.
- Khi có thể, giải thích bằng mental model trước, sau đó mới đi vào syntax hoặc API.
- Nếu dùng ngôi thứ nhất, ưu tiên nhất quán theo kiểu `Mình` thay vì lúc `Mình`, lúc `Tôi`.

### Quy tắc cực quan trọng

- Đừng hy sinh độ đúng để lấy một câu nói nhanh gọn nhưng sai.
- Nếu phải đơn giản hóa, hãy đơn giản hóa có kiểm soát.
- Với concept dễ bị hiểu nhầm, hãy nói rõ “cách nói này chỉ là mental model gần đúng” nếu cần.

Ví dụ:

- Tốt: `reference` là một giá trị tham chiếu trừu tượng dùng để đi tới object.
- Kém hơn: `reference chính là con trỏ bộ nhớ thô`.

---

## 4. Cấu trúc chuẩn của một note Markdown

Mục tiêu của cấu trúc chuẩn không phải là ép mọi note giống nhau tuyệt đối. Mục tiêu là để người đọc có thể mở bất kỳ note nào và nhanh chóng biết concept này dùng để quyết định điều gì, guarantee nào đến từ Java/JVM, ví dụ đúng/sai nằm ở đâu, và nguồn chính thức nào kiểm chứng được.

### 4.0. Mức độ bắt buộc của guidance

Khi một note đưa ra rule, hãy phân biệt rõ một trong ba mức:

| Mức | Ý nghĩa | Cách viết |
|---|---|---|
| Hard rule | Là behavior do Java/JVM/JLS/JVMS quy định hoặc rule cần giữ để tránh bug rõ ràng | dùng `phải`, `không được`, `bắt buộc` |
| Default convention | Là lựa chọn mặc định nên theo trong phần lớn project | dùng `nên`, `ưu tiên`, `mặc định chọn` |
| Local preference | Là preference của handbook hoặc team, có thể đổi theo project | nói rõ `trong handbook này` hoặc `nếu project không có convention riêng` |

Không viết preference như thể nó là Java language guarantee. Ví dụ `class name nên dùng PascalCase` là convention; còn `overload resolution xảy ra ở compile-time` là language behavior.

### 4.1. Cấu trúc handbook chuẩn cho note mental model / concept

Mỗi note nên dùng đúng các section sau, theo thứ tự này:

```markdown
# Topic Name

## What is it

## How I used to misunderstand it

## How it actually works

## Code example

## When to use / when NOT to use

## How this connects to real Java projects

## Gotchas

## Handbook rule

## Links
```

Ý nghĩa các section handbook:

| Section | Mục đích | Bắt buộc khi nào |
|---|---|---|
| `## What is it` | định nghĩa ngắn, scope rõ | luôn có |
| `## How I used to misunderstand it` | chỉ ra hiểu nhầm phổ biến | mental model / concept dễ sai |
| `## How it actually works` | cơ chế đúng hoặc model gần đúng | luôn có |
| `## Code example` | ví dụ nhỏ, chạy được nếu là code | luôn có trừ note thuần design |
| `## When to use / when NOT to use` | rule áp dụng và giới hạn | luôn có |
| `## How this connects to real Java projects` | bridge sang project thật, kể cả Spring/Maven/Gradle khi liên quan | luôn có với learning note |
| `## Gotchas` | lỗi hoặc edge case dễ gặp | luôn có |
| `## Handbook rule` | 2-5 bullet chốt rule để viết/review/debug code | luôn có với `00_Mental-Models`; nên có với concept subtle ở nơi khác |
| `## Check yourself` | câu hỏi tự kiểm tra | luôn có với learning note |
| `## Links` | official docs và note liên quan | luôn có |

Với concept cần chọn giữa nhiều hướng, thêm một section decision trước `## Handbook rule`:

```markdown
## <Topic> decision matrix
```

Tên section có thể cụ thể theo topic, ví dụ `## Collection decision matrix`, `## Concurrency decision matrix`, `## Exception decision matrix`. Không bắt buộc phải đúng literal `## Decision matrix`, nhưng phải có cụm `decision matrix` để dễ grep.

`## Handbook rule` không được lặp lại toàn bộ bài. Nó chỉ chốt những câu người đọc cần nhớ khi viết, review, hoặc debug code.

Với note dạy concept, đặc biệt là `00_Mental-Models` và `01_Core`, nên có thêm:

```markdown
## Check yourself
```

`## Handbook rule` là 2-5 bullet chốt rule để viết/review/debug Java code. Section này luôn có với `00_Mental-Models` và nên có với concept subtle ở nơi khác.

Section `## Check yourself` nên đặt:

- trước `## Links` nếu note không có `## Exercises`
- trước `## Exercises` nếu note có `## Exercises`

### 4.3. Exercises

Từ:

```text
01_Core/Collections
```

trở đi, mỗi note phải có thêm:

```markdown
## Exercises
```

Tức là với note có bài tập, thứ tự chuẩn là:

```markdown
# Topic Name

## What is it

## How I used to misunderstand it

## How it actually works

## Code example

## When to use / when NOT to use

## How this connects to real Java projects

## Gotchas

## Handbook rule

## Check yourself

## Exercises

## Links
```

### 4.4. Cấu trúc handbook chuẩn cho note convention

Mỗi note trong `00_Java-Conventions` nên dùng đúng các section sau, theo thứ tự này:

```markdown
# Convention Topic

## Why this file exists

## Rules

## <Topic> decision matrix

## <Topic> checklist

## Good examples

## Bad examples

## Exceptions

## Notes

## Official references

## Related rules
```

Ý nghĩa các section convention:

| Section | Mục đích | Bắt buộc khi nào |
|---|---|---|
| `## Why this file exists` | nêu decision/review problem | luôn có |
| `## Rules` | hard rule/default convention/local preference | luôn có |
| `## <Topic> decision matrix` | chọn cách làm khi có nhiều lựa chọn đúng | bắt buộc với topic có trade-off; optional với note rất đơn giản |
| `## <Topic> checklist` | checklist review hoặc design nhanh | nên có với topic dùng khi review code |
| `## Good examples` | ví dụ đúng hoặc shape nên theo | luôn có |
| `## Bad examples` | anti-pattern cụ thể | nên có |
| `## Exceptions` | trường hợp được phép phá rule | nên có khi rule không tuyệt đối |
| `## Notes` | nuance, version note, JVM/JDK/framework caveat | optional |
| `## Official references` | Oracle docs, JDK Javadocs, JLS, JVMS, Maven/Gradle docs, JUnit/Mockito docs, Spring docs khi liên quan | luôn có với convention dựa trên nguồn chuẩn |
| `## Related rules` | link sang convention liên quan | luôn có |

Convention note phải phân biệt `hard rule`, `default convention`, và `local preference`. Không biến Spring/team preference thành Java guarantee.

---

## 5. Quy tắc đặt tên file note

- Dùng số thứ tự 3 chữ số ở đầu file khi note nằm trong sequence học.
- Dùng `all lowercase kebab-case` cho tên file.
- Ngoại lệ cực kỳ quan trọng: **mọi file `README.md` ở mọi folder** đều được giữ nguyên là `README.md`, không đổi sang lowercase kebab-case.
- Không dùng chữ hoa trong filename, kể cả với acronym như `JVM`, `JDK`, `JMM`, `API`, `NPE`.
- Nếu title của concept có class name, method name, acronym, hoặc Java symbol quen thuộc, giữ chúng ở heading `# ...`; filename vẫn phải chuyển toàn bộ về lowercase.
- Tên file phải phản ánh đúng concept chính.
- Nếu note so sánh hai hay nhiều thứ, tên file nên thể hiện rõ cặp hoặc nhóm so sánh.

Ví dụ tốt:

```text
001-list-vs-set-vs-map.md
002-array-list-vs-linked-list.md
003-hash-map.md
004-iterator.md
```

Ví dụ không tốt:

```text
new-note.md
java-topic.md
misc.md
Problem1.md
001-HashMap.md
002-equals-and-hash-code-contract.md
```

### 5.1. Title policy cho heading `# ...`

Top-level title của note phải theo policy thống nhất sau:

- Dùng title dễ đọc cho người học, không bắt buộc lowercase như filename.
- Giữ acronym quen thuộc như `JVM`, `JDK`, `JMM`, `API`, `GC`, `NPE` trong title.
- Giữ class name, method name, annotation name, hoặc Java symbol đúng casing thật.
- Giữ `vs` ở dạng lowercase trong title so sánh.
- Không dùng title chung chung như `Java Topic`, `Advanced Note`, hoặc `Misc`.

---

## 6. Quy tắc viết từng section trong note

### What is it

- Định nghĩa concept bằng ngôn ngữ đơn giản.
- Nói rõ concept này giải quyết vấn đề gì.
- Nếu có nhiều object/API liên quan, nêu vai trò của từng cái ngắn gọn.

### How I used to misunderstand it

- Ghi lại hiểu lầm phổ biến hoặc cách nghĩ sai của người mới học.
- Nên viết thật cụ thể, không viết chung chung.
- Nếu có 2–3 hiểu nhầm phổ biến thì tốt hơn chỉ có 1 câu rất mờ.

### How it actually works

- Giải thích cơ chế thật sự.
- Ưu tiên mental model trước implementation detail.
- Chỉ đi sâu vừa đủ để hiểu cách dùng đúng.
- Nếu concept liên quan JVM, memory, object identity, equality, lifecycle, hoặc concurrency thì phải nói rõ phần dễ hiểu nhầm.
- Với concept khó, nên thêm một trong các loại scaffold sau:
  - bảng so sánh
  - decision matrix
  - ASCII flow
  - timeline
  - object graph đơn giản

### Code example

- Code phải ngắn, chạy được, và minh họa đúng điểm chính.
- Không nhồi quá nhiều concept vào một ví dụ.
- Dùng Java syntax hiện đại nhưng vẫn dễ đọc.
- Nếu output quan trọng, thêm comment hoặc giải thích ngay dưới code.
- Với inline `//` comment trong code example:
  - Nếu comment dài hoặc mang tính giải thích, đặt comment ở dòng ngay phía trên code nó mô tả.
  - Chỉ giữ comment cùng dòng khi comment thật sự ngắn, thường dưới khoảng 22 ký tự hoặc chỉ 2–4 từ.
  - Ví dụ nên giữ inline: `// 2`, `// true`, `// false`, `// null`.
  - Ví dụ không nên để inline: `// primitive is boxed because List works with reference types`.
- Với concept dễ nhầm, ưu tiên example kiểu contrast:
  - mutate vs reassign
  - equals vs identity
  - compile-time vs runtime
  - thread-safe operation vs thread-safe business logic

### When to use / when NOT to use

- Chia rõ trường hợp nên dùng và không nên dùng.
- So sánh với alternative gần nhất nếu có.
- Không chỉ nói API này tốt; phải nói trade-off.

### How this connects to real Java projects

- Mỗi note nên nối concept Java với project thật khi có liên hệ trực tiếp, ví dụ Maven/Gradle build, JDK API, library phổ biến, hoặc Spring khi concept liên quan rõ ràng.
- Không cần ép liên hệ nếu quá gượng, nhưng với `01_Core` trở đi thường nên cố gắng có một bridge hợp lý.
- Khi nhắc Spring, giữ nó như một case cụ thể, không như đại diện duy nhất cho “Java project thật”.

### Gotchas

- Liệt kê lỗi dễ gặp, edge cases, hoặc behavior gây bất ngờ.
- Mỗi bullet nên có hậu quả cụ thể.
- Ưu tiên lỗi thực tế có thể gặp khi viết Java/Spring production code.

### Handbook rule

- Chốt 2-5 bullet có thể dùng khi review/debug code thật.
- Phân biệt rõ hard rule, default convention, và local preference nếu note có cả ba.
- Không lặp lại toàn bộ nội dung note; chỉ giữ những rule người đọc cần nhớ khi ra quyết định.

### Check yourself

- Đây không phải bài tập implementation.
- Đây là 3–5 câu hỏi ngắn để người đọc tự kiểm tra xem mình có thật sự hiểu concept không.
- Câu hỏi nên đánh trúng hiểu nhầm cốt lõi của note.

Ví dụ tốt:

```markdown
## Check yourself

- Nếu một method return rồi mà object vẫn còn sống, ai đang giữ reference tới nó?
- Vì sao `final` không tự động làm object immutable?
- Khi nào `ConcurrentHashMap` vẫn chưa đủ để business logic thread-safe?
```

### Links

- Dùng để chứa official docs và note liên quan.
- Ưu tiên Oracle docs, JDK Javadocs, JLS, JVMS, Spring docs, hoặc internal note links.
- Mỗi Obsidian wiki-link phải đứng trên một dòng riêng.
- Nếu có external links, nên chọn link authoritative thay vì blog ngẫu nhiên.

---

## 7. Quy tắc pedagogy rút ra từ quá trình làm vault này

### 7.1. Khi nào phải thêm bảng hoặc diagram

Rất nên thêm bảng/diagram nếu note thuộc một trong các dạng sau:

- so sánh nhiều lựa chọn gần nhau
- khái niệm JVM/runtime khó hình dung
- lifecycle / reachability / memory model
- compile-time vs runtime
- visibility vs atomicity vs locking
- interface vs abstract class vs composition
- collection choices

### 7.2. Pattern dạy hiệu quả nhất

Khi rewrite note khó, pattern thường hiệu quả là:

1. nêu invariant hoặc takeaway một câu
2. nêu hiểu nhầm phổ biến
3. cho ví dụ rất nhỏ
4. chốt bằng quy tắc thực dụng

### 7.3. Đừng để note chỉ “đúng” mà vẫn khó học

Những dấu hiệu note cần cải thiện:

- paragraph quá dài
- nhiều thuật ngữ chưa định nghĩa
- analogy có nhưng takeaway không chốt rõ
- code example đúng nhưng không contrast với case sai
- links yếu hoặc không có official refs

### 7.4. Definition of done cho note mức handbook

Trước khi xem một note là xong, kiểm tra nhanh:

- Có trả lời được “khi gặp tình huống thật, mình nên làm gì?” không?
- Có chỉ ra ít nhất một hiểu nhầm, gotcha, hoặc anti-pattern không?
- Có ví dụ Java đủ nhỏ để copy chạy hoặc đủ cụ thể để review code không?
- Có phân biệt default convention với hard rule không?
- Có official reference nếu note nói về Java language, JVM behavior, JDK API, Javadoc, build tool, testing, hoặc style chuẩn không?
- Có link sang note liên quan nếu người đọc cần đào sâu không?
- Có dùng tiếng Việt tự nhiên nhưng giữ English technical terms đúng chỗ không?

---

## 8. Rules riêng cho `00_Mental-Models`

- Không bắt buộc `## Exercises`.
- Bắt buộc `## Check yourself`.
- Nên có visual scaffold gần như ở mọi note khó.
- Nên có official refs nếu khái niệm chạm JVM, JLS, memory, class loading, equality, hoặc concurrency.
- Cực kỳ cẩn thận với các câu dễ bị dạy sai như:
  - `reference = pointer`
  - `final = immutable`
  - `object chết khi method return`
  - `compile được nghĩa là gần như đúng`

---

## 9. Rules riêng cho `01_Core` và `02_Advanced`

- Từ `01_Core/Collections` trở đi: bắt buộc có cả `## Check yourself` và `## Exercises`.
- Với note comparison-heavy, gần như bắt buộc có bảng hoặc decision matrix.
- Với note subtle như memory, reflection, multithreading, generics, nên thêm official references.
- Với note đã có exercises, `Check yourself` không thay thế exercises; hai phần có mục đích khác nhau.

---

## 10. Rules riêng cho `03_Evergreen-Notes`, `04_Lessons-from-bugs`, và `05_Open-Questions`

### `03_Evergreen-Notes`

- Không bắt buộc dùng đủ section chuẩn của concept note nếu note là principle hoặc heuristic.
- Bắt buộc có một câu chốt rõ “ý chính cần nhớ”.
- Phải nêu phạm vi áp dụng: khi nào principle này đúng, khi nào dễ sai.
- Nên có ít nhất một ví dụ Java nhỏ hoặc một scenario thực tế.
- Nên link về note nền trong `01_Core` hoặc `02_Advanced` nếu principle dựa trên concept cụ thể.

Format khuyến nghị:

```markdown
# Evergreen idea

## Core idea

## Why it matters

## Example

## When this breaks down

## Related notes
```

### `04_Lessons-from-bugs`

- Không viết như postmortem dài nếu bug nhỏ; ưu tiên learning dễ tái dùng.
- Bắt buộc có symptom, root cause, fix/prevention, và lesson.
- Nên có minimal reproduction nếu bug liên quan Java/JVM/Spring/build-tool behavior.
- Nếu bug phụ thuộc version, framework, JVM flag, OS, hoặc build tool, phải ghi rõ environment.

Format khuyến nghị:

```markdown
# Bug lesson title

## Symptom

## Root cause

## Minimal reproduction

## Fix / prevention

## Lesson

## Related notes
```

### `05_Open-Questions`

- Không giả vờ chắc chắn khi câu hỏi chưa resolve.
- Bắt buộc có current hypothesis, evidence đã biết, phần chưa rõ, và next step.
- Khi câu hỏi được resolve, update hoặc link sang note chính thức thay vì để open question stale.

Format khuyến nghị:

```markdown
# Open question title

## Question

## Current hypothesis

## What I know so far

## What is still unclear

## Next step

## Links
```

---

## 11. Format chuẩn cho bài tập trong Markdown

Mỗi bài tập trong `## Exercises` phải có format đầy đủ như sau:

````markdown
### Exercise 1: Title

Difficulty: Easy

Problem:
Mô tả bài toán rõ ràng.

Example 1:

Input:
```text
...
```

Output:
```text
...
```

Explanation:
Giải thích vì sao output đúng.

Constraints:

- Constraint 1
- Constraint 2
- Constraint 3
````

Nếu bài có nhiều ví dụ, dùng `Example 1`, `Example 2`, v.v.

---

## 12. Quy tắc thiết kế bài tập

- Bài tập phải kiểm tra đúng concept, không chỉ kiểm tra syntax.
- Mỗi bài nên có một trọng tâm chính.
- Nên tăng độ khó dần trong cùng một note.
- Difficulty dùng một trong các mức: `Easy`, `Medium`, `Hard`.
- `Easy` kiểm tra usage cơ bản hoặc một edge case nhỏ.
- `Medium` kiểm tra lựa chọn data structure, loop, branching, hoặc kết hợp nhiều bước.
- `Hard` chỉ dùng khi thật sự cần algorithm phức tạp hơn.
- Tránh bài tập cần framework, database, network, hoặc file I/O thật.
- Input/output nên dùng Java primitives, arrays, `String`, `List`, `Map`, hoặc các collection cơ bản.
- Constraints phải đủ rõ để người implement biết giới hạn dữ liệu.
- Không yêu cầu in ra console nếu bài tập có thể return value.
- Ưu tiên method pure function: nhận input, trả output, không side effect.

---

## 13. Mapping từ note sang Java exercise skeleton

Mỗi bài tập trong note phải có một file `.java` tương ứng trong Maven project:

```text
java-exercises
```

Package root bắt buộc:

```java
dev.hoangtuan.java.exercises
```

Package con nên phản ánh chapter và topic.

Ví dụ:

```java
dev.hoangtuan.java.exercises.collections.hashmap
dev.hoangtuan.java.exercises.controlflow.loops
dev.hoangtuan.java.exercises.advanced.testing.junit5
```

---

## 14. Quy tắc đặt tên Java exercise file

- Mỗi exercise là một `public final class` riêng.
- Class name dùng PascalCase.
- Class name nên là tên bài tập rút gọn, không thêm hậu tố không cần thiết.
- Không dùng tên quá chung như `Exercise1`, `Problem1`, `Test`, `Solution`.

Ví dụ tốt:

```text
FirstDuplicate.java
CountFrequencies.java
MergeSortedLists.java
ValidateBracketSequence.java
```

---

## 15. Cấu trúc chuẩn của Java exercise skeleton

Mỗi file exercise skeleton nên theo cấu trúc này:

```java
package dev.hoangtuan.java.exercises.collections.hashmap;

/**
 * Solves the first duplicate problem.
 *
 * <p><b>Problem:</b> Given an integer array, return the first value that appears more than once.
 *
 * <p><b>Example:</b> {@code [2, 1, 3, 5, 3, 2]} returns {@code 3}.
 *
 * <p><b>Constraints:</b> input length is between 0 and 100000.
 */
public final class FirstDuplicate {
    private FirstDuplicate() {
    }

    /**
     * Returns the first duplicated value in encounter order.
     *
     * <p><b>Contract:</b> scans values from left to right and returns the first value whose second occurrence is encountered first.
     * <p><b>Why use:</b> practice choosing a lookup structure instead of nested loops.
     * <p><b>Non-obvious behavior:</b> returns {@code -1} when no duplicate exists.
     *
     * @param numbers input values
     * @return first duplicated value, or {@code -1} when none exists
     * @throws IllegalArgumentException if {@code numbers} is {@code null}
     */
    public static int solve(int[] numbers) {
        throw new UnsupportedOperationException("Not implemented yet");
    }
}
```

---

## 16. Quy tắc implementation trong skeleton

- Không viết lời giải.
- Không viết pseudo-code trong body.
- Method body luôn dùng đúng:

```java
throw new UnsupportedOperationException("Not implemented yet");
```

- Class nên là utility-style:
  - `public final class ClassName`
  - private constructor rỗng
  - static `solve(...)` method
- Method chính nên tên là `solve` để nhất quán.
- Không thêm test solution trừ khi được yêu cầu riêng.

---

## 17. Quy tắc Javadoc cho Java exercise

Áp dụng đúng Javadoc template đã thống nhất.

### Class-level Javadoc

- mô tả bài toán cần giải
- cho ví dụ input/output ngắn
- nêu constraints quan trọng
- không mô tả thuật toán lời giải

### Method-level Javadoc

```java
/**
 * Verb + object + condition if needed.
 *
 * <p><b>Contract:</b> non-obvious guarantee.
 * <p><b>Why use:</b> reason to choose this exercise or API practice.
 * <p><b>Non-obvious behavior:</b> null, exception, sentinel value, or side effect surprise.
 *
 * @param name description
 * @return description
 * @throws Type condition
 */
```

Spacing rules:

- một blank line sau summary sentence
- không có blank line giữa các `<p><b>...</b>` section
- một blank line giữa section `<p>` cuối và `@param` đầu tiên
- `@param`, `@return`, `@throws` không căn padding thừa

Inline tag rules:

- dùng `{@code null}`, `{@code true}`, `{@code false}` cho literals và keywords
- dùng `{@link ClassName}` hoặc `{@link ClassName#method}` khi reference type/method khác
- không viết bare `null`, `true`, `false` trong Javadoc prose

---

## 18. Quy tắc chọn input/output cho `solve`

- Ưu tiên signature đơn giản, gần với LeetCode.
- Dùng arrays khi bài tập thiên về index hoặc loop.
- Dùng `List<T>` khi bài tập thiên về Collections API.
- Dùng `Map<K, V>` khi bài tập cần key-value input hoặc output.
- Dùng `String` cho bài parsing, counting characters, validation.
- Tránh custom class nếu không thật sự cần.
- Nếu output có thể không tồn tại, ưu tiên sentinel rõ ràng như `-1`, empty list, empty map.
- Với skeleton cho người mới, tránh lạm dụng `Optional` nếu nó không phải concept chính của note.

---

## 19. Quy tắc verification

### Với notes

Sau khi tạo hoặc sửa note, phải kiểm tra ít nhất:

- section order đúng chưa
- `Check yourself` đã đúng vị trí chưa
- nếu note có exercises, `Exercises` còn nguyên format chưa
- links nội bộ có resolve không
- mỗi wiki-link có đứng riêng một dòng không
- nếu note so sánh nhiều thứ, đã có bảng/diagram đủ rõ chưa

### Với Java exercise skeleton

Sau khi tạo hoặc sửa Java exercise skeleton, phải compile Maven project:

```powershell
mvn -q -DskipTests compile
```

Chạy command trong thư mục:

```text
java-exercises
```

Chỉ báo hoàn thành khi compile thành công hoặc nêu rõ lỗi compile nếu có.

---

## 20. Quy tắc batching khi viết nhiều note

- Không nhất thiết viết cả chapter một lần.
- Ưu tiên batch theo nhóm concept liên quan.
- Batch nên đủ nhỏ để review được.
- Với folder lớn, ưu tiên batch theo teaching payoff, không chỉ theo alphabet.

Ví dụ payoff cao thường là:

- folder comparison-heavy như `Collections`
- folder concept-dense như `Memory`, `Multithreading`, `Generics`
- folder framework-adjacent như `Reflection-and-Annotation`

---

## 21. Definition of Done cho một note có bài tập

Một note được xem là xong khi:

- File Markdown có đủ section chuẩn.
- Nội dung giải thích đúng concept chính.
- Có ví dụ code Java ngắn và đúng trọng tâm.
- Có phần Spring connection nếu có liên hệ hợp lý.
- Có Gotchas thực tế.
- Có `Check yourself` hữu ích, không hời hợt.
- Có 3 đến 5 bài tập LeetCode-style nếu note thuộc vùng bắt buộc có exercises.
- Mỗi bài tập có title, difficulty, problem, examples, input, output, explanation, constraints.
- Có `Links` đủ mạnh: internal links hợp lý và official refs khi concept subtle.
- Không có lời giải trong note.

## 22. Definition of Done cho note không có bài tập

Áp dụng cho các note kiểu `00_Mental-Models` hoặc note khái niệm nền không yêu cầu exercises.

Một note được xem là xong khi:

- Có đủ section chuẩn, trừ `Exercises` nếu khu vực đó không yêu cầu.
- Có `Check yourself` đúng trọng tâm.
- Có ít nhất một mental-model aid nếu concept trừu tượng.
- Có official refs nếu concept chạm JVM/JLS/JVMS/runtime subtleties.
- Không oversimplify tới mức sai bản chất.

---

## 23. Definition of Done cho evergreen note, bug lesson, và open question

### Evergreen note

Một evergreen note được xem là xong khi có core idea rõ, phạm vi áp dụng, scenario cụ thể, links tới note nền liên quan, và không bị lẫn với nhật ký học ngắn hạn.

### Bug lesson

Một bug lesson được xem là xong khi có symptom cụ thể, root cause rõ, fix/prevention cụ thể, lesson tổng quát hóa được, và minimal reproduction nếu bug liên quan Java/JVM/Spring/build-tool behavior.

### Open question

Một open question được xem là đủ tốt khi có câu hỏi chính rõ, current hypothesis, evidence đã biết, phần còn thiếu, next step cụ thể, và cơ chế update/link sang note chính thức khi đã resolve.

---

## 24. Definition of Done cho Java exercise skeleton

Một Java exercise skeleton được xem là xong khi:

- Có đúng package theo chapter/topic.
- Class name rõ nghĩa, PascalCase.
- Class là `public final`.
- Có private constructor.
- Có class-level Javadoc mô tả problem/example/constraints.
- Có `public static solve(...)` method.
- Method-level Javadoc đúng template.
- Body chỉ có `throw new UnsupportedOperationException("Not implemented yet");`.
- Maven compile thành công.

---

## 25. Những điều không làm nếu không được yêu cầu

- Không viết solution cho bài tập.
- Không thêm unit tests solution.
- Không thêm framework hoặc dependency mới.
- Không refactor cấu trúc Maven project nếu skeleton hiện tại vẫn compile.
- Không đổi package root `dev.hoangtuan.java.exercises`.
- Không commit git nếu chưa được yêu cầu.
- Không để broken wiki-links mới xuất hiện.

---

## 26. Checklist nhanh trước khi báo xong

### Với note

- Note có đủ section chưa?
- `Check yourself` có chưa và đã đúng vị trí chưa?
- Nếu note có exercises, format bài tập có còn chuẩn chưa?
- Có chỗ nào quá dài, quá mơ hồ, hoặc chỉ đúng mà chưa dễ hiểu không?
- Có bảng/diagram nếu concept rất dễ nhầm không?
- Links nội bộ có resolve không?
- Official links có đủ mạnh không?

### Với Java skeleton

- Java skeleton có đúng package chưa?
- Javadoc có đúng spacing và inline tags chưa?
- Skeleton có vô tình chứa solution không?
- `mvn -q -DskipTests compile` đã chạy thành công chưa?
