在现代 C++ 中，`constexpr`、`consteval` 和 `constinit` 是三个重要的关键字，它们用于指定不同类型的常量表达式和初始化行为。这些关键字在 C++11、C++20 和 C++20 的不同版本中引入，旨在提供更强的编译时计算和初始化控制。

### `constexpr`

`constexpr` 是在 C++11 引入的关键字，用于声明可以在编译时求值的常量表达式。它可以用于变量、函数和构造函数。

#### 用法示例：

1. **变量**：
```cpp
constexpr int max_value = 100;
```

2. **函数**：
```cpp
constexpr int square(int x) {
    return x * x;
}
```

3. **构造函数**：
```cpp
struct Point {
    int x;
    int y;
    constexpr Point(int x, int y) : x(x), y(y) {}
};

constexpr Point p(1, 2);
```

### `consteval`

`consteval` 是在 C++20 引入的关键字，用于声明必须在编译时求值的函数。这意味着这样的函数在运行时调用会导致编译错误。

#### 用法示例：

```cpp
consteval int factorial(int n) {
    return (n <= 1) ? 1 : (n * factorial(n - 1));
}

int main() {
    constexpr int fact5 = factorial(5); // 正常编译，因为在编译时求值
    // int runtime_fact = factorial(5); // 编译错误，因为试图在运行时调用 consteval 函数
    return 0;
}
```

### `constinit`

`constinit` 也是在 C++20 引入的关键字，用于指示变量必须立即初始化，并且初始化必须在编译时进行。这对于确保静态或全局变量的初始化顺序非常有用。

#### 用法示例：

```cpp
constinit int global_var = 42;

struct Example {
    static constinit int static_var;
};

constinit int Example::static_var = 10;

int main() {
    // global_var 和 Example::static_var 都已在编译时正确初始化
    return 0;
}
```

### 比较和总结

- **`constexpr`**：用于声明可以在编译时求值的变量、函数和构造函数。它允许在编译时和运行时都可以使用。
- **`consteval`**：用于声明必须在编译时求值的函数。如果尝试在运行时调用 `consteval` 函数，会导致编译错误。
- **`constinit`**：用于确保变量在编译时立即初始化，特别适用于静态或全局变量，以避免初始化顺序问题。

这些关键字的引入使得 C++ 在类型安全、性能优化和初始化控制方面更加灵活和强大。通过正确使用这些关键字，你可以编写出更高效、更健壮的代码。
