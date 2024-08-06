# 学习笔记 《[C++ Best Practices by Jason-Turner](https://www.scribd.com/document/703404100/C-Best-Practices-by-Jason-Turner)》
## 1.概要（Overview）
1. 减少常见错误 Reducing common mistakes
2. 更快速的定位错误 Finding errors quickly
3. 不牺牲代码性能（用C++编程一定是出于性能考虑） Without sacrificing (and often improving) performance

## 2.最佳实践（Best Practices）
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



### 资源获取即初始化（RAII）

### 循环语句（for-loop）

### 


## 3.资料（Resources）
* https://www.alcf.anl.gov/sites/default/files/2020-04/BestPractices-101-miniQMC-CaseStudy.pdf

## 4.书籍（Books）
* 《[Functional Programming in C++](https://www.manning.com/books/functional-programming-in-c-plus-plus?a_aid=FPinCXX&a_bid=441f12cc)》
