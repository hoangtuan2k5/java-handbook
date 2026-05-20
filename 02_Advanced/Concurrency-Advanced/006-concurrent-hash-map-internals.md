# ConcurrentHashMap internals

## What is it

`ConcurrentHashMap` là concurrent map tối ưu cho truy cập đồng thời cao, nhất là khi nhiều thread cùng đọc và cập nhật key khác nhau.

Mental model đúng không phải là “`HashMap` có một cái lock to hơn”. Nó là map được thiết kế để:

- cho read path đi theo đường rất nhẹ,
- giảm contention xuống phạm vi nhỏ hơn toàn map,
- cung cấp atomic operation ở level từng key hoặc từng map method phù hợp.

## How I used to misunderstand it

Mình từng nghĩ chỉ cần thay `HashMap` bằng `ConcurrentHashMap` là mọi pattern dùng map đều thread-safe.

Sai ở chỗ `ConcurrentHashMap` thread-safe chủ yếu ở **operation level**. Nếu workflow của bạn là nhiều bước như `get` rồi quyết định `put`, bạn vẫn có thể race giữa hai thread nếu không dùng đúng API atomic như `computeIfAbsent`, `compute`, hoặc `merge`.

Hiểu nhầm thứ hai là nghĩ iteration trên `ConcurrentHashMap` giống `HashMap` với lock bên dưới. Không đúng. Iterator của nó là **weakly consistent**, tức là có thể thấy một phần update mới, bỏ lỡ một phần khác, nhưng không ném `ConcurrentModificationException` theo kiểu fail-fast quen thuộc.

## How it actually works

Với JDK hiện đại, mental model hữu ích là table chung với coordination tinh hơn ở từng vùng dữ liệu. Read path cố tránh lock toàn cục. Update path dùng CAS, volatile semantics, và synchronized ở phạm vi bin hoặc node khi cần.

Điều này dẫn tới ba takeaway lớn cho người dùng:

1. `ConcurrentHashMap` tốt cho concurrent access cao.
2. Nó không biến multi-step business workflow thành atomic magically.
3. Iteration của nó ưu tiên tiến lên được trong môi trường concurrent, chứ không hứa snapshot hoàn hảo.

### Operation-level safety khác workflow-level safety ở đâu

| Pattern | An toàn với `ConcurrentHashMap` không | Vì sao |
|---|---|---|
| `get(key)` | Có | Một operation đơn |
| `put(key, value)` | Có | Một operation đơn |
| `computeIfAbsent(key, fn)` | Có, đúng intent hơn | Atomic theo key |
| `get(key)` rồi `put(key, value)` | Không chắc | Hai bước có cửa race |
| Cập nhật nhiều key giữ invariant chung | Không đủ | Cần coordination ở mức cao hơn |

### Weakly consistent iteration nghĩa là gì

| Điều iterator hứa | Điều iterator không hứa |
|---|---|
| Không fail-fast kiểu `ConcurrentModificationException` trong use case concurrent bình thường | Không phải snapshot bất biến của map tại một thời điểm |
| Có thể duyệt được trong khi map đang đổi | Không đảm bảo thấy mọi update mới nhất |
| Hợp cho thống kê gần đúng hoặc traversal concurrent | Không hợp cho logic cần ảnh chụp chính xác tuyệt đối |

### Khi nào nên chọn API nào

| Nhu cầu | API nên nghĩ tới trước |
|---|---|
| Tạo value nếu key chưa có | `computeIfAbsent` |
| Cộng dồn vào value cũ | `merge` hoặc `compute` |
| Đọc đơn giản | `get` |
| Cần snapshot ổn định để xuất báo cáo | Copy sang cấu trúc khác trước |

## Code example

```java
import java.util.concurrent.ConcurrentHashMap;

ConcurrentHashMap<String, Integer> counts = new ConcurrentHashMap<>();

counts.compute("java", (key, value) -> value == null ? 1 : value + 1);
```

`compute` gom logic đọc và ghi vào cùng một atomic map operation. Đây chính là khác biệt lớn giữa “map thread-safe” và “workflow thread-safe”.

## When to use / when NOT to use

Dùng `ConcurrentHashMap` cho:

- local cache nhỏ,
- registry dùng chung,
- counter map theo key,
- dedup table,
- metadata được nhiều thread đụng tới.

Không dùng hoặc cần cân nhắc kỹ khi:

- bạn cần ordered iteration,
- cần `null` key hoặc `null` value,
- cần snapshot chính xác tuyệt đối trong lúc concurrent update,
- invariant business trải dài qua nhiều key hoặc nhiều cấu trúc dữ liệu.

## How this connects to real Java projects

Trong Spring, `ConcurrentHashMap` rất hay xuất hiện trong singleton bean làm local cache, metadata registry, idempotency helper, hoặc state theo key.

Nhưng đừng để nó che mất câu hỏi kiến trúc lớn hơn. Nếu state quan trọng phải survive restart, phải share giữa nhiều instance, hoặc phải transactionally consistent với database, in-memory `ConcurrentHashMap` chỉ là một phần rất nhỏ của câu chuyện.

## Gotchas

- `ConcurrentHashMap` không cho `null` key hoặc `null` value.
- `get` rồi `put` riêng vẫn race. Hãy nghĩ theo atomic API của map trước.
- Iterator là weakly consistent, không phải snapshot ổn định.
- Thread-safe ở level operation không có nghĩa toàn bộ business workflow đã thread-safe.

## Handbook rule

- `ConcurrentHashMap` không cho `null` key/value; thiết kế API phù hợp.
- Check-then-act phải dùng atomic API (`putIfAbsent`, `compute`, `merge`); không split `get` rồi `put`.
- Iterator weakly consistent, không phải snapshot ổn định.
- Per-entry atomicity không đảm bảo workflow business thread-safe.
- Cần ordered iteration phải đổi sang `ConcurrentSkipListMap` hoặc đổi data structure.

## Check yourself

- Vì sao thay `HashMap` bằng `ConcurrentHashMap` chưa đủ để sửa mọi race condition quanh map?
- `computeIfAbsent` khác `get` rồi `put` ở điểm atomic nào?
- Weakly consistent iteration phù hợp cho loại use case nào, và không hợp cho loại nào?
- Nếu logic cần giữ invariant giữa nhiều key, vì sao `ConcurrentHashMap` một mình chưa đủ?
- Vì sao `null` key/value bị cấm lại hữu ích cho API concurrent?

## Exercises

### Exercise 1: Count Bin Collisions

Độ khó: Easy

Đề bài:
Cho `bucketCount` và mảng `bucketIndexes`, trong đó mỗi phần tử là bucket mà một key rơi vào. Trả về tổng số collision, nghĩa là số key vượt quá key đầu tiên của mỗi bucket.

Ví dụ 1:

Đầu vào:
```text
bucketCount = 4
bucketIndexes = [0, 1, 1, 3, 1]
```

Đầu ra:
```text
2
```

Giải thích:
Bucket `1` chứa `3` key, nên tạo `2` collision. Các bucket khác không tạo collision.

Ràng buộc:

- `1 <= bucketCount <= 100000`
- `0 <= bucketIndexes[i] < bucketCount`
- `0 <= bucketIndexes.length <= 100000`

### Exercise 2: Should Resize Concurrent Table

Độ khó: Easy

Đề bài:
Cho `size`, `capacity`, và `threshold`. Trả về `true` nếu table nên resize theo rule `size >= threshold` và `capacity > 0`, ngược lại trả về `false`.

Ví dụ 1:

Đầu vào:
```text
size = 12
capacity = 16
threshold = 12
```

Đầu ra:
```text
true
```

Giải thích:
Khi số phần tử chạm threshold, resize nên được kích hoạt.

Ràng buộc:

- `0 <= size, capacity, threshold <= 1000000`
- Nếu `capacity = 0`, luôn trả về `false`
- Không cần mô phỏng resize thật

### Exercise 3: Select Concurrent Map API

Độ khó: Medium

Đề bài:
Cho ba cờ `createIfMissing`, `accumulateValue`, `orderedIterationRequired`. Nếu cần ordered iteration, trả về `"not-concurrent-hash-map"`. Nếu cần cộng dồn vào value hiện có, trả về `"merge"`. Nếu chỉ cần tạo khi key chưa tồn tại, trả về `"computeIfAbsent"`. Ngược lại trả về `"get"`.

Ví dụ 1:

Đầu vào:
```text
createIfMissing = true
accumulateValue = false
orderedIterationRequired = false
```

Đầu ra:
```text
"computeIfAbsent"
```

Giải thích:
Đây là compound action cần atomic initialization theo key.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về một trong bốn nhãn đã nêu
- Không dùng `null` như sentinel output

## Links

- [[../../01_Core/Collections/010-concurrent-hash-map-vs-hash-map]]
- [[../../01_Core/Collections/004-iterator]]
- [[../../01_Core/Multithreading/002-race-condition]]
- [[../../01_Core/Multithreading/005-atomic-classes]]
- [[001-reentrant-lock-vs-synchronized]]
- [JDK 21 `ConcurrentHashMap`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html)
- [JDK 21 `ConcurrentMap`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentMap.html)
- [JDK 21 `Map`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Map.html)
