---
title: 虚函数和CRTP
date: 2025-10-18 21:49:04
tags: C++
categories: C++
---

# 虚函数

## 概念

虚函数是C++实现运行时多态的核心机制。通过在基类中使用`virtual`关键字声明函数，可以让派生类重写（override）该函数，从而在运行时根据对象的实际类型调用相应的函数实现。

## 特点

- **动态绑定**：函数调用在运行时根据对象的实际类型确定，而不是编译时的静态类型
- **虚函数表（vtable）**：编译器为每个包含虚函数的类生成一个虚函数表，通过虚指针（vptr）指向该表
- **性能开销**：由于需要通过虚指针查找虚函数表，存在一定的运行时开销
- **灵活性**：派生类可以选择性地重写虚函数，未重写时使用基类的默认实现

## 使用场景

当你需要通过基类指针或引用操作不同派生类对象，并根据对象的实际类型执行不同行为时，虚函数是最佳选择。

## 示例代码

```
#include <iostream>
#include <memory>

// 基类
class Animal {
public:
    virtual ~Animal() = default;  // 虚析构函数
    
    virtual void makeSound() const {
        std::cout << "Animal makes a sound" << std::endl;
    }
    
    virtual void move() const {
        std::cout << "Animal moves" << std::endl;
    }
};

// 派生类：狗
class Dog : public Animal {
public:
    void makeSound() const override {
        std::cout << "Dog barks: Woof! Woof!" << std::endl;
    }
    
    void move() const override {
        std::cout << "Dog runs on four legs" << std::endl;
    }
};

// 派生类：猫
class Cat : public Animal {
public:
    void makeSound() const override {
        std::cout << "Cat meows: Meow! Meow!" << std::endl;
    }
    
    void move() const override {
        std::cout << "Cat walks gracefully" << std::endl;
    }
};

// 派生类：鸟
class Bird : public Animal {
public:
    void makeSound() const override {
        std::cout << "Bird chirps: Tweet! Tweet!" << std::endl;
    }
    
    void move() const override {
        std::cout << "Bird flies in the sky" << std::endl;
    }
};

// 使用多态
void demonstratePolymorphism(const Animal& animal) {
    animal.makeSound();
    animal.move();
    std::cout << "---" << std::endl;
}

int main() {
    Dog dog;
    Cat cat;
    Bird bird;
    
    std::cout << "=== 运行时多态演示 ===" << std::endl;
    demonstratePolymorphism(dog);
    demonstratePolymorphism(cat);
    demonstratePolymorphism(bird);
    
    // 使用指针
    std::cout << "=== 使用指针 ===" << std::endl;
    Animal* animals[] = {&dog, &cat, &bird};
    for (auto* animal : animals) {
        animal->makeSound();
    }
    
    return 0;
}
```

# CRTP

## 概念

CRTP（Curiously Recurring Template Pattern，奇异递归模板模式）是一种编译时多态技术。其核心思想是让派生类作为模板参数传递给基类模板，使得基类可以在编译时访问派生类的成员。

## 特点

- **静态多态**：函数调用在编译时就已确定，无需运行时查找
- **零开销**：不需要虚函数表和虚指针，没有运行时性能损耗
- **编译期类型检查**：如果派生类没有实现所需的函数，会在编译时报错
- **代码膨胀**：每个派生类都会生成独立的代码，可能导致二进制文件增大

## 使用场景

当你需要在编译期实现多态，追求零开销抽象，且类型关系在编译期就能确定时，CRTP是理想选择。常用于性能敏感的场景，如模板库、数学库等。

## 示例代码

```
template <typename Derived>
class Base {
public:
    void interface() {
        // 调用派生类的实现
        static_cast<Derived*>(this)->implementation();
    }
};

class Derived : public Base<Derived> {
public:
    void implementation() {
        // 具体实现
    }
};
```


# 虚函数 vs CRTP 对比

## 主要区别

| 特性 | 虚函数 | CRTP |
|------|--------|------|
| 多态类型 | 运行时多态 | 编译时多态 |
| 性能开销 | 有（虚函数表查找） | 无（编译期展开） |
| 灵活性 | 可选择性重写 | 必须实现所需函数 |
| 内存开销 | 每个对象有虚指针 | 无额外指针开销 |
| 代码大小 | 较小 | 可能较大（模板展开） |
| 类型安全 | 运行时 | 编译时 |
| 异构容器 | 支持（可用基类指针容器） | 不支持（类型不同） |

## 选择建议

**使用虚函数的情况：**
- 需要在运行时动态确定对象类型
- 需要将不同类型的对象放在同一个容器中
- 派生类可以选择性地实现某些功能
- 性能开销可以接受

**使用CRTP的情况：**
- 追求极致性能，无法接受虚函数开销
- 类型关系在编译期就已确定
- 需要编译期类型检查和错误提示
- 所有派生类都必须实现相同的接口

## 个人理解

如果虚函数基类中的某一个`virtual`函数会被所有子类`override`，那其实就可以用CRTP平替。CRTP要求所有子类都必须实现被调用的函数（否则编译错误），而虚函数的子类可以选择不实现某个虚函数（使用基类的默认实现）。因此，虚函数更灵活，CRTP更高效。