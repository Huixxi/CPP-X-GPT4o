Sanitizers 是一组动态检测工具，主要用于在运行时发现各种类型的内存错误和未定义行为。它们是由 LLVM 项目提供的，并且与 Clang 和 GCC 编译器紧密集成。这些工具对 C 和 C++ 项目特别有用。

以下是几个常见的 Sanitizers 及其作用：

### AddressSanitizer (ASan)

**作用**：检测内存错误，例如：
- 越界访问（heap buffer overflow, stack buffer overflow）
- 使用已释放的内存（use-after-free）
- 使用未初始化的堆内存
- 重复释放内存（double-free）

**使用方法**：
在编译时启用 ASan：
```bash
clang -fsanitize=address -g -o my_program my_program.c
```
或者使用 GCC：
```bash
gcc -fsanitize=address -g -o my_program my_program.c
```
运行程序后，ASan 会在检测到问题时输出详细的错误报告。

### UndefinedBehaviorSanitizer (UBSan)

**作用**：检测未定义行为，例如：
- 整数溢出
- 访问空指针
- 非法类型转换
- 越界访问数组

**使用方法**：
在编译时启用 UBSan：
```bash
clang -fsanitize=undefined -g -o my_program my_program.c
```
或者使用 GCC：
```bash
gcc -fsanitize=undefined -g -o my_program my_program.c
```
运行程序后，UBSan 会在检测到未定义行为时输出详细的错误报告。

### MemorySanitizer (MSan)

**作用**：检测未初始化内存的使用（use-of-uninitialized-memory）。

**使用方法**：
在编译时启用 MSan：
```bash
clang -fsanitize=memory -g -o my_program my_program.c
```
注意：MSan 仅支持 Clang，不支持 GCC。

### ThreadSanitizer (TSan)

**作用**：检测数据竞争和死锁等并发问题。

**使用方法**：
在编译时启用 TSan：
```bash
clang -fsanitize=thread -g -o my_program my_program.c
```
或者使用 GCC：
```bash
gcc -fsanitize=thread -g -o my_program my_program.c
```
运行程序后，TSan 会在检测到数据竞争或死锁时输出详细的错误报告。

### LeakSanitizer (LSan)

**作用**：检测内存泄漏。

**使用方法**：
LeakSanitizer 通常与 AddressSanitizer 一起使用。在编译时启用 ASan 会自动启用 LSan：
```bash
clang -fsanitize=address -g -o my_program my_program.c
```
或者使用 GCC：
```bash
gcc -fsanitize=address -g -o my_program my_program.c
```
运行程序后，LSan 会在程序退出时输出内存泄漏报告。

### 启用多种 Sanitizers

你可以在编译时同时启用多个 Sanitizers。例如，启用 AddressSanitizer 和 UndefinedBehaviorSanitizer：
```bash
clang -fsanitize=address,undefined -g -o my_program my_program.c
```
或者使用 GCC：
```bash
gcc -fsanitize=address,undefined -g -o my_program my_program.c
```

### 总结

Sanitizers 是一组强大的动态检测工具，可以帮助开发者在运行时发现各种类型的内存错误和未定义行为。它们与 Clang 和 GCC 编译器紧密集成，使得在开发过程中检测和修复这些问题变得更加容易。通过使用 Sanitizers，可以显著提高代码质量和稳定性。
