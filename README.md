# Java

Written by Hoàng Tuấn  
Contact: tuan@hoangtuan.dev

## Purpose

Folder này là knowledge base Java trong vault: từ mental model nền, convention, core language, JVM/runtime, tới bài học từ bug và exercise skeleton.

Mục tiêu không phải là học thuộc API rời rạc, mà là xây một hệ thống note giúp mình:

- hiểu đúng Java language và JVM runtime
- viết code Java/Spring dễ đọc hơn
- tra cứu nhanh khi gặp bug hoặc design trade-off
- luyện tập bằng exercise skeleton khi concept cần thực hành

## Scope

README này là entry point cho toàn bộ folder Java. Nó không thay thế note chi tiết bên trong; mục tiêu của nó là giúp điều hướng, chọn thứ tự đọc, và nhắc lại các rule làm việc quan trọng.

## Suggested reading path

Nếu học từ đầu, nên đi theo thứ tự:

1. `00_Mental-Models/`
2. `00_Java-Conventions/`
3. `01_Core/Types-and-Variables/`
4. `01_Core/Control-Flow/`
5. `01_Core/Object-Methods/`
6. `01_Core/Collections/`
7. `01_Core/Generics/`
8. `01_Core/Functional/`
9. `01_Core/Memory/`
10. `01_Core/Multithreading/`
11. `02_Advanced/`

Các folder `03_Evergreen-Notes/`, `04_Lessons-from-bugs/`, và `05_Open-Questions/` nên dùng như reference khi gặp vấn đề thật.

## Folder map

- `00_Mental-Models/`: model nền để tránh hiểu sai Java/JVM.
- `00_Java-Conventions/`: convention tổ chức, naming, exception, logging, Javadoc.
- `01_Core/`: kiến thức Java nền cần dùng thường xuyên.
- `02_Advanced/`: JVM internals, concurrency nâng cao, performance, security, testing, design patterns.
- `03_Evergreen-Notes/`: principle và heuristic có giá trị lâu dài.
- `04_Lessons-from-bugs/`: bug lessons từ lỗi Java phổ biến.
- `05_Open-Questions/`: câu hỏi chưa resolve hoặc cần research thêm.
- `java-exercises/`: Maven project chứa skeleton bài tập.

## Working rules

- Khi thêm/sửa note, đọc `note-and-exercise-guidelines.md` trước.
- Với concept subtle, phân biệt rõ Java language, JVM/JDK, build tool, và Spring/framework behavior.
- Với exercise skeleton, không viết solution và phải giữ project compile được.

## How to use this folder

- Nếu đang học từ đầu, đi theo reading path rồi xuống README của từng subfolder.
- Nếu đang debug bug thật, đi thẳng vào nhóm concept liên quan rồi quay lại mental model khi cần.
- Nếu đang thêm note mới, kiểm tra guideline trước để giữ structure và pedagogy nhất quán.

## Related notes

[[note-and-exercise-guidelines]]
