# JIT Compiler

## What is it

`JIT compiler` là thành phần của JVM biên dịch `bytecode` thành machine code trong lúc chương trình đang chạy. Ý tưởng cốt lõi là: không phải mọi đoạn code đều đáng tối ưu ngay từ đầu, nên JVM quan sát hành vi runtime rồi mới đầu tư compile sâu cho phần `hot code`.

Đây là lý do Java vừa giữ được portability của `bytecode`, vừa có thể đạt performance rất gần native code ở steady state.

## How I used to misunderstand it

Mình từng nghĩ JVM hoặc interpret hết, hoặc compile hết.

Thực tế HotSpot đi theo hướng phân tầng hơn: chạy sớm, thu profile, rồi tối ưu dần khi thấy code đủ nóng. Vì vậy request đầu tiên, benchmark ngắn, hoặc test chạy một vòng thường không đại diện cho throughput dài hạn.

## How it actually works

Một flow mental model vừa đủ để debug performance là:

```text
Bytecode starts running
      |
      v
JVM gathers counters and profiles
      |
      v
Hot methods / loops become compile candidates
      |
      v
JIT compiles optimized machine code
      |
      v
If assumptions break, JVM may deoptimize
```

Điểm hay của JIT là nó thấy dữ liệu runtime thật. Nó có thể tối ưu dựa trên call frequency, branch profile, receiver type profile, và escape behavior của object.

| Tối ưu thường gặp | Tác dụng | Điều kiện hay gặp |
| --- | --- | --- |
| `Inlining` | Bỏ chi phí call, mở đường cho tối ưu tiếp theo | Method nhỏ, call site ổn định |
| Branch optimization | Tối ưu nhánh chạy nhiều | Profile đủ ổn định |
| Type-based optimization | Tối ưu virtual dispatch | Call site không quá đa hình |
| Escape analysis | Có thể loại hoặc giảm allocation | Object không escape khỏi scope phù hợp |

Nhưng JIT không phải phép màu miễn phí. Nó dựa trên assumption. Nếu traffic đổi shape, call site thành `megamorphic`, hoặc branch trước đây hiếm nay trở nên phổ biến, JVM có thể `deoptimize` để quay về con đường ít tối ưu hơn nhưng đúng.

Contrast quan trọng nhất khi benchmark:

| Giai đoạn | Điều dễ xảy ra | Kết luận sai thường gặp |
| --- | --- | --- |
| Startup / request đầu | Class loading, interpreter, proxy init, cache cold | “Java chậm” |
| Warmup | Profile đang tích lũy, compile đang diễn ra | “Kết quả benchmark không ổn định” nhưng không biết vì sao |
| Steady state | JIT đã tối ưu nhiều hơn | Đây mới gần workload dài hạn hơn |

## Code example

```java
long sum = 0;

for (int i = 0; i < 1_000_000; i++) {
    sum += i;
}
```

Một loop như thế này nếu được gọi lặp đi lặp lại rất dễ trở thành `hot code`.

Điều cần nhớ không phải là “loop nào cũng sẽ siêu nhanh”, mà là JVM cần thời gian quan sát trước khi quyết định đoạn nào đáng tối ưu. Vì vậy cùng một đoạn code có thể cho latency rất khác ở vòng chạy đầu và sau warmup.

## When to use / when NOT to use

Hãy dùng mental model này khi:

- benchmark Java,
- so sánh startup latency với steady-state throughput,
- tối ưu CPU hotspot trong service chạy lâu,
- đọc output từ JMH, JFR, hoặc compiler logging.

Không nên kết luận performance từ một lần chạy ngắn, từ debug mode, hoặc từ microbenchmark viết tay không có warmup tử tế.

## How this connects to Spring

Spring Boot service thường chạy lâu, nên phần lớn giá trị performance nằm ở steady state. Request đầu sau deploy có thể chậm vì class loading, bean init, proxy creation, cache cold, và JIT chưa tối ưu đủ.

Điều này đặc biệt quan trọng khi so controller, service layer, JSON mapping, hoặc AOP overhead. Nếu benchmark bỏ qua warmup, bạn rất dễ quy sai nguyên nhân cho framework abstraction.

## Gotchas

- Benchmark không warmup gần như luôn dễ sai.
- `Inlining` có thể làm stack trace hoặc profiler output khó đọc hơn.
- Profile test nhỏ có thể khác xa production traffic thật.
- `Deoptimization` làm performance thay đổi theo traffic pattern, không chỉ theo source code.

## Check yourself

- Vì sao request đầu hoặc vòng benchmark đầu thường không đại diện cho hiệu năng dài hạn?
- JIT cần “thấy” những tín hiệu nào trước khi compile sâu hơn?
- `Inlining` giúp gì ngoài chuyện bỏ chi phí gọi method?
- Khi nào JVM phải `deoptimize`?
- Vì sao benchmark trong debug mode dễ gây hiểu nhầm?

## Exercises

### Exercise 1: Find First Hot Method

Độ khó: Easy

Đề bài:
Cho `String[] methodNames`, `int[] invocationCounts`, và `int hotThreshold`. Hãy trả về tên method đầu tiên có số lần gọi lớn hơn hoặc bằng `hotThreshold`. Nếu không có method nào đủ nóng, trả về chuỗi rỗng `""`.

Ví dụ 1:

Đầu vào:
```text
methodNames = ["parse", "render", "save"]
invocationCounts = [120, 9000, 40]
hotThreshold = 1000
```

Đầu ra:
```text
"render"
```

Giải thích:
`render` là method đầu tiên vượt ngưỡng hot.

Ràng buộc:

- `0 <= methodNames.length <= 100000`
- Hai mảng phải có cùng độ dài
- Nếu không có kết quả, trả về `""`

### Exercise 2: Count Warmup Outliers

Độ khó: Medium

Đề bài:
Cho `int[] firstRunMicros`, `int[] steadyStateMicros`, và `int multiplier`. Một sample được xem là warmup outlier nếu `firstRunMicros[i] >= steadyStateMicros[i] * multiplier`. Hãy đếm số sample như vậy.

Ví dụ 1:

Đầu vào:
```text
firstRunMicros = [900, 300, 1000]
steadyStateMicros = [300, 200, 250]
multiplier = 3
```

Đầu ra:
```text
2
```

Giải thích:
Sample `0` và `2` chậm hơn hoặc bằng theo ngưỡng warmup đã cho.

Ràng buộc:

- `0 <= firstRunMicros.length <= 100000`
- Hai mảng phải có cùng độ dài
- `1 <= multiplier <= 100`

### Exercise 3: Classify Deoptimization Risk

Độ khó: Medium

Đề bài:
Cho ba cờ `unstableBranchProfile`, `megamorphicCallSite`, và `heavyReflection`. Hãy trả về `"high"` nếu có ít nhất hai cờ là `true`, `"medium"` nếu có đúng một cờ là `true`, ngược lại trả về `"low"`.

Ví dụ 1:

Đầu vào:
```text
unstableBranchProfile = true
megamorphicCallSite = false
heavyReflection = true
```

Đầu ra:
```text
"high"
```

Giải thích:
Có hai tín hiệu cho thấy optimized assumptions dễ bị phá vỡ.

Ràng buộc:

- Mỗi input là một boolean
- Chỉ trả về `"low"`, `"medium"`, hoặc `"high"`
- Không cần mô phỏng JIT thật

## Links

- [[002-bytecode-and-class-file]]
- [[006-JVM-flags]]
- [Oracle, Java HotSpot VM Performance Enhancements](https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html)
- [HotSpot Glossary](https://openjdk.org/groups/hotspot/docs/HotSpotGlossary.html)
- [JMH Project](https://openjdk.org/projects/code-tools/jmh/)
