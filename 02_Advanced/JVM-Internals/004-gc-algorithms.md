# GC Algorithms

## What is it

`GC algorithms` là các chiến lược JVM dùng để tìm object không còn reachable và thu hồi memory. Cùng là `garbage collection`, nhưng collector khác nhau tối ưu cho mục tiêu khác nhau: `throughput`, `pause time`, heap rất lớn, hoặc tính ổn định của latency.

Không có collector “tốt nhất” cho mọi workload. Mỗi collector là một trade-off engine.

## How I used to misunderstand it

Mình từng nghĩ GC chỉ là “lâu lâu dọn rác”, nên collector nào chắc cũng gần giống nhau.

Thực tế collector quyết định rất nhiều chuyện: chia heap ra sao, có ưu tiên `young generation` không, phần nào làm concurrent, phần nào `stop the world`, có move object hay không, và cố đạt loại mục tiêu nào.

## How it actually works

Nhiều collector dựa trên giả định `most objects die young`, nên heap thường được tổ chức để dọn object ngắn sống nhanh hơn.

Một flow rất đáng nhớ:

```text
Objects allocated
      |
      v
Most die in young generation
      |
      v
Survivors age across collections
      |
      v
Some get promoted to old generation
      |
      v
Old generation pressure drives heavier collections
```

| Khái niệm | Ý chính | Vì sao quan trọng |
| --- | --- | --- |
| `Minor GC` | Dọn vùng trẻ | Thường xảy ra thường xuyên hơn và rẻ hơn |
| `Promotion` | Object sống đủ lâu được đẩy sang old generation | Dự báo áp lực dài hạn |
| `Full GC` hoặc old-gen heavy work | Dọn sâu hơn, thường đắt hơn | Hay là tín hiệu có vấn đề workload hoặc sizing |

So sánh ở mức đủ dùng giữa các collector phổ biến:

| Collector | Thế mạnh chính | Đổi lại |
| --- | --- | --- |
| `Serial GC` | Đơn giản, hợp môi trường nhỏ | Pause dài vì một thread GC |
| `Parallel GC` | Ưu tiên throughput | Pause có thể kém đẹp hơn collector thiên low-pause |
| `G1` | Cân bằng pause goal và heap lớn vừa phải | Tuning và behavior phức tạp hơn `Serial`/`Parallel` |
| `ZGC` | Pause rất thấp trên heap lớn | Overhead và yêu cầu môi trường phù hợp hơn |
| `Shenandoah` | Low-pause, concurrent mạnh | Tùy JDK/distribution mà hỗ trợ và behavior khác nhau |

Điểm nên nhớ không phải là thuộc lòng từng internal detail, mà là biết collector hiện tại đang tối ưu cho điều gì, và dấu hiệu nào cho thấy nó không hợp workload.

## Code example

```java
byte[][] buffers = new byte[10_000][];

for (int i = 0; i < buffers.length; i++) {
    buffers[i] = new byte[1024];
}
```

Đoạn code này tạo áp lực allocation đáng kể. Nếu các object chết sớm, `young generation` behavior trở nên rất quan trọng. Nếu chúng bị giữ lâu hơn dự kiến, áp lực sẽ lan sang old generation và làm collector phải trả chi phí khác.

## When to use / when NOT to use

Hãy dùng note này khi:

- cần chọn collector theo goal vận hành,
- thấy pause time, GC CPU, hoặc tail latency tăng,
- muốn hiểu vì sao cùng heap size nhưng collector khác nhau cho kết quả khác nhau.

Không nên đổi collector chỉ vì một bài blog nói collector nào đó “luôn tốt hơn”. Không có collector thắng mọi shape traffic.

## How this connects to real Java projects

Spring apps thường tạo nhiều object ngắn sống ở web layer, JSON serialization, mapping, logging, hoặc reactive pipeline. Vì vậy behavior của vùng trẻ ảnh hưởng trực tiếp tới request latency.

Nếu app chạy trong container với memory budget chặt, chọn collector sai có thể khiến average latency trông ổn nhưng tail latency hoặc CPU under load lại tệ.

## Gotchas

- Pause thấp hơn thường đi kèm trade-off khác về overhead hoặc memory.
- Tăng young pressure có thể đẩy vấn đề sang promotion và old generation.
- `Full GC` lặp lại thường là symptom mạnh, không nên xem như chuyện bình thường.
- Average latency đẹp không cứu được SLA nếu tail latency xấu.

## Handbook rule

- Chọn collector theo goal (latency/throughput/memory), không theo ý kiến chung chung.
- Pause thấp hơn thường đổi bằng overhead/memory; biết trade-off trước khi quyết.
- Full GC lặp là symptom; không bỏ qua.
- Average latency không phản ánh tail latency; đo p99/p999 cho service nhạy độ trễ.
- Tăng young pressure có thể đẩy vấn đề sang promotion/old gen; theo dõi cả pipeline.

## Check yourself

- Vì sao “most objects die young” lại ảnh hưởng mạnh tới thiết kế collector?
- `Minor GC`, promotion, và old-generation pressure liên quan với nhau thế nào?
- Collector thiên throughput khác collector thiên low-pause ở mục tiêu gì?
- Vì sao cùng heap size nhưng collector khác có thể cho kết quả rất khác?
- Khi nào thấy `Full GC` là dấu hiệu phải điều tra sâu hơn?

## Exercises

### Exercise 1: Choose Collector Goal

Độ khó: Easy

Đề bài:
Cho ba cờ `prioritizeLowPause`, `prioritizeThroughput`, và `hugeHeap`. Hãy trả về collector goal phù hợp nhất theo quy tắc sau: nếu `prioritizeLowPause` là `true` và `hugeHeap` là `true`, trả về `"low-pause-huge-heap"`; nếu chỉ `prioritizeLowPause` là `true`, trả về `"low-pause"`; nếu `prioritizeThroughput` là `true`, trả về `"throughput"`; ngược lại trả về `"balanced"`.

Ví dụ 1:

Đầu vào:
```text
prioritizeLowPause = true
prioritizeThroughput = false
hugeHeap = true
```

Đầu ra:
```text
"low-pause-huge-heap"
```

Giải thích:
Workload ưu tiên pause thấp trên heap lớn cần goal riêng hơn collector throughput.

Ràng buộc:

- Mỗi input là một boolean
- Trả về đúng một trong bốn nhãn hợp lệ
- Không cần map sang tên collector cụ thể

### Exercise 2: Count Promoted Objects

Độ khó: Medium

Đề bài:
Cho `int[] objectAges` và `int promotionAgeThreshold`. Mỗi phần tử là số lần object đã sống sót qua `minor GC`. Hãy đếm bao nhiêu object sẽ được xem là đủ tuổi để promote nếu `objectAges[i] >= promotionAgeThreshold`.

Ví dụ 1:

Đầu vào:
```text
objectAges = [1, 2, 5, 3, 6]
promotionAgeThreshold = 4
```

Đầu ra:
```text
2
```

Giải thích:
Chỉ object có tuổi `5` và `6` đạt ngưỡng promote.

Ràng buộc:

- `0 <= objectAges.length <= 100000`
- `0 <= objectAges[i] <= 1000`
- `1 <= promotionAgeThreshold <= 1000`

### Exercise 3: Detect Full GC Pressure

Độ khó: Medium

Đề bài:
Cho `int[] oldGenUsagePercent` và `int fullGcThreshold`. Hãy trả về index đầu tiên mà usage của `old generation` lớn hơn hoặc bằng `fullGcThreshold`. Nếu không có, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
oldGenUsagePercent = [55, 68, 79, 93]
fullGcThreshold = 90
```

Đầu ra:
```text
3
```

Giải thích:
Index `3` là thời điểm đầu tiên áp lực `old generation` chạm ngưỡng đáng lo.

Ràng buộc:

- `0 <= oldGenUsagePercent.length <= 100000`
- `0 <= oldGenUsagePercent[i] <= 100`
- Nếu không chạm ngưỡng, trả về `-1`

## Links

- [[../../01_Core/Memory/001-GC]]
- [[005-GC-tuning]]
- [[006-JVM-flags]]
- [Oracle GC Tuning Guide, Available Collectors](https://docs.oracle.com/en/java/javase/21/gctuning/available-collectors.html)
- [Oracle GC Tuning Guide, Garbage First](https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html)
- [Oracle GC Tuning Guide, Z Garbage Collector](https://docs.oracle.com/en/java/javase/21/gctuning/z-garbage-collector.html)
