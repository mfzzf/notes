# 现代C++实战

## 栈、堆

### 概念

堆，英文是 heap，在内存管理的语境下，指的是动态分配内存的区域。这里的内存，被分配之后需要手工释放，否则，就会造成内存泄漏。

栈，英文是 stack，在内存管理的语境下，指的是函数调用过程中产生的本地变量和调用数据的区域。这个栈和数据结构里的栈高度相似，都满足“后进先出”（last-in-first-out 或
LIFO）。

### 栈与堆的区别
| **特性**     | **栈（Stack）**      | **堆（Heap）**               |
| ------------ | -------------------- | ---------------------------- |
| **内存管理** | 自动分配和释放       | 手动分配和释放               |
| **分配速度** | 快                   | 慢                           |
| **生命周期** | 随函数调用结束而销毁 | 由程序员控制                 |
| **内存大小** | 较小，空间有限       | 大，空间相对较大             |
| **使用场景** | 局部变量、函数调用栈 | 动态分配的大块内存、全局对象 |

---

C++ 标准里一个相关概念是自由存储区，英文是 free store，特指使用 new 和 delete 来
分配和释放内存的区域。一般而言，这是堆的一个子集：

`new` 和 `delete` 操作的区域是 `free store`
`malloc` 和 `free` 操作的区域是` heap`

## 对象切片错误

对象切片（object slicing）是一种在 C++ 中经常遇到的错误，发生在将一个**派生类对象赋值**或传递给一个**基类对象**时，导致**派生类对象的特有数据或行为丢失**。

### 举例说明

```cpp
class shape {
public:
    virtual void draw() const {
        std::cout << "Drawing a shape" << std::endl;
    }
};

class circle : public shape {
public:
    void draw() const override {
        std::cout << "Drawing a circle" << std::endl;
    }

    void specific_circle_method() {
        std::cout << "Specific method for circle" << std::endl;
    }
};

int main() {
    circle c;
    shape s = c;  // 对象切片发生
    s.draw();     // 调用的是基类的 draw() 方法，而不是 circle 的 draw()
}
```

### 在这个例子中：

- `circle c` 是一个派生类对象，它有自己的 `draw()` 方法，并且可能有其他派生类独有的成员（例如 `specific_circle_method()`）。
- 当 `circle c` 赋值给基类 `shape s` 时，只保留了基类 `shape` 的部分，派生类 `circle` 的特有部分（如 `specific_circle_method()`）被“切掉”了。结果，`s.draw()` 调用的是基类的 `draw()` 方法，而不是派生类的 `draw()` 方法。

### 避免对象切片的方法：
1. **使用指针或引用**来代替直接赋值。例如：

   ```cpp
   void draw_shape(const shape& s) {
       s.draw();  // 多态调用，派生类的 draw() 方法将被调用
   }

   int main() {
       circle c;
       draw_shape(c);  // 传递引用，避免对象切片
   }
   ```

   在这种情况下，`draw_shape()` 函数接收的是基类的引用，而不是值，因此可以正确调用派生类的 `draw()` 方法。

2. **将类声明为抽象类**，这样就不会直接创建基类对象。例如：

   ```cpp
   class shape {
   public:
       virtual void draw() const = 0;  // 纯虚函数，shape 成为抽象类
   };
   ```

   抽象类不能直接实例化，因此就避免了将派生类对象赋值给基类对象导致的对象切片问题。

## 由对象切片导致派生类的析构函数不被调用

**对象切片**本身不会直接导致内存泄漏，但它会导致多态失效，进而可能间接引发一些与内存管理相关的问题。在 C++ 中，内存泄漏通常是由于动态分配的内存没有被正确释放造成的，而对象切片与这个行为并没有直接关联。不过，下面是一些与对象切片相关的潜在问题，它们可能导致内存管理错误：

对象切片发生时，派生类的成员被丢弃，导致只保留基类的部分。由于切片之后丢失了派生类的特有成员，当你用基类来管理派生类时，派生类的行为（如析构函数）可能不会被正确调用。这种情况下，如果基类析构函数不是虚函数，释放资源时会发生问题。

### 例子：
```cpp
class shape {
public:
    virtual ~shape() { std::cout << "Base destructor called" << std::endl; }
};

class circle : public shape {
public:
    ~circle() { std::cout << "Circle destructor called" << std::endl; }
    int* data;

    circle() { data = new int[100]; }  // 动态分配资源
    ~circle() { delete[] data; }       // 释放资源
};

int main() {
    circle c;
    shape s = c;  // 对象切片
    // 派生类 circle 的部分被“切掉”
    // 只有 shape 的部分保留，circle 中的动态资源管理失效
}
```

在这个例子中，`circle` 类中的 `data` 指针动态分配了内存，并在析构函数中释放。然而，由于对象切片，`circle` 中的析构函数不会被调用，导致内存泄漏，因为 `data` 的内存没有被释放。

## RAII

RAII（Resource Acquisition Is Initialization）是一种在 C++ 中常用的资源管理机制，旨在通过**对象的生命周期**管理资源。RAII 的核心思想是将**资源的获取与释放绑定到对象的创建和销毁上**，这样就可以确保无论何时对象的生命周期结束，相关资源都能被正确释放。

### 关键概念

- **资源**：资源可以是任何需要手动管理的外部资源，例如内存、文件句柄、数据库连接、互斥锁等。
- **获取与释放绑定到对象的生命周期**：在 RAII 中，资源的获取（如动态分配内存、打开文件等）在对象构造函数中进行，而资源的释放（如释放内存、关闭文件等）则在析构函数中进行。

### 实现机制

在 C++ 中，通过构造函数和析构函数来管理资源的获取与释放：
1. **构造函数**：当对象被创建时，资源会在构造函数中被初始化。
2. **析构函数**：当对象的生命周期结束时，析构函数会自动调用，用于释放相关的资源。

这种机制与 C++ 的作用域管理结合紧密，因为一旦对象超出了其作用域，析构函数会被自动调用，确保资源得到释放。

### RAII 的好处

1. **自动管理资源**：通过 RAII，可以确保在异常发生时也能正确释放资源，避免资源泄漏。例如，如果在某个函数内分配了资源，而该函数抛出异常，RAII 可以保证对象的析构函数仍然会被调用，资源会被释放。
  
2. **简化代码**：开发者不再需要显式调用释放资源的代码（例如 `delete`、`close()`），减少了管理资源的复杂性，避免了诸如内存泄漏等问题。

3. **异常安全性**：RAII 与 C++ 的异常处理机制结合紧密，在异常情况下自动释放资源，保证程序的健壮性。

### RAII 示例

以下是一个简单的 RAII 示例，展示如何通过 RAII 管理文件句柄的打开和关闭：

```cpp
#include <iostream>
#include <fstream>

class File {
public:
    // 构造函数：打开文件
    File(const std::string& filename) {
        file_.open(filename);
        if (!file_) {
            throw std::runtime_error("Failed to open file");
        }
        std::cout << "File opened\n";
    }

    // 析构函数：关闭文件
    ~File() {
        if (file_.is_open()) {
            file_.close();
            std::cout << "File closed\n";
        }
    }

private:
    std::fstream file_;
};

int main() {
    try {
        File file("example.txt");
        // 在这里使用文件
    } catch (const std::exception& e) {
        std::cerr << e.what() << std::endl;
    }
    // 文件对象销毁时，析构函数会自动调用，关闭文件
    return 0;
}
```

在这个示例中，`File` 类在构造函数中打开文件，并在析构函数中关闭文件。即使在 `main()` 中发生异常，文件仍然会被正确关闭。

### RAII 常见应用场景

1. **内存管理**：通过智能指针（如 `std::unique_ptr`、`std::shared_ptr`）来管理动态分配的内存。
   
2. **文件操作**：通过 RAII 管理文件的打开与关闭，避免文件句柄泄漏。
   
3. **线程锁管理**：通过 RAII 方式管理互斥锁，如 `std::lock_guard`，确保锁在作用域结束时自动释放，避免死锁问题。



## 智能指针

```cpp
class shape_wrapper
{
public:
    explicit shape_wrapper(
        shape *ptr = nullptr)
        : ptr_(ptr) {}
    ~shape_wrapper()
    {
        delete ptr_;
    }
    shape *get() const { return ptr_; }

private:
    shape *ptr_;
};
```

上一讲中，给出了这个类，这个类可以完成智能指针的最基本的功能：对超出作用域的对象进行释放。但它缺了点东西：

1.  这个类只适用于 shape 类
2.  该类对象的行为不够像指针
3.  拷贝该类对象会引发程序行为异常

要能包装任何类型的指针，我们需要使用类模板。

要使他的行为像指针，我们需要加上成员函数。

```cpp
template <typename T>
class smart_ptr
{
public:
    explicit smart_ptr(
        T *ptr = nullptr)
        : ptr_(ptr) {}
    ~smart_ptr()
    {
        delete ptr_;
    }
    T *get() const { return ptr_; }

    T* operator->() const {return ptr_;}
    T& operator*() const {return *ptr_;}
    operator bool() const {return ptr_; }
private:
    T *ptr_;
};
```



## 现代 C++ 中的 RAII

在现代 C++（C++11 及以后）中，智能指针（如 `std::unique_ptr`、`std::shared_ptr`）是 RAII 的典型实现，用来简化内存管理并防止内存泄漏。

例如，`std::unique_ptr` 通过 RAII 自动释放内存：

```cpp
#include <memory>

int main() {
    std::unique_ptr<int> ptr(new int(42));
    // 无需手动 delete，ptr 超出作用域时自动释放内存
    return 0;
}
```

### 总结

RAII 是 C++ 资源管理的一种核心机制，通过构造函数和析构函数将资源的获取和释放绑定到对象的生命周期，确保资源能够被正确管理。RAII 的引入减少了手动管理资源的负担，提高了代码的健壮性和异常安全性。