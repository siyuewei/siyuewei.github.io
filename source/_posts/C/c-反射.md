---
title: C#反射
date: 2025-10-18 23:05:12
tags: C#
categories: C#
---

# C#反射

## 概念

反射（Reflection）是C#中一种强大的机制，它允许程序在运行时检查和操作类型的元数据信息。通过反射，可以在运行时动态地获取类型信息、创建对象实例、调用方法、访问属性和字段等。

反射的核心是`System.Reflection`命名空间，其中`Type`类是反射的基础。

## 个人理解

本质就是可以通过`Type`，在程序运行时去动态获取当前类里面的信息。运行前可能不知道类的具体信息，运行中可以直接获取。

## 主要功能

反射可以实现以下功能：

1. **获取类型信息**：获取类、接口、结构体等的元数据
2. **动态创建对象**：在运行时创建类型的实例
3. **动态调用方法**：在运行时调用对象的方法
4. **访问成员**：读取或设置字段、属性的值
5. **查看特性（Attribute）**：获取类型上标注的特性信息
6. **遍历程序集**：查看程序集中包含的所有类型

## 常用API

### 获取Type对象的方式

```csharp
// 1. 使用 typeof 关键字
Type type1 = typeof(MyClass);

// 2. 使用 GetType() 方法
MyClass obj = new MyClass();
Type type2 = obj.GetType();

// 3. 使用 Type.GetType() 方法
Type type3 = Type.GetType("命名空间.MyClass");
```

### 常用反射方法

```csharp
// 获取所有公共方法
MethodInfo[] methods = type.GetMethods();

// 获取所有公共属性
PropertyInfo[] properties = type.GetProperties();

// 获取所有公共字段
FieldInfo[] fields = type.GetFields();

// 获取所有构造函数
ConstructorInfo[] constructors = type.GetConstructors();

// 创建实例
object instance = Activator.CreateInstance(type);

// 调用方法
MethodInfo method = type.GetMethod("MethodName");
method.Invoke(instance, new object[] { param1, param2 });

// 获取/设置属性值
PropertyInfo property = type.GetProperty("PropertyName");
object value = property.GetValue(instance);
property.SetValue(instance, newValue);
```

## 使用场景

1. **插件系统**：动态加载和使用外部程序集中的类型
2. **依赖注入（DI）容器**：如ASP.NET Core的依赖注入框架
3. **序列化/反序列化**：JSON、XML等序列化库
4. **ORM框架**：如Entity Framework，动态映射对象和数据库
5. **单元测试框架**：如NUnit、xUnit，动态发现和执行测试方法
6. **自动化工具**：根据配置文件动态创建和调用对象
7. **代码生成工具**：分析现有类型生成代码
8. **特性处理**：处理自定义特性标记的类和方法

## 示例代码

```csharp
using System;
using System.Reflection;

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public void SayHello()
    {
        Console.WriteLine($"你好，我是{Name}，{Age}岁。");
    }
    
    public string GetInfo(string prefix)
    {
        return $"{prefix}: {Name}, {Age}岁";
    }
}

class Program
{
    static void Main()
    {
        // 获取Type对象
        Type type = typeof(Person);
        Console.WriteLine($"类型名称: {type.Name}");
        Console.WriteLine($"完整名称: {type.FullName}");
        
        // 创建实例
        object personObj = Activator.CreateInstance(type);
        
        // 设置属性值
        PropertyInfo nameProp = type.GetProperty("Name");
        PropertyInfo ageProp = type.GetProperty("Age");
        nameProp.SetValue(personObj, "张三");
        ageProp.SetValue(personObj, 25);
        
        // 获取属性值
        Console.WriteLine($"姓名: {nameProp.GetValue(personObj)}");
        Console.WriteLine($"年龄: {ageProp.GetValue(personObj)}");
        
        // 调用无参方法
        MethodInfo sayHelloMethod = type.GetMethod("SayHello");
        sayHelloMethod.Invoke(personObj, null);
        
        // 调用有参方法
        MethodInfo getInfoMethod = type.GetMethod("GetInfo");
        object result = getInfoMethod.Invoke(personObj, new object[] { "个人信息" });
        Console.WriteLine(result);
        
        // 列出所有公共属性
        Console.WriteLine("\n所有公共属性:");
        foreach (PropertyInfo prop in type.GetProperties())
        {
            Console.WriteLine($"- {prop.Name} ({prop.PropertyType.Name})");
        }
        
        // 列出所有公共方法
        Console.WriteLine("\n所有公共方法:");
        foreach (MethodInfo method in type.GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly))
        {
            Console.WriteLine($"- {method.Name}");
        }
    }
}
```

## 优点

1. **动态性**：可以在运行时处理编译时未知的类型
2. **灵活性**：适合构建通用框架和工具
3. **解耦**：降低模块间的直接依赖
4. **可扩展性**：便于实现插件机制

## 缺点

1. **性能开销**：反射操作比直接调用慢很多（通常慢10-100倍）
2. **类型安全性降低**：编译时无法检查类型错误
3. **代码可读性差**：反射代码通常比直接调用难理解
4. **安全问题**：可以访问私有成员，可能破坏封装性
5. **调试困难**：反射调用的错误在运行时才能发现

## 为什么反射的性能开销大？

反射的性能开销主要来自以下几个方面：

### 1. 元数据查找开销
- **运行时搜索**：反射需要在运行时遍历类型的元数据来查找方法、属性、字段等信息
- **字符串匹配**：使用字符串名称查找成员（如`GetMethod("MethodName")`）需要进行字符串比较
- **无编译器优化**：编译器无法对反射调用进行优化，而直接调用在编译时就已经确定了地址

### 2. 类型检查和验证
- **参数验证**：每次反射调用都需要验证参数类型是否匹配
- **访问权限检查**：需要检查是否有权限访问该成员（public、private等）
- **类型转换**：反射调用的参数通常是`object[]`，需要进行装箱/拆箱操作

### 3. 安全检查
- **权限验证**：CLR需要验证调用者是否有足够的权限执行反射操作
- **安全栈遍历**：可能需要遍历调用栈来验证安全性

### 4. 间接调用
- **无法内联**：直接调用可以被编译器内联优化，而反射调用无法内联
- **动态调度**：反射调用通过间接的方式执行，无法利用CPU的分支预测等优化

### 5. 对象创建开销
- **装箱操作**：值类型参数需要装箱成`object`
- **数组分配**：参数需要封装到`object[]`数组中
- **临时对象**：会产生额外的临时对象和GC压力

### 性能对比示例

```csharp
// 直接调用：约 1-2 纳秒
person.SayHello();

// 反射调用：约 100-200 纳秒
methodInfo.Invoke(person, null);

// 性能差距：约 100 倍
```

**总结**：反射的性能开销主要是因为它将编译时的静态操作推迟到了运行时，需要进行大量的查找、验证、类型检查等动态操作，而这些操作在直接调用中都是在编译时完成的。

## 性能优化建议

1. **缓存Type和MemberInfo对象**：避免重复获取
2. **使用表达式树（Expression Trees）**：比反射快，但仍比直接调用慢
3. **考虑使用IL Emit**：动态生成IL代码，性能接近原生
4. **谨慎使用**：仅在必要时使用反射，能用泛型就用泛型

## BindingFlags 常用标志

```csharp
// 公共成员
BindingFlags.Public

// 非公共成员（private, protected, internal）
BindingFlags.NonPublic

// 实例成员
BindingFlags.Instance

// 静态成员
BindingFlags.Static

// 仅包含当前类声明的成员（不包括继承的）
BindingFlags.DeclaredOnly

// 组合使用示例
type.GetMethods(BindingFlags.Public | BindingFlags.Instance)
type.GetFields(BindingFlags.NonPublic | BindingFlags.Static)
```

## 小结

反射是C#中非常强大的特性，它让程序具有了自省和动态操作的能力。虽然存在性能开销，但在很多场景下（如框架开发、插件系统），反射带来的灵活性是无可替代的。关键是要在灵活性和性能之间找到合适的平衡点。