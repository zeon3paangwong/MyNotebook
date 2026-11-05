---
title: Qt 元对象系统
tags:
  - Qt
  - 元对象
  - 反射
created: 2026-11-05
---
# Qt 元对象系统（Meta-Object System）

> [!abstract] 核心定义
> **元对象（QMetaObject）**：保存一个类完整的类型信息（类名、信号、槽、属性、方法等），为 Qt 提供**运行时反射**能力。
> 启用元对象系统有两个硬性要求：
> 1. 类必须继承 `QObject`
> 2. 类声明内部必须添加 `Q_OBJECT` 宏

> [!warning] 拼写勘误
> 笔记常见笔误：`Q_INBOKABEL` **正确宏名是 `Q_INVOKABLE`**

---
## 1. 反射三要素：字段 / 方法 / 反射类
### 1.1 反射字段（Q_PROPERTY）
想要一个成员变量成为可反射的属性，满足两点：
1. 字段必须配套**读访问器 / 写访问器**；
2. 通过 `Q_PROPERTY` 将访问器与成员变量绑定。

### 1.2 反射方法
把一个普通成员函数纳入元对象系统、支持运行时字符串调用，三种可选方式：
1. 声明为 `signal`
2. 声明为 `slots`
3. 函数加上 `Q_INVOKABLE` 宏（最常用，普通成员函数推荐）

### 1.3 反射类（运行时动态创建）
C++ 普通编译期创建对象的前提：
1. 编译阶段拿到类的完整信息（导入头文件）；
2. 可以访问类的构造函数。

Qt 反射创建对象：**无需提前知晓具体类类型**，依靠元对象在运行时构造实例：
- 带有默认构造函数的类，可以给构造函数标记 `Q_INVOKABLE`
- 通过 `QMetaObject::newInstance()` 反射创建对象

> [!note] 原理补充
> 当实例化 QObject 派生类对象时，Qt 会自动生成配套的 `QMetaObject` 元对象。
> `QMetaObject::newInstance()` 是 **反射式对象构造**，和 `invokeMethod` 属于同一套元对象机制：
> - `invokeMethod`：运行时按字符串**调用函数**
> - `newInstance`：运行时按字符串 / 元对象**构造对象**
> 不再直接使用 `new 类名()` 硬编码创建。

---
## 2. Q_INVOKABLE 宏作用
> [!important]
> `Q_INVOKABLE` 把普通成员函数注册进 Qt 元对象系统，实现：
> 1. **反射调用**：可以通过 `QMetaObject::invokeMethod` 传入函数字符串动态调用；
> 2. 被 **QML、脚本引擎、Qt 属性系统**识别使用。

> [!tip] 使用建议
> 普通成员函数想要被反射调用，优先 `Q_INVOKABLE`；
> 通信交互场景优先使用信号槽 `signal/slots`。

---
## 3. QMetaObject::invokeMethod 三大用途
`QMetaObject::invokeMethod` 是元对象系统核心反射调用接口，主要场景：
1. 调用被 `Q_INVOKABLE` 修饰的成员函数；
2. **跨线程**异步调用其他对象的槽函数（Qt 跨线程通信经典方案）；
3. 实现函数的**延迟调用（QueuedConnection）**。

---
## 4. 概念速查总结
| 概念 | 用途 |
| ---- | ---- |
| Q_OBJECT | 开启当前类的元对象代码生成，反射系统基础 |
| Q_PROPERTY | 定义可反射的**属性（字段）** |
| Q_INVOKABLE | 标记普通成员函数，允许反射调用 |
| signal/slots | 信号槽，本身天然被元对象识别 |
| QMetaObject | 承载类的所有运行时类型信息 |
| QMetaObject::newInstance | 运行时反射创建对象实例 |
| QMetaObject::invokeMethod | 运行时反射调用函数，支持跨线程、延迟调用 |

> [!question] 常见疑问
> **普通 C++ 函数默认不支持反射，必须依靠 QObject体系 + 对应宏，才能被 Qt 元对象系统识别。**