---
title: C#基础知识
date: 2026-05-30 14:30:00
tags: C#
category: C#
---

## 一、C#简介

### 1. C#是什么？

​	C#（读作 C Sharp）是微软推出的一种**面向对象**的编程语言，运行于 .NET 平台之上。

​	由 Anders Hejlsberg 主导设计，语法风格与 Java、C++ 相近，学习 Java 后再学 C# 上手较快。

​	官网：https://dotnet.microsoft.com/

### 2. .NET 与 C# 的关系

- **.NET**：跨平台的开发框架，提供运行时、类库和工具链
- **CLR**（Common Language Runtime）：公共语言运行时，负责内存管理、垃圾回收、类型安全等
- **.NET SDK**：开发工具包，包含编译器、运行时、NuGet 包管理器等
- **C#**：运行在 .NET 上的一种编程语言

关系可以理解为：C# 是语言，.NET 是运行和开发平台，二者配合完成应用开发。

### 3. 为什么学习 C#

- 与 Windows 桌面、ASP.NET Web、Unity 游戏开发等场景结合紧密
- 语法现代，持续演进（如 LINQ、async/await、模式匹配、记录类型等）
- .NET 已支持跨平台，可在 Windows、Linux、macOS 上运行
- 企业级开发、云服务（Azure）生态完善

## 二、开发环境

### 1. 环境要求

- .NET SDK 6.0 或 8.0 及以上
- 推荐 IDE：Visual Studio、Visual Studio Code（安装 C# 扩展）、Rider

### 2. 安装与验证

从官网下载并安装 .NET SDK 后，在终端执行：

```bash
dotnet --version
```

### 3. 创建第一个项目

```bash
dotnet new console -n HelloCSharp
cd HelloCSharp
dotnet run
```

## 三、第一个 C# 程序

```csharp
using System;

namespace HelloCSharp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, C#!");
        }
    }
}
```

说明：

- `using System;`：引入命名空间
- `namespace`：命名空间，用于组织代码
- `class Program`：定义类
- `static void Main`：程序入口方法
- `Console.WriteLine`：向控制台输出内容

## 四、基本语法

### 1. 注释

```csharp
// 单行注释

/*
  多行注释
*/

/// <summary>
/// XML 文档注释，可用于生成 API 文档
/// </summary>
```

### 2. 变量与常量

```csharp
int age = 18;              // 整型变量
double price = 99.9;       // 双精度浮点
string name = "YukiCCC";   // 字符串
bool isOk = true;          // 布尔值
var count = 10;              // 隐式类型推断（编译期确定类型）

const double PI = 3.14159; // 常量，不可修改
```

### 3. 常见数据类型

| 类型 | 说明 | 示例 |
| --- | --- | --- |
| `int` | 32 位整数 | `int a = 10;` |
| `long` | 64 位整数 | `long b = 100L;` |
| `float` | 单精度浮点 | `float c = 3.14f;` |
| `double` | 双精度浮点 | `double d = 3.14;` |
| `decimal` | 高精度小数 | `decimal money = 99.99m;` |
| `char` | 单个字符 | `char ch = 'A';` |
| `bool` | 布尔值 | `bool flag = false;` |
| `string` | 字符串 | `string s = "hello";` |

### 4. 值类型与引用类型

- **值类型**：直接存储数据，如 `int`、`double`、`struct`、`enum`，赋值时复制一份
- **引用类型**：存储对象的引用，如 `class`、`string`、数组、接口，赋值时复制引用

### 5. 字符串常用操作

```csharp
string s1 = "Hello";
string s2 = "World";
string s3 = s1 + " " + s2;           // 拼接
string s4 = $"{s1} {s2}";              // 字符串插值
int len = s1.Length;                   // 长度
bool eq = s1.Equals("hello");        // 比较
string sub = "Hello C#".Substring(0, 5); // 截取
```

## 五、运算符

### 1. 算术运算符

`+`、`-`、`*`、`/`、`%`（取余）、`++`、`--`

### 2. 关系运算符

`==`、`!=`、`>`、`<`、`>=`、`<=`

### 3. 逻辑运算符

`&&`（与）、`||`（或）、`!`（非）

### 4. 其他常用运算符

- `??`：空合并运算符
- `?.`：空条件运算符
- `??=`：空合并赋值

```csharp
string? name = null;
string display = name ?? "匿名";   // 若 name 为 null，则使用 "匿名"

int? len = name?.Length;           // 若 name 为 null，则 len 为 null
```

## 六、流程控制

### 1. 条件语句

```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("优秀");
}
else if (score >= 60)
{
    Console.WriteLine("及格");
}
else
{
    Console.WriteLine("不及格");
}
```

### 2. switch 语句

```csharp
int day = 3;

switch (day)
{
    case 1:
        Console.WriteLine("星期一");
        break;
    case 2:
        Console.WriteLine("星期二");
        break;
    default:
        Console.WriteLine("其他");
        break;
}
```

C# 还支持 **switch 表达式**：

```csharp
string result = day switch
{
    1 => "星期一",
    2 => "星期二",
    _ => "其他"
};
```

### 3. 循环语句

```csharp
// for 循环
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);
}

// while 循环
int j = 0;
while (j < 5)
{
    Console.WriteLine(j);
    j++;
}

// foreach 循环
int[] nums = { 1, 2, 3, 4, 5 };
foreach (int n in nums)
{
    Console.WriteLine(n);
}
```

## 七、数组

```csharp
// 一维数组
int[] arr = new int[3] { 1, 2, 3 };
int[] arr2 = { 10, 20, 30 };

// 二维数组
int[,] matrix = {
    { 1, 2 },
    { 3, 4 }
};

// 交错数组
int[][] jagged = new int[2][];
jagged[0] = new int[] { 1, 2 };
jagged[1] = new int[] { 3, 4, 5 };
```

## 八、面向对象

### 1. 三大特性

- **封装**：将数据和方法包装在类中，通过访问修饰符控制可见性
- **继承**：子类继承父类的属性和方法
- **多态**：同一方法在不同对象上表现出不同行为

### 2. 访问修饰符

| 修饰符 | 说明 |
| --- | --- |
| `public` | 任何地方都可访问 |
| `private` | 仅当前类内部可访问 |
| `protected` | 当前类及子类可访问 |
| `internal` | 当前程序集内可访问 |

### 3. 定义类与对象

```csharp
public class Person
{
    // 字段
    private string name;

    // 属性（推荐写法）
    public string Name
    {
        get { return name; }
        set { name = value; }
    }

    // 自动属性
    public int Age { get; set; }

    // 构造函数
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }

    // 方法
    public void SayHello()
    {
        Console.WriteLine($"你好，我是 {Name}，今年 {Age} 岁");
    }
}

// 使用
Person p = new Person("张三", 20);
p.SayHello();
```

### 4. 继承

```csharp
public class Student : Person
{
    public string School { get; set; }

    public Student(string name, int age, string school) : base(name, age)
    {
        School = school;
    }

    public override void SayHello()
    {
        Console.WriteLine($"我是学生 {Name}，来自 {School}");
    }
}
```

- `base`：访问父类成员
- `override`：重写父类虚方法
- `virtual`：声明可被重写的方法

### 5. 抽象类与接口

**抽象类**：不能实例化，可包含抽象方法和具体方法

```csharp
public abstract class Animal
{
    public abstract void Speak();

    public void Eat()
    {
        Console.WriteLine("动物在吃东西");
    }
}
```

**接口**：定义行为规范，类可以实现多个接口

```csharp
public interface IFlyable
{
    void Fly();
}

public class Bird : Animal, IFlyable
{
    public override void Speak()
    {
        Console.WriteLine("鸟在叫");
    }

    public void Fly()
    {
        Console.WriteLine("鸟在飞");
    }
}
```

### 6. static 关键字

```csharp
public class MathHelper
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}

// 通过类名直接调用
int sum = MathHelper.Add(1, 2);
```

## 九、常用集合

### 1. List<T>

```csharp
using System.Collections.Generic;

List<string> list = new List<string>();
list.Add("Java");
list.Add("C#");
list.Add("Python");

foreach (string item in list)
{
    Console.WriteLine(item);
}

list.Remove("Java");
Console.WriteLine(list.Count);
```

### 2. Dictionary<TKey, TValue>

```csharp
Dictionary<int, string> map = new Dictionary<int, string>();
map.Add(1, "张三");
map[2] = "李四";

foreach (var kv in map)
{
    Console.WriteLine($"key:{kv.Key}, value:{kv.Value}");
}
```

### 3. 与 Java 集合对照

| C# | Java |
| --- | --- |
| `List<T>` | `ArrayList` / `List` |
| `Dictionary<K,V>` | `HashMap` |
| `HashSet<T>` | `HashSet` |
| `Queue<T>` | `Queue` |
| `Stack<T>` | `Stack` |

## 十、异常处理

```csharp
try
{
    int a = 10;
    int b = 0;
    int c = a / b;
}
catch (DivideByZeroException ex)
{
    Console.WriteLine("除数不能为 0：" + ex.Message);
}
catch (Exception ex)
{
    Console.WriteLine("发生异常：" + ex.Message);
}
finally
{
    Console.WriteLine("无论是否异常都会执行");
}
```

自定义异常：

```csharp
public class BusinessException : Exception
{
    public BusinessException(string message) : base(message)
    {
    }
}
```

## 十一、LINQ 简介

LINQ（Language Integrated Query）是 C# 中用于**查询集合数据**的语法，风格类似 SQL。

```csharp
using System.Linq;

List<int> nums = new List<int> { 1, 2, 3, 4, 5, 6 };

// 查询偶数并排序
var result = nums
    .Where(n => n % 2 == 0)
    .OrderByDescending(n => n)
    .ToList();

foreach (int n in result)
{
    Console.WriteLine(n);
}
```

常用 LINQ 方法：

- `Where`：筛选
- `Select`：投影/转换
- `OrderBy` / `OrderByDescending`：排序
- `First` / `FirstOrDefault`：取第一个元素
- `Count` / `Sum` / `Max` / `Min`：聚合统计

## 十二、异步编程

C# 使用 `async` 和 `await` 简化异步编程，避免阻塞主线程。

```csharp
using System.Net.Http;
using System.Threading.Tasks;

public class Demo
{
    public static async Task Main(string[] args)
    {
        string content = await GetPageContentAsync("https://example.com");
        Console.WriteLine(content);
    }

    public static async Task<string> GetPageContentAsync(string url)
    {
        using HttpClient client = new HttpClient();
        return await client.GetStringAsync(url);
    }
}
```

说明：

- `async`：标记异步方法
- `await`：等待异步任务完成
- `Task` / `Task<T>`：表示异步操作

## 十三、命名空间与 using

```csharp
namespace MyApp.Services
{
    using System;
    using System.Collections.Generic;

    public class UserService
    {
        public void Show()
        {
            Console.WriteLine("UserService");
        }
    }
}
```

- 命名空间用于避免类名冲突
- `using` 引入命名空间，减少全限定名书写
- .NET 6+ 支持**文件范围命名空间**和**顶级语句**，可简化小程序结构

## 十四、小结

C# 是一门语法清晰、生态完善的现代语言。学习时可按以下路径循序渐进：

1. 掌握基本语法、数据类型和流程控制
2. 理解面向对象：类、继承、多态、接口
3. 熟悉常用集合与异常处理
4. 学习 LINQ 与 async/await 等 C# 特色能力
5. 结合具体方向深入：ASP.NET Web、WPF/WinForms 桌面、Unity 游戏等

与 Java 相比，C# 在属性语法、LINQ、异步模型等方面更加简洁；有 Java 基础后转 C#，重点放在 .NET 生态和语言特性差异上即可快速上手。
