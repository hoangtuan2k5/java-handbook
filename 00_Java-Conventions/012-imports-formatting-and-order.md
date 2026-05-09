# Imports, Formatting, and Order

## Why this file exists

Nhiều rule formatting nên để tool lo, nhưng mình vẫn cần một baseline để không tạo diff lộn xộn hoặc import vô nghĩa.

## Rules

- Không dùng wildcard import trừ khi team đã thống nhất formatter cho phép.
- Xóa unused imports.
- Giữ formatting nhất quán với formatter của project nếu có.
- Đừng tạo style cá nhân khác hẳn codebase đang có.
- Ưu tiên auto-format hơn tranh luận bằng tay về khoảng trắng nhỏ.

## Good examples

```java
import java.util.List;
import java.util.Map;
```

## Bad examples

```java
import java.util.*;
import java.io.*;
```

## Notes

Formatting quan trọng, nhưng consistency quan trọng hơn preference cá nhân.

Wildcard import không làm runtime chậm hơn; vấn đề chính là readability và diff stability. Nó làm người đọc khó thấy type đến từ đâu, dễ tạo conflict tên, và khiến diff thay đổi bất ngờ khi formatter collapse/expand import. Nếu project có formatter thống nhất threshold wildcard rõ ràng, hãy theo formatter thay vì tranh luận thủ công.

## Formatting decision matrix

| Situation | Prefer | Avoid |
|---|---|---|
| Project has formatter | follow formatter | hand-format style debates |
| Imports unused | remove | leaving noise for IDE/build |
| Many imports from same package | explicit imports unless team allows wildcard | surprise wildcard imports |
| Static imports in tests | acceptable for assertions | overusing in production code |
| Existing codebase style differs | match local style | drive-by formatting unrelated files |

## Official references

- [Java Code Conventions: Declarations and Statements](https://www.oracle.com/java/technologies/javase/codeconventions-declarations.html)
- [Google Java Style Guide: Imports](https://google.github.io/styleguide/javaguide.html#s3.3-import-statements)

## Related rules

[[011-code-organization]]

[[015-javadoc-and-comments]]
