# Bytecode and Class File

## What is it

Java source code không chạy trực tiếp trên JVM. `javac` biên dịch source thành `bytecode`, rồi lưu trong `.class` file có format chuẩn được mô tả trong JVMS.

Điều JVM đọc đầu tiên không phải Java syntax, mà là `class file structure`: magic number, version, constant pool, access flags, field info, method info, attributes, rồi instruction stream của từng method.

Vì vậy nếu muốn hiểu runtime ở mức thấp hơn source, bạn phải nhìn sang `bytecode` và `.class` file.

## How I used to misunderstand it

Mình từng nghĩ `.class` file chỉ là “source đã máy hóa”. Không đủ chính xác.

Thực tế `.class` là một binary format có grammar riêng. Hai đoạn source trông gần giống nhau có thể sinh bytecode khác đáng kể. Và tool như proxy library, AOP enhancer, Byte Buddy, ASM, hay compiler khác có thể sinh bytecode trực tiếp mà không cần đi qua chính source Java của bạn.

## How it actually works

Một `.class` file thường được hình dung tốt bằng sơ đồ này:

```text
.class file
  |
  +-- magic + version
  +-- constant pool
  +-- access flags
  +-- this_class / super_class
  +-- interfaces
  +-- fields
  +-- methods
  +-- attributes
```

Trong đó `constant pool` là trung tâm. Nó không chỉ chứa string literal, mà còn chứa symbolic references tới class, field, method, name, descriptor, và nhiều metadata khác.

| Thành phần | Giữ gì | Vì sao quan trọng |
| --- | --- | --- |
| `magic` và `version` | Dấu hiệu nhận diện file và version class file | Giải thích `UnsupportedClassVersionError` |
| `constant pool` | Symbolic references và constants | Bytecode trỏ vào đây thay vì nhúng mọi thứ trực tiếp |
| `methods` | Bytecode của từng method | JVM execute hoặc JIT compile từ đây |
| `attributes` | Metadata bổ sung như line number, stack map frames | Quan trọng cho verify, debug, reflection, tooling |

Ở mức instruction, JVM làm việc với opcode như `aload_0`, `getfield`, `invokevirtual`, `ifnonnull`, `ireturn`.

Một đối chiếu rất hữu ích khi học note này:

| Ở source Java | Ở bytecode / class file |
| --- | --- |
| Tên biến local có thể nhìn rất rõ | Thường biến thành slot index hoặc metadata debug |
| Generic type nhìn như có thật | Nhiều chỗ bị `type erasure` trong runtime view |
| Method call nhìn đơn giản | Thành `invokevirtual`, `invokestatic`, `invokeinterface`, hoặc `invokespecial` |
| `if`, `for`, `while` | Thành jump instruction và labels |

JVM còn phải `verify` bytecode trước khi cho chạy. Đây là lý do bytecode sai format, stack usage không hợp lệ, hoặc metadata không khớp có thể fail trước cả khi method thật sự chạy.

## Code example

```java
public class BytecodeDemo {
    public int increment(int value) {
        return value + 1;
    }
}
```

Nếu disassemble bằng `javap -c -v BytecodeDemo`, bạn sẽ không còn thấy Java syntax ở mức thực thi. Bạn sẽ thấy version, constant pool, descriptor của method, rồi instruction stream kiểu:

```text
iload_1
iconst_1
iadd
ireturn
```

Ý chính cần nhớ là: JVM reasoning ở tầng thấp hơn source, nên khi debug version mismatch, proxy generation, verify error, hoặc performance, nhìn source thôi chưa đủ.

## When to use / when NOT to use

Hãy dùng mental model này khi:

- đọc output từ `javap`,
- debug lỗi liên quan tới class file version, verification, instrumentation,
- học proxy generation, agent, AOT, framework enhancement.

Không cần đi sâu từng opcode nếu mục tiêu trước mắt chỉ là viết business logic CRUD. Khi đó hiểu layout lớn và vài failure mode quan trọng là đủ.

## How this connects to real Java projects

Spring và ecosystem xung quanh thường tạo proxy hoặc enhance class bằng bytecode, không phải bằng cách sửa source. CGLIB, Byte Buddy, Hibernate enhancement, và nhiều AOP path đều hoạt động ở tầng này.

Vì vậy khi app fail vì module access, incompatible class version, hoặc generated proxy lạ, đọc được `class file` model sẽ giúp bạn hiểu log nhanh hơn thay vì đổ lỗi mơ hồ cho framework.

## Gotchas

- Compile bằng JDK mới rồi chạy bằng runtime cũ sẽ gây `UnsupportedClassVersionError`.
- `constant pool` không chỉ chứa constants kiểu string hay number.
- Bytecode hợp lệ không nhất thiết phải đến từ `javac`.
- Source code nhìn “tương đương” chưa chắc có bytecode tương đương.

## Handbook rule

- Compile JDK mới chạy runtime cũ gây `UnsupportedClassVersionError`; thống nhất `--release`.
- Bytecode hợp lệ không nhất thiết đến từ `javac`; framework có thể generate.
- Source “tương đương” không có nghĩa bytecode tương đương; xác nhận bằng `javap` khi cần.
- Constant pool chứa nhiều loại entry; đừng coi nó chỉ là string/number.
- Đi sâu opcode chỉ khi debug instrumentation/AOT/agent, không phải để viết CRUD.

## Check yourself

- Vì sao JVM không cần Java source để chạy chương trình?
- `constant pool` khác gì với cách hiểu hẹp là “bảng constant”?
- Vì sao compile bằng JDK mới hơn có thể fail trên runtime cũ hơn?
- `invokevirtual` và `invokestatic` gợi ý điều gì về call site?
- Khi nào nhìn `javap` hữu ích hơn nhìn source thuần?

## Exercises

### Exercise 1: Classify Bytecode Opcode

Độ khó: Easy

Đề bài:
Cho một opcode `String opcode`. Hãy phân loại nó thành một trong các nhóm sau: `"load"`, `"store"`, `"invoke"`, `"control"`, hoặc `"other"`. Quy ước: opcode bắt đầu bằng `"iload"`, `"aload"`, `"lload"`, `"fload"`, `"dload"` là `load`; bắt đầu bằng `"istore"`, `"astore"`, `"lstore"`, `"fstore"`, `"dstore"` là `store`; bắt đầu bằng `"invoke"` là `invoke`; bắt đầu bằng `"if"`, hoặc bằng `"goto"`, `"tableswitch"`, `"lookupswitch"` là `control`.

Ví dụ 1:

Đầu vào:
```text
opcode = "invokevirtual"
```

Đầu ra:
```text
"invoke"
```

Giải thích:
`invokevirtual` là instruction gọi method.

Ràng buộc:

- `opcode` là chuỗi không rỗng
- So sánh phân biệt hoa thường
- Nếu không thuộc nhóm nào ở trên, trả về `"other"`

### Exercise 2: Find Unsupported Class File Version

Độ khó: Medium

Đề bài:
Cho `int[] classFileMajors` và `int maxSupportedMajor`. Hãy trả về index đầu tiên của `class file` có major version lớn hơn runtime hỗ trợ. Nếu tất cả đều hợp lệ, trả về `-1`.

Ví dụ 1:

Đầu vào:
```text
classFileMajors = [52, 55, 61]
maxSupportedMajor = 55
```

Đầu ra:
```text
2
```

Giải thích:
File ở index `2` cần runtime mới hơn.

Ràng buộc:

- `0 <= classFileMajors.length <= 100000`
- `45 <= classFileMajors[i] <= 100`
- Nếu không có version mismatch, trả về `-1`

### Exercise 3: Count Invoke Instructions

Độ khó: Medium

Đề bài:
Cho `String[] opcodes`. Hãy đếm số instruction là method invocation, tức là opcode bắt đầu bằng `"invoke"`.

Ví dụ 1:

Đầu vào:
```text
opcodes = ["aload_0", "invokevirtual", "iconst_1", "invokestatic"]
```

Đầu ra:
```text
2
```

Giải thích:
Có hai instruction gọi method là `invokevirtual` và `invokestatic`.

Ràng buộc:

- `0 <= opcodes.length <= 100000`
- Mỗi opcode là `String` không rỗng
- Chỉ cần đếm theo prefix, không cần parse descriptor

## Links

- [[001-classloader-mechanism]]
- [[003-JIT-compiler]]
- [[../../01_Core/Generics/003-type-erasure]]
- [JVMS Chapter 4, The `class` File Format](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-4.html)
- [JVMS Chapter 6, The Java Virtual Machine Instruction Set](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-6.html)
- [`javap` Tool Guide](https://docs.oracle.com/en/java/javase/21/docs/specs/man/javap.html)
