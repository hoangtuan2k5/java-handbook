# Off-Heap Memory

## What is it

`Off-heap memory` là memory được dùng ngoài Java heap, ví dụ:

- direct `ByteBuffer`
- memory-mapped file
- native library allocation
- một số JVM runtime structure ngoài heap thông thường

Mental model:

- không nằm trong heap không có nghĩa là miễn phí
- nó vẫn là RAM thật
- lifecycle của nó thường khó nhìn hơn heap object

## How I used to misunderstand it

Hiểu nhầm phổ biến nhất là nghĩ Java memory issue chỉ cần nhìn heap. Thực tế process Java có thể hết memory dù heap dump không lớn nếu phần non-heap hoặc native tăng mạnh.

Ví dụ, app có thể tốn memory ở:

- direct buffers
- native allocations từ library
- memory-mapped files
- thread stacks
- metaspace
- tổng memory limit của container

## How it actually works

Ví dụ quen thuộc nhất là direct `ByteBuffer`.

Khi gọi `ByteBuffer.allocateDirect(...)`, JVM tạo một wrapper object trên heap, nhưng phần byte storage chính nằm ngoài heap. Điều này hữu ích trong một số bài toán I/O hoặc native interop vì có thể giảm copy trung gian.

### Heap vs off-heap rất ngắn gọn

| Câu hỏi | Heap | Off-heap |
|---|---|---|
| Object Java bình thường ở đây? | Có | Không phải mặc định |
| Heap dump có thấy đầy đủ không? | Thường có | Thường không đầy đủ |
| GC có trực tiếp quản lý byte storage như object thường không? | Có, với object heap bình thường | Không theo cách trực tiếp như heap object; thường đi qua wrapper và cleanup mechanism |
| Có tính vào process/container memory limit không? | Có | Có |

### Tiny mental diagram

```text
heap object wrapper  ---->  native/off-heap bytes
     ByteBuffer                 [ .... data .... ]

GC thấy wrapper trên heap,
nhưng phần data chính nằm ngoài heap.
```

Điểm quan trọng là memory off-heap thường được giải phóng gắn với lifecycle của wrapper và cơ chế cleanup nội bộ, nhưng timing không giống việc thu hồi object heap bình thường. Vì vậy bạn có thể thấy heap chưa cao lắm mà process memory vẫn phình to.

## Code example

```java
import java.nio.ByteBuffer;

ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
buffer.put((byte) 65);
buffer.flip();

System.out.println(buffer.get()); // 65
```

Đoạn code trông rất giống dùng buffer bình thường, nhưng phần memory phía sau `buffer` lại nằm ngoài Java heap.

## When to use / when NOT to use

Dùng off-heap khi framework, thư viện, hoặc trường hợp sử dụng I/O thực sự có lý do rõ ràng.

Ví dụ hợp lý:

- networking stack như Netty
- zero-copy hoặc reduced-copy I/O pattern
- native interop

Không dùng direct buffer chỉ vì nghe nói “nhanh hơn”. Với nhiều ứng dụng business bình thường, heap object đơn giản dễ đo, dễ debug, và đủ tốt hơn.

Nếu chạy trong container, luôn tính tổng memory, không chỉ `-Xmx`.

## How this connects to Spring

Spring app có thể dùng off-heap gián tiếp qua Netty, WebFlux, driver database, TLS stack, compression library, hoặc cache/native dependency.

Nếu container bị OOM mà heap chưa đầy, hướng điều tra đúng thường là:

- direct memory usage
- metaspace
- thread count và thread stack
- native allocations
- tổng process RSS

## Gotchas

- Heap dump không cho bạn toàn bộ bức tranh off-heap.
- Direct buffer allocation quá nhiều có thể gây `OutOfMemoryError` ngoài heap.
- Container memory limit tính cả heap lẫn non-heap/native memory.
- Tăng `-Xmx` đôi khi còn làm tổng memory tệ hơn nếu bạn quên phần off-heap.

## Check yourself

- Vì sao heap nhỏ không đồng nghĩa process memory nhỏ?
- `ByteBuffer.allocateDirect()` khác gì với object Java bình thường về nơi chứa data?
- Vì sao heap dump có thể chưa đủ để chẩn đoán memory issue kiểu này?
- Trong container, vì sao chỉ nhìn `-Xmx` là chưa đủ?
- Khi nào direct buffer có lý do tồn tại thật, khi nào chỉ là overkill?

## Exercises

### Bài 1: Estimate Process Memory
Độ khó: Dễ

Đề bài:
Cho mức dùng heap và off-heap tính bằng MB, trả về tổng process memory theo MB.

Ví dụ 1:
Đầu vào:
```text
heapMb = 512, offHeapMb = 128
```

Đầu ra:
```text
640
```

Giải thích:
Process memory bao gồm cả heap lẫn off-heap usage.

Ràng buộc:
- heapMb >= 0
- offHeapMb >= 0
- Kết quả nằm trong int range

### Bài 2: Detect Direct Buffer Pressure
Độ khó: Trung bình

Đề bài:
Cho direct buffer usage và direct memory limit theo MB, trả về `true` nếu usage lớn hơn limit.

Ví dụ 1:
Đầu vào:
```text
usedMb = 300, limitMb = 256
```

Đầu ra:
```text
true
```

Giải thích:
Direct memory usage đã vượt quá limit được cấu hình.

Ràng buộc:
- usedMb >= 0
- limitMb >= 0
- Trả về một boolean

### Bài 3: Classify Memory Area
Độ khó: Trung bình

Đề bài:
Cho một memory area label, trả về `"heap"` cho `"object"` và `"off-heap"` cho `"direct-buffer"`, `"mmap"`, hoặc `"native"`.

Ví dụ 1:
Đầu vào:
```text
area = "direct-buffer"
```

Đầu ra:
```text
"off-heap"
```

Giải thích:
Direct buffer dùng memory nằm ngoài Java heap.

Ràng buộc:
- area là một trong các label đã được liệt kê
- Chỉ trả về `"heap"` hoặc `"off-heap"`
- So khớp có phân biệt chữ hoa/thường

## Links

- [[001-GC]]
- [[002-Memory-leak]]
- [[005-strong-soft-weak-phantom-reference]]
- [Java SE 21, `ByteBuffer` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/ByteBuffer.html)
- [Java SE 21, `Buffer` Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/Buffer.html)
- [Java SE Troubleshooting Guide, Native Memory Tracking](https://docs.oracle.com/en/java/javase/21/troubleshoot/diagnostic-tools.html)

