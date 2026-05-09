# Method Naming

## Why this file exists

Method name là phần người đọc nhìn nhiều nhất trong code Java. Nó quyết định API có đọc như tiếng Anh tự nhiên hay không.

## Rules

- Dùng lowerCamelCase.
- Ưu tiên verb-first.
- Tên method nên diễn tả outcome hoặc intent, không chỉ diễn tả implementation detail.
- Boolean-returning method nên bắt đầu bằng `is`, `has`, `can`, `should` khi hợp lý.
- Tránh tên quá ngắn kiểu `doIt`, `handle`, `process` nếu không có context mạnh đi kèm.

## Good examples

```java
calculateTotal()
findUserByEmail()
isExpired()
hasPermission()
shouldRetry()
```

## Bad examples

```java
doWork()
handleData()
process()
check()
runThing()
```

## Notes

Tên method tốt giúp giảm nhu cầu comment. Nếu method cần comment chỉ để giải thích nó làm gì, thường tên chưa đủ rõ.

## Official references

- [Java Code Conventions: Methods](https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html)
- [JLS: Methods](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.4)

## Related rules

[[009-variable-parameter-field-naming]]

[[011-code-organization]]
