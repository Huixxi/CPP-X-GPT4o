# 学习笔记 《[C++ Best Practices by Jason-Turner](https://www.scribd.com/document/703404100/C-Best-Practices-by-Jason-Turner)》
## 1.概要（Overview）
1. 减少常见错误 Reducing common mistakes
2. 更快速的定位错误 Finding errors quickly
3. 不牺牲代码性能（用C++编程一定是出于性能考虑） Without sacrificing (and often improving) performance

## 2.使用工具（Use The Tools）
1. 假如一个解决方案庞大又复杂，请停下来。 If the solution seems large or complex, stop.
2. 花在写代码上的时间越少，用来思考的时间就越多。 The less time I spend programming, and the more time I spend thinking.
3. C++不止是一个面向对象语言。 C++ is not an object-oriented language(OOL).
4. 学习其他编程语言。 Learn Another Language.
5. C++20和C++23标准库中的一些重要改变：
   * 极大的扩充了date/time库。 greatly expanded date/time library (with calendar).
   * 很多并发和线程支持的改变。 many changes to concurrency and threading support.
   * `std::format`
   * `ranges`
   * `concepts`
   * `<stacktrace>` 和其他诊断工具。 other diagnostics helpers.
6. 不要引发未定义行为。Don’t Invoke Undefind Behavior(UB).
   * `-Wtautological-undefined-compare`
   * 不要检查`this == nullptr`
   * 不要检查一个`&thing == nullptr`
7. 使用自动化测试工具（Automated Tests Tools）
   * [Catch2](https://github.com/catchorg/Catch2)
   * [Doctest](https://github.com/doctest/doctest)
   * [Googletest](https://github.com/google/googletest)
8. 使用编译器警告（Compiler Warnings）
   * 强烈考虑`-Wpedantic`。 Strongly consider `-Wpedantic` (GCC/Clang).
   * MSVC编译使用/W1～/W4
9. 使用静态分析工具（Static Analysis）
   * cppcheck：一个轻量级的 C/C++ 静态代码分析工具，专注于发现错误和漏洞。
   * clang-tidy：一个强大的 C++ 静态代码分析工具，提供了丰富的检查规则和自动修复功能。
   * CodeQL：GitHub 提供的代码分析引擎，通过查询语言来发现代码中的安全和性能问题。
10. 使用运行时分析工具（Sanitizers）
    * [使用手册](https://github.com/Huixxi/CPP-X-GPT4o/blob/main/C%2B%2B23%20Best%20Practices/gpt-sanitizers.md)
    * 使用Dr Memory或Valgrind
    * Address (ASan)
    * Undefined Behavior (UBSan) (More on Undefined Behavior later)
    * Thread (TSan)
    * DataFlow (use for code analysis, not finding bugs)
    * Lib Fuzzer (addressed in a later chapter)
11. 使用多重编译器 Multiple Compilers
    * 在GCC和Clang二者间切换编译
12. 使用模糊测试和变异测试 Fuzz test and Mutation test
    * Fuzz testers generate strings of random data of various lengths.
      * [LLVM’s libFuzzer](https://www.llvm.org/docs/LibFuzzer.html)
      * [Google FuzzTest](https://github.com/google/fuzztest)
      * [DeepState](https://github.com/trailofbits/deepstate)
      * [oss-fuzz](https://github.com/google/oss-fuzz)
    * Mutation testing works by modifying conditionals and constants in the codebeing tested.
      * [Mutate++](https://github.com/nlohmann/mutate_cpp)
13. 使用代码构建生成器 Build Generators
    * 写好自己的编译脚本 build scripts
    * [CMake](https://cmake.org)
    * [Meson](https://mesonbuild.com/)
    * [Bazel](https://bazel.build/)
    * [Others](https://github.com/lefticus/cppbestpractices/blob/master/02-Use_the_Tools_Available.md)
14. 使用打包管理器 Package Managers
    * [Vcpkg](https://github.com/Microsoft/vcpkg)
    * [Conan](https://conan.io/)
    * [CPM](https://github.com/cpm-cmake/CPM.cmake)

## 3.API和代码设计指南（API and CodeDesign Guidelines）
1. 让你的接口很难被用错 Make your interfaces hardto use wrong.
2. 考虑在错误使用了API的时候会引发未定义的行为 Consider If Using the APIWrong Invokes UndefinedBehavior
3. 对全局状态保持敬畏 Be Afraid of Global State
   * 任何`non-const static`值或`std::shared_ptr<>`都可能是全局状态。永远不知道谁会更新该值，也不知道这样做是否是线程安全的。
   * 尽量避免使用单例模式（singletons）

### 使用强类型（Strongly Typed API）
```c++
// Poorly defined constructor.
Rectangle(int, int, int, int);
// vs
struct Position {
  int x;
  int y;
};

struct Size {
  int width;
  int height;
};

struct Rectangle {
  Position position;
  Size size;
};

Rectangle(Position, Size);
```

### 避免使用Boolean类型参数（Avoid Boolean Arguments）
```c++
struct Widget {
  // this constructor is easy to use wrong, we
  // can easily transpose the parameters
  Widget(bool visible, bool resizable);
}

struct Widget {
  enum struct Visible { True, False };
  enum struct Resizable { True, False };
  // still possible to use this wrong, but MUCH harder
  Widget(Visible visible, Resizable resizable);
}
```

### 考虑用 =delete 去规避可能造成问题的类型转换
```c++
// Deleting a problematic accidental promotion from ‘float‘ to ‘double‘
double high_precision_thing(double);
double high_precision_thing(float) = delete;
```

### 自由的使用`[[nodiscard]]`关键字
[[nodiscard]]是一个C++属性，它告诉编译器在忽略返回值时发出警告。它可用于以下功能：
```c++
[[nodiscard]] int get_value();
int main() {
  // warning, [[nodiscard]] value ignored
  get_value();
}
```
```c++
// ‘[[nodiscard]]‘ on types
struct [[nodiscard]] ErrorCode{};
ErrorCode get_value();
int main() {
  // warning, [[nodiscard]] value ignored
  get_value();
}
```
```c++
// C++20’s‘[[nodiscard]]‘with description
[[nodiscard("Ignoring this result leaks resources")]]
```

### C++23中引入的 modules 的实现和使用
略.

### 栈优先于堆（Prefer Stack Over Heap）
`std::string`本身可能会在内部分配，并使用heap，应尽量避免不必要的堆内存开销。通过`new`或者`make_unique`或`make_shared`创建的对象会分配在堆上。
使用heap profiling工具查找代码中堆内存滥用的情况。
* [heaptrack - a heap memory profiler for Linux](https://github.com/KDE/heaptrack)
* [Valgrind Massif: a heap profiler](https://valgrind.org/info/tools.html#massif)

### 不要返回裸指针
能否通过返回一个值（value）、引用（reference）或`std::unique_ptr`作为替换。

### `std:array<>`优先于`std::vector<>`
略.

### Be Aware of Custom Allocation And Polymorphic Memory Resources(PMR)
略.

### 使用Concepts
cppreference提供了一个[concepts列表](https://en.cppreference.com/w/cpp/concepts)。
```c++
import std;
template <typename Numerator, typename Denominator>
[[nodiscard]] constexpr auto divide(Numerator numerator, Denominator denominator) {
  if constexpr (std::is_integral_v<Numerator> &&
                std::is_integral_v<Denominator>) {
    // is integral division
    if (denominator == 0) {
      throw std::runtime_error("divide by 0!");
    }
  }
  return numerator / denominator;
}
```
Conceptsin ‘requires‘ clause.
```c++
import std;
// overload resolution will pick the most specific version
template <typename Numerator, typename Denominator>
[[nodiscard]] constexpr auto divide(Numerator numerator, Denominator denominator) requires(
    std::is_integral_v<Numerator>
    && std::is_integral_v<Denominator>) {
  // is integral division
  if (denominator == 0) {
    throw std::runtime_error("divide by 0!");
  }
  return numerator / denominator;
}

template <typename Numerator, typename Denominator>
[[nodiscard]] constexpr auto divide(Numerator numerator, Denominator denominator) {
  return numerator / denominator;
}
```
Terse concepts requirement syntax, C++20 even has an “autoconcept,” which is an implicit template function.
```c++20
import std;
[[nodiscard]] constexpr auto divide(std::integral auto numerator, std::integral auto denominator) {
  // is integer division
  if (denominator == 0) {
    throw std::runtime_error("divide by 0!");
  }
  return numerator / denominator;
}

[[nodiscard]] constexpr auto divide(auto numerator, auto denominator) {
  // is floating point division
  return numerator / denominator;
}
```
代码中，`std::integral` 是一个概念（`concept`），用于限制模板参数的类型。C++20 引入了概念（`concepts`），这是模板编程的一种新特性，旨在提供更好的类型约束和更友好的编译器错误信息。  
`std::integral` 是标准库中定义的一个概念，用于检查一个类型是否是整型。它是 `std::concepts` 命名空间的一部分：
```c++
namespace std::concepts {
    template<class T>
    concept integral = std::is_integral_v<T>;
}
```
在代码中，`std::integral auto` 限制了 `divide` 函数的参数 `numerator` 和 `denominator` 必须是整型类型。这意味着你只能用整型类型（例如 `int, long, short, unsigned int` 等）来调用这个函数。

### 理解`consteval`和`constinit`
详见：


### 资源获取即初始化（RAII）

### 循环语句（for-loop）

### 


## 3.资料（Resources）
* https://www.alcf.anl.gov/sites/default/files/2020-04/BestPractices-101-miniQMC-CaseStudy.pdf

## 4.书籍（Books）
* 《[Functional Programming in C++](https://www.manning.com/books/functional-programming-in-c-plus-plus?a_aid=FPinCXX&a_bid=441f12cc)》
