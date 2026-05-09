# Javadoc and Comments

## Why this file exists

Comment tốt giúp giải thích intent và contract. Comment tệ chỉ lặp lại code hoặc che dấu naming kém.

## Rules

- Chỉ viết Javadoc khi API public hoặc contract không hiển nhiên.
- Comment nên giải thích `why` hoặc `contract`, không chỉ lặp lại `what`.
- Nếu code cần comment chỉ để đọc hiểu basic behavior, thử đổi tên trước.
- Với public API hoặc method có contract không hiển nhiên, Javadoc nên nhất quán và nói rõ guarantee quan trọng.
- Không để comment cũ sai với code hiện tại.

## Good examples

```java
/**
 * Returns the first duplicated value in encounter order.
 */
```

```java
// Giữ nhánh này riêng vì downstream billing cần idempotency key khác.
```

## Bad examples

```java
// tăng i
i++;
```

```java
// Method này tính total
BigDecimal calculateTotal() { ... }
```

## Notes

Comment nên là phần bổ sung cho naming và structure, không phải miếng vá cho code khó đọc.

## Javadoc decision matrix

| Situation | Prefer | Avoid |
|---|---|---|
| Public API | Javadoc contract + params/returns/throws | no docs for non-obvious behavior |
| Private helper obvious from name | no Javadoc | comment noise |
| Method has null/exception/sentinel behavior | document non-obvious behavior | making caller read implementation |
| Comment repeats code | rename/extract instead | stale comments |
| Exercise skeleton | use agreed template exactly | solution walkthrough |

## Javadoc template reminder

```java
/**
 * Verb + object + condition if needed.
 *
 * <p><b>Contract:</b> non-obvious guarantee.
 * <p><b>Why use:</b> reason to choose this over alternatives.
 * <p><b>Non-obvious behavior:</b> null, exception, side effect, or sentinel behavior.
 *
 * @param name description
 * @return description
 * @throws Type condition
 */
```

## Official references

- [Oracle: How to Write Doc Comments](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)
- [Javadoc tool documentation](https://docs.oracle.com/en/java/javase/21/javadoc/javadoc.html)
- [Java Code Conventions: Comments](https://www.oracle.com/java/technologies/javase/codeconventions-comments.html)

## Related rules

[[008-method-naming]]

[[011-code-organization]]
