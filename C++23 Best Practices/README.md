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
10. 使用运行时分析工具（Sanitizers，详见：[gpt-sanitizers.md](https://github.com/Huixxi/CPP-X-GPT4o/blob/main/C%2B%2B23%20Best%20Practices/gpt-sanitizers.md)）
    * 使用Dr Memory或Valgrind Memcheck
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
与SFINAE相比，概念（concept）将（最终）产生更好的错误消息和更好的编译时间，比SFINAE更易读的代码。cppreference提供了一个[concepts列表](https://en.cppreference.com/w/cpp/concepts)。
> SFINAE（Substitution Failure Is Not An Error）是 C++ 模板编程中的一个重要概念。它的意思是：在模板实例化过程中，如果模板参数替换导致了不匹配或者无效的表达式，那么这种替换失败不会被视为编译错误，而是会导致模板被忽略，然后编译器会尝试其他匹配的模板。
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
Concepts in ‘requires‘ clause.
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
详见：[gpt-constxxx.md](https://github.com/Huixxi/CPP-X-GPT4o/blob/main/C%2B%2B23%20Best%20Practices/gpt-constxxx.md)

### 使用Spaceships`<=>`
```c++
struct MyData {
  int i;
  int j;
  // provide all comparisons
  friend auto operator<=>(const MyData &, const MyData &) = default;
};
```

### 遵循Rule0/3/5
**Rule 0:**  
大多数类不应该有任何的用户定义的拷贝/移动构造函数，赋值操作符和析构函数（copy/move constructor、assignment operator or destructor）   
**Rule 5:**  
如果你提供析构函数是因为`std::unique_ptr`对你的用例没有意义，你必须`=delete`、`=default`或实现其他特殊成员函数。

## 4.代码实现指南（Code Implementation Guidelines）
不要复制粘贴代码。使用[PMD CPD(copy-paste-detectors) tool](https://pmd.github.io/latest/pmd_userdocs_cpd.html)

### 相比`iostream`或`c-formatting`更偏向使用`format`
C++20添加了`<format>`头，它提供了`std::format`方法。
```c++
std::format("Hello {}!\n", "Jason");
```

### 使用`constexpr`
* 使用`constexpr`替换`const`
* 使用`static constexpr`替换`static const`
* 将头文件当中的全局变量编程`inline constexpr`
```c++
// my_library.hpp
// the object `dataset` will exist once in the entire binary
inline constexpr auto dataset = make_data();
```

### `cosnt`一切事情
它迫使我们思考影响性能的对象的初始化和生命周期，并向代码的读者传达意义。
```c++
// Prefer not naming temporaries
ReturnType some_function(int value) {
  // if the types of result and ReturnType differ,
  // and an implicit conversion exists, you get
  // implicit moves.
  // If they are the same, then guaranteed
  // copy/move elision applies from C++17
  return get_value(value + 42);
}
```

### 在多数情况下都选择`auto`
```c++
const auto result = std::count( /* stuff */ );
auto const result = std::count( /* stuff */ );
```

### 永远要初始化`non-const`, `non-auto`的变量
略.

### 使用Ranges和Views为了正确性和可读性
```c++
void print_all_but_first(const std::vector<int> &values) {
  for (const auto val : values | std::views::drop(1)) {
    std::println("{}", val);
  }
}
// 但不要重复使用Views
import std;
int main() {
  std::list<int> values{1,2,3,4,5,6,7,8,9};
  auto drop_2 = []{ return values | std::views::drop(2); };
  std::println("{}", drop_2());
  values.erase(values.begin());
  std::println("{}", drop_2());
}
```

### 相比于循环更倾向于Algorithms
Algorithms能传达意义，并帮助我们应用“const All the Things”规则。在C++20中，我们得到了Ranges，这使得Algorithms更易于使用。
```c++
const auto has_value = std::any_of(container, greater_than(12));
const auto has_value = std::any_of(begin(container), end(container), greater_than(12));
```

### 使用range-for循环
使用clang-tidy的[`modernize-loop-convert`](https://clang.llvm.org/extra/clang-tidy/checks/modernize-loop-convert.html)检查。
```c++
for(const auto &element : container) {
  // eliminates both other problems
}
```
使用偏好：
* `const auto &` for non-mutating loops
* `auto &` for mutating loops
* `auto &&` only when you have to work with weird types like`std::vector<bool>`, or if moving elements out of the container

```c++
std::map<std::string, int> get_map();
using element_type = std::pair<std::string, int>;
for (const element_type & : get_map()) {
}
```

### 避免default语句在switch语句中
```c++
enum class Values {
  val1,
  val2,
  val3 // added a new value
};
[[nodiscard]] constexpr std::string_view get_name(Values value) {
  switch (value) {
  case Values::val1:
    return "val1";
  case Values::val2:
    return "val2";
  }
  // unhandled enum value warning now
  return "unknown";
}
```

### 使用scoped enum
```c++
enum class Choices {
  option1
};

enum class OtherChoices {
  option2
};

int main() {
  int val = option1; // cannot compile, need scope
  int val2 = Choices::option1; // cannot compile, wrong type
  Choices val = Choices::option1; // compiles
  val = OtherChoices::option2; // cannot compile, wrong type
}
```

### 去模板化你的通用代码
尽可能将通用代码移出模板，去模板化将缩短编译时间并减少二进制文件大小。
```c++
constexpr auto some_function() { /* do things*/ }
template<typename T>
constexpr void do_things() {
  auto value = some_function();
}
```
了解Bloaty McBloatface and `-ftime-trace`。详见：[gpt-shrinkbinary.md](https://github.com/Huixxi/CPP-X-GPT4o/blob/main/C%2B%2B23%20Best%20Practices/gpt-shrinkbinary.md)

### 使用Lippincott Functions
```c++
void handle_exception() {
  try {
    throw; // re-throw exception already in flight
  } catch (const std::runtime_error &) {
  } catch (const std::exception &) { }
}

void use_thing() {
  try {
    do_thing();
  } catch (...) {
    handle_exception();
  }
}

void use_other_thing() {
  try {
    do_other_thing();
  } catch (...) {
    handle_exception();
  }
}
```

### 不要再使用`new`
使用clang-tidy自动将`new`语句转成`make_unique<>``make_shared<>`调用
* 使用`std::unique<>()`
* 极少数情况使用`std::shared<>()`

### 避免使用std::bind和std::function
略.

### 对非传统类型变量不要使用列表初始化方法（initializer_list）
略.

### 在C++20标准中使用Designated Initializers
详见：[gpt-designated_initializers.md](https://github.com/Huixxi/CPP-X-GPT4o/blob/main/C%2B%2B23%20Best%20Practices/gpt-designated_initializers.md)

## 3.资料（Resources）
* https://www.alcf.anl.gov/sites/default/files/2020-04/BestPractices-101-miniQMC-CaseStudy.pdf

## 4.书籍（Books）
* 《[Functional Programming in C++](https://www.manning.com/books/functional-programming-in-c-plus-plus?a_aid=FPinCXX&a_bid=441f12cc)》
