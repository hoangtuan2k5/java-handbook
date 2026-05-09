# ConcurrentHashMap vs HashMap

## What is it

`HashMap` là map nhanh cho single-thread hoặc external synchronization.

`ConcurrentHashMap` là map được thiết kế để nhiều thread đọc và ghi cùng lúc an toàn hơn mà không cần lock toàn bộ map cho mọi operation.

Nói đơn giản: `HashMap` giống sổ tay cá nhân, còn `ConcurrentHashMap` giống bảng công việc chung có cơ chế để nhiều người cập nhật mà không giẫm lên nhau quá dễ.

## How I used to misunderstand it

Nhiều dev nghĩ chỉ cần dùng `ConcurrentHashMap` là mọi logic liên quan map đều thread-safe.

Không đúng. Từng operation như `put()`, `get()`, `computeIfAbsent()` có guarantee riêng, nhưng chuỗi nhiều bước kiểu `if (!map.containsKey(k)) map.put(k, v)` vẫn có race nếu không dùng atomic method.

Hiểu nhầm khác là `HashMap` chỉ nguy hiểm khi có rất nhiều thread. Thật ra chỉ cần concurrent write không đồng bộ là state đã không còn đáng tin.

## How it actually works

`HashMap` không có cơ chế bảo vệ internal table khỏi concurrent mutation. Nếu nhiều thread cùng `put()`, resize, hoặc update bucket, behavior không còn đáng tin.

`ConcurrentHashMap` chia nhỏ chiến lược đồng bộ và dùng atomic hoặc locking ở mức cần thiết để cho phép nhiều thread thao tác tốt hơn so với lock toàn map.

Iterator của `ConcurrentHashMap` thường weakly consistent. Nó không ném `ConcurrentModificationException` chỉ vì map đang thay đổi, nhưng cũng không hứa snapshot hoàn hảo tại một thời điểm.

Ngoài ra, `ConcurrentHashMap` không cho phép `null` key hoặc `null` value để tránh ambiguity. `get()` trả `null` phải luôn có nghĩa là không có mapping.

### Comparison table

| Concern | `HashMap` | `ConcurrentHashMap` |
|---|---|---|
| Single-thread local use | Best fit | Có thể, nhưng thường không cần |
| Shared mutable state across threads | Không an toàn | Best fit tối thiểu |
| `null` key hoặc value | Cho phép | Không cho phép |
| Iterator behavior | Thường fail-fast | Weakly consistent |
| Atomic update helpers | Không | Có như `compute()`, `computeIfAbsent()`, `merge()` |

### Race vs atomic update

```text
// check-then-act race
Bad:  containsKey -> then put
// single atomic map operation
Good: computeIfAbsent or compute
```

## Code example

```java
import java.util.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        Map<String, Integer> counts = new ConcurrentHashMap<>();

        // atomic update
        counts.compute("login", (key, oldValue) -> oldValue == null ? 1 : oldValue + 1);
        // atomic create-if-missing
        counts.computeIfAbsent("logout", key -> 0);

        System.out.println(counts.get("login"));
    }
}
```

## When to use / when NOT to use

Dùng `HashMap` cho local variable, request-scoped data, build response tạm, hoặc map không bị nhiều thread mutate.

Dùng `ConcurrentHashMap` cho shared mutable map trong singleton service, cache đơn giản, counters, registry, hoặc state được nhiều thread truy cập.

Không dùng `ConcurrentHashMap` để che giấu thiết kế shared mutable state phức tạp. Nếu cần eviction, size limit, TTL, hoặc metrics tốt, hãy cân nhắc cache library như Caffeine.

## How this connects to Spring

Trong Spring Boot, singleton beans được share giữa request threads. Nếu bean giữ một `HashMap` mutable làm cache hoặc registry và nhiều request cùng update, đó là race condition thật.

`ConcurrentHashMap` thường là bước tối thiểu tốt hơn, nhưng vẫn phải dùng atomic methods như `computeIfAbsent()`.

Với cache production, Spring Cache cộng với provider phù hợp thường tốt hơn tự quản lý map trong service.

## Gotchas

- `ConcurrentHashMap` không cho phép `null` key hoặc `null` value.
- Check-then-act nhiều bước vẫn race. Dùng `compute()`, `computeIfAbsent()`, hoặc `merge()` khi phù hợp.
- Thread-safe map không làm object value bên trong thread-safe. Mutable value vẫn có thể bị race.

## Check yourself

- Vì sao `HashMap` local trong method thường an toàn, còn `HashMap` field trong singleton bean lại nguy hiểm?
- Vì sao `ConcurrentHashMap` chưa tự động làm toàn bộ business logic thread-safe?
- Nếu code đang làm `containsKey()` rồi mới `put()`, race nằm ở đâu?
- Vì sao `ConcurrentHashMap` cấm `null`?
- Khi requirement có TTL và max size, vì sao chỉ đổi `HashMap` sang `ConcurrentHashMap` vẫn chưa đủ?

## Exercises

### Bài 1: Local Map Safety
Độ khó: Dễ

Đề bài:
Cho một method tạo `HashMap` như local variable, hãy quyết định xem điều đó có an toàn trong một multi-request Spring app hay không.

Ví dụ 1:
Đầu vào:
```text
scope = "local variable", sharedAcrossThreads = false
```

Đầu ra:
```text
"Safe"
```

Giải thích:
Mỗi lần gọi method nhận một map instance riêng, nên các request thread không share cùng một mutable map.

Ràng buộc:
- Map không được escape ra khỏi method
- Không có reference nào tới map được lưu trong field hoặc static variable

### Bài 2: Atomic Cache Creation
Độ khó: Trung bình

Đề bài:
Refactor việc khởi tạo cache theo kiểu check-then-act thành một atomic operation của `ConcurrentHashMap`.

Ví dụ 1:
Đầu vào:
```text
key = "u1", map does not contain key
```

Đầu ra:
```text
created value is stored exactly once for that key
```

Giải thích:
`computeIfAbsent()` tránh race condition giữa bước checking và putting.

Ràng buộc:
- Nhiều thread có thể cùng request một key
- Value creation function chỉ nên được gọi khi thật sự cần

### Bài 3: Null Value In ConcurrentHashMap
Độ khó: Trung bình

Đề bài:
Xác định điều gì xảy ra khi code cố gắng `put` một giá trị `null` vào `ConcurrentHashMap`.

Ví dụ 1:
Đầu vào:
```text
map.put("a", null)
```

Đầu ra:
```text
NullPointerException
```

Giải thích:
`ConcurrentHashMap` không cho phép `null` để tránh mơ hồ giữa key không tồn tại và key được map tới `null`.

Ràng buộc:
- Key là non-null
- Value là null

### Bài 4: Design User Profile Cache
Độ khó: Khó

Đề bài:
Thiết kế một cache cho user profile trong Spring singleton service. So sánh cách dùng trực tiếp `ConcurrentHashMap` với cách dùng Spring Cache backed by Caffeine.

Ví dụ 1:
Đầu vào:
```text
requirements = ["concurrent reads", "TTL", "max size", "eviction"]
```

Đầu ra:
```text
"Prefer Caffeine or Spring Cache provider"
```

Giải thích:
`ConcurrentHashMap` xử lý concurrent access, nhưng tự nó không cung cấp TTL, eviction policy, hoặc size management.

Ràng buộc:
- Cache được share giữa các request thread
- Entry phải hết hạn theo thời gian
- Memory growth phải được giới hạn

## Links

- [[003-hash-map]]
- [[../Multithreading/002-race-condition]]
- [[../Multithreading/004-volatile]]
- `ConcurrentHashMap` Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html
