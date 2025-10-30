---
title: C#基本概念汇总
date: 2025-10-30 18:48:20
tags: C#
categories: C#
---

# 创建属性

属性本质上是对变量的 get 和 set 访问器的封装。

对于需要公开访问的字段，直接使用 public 修饰符存在安全隐患，而使用属性可以带来以下优势：

1. 保护私有数据不被直接修改
2. 在 get 和 set 访问器中进行安全检测
3. 支持定义只读或只写属性

此外，属性也可以理解为两个方法的组合。例如下面的 Level 属性，实际上就是提供了两个方法来修改或获取内部的 experience 数据。

```
public class PlayerController
{
    private int experience;

    public int Experience
    {
        get
        {
            return experience;
        }
        set
        {
            experience = value;
        }
    }
    
    public int Level
    {
        get
        {
            return experience / 1000;
        }
        set
        {
            experience = value * 1000;
        }
    }
    
    public int Health { get; set; }
}


```

## 特别注意

- 在接口中：{ get; set; } 是一个抽象声明，它不包含任何实现或数据存储。它强制实现类去提供实现。

- 在类中：{ get; set; } 是一个完整的实现，它自动为数据存储和访问逻辑。

# 虚函数

C# 支持单继承，子类可以通过 `base` 关键字访问父类的成员。

虚函数（Virtual Function）是实现多态的核心机制：
- 父类使用 `virtual` 关键字声明可被重写的方法
- 子类使用 `override` 关键字重写父类的虚方法

**关键特性**：当子类对象被强制转换为父类类型时，只能访问父类定义的成员，但调用虚函数时仍会执行子类重写的版本，这体现了运行时多态。

代码示例如下：

```csharp
using UnityEngine;

// 父类：定义基础行为
public class Actor
{
    private int _health;
    
    // 属性：封装健康值
    public int Health
    {
        get { return _health; }
        set { _health = value; }
    }
    
    // 虚方法：可被子类重写
    public virtual void Move(float x, float y)
    {
        Debug.Log("Actor is moving to: " + x + ", " + y);
    }
}

// 子类：继承并扩展父类功能
public class PlayerController : Actor
{
    // 子类特有的属性
    public int jumpHeight;

    public PlayerController()
    {
        jumpHeight = 2;
    }
    
    // 重写父类的虚方法
    public override void Move(float x, float y)
    {
        Debug.Log("PlayerController is moving to: " + x + ", " + y);
    }
}

// 测试类：演示多态行为
namespace Test
{
    public class TestPlayer : MonoBehaviour
    {
        void Start()
        {
            PlayerController player = new PlayerController();
            
            // 通过子类引用调用
            player.Move(5.0f, 10.0f);           // 输出：PlayerController is moving to: 5, 10
            Debug.Log(player.jumpHeight);        // 输出：2（子类特有属性可访问）
            Debug.Log(player.Health);            // 输出：0（继承自父类的属性）
            
            // 将子类对象强转为父类类型
            Actor actor = (Actor)player;
            
            // 虚方法调用：仍然执行子类的重写版本（多态）
            actor.Move(15.0f, 20.0f);           // 输出：PlayerController is moving to: 15, 20
            
            // 子类特有成员无法访问（编译错误）
            // Debug.Log(actor.jumpHeight);      // 错误：Actor 类型没有 jumpHeight 成员
        }
    }
}
```

# 成员隐藏

成员隐藏（Member Hiding）是通过 `new` 关键字在子类中重新定义父类的成员或方法，从而"隐藏"父类的实现。

**核心区别**：
- **虚函数（多态）**：使用 `virtual` 和 `override`，强转后调用的是子类重写的方法
- **成员隐藏**：使用 `new` 关键字，强转后调用的是父类原有的方法

**行为特点**：当子类对象被强制转换为父类类型时，调用的成员或方法都是父类的版本，而非子类隐藏的版本。这是因为成员隐藏是编译时绑定，而非运行时多态。

代码示例如下：

```csharp
using UnityEngine;

// 父类
public class Warrior
{
    public int damage = 10;
    
    // 普通方法（非虚方法）
    public void Attack()
    {
        Debug.Log("Warrior attacks with damage: " + damage);
    }
    
    public void Defend()
    {
        Debug.Log("Warrior defends!");
    }
}

// 子类：使用 new 关键字隐藏父类成员
public class Paladin : Warrior
{
    // 使用 new 隐藏父类的字段
    public new int damage = 20;
    
    // 使用 new 隐藏父类的方法
    public new void Attack()
    {
        Debug.Log("Paladin attacks with holy damage: " + damage);
    }
    
    public new void Defend()
    {
        Debug.Log("Paladin defends with shield!");
    }
}

// 测试类：对比成员隐藏的行为
namespace Test
{
    public class TestHiding : MonoBehaviour
    {
        void Start()
        {
            Paladin paladin = new Paladin();
            
            // 通过子类引用访问
            Debug.Log(paladin.damage);      // 输出：20（访问子类的 damage）
            paladin.Attack();                // 输出：Paladin attacks with holy damage: 20
            paladin.Defend();                // 输出：Paladin defends with shield!
            
            Debug.Log("--- 强转为父类后 ---");
            
            // 将子类对象强转为父类类型
            Warrior warrior = (Warrior)paladin;
            
            // 关键：强转后访问的是父类的成员（编译时绑定）
            Debug.Log(warrior.damage);       // 输出：10（访问父类的 damage）
            warrior.Attack();                // 输出：Warrior attacks with damage: 10
            warrior.Defend();                // 输出：Warrior defends!
            
            // 注意：虽然 warrior 和 paladin 引用的是同一个对象，
            // 但由于使用了 new 隐藏而非 override 重写，
            // 所以通过父类引用调用的是父类版本的方法
        }
    }
}
```

**总结**：
- 成员隐藏打破了多态性，调用哪个版本取决于引用类型而非对象类型
- 如果需要多态行为，应使用 `virtual` 和 `override`
- `new` 关键字主要用于有意隐藏父类成员，或解决命名冲突

# 接口（Interface）

接口是一种定义契约的方式，只包含成员的声明，不包含任何实现。实现接口的类必须提供所有成员的具体实现。

**接口的特点**：
- 只包含方法、属性、事件和索引器的声明，不包含实现
- 接口成员默认是 `public` 的，不需要显式声明访问修饰符
- 一个类可以实现多个接口，弥补 C# 单继承的限制
- 接口支持多态，通过接口引用可以调用不同实现类的方法

**属性与接口**：属性可以定义在接口中，因为属性本质上就是 `get` 和 `set` 方法的声明。接口可以要求实现类提供特定的属性访问器。

## 接口属性的本质与演进

### 传统接口（C# 8.0 之前）

在 C# 8.0 之前，接口**只能包含声明，不能包含实现**。

```csharp
// 传统接口：只有抽象声明
public interface IWeapon
{
    // 这是抽象属性声明，不是实现
    int BulletNumber { get; set; }
    
    // 等同于声明这两个抽象方法：
    // int get_BulletNumber();
    // void set_BulletNumber(int value);
}

// 实现类必须提供完整实现
public class Pistol : IWeapon
{
    // 方式一：自动属性（编译器自动生成后备字段和实现）
    public int BulletNumber { get; set; }
    
    // 方式二：完整属性（手动定义后备字段和逻辑）
    // private int _bulletNumber;
    // public int BulletNumber 
    // { 
    //     get { return _bulletNumber; }
    //     set { _bulletNumber = value; }
    // }
}
```

**关键点**：
- 接口中的 `{ get; set; }` 是**抽象声明**，强制实现类提供实现和数据存储
- 类中的 `{ get; set; }` 是**自动属性**，编译器自动生成后备字段和完整实现

### 接口默认实现（C# 8.0+）

从 C# 8.0 开始，接口可以包含**默认实现**，主要用于 API 的版本演进。

**引入原因**：解决接口扩展时的破坏性更改问题。

```csharp
// 场景：为已有接口添加新功能，但不破坏现有实现类
public interface IWeapon
{
    void Attack();  // 原有方法
    
    // C# 8.0+：为新方法提供默认实现
    // 现有实现类无需修改，新类可以选择重写
    void Reload()
    {
        Console.WriteLine("Default reload behavior");
    }
}
```

**属性的默认实现示例**：

```csharp
public interface IWeapon
{
    // 1. 抽象属性：必须由实现类提供（用于存储状态）
    int CurrentAmmo { get; set; }
    int MaxAmmo { get; }
    
    // 2. 默认实现的计算属性：基于其他抽象成员
    bool IsEmpty
    {
        get { return CurrentAmmo <= 0; }  // ✅ 正确：依赖抽象属性
    }
    
    // 3. 默认实现的方法：操作抽象成员
    void Shoot()
    {
        if (!IsEmpty)
        {
            CurrentAmmo--;
        }
    }
}
```

### ⚠️ 常见错误：接口默认实现的递归陷阱

**错误示例**（会导致 `StackOverflowException`）：

```csharp
public interface IWeapon
{
    // ❌ 错误：接口不能包含实例字段（后备字段）
    int bulletNumber 
    {
        get
        {
            // 无限递归！调用 get 来获取 bulletNumber，而 bulletNumber 又调用 get...
            return bulletNumber;
        }
        set
        {
            // 无限递归！调用 set 来设置 bulletNumber，而 bulletNumber 又调用 set...
            bulletNumber = value;
        }
    }
}
```

**原因**：接口不能包含实例状态（字段）。上述代码中没有后备字段来存储数据，访问属性会无限递归调用自身。

**正确的做法**：
- **抽象声明**：让实现类提供存储
- **默认实现**：基于其他抽象成员进行计算或逻辑处理
