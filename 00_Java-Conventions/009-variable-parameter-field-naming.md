# Variable, Parameter, and Field Naming

## Why this file exists

Code Java thường dài hơn một số ngôn ngữ khác, nên tên biến tốt rất quan trọng để giảm noise mà vẫn giữ rõ nghĩa.

## Rules

- Dùng lowerCamelCase.
- Tên biến nên đủ rõ để người đọc không phải nhảy đi tìm type liên tục.
- Parameter nên khớp domain meaning, không nên đặt chỉ theo type.
- Tránh viết tắt khó đoán.
- Field của class nên đặt theo domain state, không đặt theo UI label hoặc tên tạm.

## Good examples

```java
orderTotal
retryCount
customerEmail
requestTimeoutMillis
```

## Bad examples

```java
tmp
data
obj
str
val
```

## Special cases

- Loop index `i`, `j` chấp nhận được trong loop ngắn.
- `id` là tên tốt khi domain đã rõ.
- `dto`, `entity`, `req`, `res` chỉ nên dùng khi context đã thực sự mạnh.

## Naming decision matrix

| Situation | Prefer | Avoid |
|---|---|---|
| Domain value | `customerEmail`, `orderTotal` | `str`, `val`, `data` |
| Time/duration | include unit like `timeoutMillis` | ambiguous `timeout` |
| Boolean | `isEnabled`, `hasPermission` | `flag`, `check` |
| Short loop | `i`, `j` acceptable | verbose noise in tiny scope |
| Wider scope | descriptive domain name | abbreviation only author understands |

## Official references

- [Java Code Conventions: Variables](https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html)

## Related rules

[[008-method-naming]]

[[010-constants-and-generics-naming]]
