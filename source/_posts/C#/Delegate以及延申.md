---
title: Delegate以及延申
date: 2025-11-14 18:21:20
tags: 
    - C#
    - Unity
categories: C#
---

# 基本关系介绍

Delegate类似一个函数的类型声明，与之相关的内容有Func，Action，Event，UnityEvent。
```
┌─────────────────────────────────────┐
│     delegate (C# 的基础特性)         │  ← 最底层
└─────────────────┬───────────────────┘
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
┌─────────────┐      ┌─────────────┐
│ Func/Action │      │ 自定义delegate│
│  (C# 内置)  │      │  (你定义的) │
└──────┬──────┘      └──────┬──────┘
       │                    │
       └────────┬───────────┘
                ↓
        ┌─────────────┐
        │    event    │  ← 关键字，不是类型
        │  (访问限制)  │
        └──────┬──────┘
               │
               ↓
        ┌─────────────┐
        │ UnityEvent  │  ← Unity 特有封装
        │(Unity特有)  │
        └─────────────┘
```

| 名称 | 类型/关键字 | 来源 | 用途 | Inspector可见 |
|------|------------|------|------|---------------|
| `delegate` | 关键字 | C# | 定义委托类型 | ❌ |
| `Func` | 预定义委托 | C#/.NET | 有返回值的通用委托 | ❌ |
| `Action` | 预定义委托 | C#/.NET | 无返回值的通用委托 | ❌ |
| `event` | 关键字/修饰符 | C# | 封装委托，限制访问 | ❌ |
| `UnityEvent` | 类 | Unity | Unity封装的事件系统 | ✅ |

# 详细解释
## 1. delegate - C#的基础特性

**定义**：函数的类型定义，是对函数签名（输入参数+返回值）的抽象

```csharp
// 1. 定义委托类型（就像定义一个类型）
public delegate int Calculator(int a, int b);

// 2. 定义符合委托签名的方法
public int Add(int a, int b) 
{
    return a + b;
}

public int Multiply(int a, int b) 
{
    return a * b;
}

// 3. 使用委托
Calculator calc = Add;           // 将方法赋值给委托变量
int result = calc(5, 3);         // 调用委托，结果为 8

calc = Multiply;                 // 更换为另一个方法
result = calc(5, 3);             // 调用委托，结果为 15
```
## 2. Func和Action - C#预定义的泛型委托
- Func和Action都是C#基于Delegate预先定义好的类型
- 主要区别在于Func是有返回值的，Action是没有返回值的

```
它们在.NET Framework/Core中的定义类似下面

//Action：无返回值的委托
public delegate void Action();
public delegate void Action<T>(T arg);
public delegate void Action<T1,T2>(T1 arg1,T2 args);
//...最多16个参数

//Func：有返回值的委托，最后一个参数是返回值
public delegate TResult Func<TResult>();
public delegate TResult Func<T,TResult>(T arg);
public delegate TResult Func<T1,T2,TResult>(T1 arg1,T2 arg2);
//...最多16个参数+1个返回值
```

## 3. event - C#的访问控制关键字
- event不是一个类型，它是修饰符关键字
```
public delegate void Action(); //C#中已有代码，不需要我们再写
public event Action myAction;  //event修饰Action

public delegate int MyDelegate(bool b); //自己随便定义的委托
public event MyDelegate myDelegate; //event修饰自定义的委托
```

有无event的对比：
```
//没有event，外部可以随机操作
public delegate int MyDelegate(bool b);
public MyDelegate myDelegate;

//外部代码可以：
obj.myDelegate += Handler;  // ✅ 订阅
obj.myDelegate -= Handler;  // ✅ 取消订阅
obj.myDelegate();           // ❌ 但也能直接调用！
obj.myDelegate = null;      // ❌ 甚至能清空所有订阅者！
```

```
有event修饰，外部的访问权限部分被限制
public delegate int MyDelegate(bool b);
public MyDelegate myDelegate;

//外部代码只能：
obj.myDelegate += Handler;  // ✅ 订阅
obj.myDelegate -= Handler;  // ✅ 取消订阅
obj.myDelegate();           // ❌ 编译错误！不能调用
obj.myDelegate = null;      // ❌ 编译错误！不能赋值
```

## 4. UnityEvent - Unity特有的封装事件类
```
using UnityEngine.Events; //Unity特有的命名空间

//UnityEvent是Unity封装的事件类
public UnityEvent onButtonClick;       //无参数
public UnityEvent<int> onScoreChanged; //带参数
```

- Unity特有，不是C#标准
- 可以在inspector窗口中可视化配置
- 可以序列化
- 使用反射，性能比较差

### UnityEvent使用反射
#### 1.Inspector配置中的反射
```
using UnityEngine;
using UnityEngine.Events;

public class MyButton : MonoBehaviour
{
    public UnityEvent onClick;  // 在 Inspector 中可见
}
```
Inspector 做的事情：
- 使用反射扫描 GameObject 上的所有组件
- 使用反射获取每个组件的所有 public 方法
- 展示在下拉列表中供你选择

#### 2.运行时的反射调用
Unity 将 Inspector 配置序列化存储：
```csharp
// 存储的配置信息（字符串形式）
{
    "target": ScoreManager 组件引用,
    "methodName": "UpdateScore",
    "argument": 100
}
```

当调用 `onClick.Invoke()` 时，Unity 读取这些配置并使用反射调用：

```csharp
// 简化的内部实现
public void Invoke()
{
    // 1. 获取 Inspector 配置的目标对象和方法名
    Object target = config.target;
    string methodName = config.methodName;
    
    // 2. 使用反射获取方法
    Type type = target.GetType();
    MethodInfo method = type.GetMethod(methodName);
    
    // 3. 使用反射调用方法
    method.Invoke(target, new object[] { config.argument });
}
```

# 使用场景
一般而言，Delegate有以下使用场景：
- 事件系统：
  - event Action         ████████████████████ 80%  (最常用)
  - event 自定义delegate  ████████ 20%

- 回调/策略：
  - Func/Action          ████████████ 60%
  - 自定义 delegate       ████████ 40%

- 有返回值的多播（一般来说可能是验证链）：
   - 自定义 delegate + 手动处理 GetInvocationList

## 1.事件系统
优先event Action，当参数很多，需要参数名提高可读性的时候，会用到自定义delegate
```
// 当参数很多，需要参数名提高可读性时
public delegate void OnTowerUpgraded(
    TowerActor tower,
    int oldLevel,
    int newLevel,
    UpgradeType type,
    float cost
);
public event OnTowerUpgraded onTowerUpgraded;

// 对比 Action：
public event Action<TowerActor, int, int, UpgradeType, float> onTowerUpgraded;
// ❌ 看不懂每个参数的含义
```

## 2.回调/策略
简单场景，使用 Func/Action
```
// ✅ 简单回调用 Func/Action
public void LoadAsync(string url, Action<string> onSuccess)
{
    // ...
}

public void ProcessData(Func<int, int> transformer)
{
    // ...
}

// LINQ 风格
public List<T> Filter<T>(List<T> list, Func<T, bool> predicate)
{
    // ...
}
```

领域概念/复杂场景 → 自定义 delegate
```
// ✅ 表达领域概念用自定义 delegate
public delegate EnemyActor TargetingLogicFuncType(
    List<EnemyActor> enemies,
    TowerActor tower
);

private Dictionary<TargetingType, TargetingLogicFuncType> _logics;

// 对比 Func：
private Dictionary<TargetingType, Func<List<EnemyActor>, TowerActor, EnemyActor>> _logics;
// ❌ 太长，可读性差
```

## 3.多播 - 验证链（有返回值）
```
public class ValidationSystem
{
    // ✅ 有返回值，不用 event
    public delegate bool Validator(string input);
    public Validator validators;  // ⚠️ 没有 event 关键字
    
    // 也可以用 Func（少见）
    // public Func<string, bool> validators;
    
    public bool ValidateAll(string input)
    {
        if (validators == null) return true;
        
        // 手动处理所有返回值
        foreach (Validator v in validators.GetInvocationList())
        {
            if (!v(input))
            {
                Debug.Log($"验证失败：{v.Method.Name}");
                return false;
            }
        }
        return true;
    }
}

// 使用
validationSystem.validators += ValidateLength;
validationSystem.validators += ValidateFormat;
bool isValid = validationSystem.ValidateAll("test@email.com");
```

有返回值的多播，通常不加 event，需要手动控制调用逻辑。其实在代码上没有任何问题
```
public class ValidationSystem
{
    public delegate bool Validator(string input);
    public event Validator validators;  // ✅ 封装良好
    
    // 必须提供方法
    public bool ValidateAll(string input)
    {
        if (validators == null) return true;
        foreach (Validator v in validators.GetInvocationList())
        {
            if (!v(input)) return false;
        }
        return true;
    }
}
```
但是语义怪异（event 暗示"事件"），违反开发者直觉，必须提供包装方法

实际上是一种约定俗成

如果要实现event那种封装，可以用其他的办法：
```
public class ValidationSystem
{
    public delegate bool Validator(string input);
    private Validator _validators;  // ✅ 私有化
    
    // 提供受控的订阅接口
    public void AddValidator(Validator validator)
    {
        _validators += validator;
    }
    
    public void RemoveValidator(Validator validator)
    {
        _validators -= validator;
    }
    
    // 提供调用接口
    public bool ValidateAll(string input)
    {
        if (_validators == null) return true;
        foreach (Validator v in _validators.GetInvocationList())
        {
            if (!v(input)) return false;
        }
        return true;
    }
}
```