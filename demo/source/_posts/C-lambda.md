---
title: C++ lambda
date: 2026-02-18 11:16:18
tags:
---
以下是 Markdown 格式的 C++11 技术博客：

```markdown
# C++11：现代 C++ 的里程碑式变革

> "C++11 感觉像是一门全新的语言。" —— Bjarne Stroustrup

C++11 于 2011 年发布，是 C++ 标准自 1998 年以来最重大的更新。它不仅修复了诸多历史遗留问题，更引入了大量现代编程范式，让 C++ 在保证性能的同时，大幅提升了开发效率和代码安全性。本文将深入解析 C++11 的核心特性，帮助你掌握现代 C++ 编程的精髓。

---

## 一、自动类型推导：`auto` 与 `decltype`

### 1.1 `auto` 关键字

C++11 引入了 `auto` 关键字，让编译器自动推导变量类型，告别冗长的类型声明。

```cpp
// 传统写法
std::unordered_map<std::string, int>::iterator it = myMap.begin();

// C++11 写法
auto it = myMap.begin();
```

**最佳实践**：
- 使用 `auto` 避免类型不匹配问题。例如，在范围 for 循环中，`std::pair<const std::string, int>` 与 `std::pair<std::string, int>` 的细微差别可能导致不必要的拷贝
- 始终初始化 `auto` 变量，否则编译器无法推导类型

```cpp
auto x = 0;        // ✓ 正确：int
auto y;            // ✗ 错误：无法推导
auto z = {1, 2};   // ✓ std::initializer_list<int>
```

### 1.2 列表初始化（Uniform Initialization）

C++11 统一了初始化语法，使用花括号 `{}` 进行初始化：

```cpp
std::vector<int> vec{1, 2, 3};
int arr[]{1, 2, 3};
double pi{3.14159};

// 防止窄化转换（narrowing conversion）
int x{3.14};  // ✗ 编译错误：double 到 int 的转换会丢失数据
```

花括号初始化还能避免"最棘手的解析"（Most Vexing Parse）问题：

```cpp
Widget w1(10);   // 调用构造函数，传入参数 10
Widget w2();     // ✗ 声明了一个返回 Widget 的函数！
Widget w3{};     // ✓ 调用默认构造函数
```

---

## 二、Lambda 表达式：匿名函数的优雅

Lambda 表达式是 C++11 最激动人心的特性之一，它允许在代码中内联定义匿名函数。

### 2.1 基本语法

```cpp
[capture](parameters) -> return_type { body }
```

**示例**：

```cpp
std::vector<int> nums{1, 2, 3, 4, 5};

// 使用 lambda 进行排序
std::sort(nums.begin(), nums.end(), 
    [](int a, int b) -> bool { return a > b; });

// 捕获外部变量
int threshold = 3;
auto count = std::count_if(nums.begin(), nums.end(),
    [threshold](int n) { return n > threshold; });
```

### 2.2 捕获方式

| 捕获方式 | 含义 |
|---------|------|
| `[]` | 不捕获任何外部变量 |
| `[=]` | 以值捕获所有外部变量 |
| `[&]` | 以引用捕获所有外部变量 |
| `[this]` | 捕获当前对象的指针 |
| `[=, &x]` | 默认以值捕获，但 `x` 以引用捕获 |

### 2.3 存储 Lambda

使用 `auto` 存储 lambda 比 `std::function` 更高效：

```cpp
// 推荐：使用 auto，无额外开销
auto multiply = [](int a, int b) -> int { return a * b; };

// 避免：std::function 可能有堆分配和调用开销
std::function<int(int, int)> multiply2 = [](int a, int b) -> int { return a * b; };
```

`auto` 定义的 lambda 调用速度更快，且不会抛出 `bad_alloc` 异常。

---

## 三、智能指针：内存管理的革命

C++11 废弃了有缺陷的 `std::auto_ptr`，引入了三款强大的智能指针，彻底改变了 C++ 的内存管理方式。

### 3.1 `std::unique_ptr`：独占所有权

`unique_ptr` 是轻量级的智能指针，大小与原始指针相同，保证独占所有权。

```cpp
#include <memory>

void uniquePtrDemo() {
    // 推荐：使用 make_unique（C++14 起，C++11 可手动实现）
    auto ptr = std::unique_ptr<int>(new int(42));
    
    // 自动释放内存，无需手动 delete
    std::cout << *ptr << std::endl;
    
    // 转移所有权
    auto ptr2 = std::move(ptr);
    // ptr 现在为 nullptr，ptr2 拥有资源
    
    // 自定义删除器
    auto filePtr = std::unique_ptr<FILE, decltype(&fclose)>(
        fopen("test.txt", "r"), &fclose);
}
```

**关键特性**：
- 不可复制，只可移动（Move-only）
- 零开销抽象：无额外内存占用（针对无状态删除器）
- 支持数组特化：`std::unique_ptr<int[]>`

### 3.2 `std::shared_ptr`：共享所有权

`shared_ptr` 使用引用计数管理资源，当最后一个引用消失时自动释放。

```cpp
void sharedPtrDemo() {
    auto ptr1 = std::make_shared<int>(42);
    {
        auto ptr2 = ptr1;  // 引用计数 +1
        std::cout << "Use count: " << ptr1.use_count() << std::endl;  // 输出 2
    }  // ptr2 销毁，引用计数 -1
    
    // 引用计数为 1，内存尚未释放
    std::cout << "Use count: " << ptr1.use_count() << std::endl;  // 输出 1
}  // ptr1 销毁，引用计数为 0，释放内存
```

**注意**：`shared_ptr` 需要额外的控制块（control block），内存开销比 `unique_ptr` 大。

### 3.3 `std::weak_ptr`：弱引用

`weak_ptr` 用于打破 `shared_ptr` 的循环引用问题：

```cpp
class Node {
public:
    std::shared_ptr<Node> next;      // 危险：可能导致循环引用
    std::weak_ptr<Node> parent;       // 安全：不增加引用计数
    
    ~Node() { std::cout << "Node destroyed\n"; }
};

void weakPtrDemo() {
    auto node1 = std::make_shared<Node>();
    auto node2 = std::make_shared<Node>();
    
    node1->next = node2;      // shared_ptr：引用计数 +1
    node2->parent = node1;    // weak_ptr：引用计数不变
    
    // 离开作用域时，node1 和 node2 都能正确销毁
}
```

### 3.4 智能指针使用准则

1. **默认使用 `unique_ptr`**：除非需要共享所有权，否则优先使用 `unique_ptr`
2. **使用 `make_shared` 和 `make_unique`**：避免显式 `new`，减少内存泄漏风险
3. **避免 `shared_ptr` 的循环引用**：使用 `weak_ptr` 打破循环
4. **不要混用智能指针和原始指针**：不要将同一个原始指针交给多个智能指针管理

---

## 四、右值引用与移动语义

### 4.1 左值与右值

- **左值（Lvalue）**：有名称、有持久地址的对象（如变量）
- **右值（Rvalue）**：临时对象、字面量、即将销毁的值

### 4.2 右值引用

C++11 引入 `&&` 表示右值引用，允许"窃取"临时对象的资源：

```cpp
std::vector<int> createVector() {
    return std::vector<int>{1, 2, 3, 4, 5};
}

void rvalueRefDemo() {
    std::vector<int> v1{1, 2, 3};
    std::vector<int> v2 = v1;                    // 拷贝构造：深拷贝，O(n)
    std::vector<int> v3 = createVector();        // 移动构造：转移资源，O(1)
    std::vector<int> v4 = std::move(v1);         // 强制移动：v1 变为空
}
```

### 4.3 完美转发（Perfect Forwarding）

使用 `std::forward` 在模板中保持参数的值类别：

```cpp
template<typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

**注意**：`std::move` 无条件转为右值，`std::forward` 根据模板参数决定是否保持原值类别。

---

## 五、并发支持：标准线程库

C++11 首次在标准库中提供了跨平台的线程支持：

```cpp
#include <thread>
#include <mutex>
#include <condition_variable>
#include <future>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; });  // 等待条件满足
    
    std::cout << "Worker processing...\n";
}

int main() {
    std::thread t(worker);
    
    // 主线程准备数据
    std::this_thread::sleep_for(std::chrono::seconds(1));
    
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one();  // 通知工作线程
    
    t.join();
    return 0;
}
```

**关键组件**：
- `std::thread`：线程管理
- `std::mutex` / `std::lock_guard` / `std::unique_lock`：互斥锁
- `std::condition_variable`：条件变量
- `std::future` / `std::promise`：异步结果传递
- `std::async`：异步任务

---

## 六、其他重要特性

### 6.1 `nullptr`：类型安全的空指针

```cpp
void foo(int x) { std::cout << "int\n"; }
void foo(char* p) { std::cout << "pointer\n"; }

foo(NULL);      // ✗ 歧义：可能调用 foo(int)
foo(nullptr);   // ✓ 明确调用 foo(char*)
```

### 6.2 `constexpr`：编译期计算

```cpp
constexpr int square(int x) { return x * x; }
constexpr int arr[square(5)];  // 编译期确定数组大小：25
```

### 6.3 范围 for 循环

```cpp
std::vector<int> nums{1, 2, 3, 4, 5};
for (const auto& n : nums) {  // 避免拷贝，使用 const 引用
    std::cout << n << " ";
}
```

### 6.4 强类型枚举（Scoped Enums）

```cpp
enum class Color { Red, Green, Blue };  // 不会污染命名空间
Color c = Color::Red;  // 必须显式指定作用域
```

---

## 七、总结与最佳实践

C++11 的引入标志着 C++ 进入现代编程语言行列。以下是核心要点：

| 特性 | 使用建议 |
|------|---------|
| `auto` | 优先使用，特别是在迭代器和 lambda 中 |
| 智能指针 | 默认 `unique_ptr`，需要共享时用 `shared_ptr`，注意循环引用 |
| Lambda | 捕获尽量明确，避免默认捕获大型对象 |
| 移动语义 | 为自定义类实现移动构造函数和移动赋值运算符 |
| 并发 | 优先使用标准库线程，而非平台特定 API |

**编译器支持**：GCC 4.8+、Clang 3.3+、MSVC 2013+ 均完整支持 C++11。

掌握 C++11 不仅是学习新语法，更是思维方式的转变——从手动管理资源到利用 RAII 自动管理，从繁琐的类型声明到让编译器推导，从危险的原始指针到安全的智能指针。这些改变让 C++ 代码更简洁、更安全、更高效。

---

**参考资源**：
- 《Effective Modern C++》by Scott Meyers
- [cppreference.com](https://cppreference.com)
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)
```