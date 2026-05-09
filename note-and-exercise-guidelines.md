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
- `java-exercises/`

---

## 2. Mục tiêu theo từng khu vực

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

### 4.1. Cấu trúc chuẩn tối thiểu cho note khái niệm

Mỗi note nên dùng đúng các section sau, theo thứ tự này:

```markdown
# Topic Name

## What is it

## How I used to misunderstand it

## How it actually works

## Code example

## When to use / when NOT to use

## How this connects to Spring

## Gotchas

## Links
```

### 4.2. Self-check scaffold

Với note dạy concept, đặc biệt là `00_Mental-Models` và `01_Core`, nên có thêm:

```markdown
## Check yourself
```

Section này nên đặt:

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

## How this connects to Spring

## Gotchas

## Check yourself

## Exercises

## Links
```

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

### How this connects to Spring

- Mỗi note nên nối concept Java với Spring nếu có liên hệ thực tế.
- Không cần ép liên hệ nếu quá gượng, nhưng với `01_Core` trở đi thường nên cố gắng có một bridge hợp lý.

### Gotchas

- Liệt kê lỗi dễ gặp, edge cases, hoặc behavior gây bất ngờ.
- Mỗi bullet nên có hậu quả cụ thể.
- Ưu tiên lỗi thực tế có thể gặp khi viết Java/Spring production code.

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

## 10. Format chuẩn cho bài tập trong Markdown

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

## 11. Quy tắc thiết kế bài tập

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

## 12. Mapping từ note sang Java exercise skeleton

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

## 13. Quy tắc đặt tên Java exercise file

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

## 14. Cấu trúc chuẩn của Java exercise skeleton

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

## 15. Quy tắc implementation trong skeleton

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

## 16. Quy tắc Javadoc cho Java exercise

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

## 17. Quy tắc chọn input/output cho `solve`

- Ưu tiên signature đơn giản, gần với LeetCode.
- Dùng arrays khi bài tập thiên về index hoặc loop.
- Dùng `List<T>` khi bài tập thiên về Collections API.
- Dùng `Map<K, V>` khi bài tập cần key-value input hoặc output.
- Dùng `String` cho bài parsing, counting characters, validation.
- Tránh custom class nếu không thật sự cần.
- Nếu output có thể không tồn tại, ưu tiên sentinel rõ ràng như `-1`, empty list, empty map.
- Với skeleton cho người mới, tránh lạm dụng `Optional` nếu nó không phải concept chính của note.

---

## 18. Quy tắc verification

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

## 19. Quy tắc batching khi viết nhiều note

- Không nhất thiết viết cả chapter một lần.
- Ưu tiên batch theo nhóm concept liên quan.
- Batch nên đủ nhỏ để review được.
- Với folder lớn, ưu tiên batch theo teaching payoff, không chỉ theo alphabet.

Ví dụ payoff cao thường là:

- folder comparison-heavy như `Collections`
- folder concept-dense như `Memory`, `Multithreading`, `Generics`
- folder framework-adjacent như `Reflection-and-Annotation`

---

## 20. Definition of Done cho một note có bài tập

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

## 21. Definition of Done cho note không có bài tập

Áp dụng cho các note kiểu `00_Mental-Models` hoặc note khái niệm nền không yêu cầu exercises.

Một note được xem là xong khi:

- Có đủ section chuẩn, trừ `Exercises` nếu khu vực đó không yêu cầu.
- Có `Check yourself` đúng trọng tâm.
- Có ít nhất một mental-model aid nếu concept trừu tượng.
- Có official refs nếu concept chạm JVM/JLS/JVMS/runtime subtleties.
- Không oversimplify tới mức sai bản chất.

---

## 22. Definition of Done cho Java exercise skeleton

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

## 23. Những điều không làm nếu không được yêu cầu

- Không viết solution cho bài tập.
- Không thêm unit tests solution.
- Không thêm framework hoặc dependency mới.
- Không refactor cấu trúc Maven project nếu skeleton hiện tại vẫn compile.
- Không đổi package root `dev.hoangtuan.java.exercises`.
- Không commit git nếu chưa được yêu cầu.
- Không để broken wiki-links mới xuất hiện.

---

## 24. Checklist nhanh trước khi báo xong

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
