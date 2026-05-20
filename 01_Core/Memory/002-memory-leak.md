# Memory leak

## What is it

Memory leak trong Java là tình huống object không còn cần về mặt business, nhưng vẫn còn `reachable`, nên GC không thể thu hồi.

Mental model:

- Java không leak vì “quên free” như trong ngôn ngữ manual memory management.
- Java leak vì code vẫn giữ reference quá lâu.

Nói ngắn gọn, leak trong Java là bug về `lifetime` và `ownership`, không phải bug về cú pháp cấp phát.

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là nghĩ Java có GC nên không có memory leak. Sai. GC chỉ nhìn object graph, không hiểu object nào “đã hết hạn dùng” về mặt business.

Nếu object còn bị giữ bởi:

- static collection
- cache không eviction
- listener chưa unregister
- `ThreadLocal` chưa `remove()`
- queue hoặc map sống lâu

thì với GC, object đó vẫn còn hợp lệ để giữ lại.

## How it actually works

Leak thường biểu hiện bằng heap usage tăng dần theo traffic hoặc thời gian chạy. Triệu chứng hay gặp là old generation phình to, GC chạy nhiều hơn, pause time xấu hơn, hoặc cuối cùng đụng `OutOfMemoryError`.

Điểm quan trọng là nhìn vào `retained path`, tức đường reference từ `GC root` tới object bị giữ lại.

### Leak chain điển hình

```text
GC Root
  |
  v
static CACHE
  |
  v
Map entry
  |
  v
UserSession
  |
  v
Large DTO / byte[] / graph
```

GC không thấy đây là “rác”. Nó chỉ thấy cả graph đó vẫn còn reachable.

### Leak source phổ biến

| Nguồn giữ reference | Vì sao dễ leak | Cách nghĩ đúng |
|---|---|---|
| `static` field / static collection | Sống gần như suốt app lifetime | Chỉ giữ dữ liệu thật sự global |
| Cache không giới hạn | Key space tăng theo traffic | Phải có size limit, TTL, hoặc eviction |
| `ThreadLocal` | Thread pool reuse thread rất lâu | Set xong phải cleanup |
| Listener / callback | Quên unregister | Ai đăng ký thì phải có lifecycle cleanup |
| Collection trong singleton bean | Bean sống lâu, data không được dọn | Tách state ngắn hạn khỏi bean dài hạn |

Debug leak thường cần heap dump hoặc profiler để trả lời hai câu hỏi:

1. Object nào chiếm retained size lớn?
2. Ai đang giữ nó sống?

## Code example

```java
import java.util.HashMap;
import java.util.Map;

class UserCache {
    private final Map<String, byte[]> values = new HashMap<>();

    void put(String id, byte[] data) {
        // if id space grows forever and nothing is evicted, memory keeps growing
        values.put(id, data);
    }
}
```

Đoạn code này không sai về cú pháp. Vấn đề nằm ở policy. Nếu `id` tăng mãi theo traffic và không có eviction, `values` sẽ giữ mọi object quá lâu và tạo hành vi leak.

## When to use / when NOT to use

Dùng mental model này khi bạn thấy memory tăng nhưng không giảm về baseline sau nhiều request hoặc nhiều batch.

Các rule thực tế:

- cache phải có eviction hoặc giới hạn rõ
- không giữ request object trong static field
- `ThreadLocal` trong thread pool phải cleanup
- đo retained size, đừng đoán bằng cảm giác

Không nên gắn nhãn “memory leak” chỉ vì memory tăng tạm thời. Có thể đó là warm-up, cache có chủ đích, hoặc allocation burst bình thường. Phải nhìn xu hướng và retained path.

## How this connects to Spring

Spring singleton beans sống cùng application context. Nếu singleton vô tình giữ request-specific data hoặc collection tăng mãi, leak sẽ tích lũy rất đều.

Các điểm dễ dính trong Spring:

- `@Cacheable` nhưng key space quá lớn
- listener / observer không cleanup
- lưu dữ liệu request vào bean singleton
- `ThreadLocal` trong filter, interceptor, async code

## Gotchas

- Static collection là leak source rất phổ biến.
- `ThreadLocal` leak nguy hiểm hơn khi thread được reuse trong pool.
- Cache không giới hạn có thể là leak theo behavior dù code vẫn “đúng syntax”.
- Heap lớn hơn chưa chắc sửa được leak, nhiều khi chỉ làm lỗi xuất hiện muộn hơn.

## Check yourself

- Vì sao Java vẫn có memory leak dù có GC?
- “Object không còn cần” và “object không còn reachable” khác nhau ở đâu?
- Vì sao retained path quan trọng hơn việc nhìn riêng object bị to?
- Một cache tăng mãi khác gì với cache có size limit hoặc TTL?
- Vì sao `ThreadLocal` đặc biệt nguy hiểm trong thread pool?

## Exercises

### Bài 1: Detect Unbounded Cache Risk
Độ khó: Dễ

Đề bài:
Cho maximum size của một cache, trả về `true` nếu cache này rủi ro vì maximum size là `0` hoặc âm.

Ví dụ 1:
Đầu vào:
```text
maxSize = 0
```

Đầu ra:
```text
true
```

Giải thích:
Max size không dương đồng nghĩa với việc không có ràng buộc hữu ích trong bài này.

Ràng buộc:
- -1000 <= maxSize <= 1000000
- Trả về một boolean
- Chỉ dùng đúng rule đã cho

### Bài 2: Count Leaked Entries
Độ khó: Trung bình

Đề bài:
Cho array tuổi của entry và một TTL, đếm các entry có tuổi lớn hơn TTL.

Ví dụ 1:
Đầu vào:
```text
ages = [10, 50, 100], ttl = 30
```

Đầu ra:
```text
2
```

Giải thích:
Các entry có tuổi `50` và `100` đáng lẽ phải bị evict.

Ràng buộc:
- ages là non-null
- 0 <= ages.length <= 100000
- ttl >= 0

### Bài 3: Should Remove ThreadLocal
Độ khó: Dễ

Đề bài:
Cho biết code có chạy trong thread pool hay không và value có phải request-scoped hay không, trả về `true` khi nên gọi `ThreadLocal.remove()`.

Ví dụ 1:
Đầu vào:
```text
threadPool = true, requestScoped = true
```

Đầu ra:
```text
true
```

Giải thích:
Request-scoped data trên thread được reuse phải được cleanup.

Ràng buộc:
- Input là các giá trị boolean
- Return `true` only when both inputs are `true`
- Không mô hình hóa framework cleanup

## Links

- [[001-GC]]
- [[005-strong-soft-weak-phantom-reference]]
- [[006-off-heap-memory]]
- [[../../00_Mental-Models/008-object-lifecycle]]
- [Java SE 21, `ThreadLocal` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ThreadLocal.html)
- [Java SE 21, `WeakHashMap` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/WeakHashMap.html)
- [Java SE 21 Troubleshooting Guide](https://docs.oracle.com/en/java/javase/21/troubleshoot/)
