---
title: WPF基础知识
date: 2026-06-08 22:18:46
tags: Csharp
category: Csharp
---

### 一、WPF 简介

#### 1. WPF 是什么

WPF（Windows Presentation Foundation）是微软推出的**桌面 UI 框架**，基于 .NET，用于构建 Windows 桌面应用程序。

它使用 **XAML** 描述界面，使用 **Csharp** 编写业务逻辑，通过强大的**数据绑定**和**样式模板**机制，适合开发现代化、可定制的 Windows 客户端程序。

#### 2. WPF 与 WinForms 的区别

| 对比项 | WinForms | WPF |
| --- | --- | --- |
| UI 描述 | 可视化拖拽 + 代码 | XAML + 代码 |
| 渲染方式 | GDI+ | DirectX（矢量渲染） |
| 数据绑定 | 较弱 | 强大，MVVM 友好 |
| 界面表现力 | 传统控件风格 | 样式、模板、动画丰富 |
| 分辨率适配 | 一般 | 矢量缩放效果更好 |
| 学习曲线 | 较低 | 较高 |

新项目若需要更现代的 UI 和 MVVM 架构，通常优先选择 WPF 或 MAUI。

#### 3. 常见应用场景

- 企业内部管理系统（ERP、MES 客户端）
- 工控上位机、产线配置工具
- 数据可视化与监控软件
- 与硬件设备交互的桌面工具

本博客《网线烧录芯片与Csharp传输实现》《MVVM设计模式》中的上位机场景，WPF 是常见技术选型之一。

### 二、开发环境

#### 1. 环境要求

- Windows 10 / 11
- .NET 6 / .NET 8 SDK（或 .NET Framework 4.x 老项目）
- Visual Studio 2022（安装「.NET 桌面开发」工作负载）

#### 2. 创建项目

```bash
dotnet new wpf -n MyWpfApp
cd MyWpfApp
dotnet run
```

或在 Visual Studio 中：**创建新项目 → WPF 应用**。

#### 3. 项目结构

```
MyWpfApp/
├── App.xaml          # 应用程序入口与全局资源
├── App.xaml.cs
├── MainWindow.xaml   # 主窗口界面
├── MainWindow.xaml.cs
└── MyWpfApp.csproj
```

### 三、XAML 基础

#### 1. XAML 是什么

XAML（eXtensible Application Markup Language）是一种基于 XML 的标记语言，用于声明式描述 UI 布局、样式和资源。

界面与逻辑分离：

- **XAML**：长什么样
- **Csharp**：怎么行为

#### 2. 基本窗口示例

```xml
<Window x:Class="MyWpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="我的 WPF 应用" Height="400" Width="600">
    <Grid>
        <TextBlock Text="Hello WPF" 
                   HorizontalAlignment="Center" 
                   VerticalAlignment="Center"
                   FontSize="24"/>
    </Grid>
</Window>
```

#### 3. 常用布局控件

| 控件 | 用途 |
| --- | --- |
| Grid | 网格布局，最常用 |
| StackPanel | 水平或垂直排列 |
| DockPanel | 停靠布局 |
| WrapPanel | 自动换行排列 |
| Canvas | 绝对坐标定位 |

#### 4. 常用基础控件

| 控件 | 用途 |
| --- | --- |
| Button | 按钮 |
| TextBox | 文本输入 |
| TextBlock | 文本显示 |
| Label | 标签 |
| ComboBox | 下拉选择 |
| ListBox / ListView | 列表 |
| DataGrid | 表格数据 |
| Image | 图片 |

### 四、代码后置与事件

#### 1. 代码后置模式

XAML 中定义控件，在 `.xaml.cs` 中写事件处理：

```xml
<Button Content="点击我" Click="Button_Click"/>
```

```csharp
private void Button_Click(object sender, RoutedEventArgs e)
{
    MessageBox.Show("按钮被点击了");
}
```

适合快速入门，但不利于测试和大型项目维护。

#### 2. 路由事件

WPF 事件支持**冒泡**和**隧道**机制，例如 `PreviewMouseDown`（隧道）、`MouseDown`（冒泡），便于在父容器统一处理子控件事件。

### 五、数据绑定

#### 1. 绑定语法

```xml
<TextBox Text="{Binding Username}" />
<TextBlock Text="{Binding WelcomeMessage}" />
```

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        DataContext = new UserViewModel();
    }
}
```

`DataContext` 即绑定源对象，通常是 ViewModel。

#### 2. 绑定模式

| 模式 | 说明 |
| --- | --- |
| OneWay | 源 → 目标 |
| TwoWay | 源 ↔ 目标 |
| OneTime | 仅初始化时绑定一次 |
| OneWayToSource | 目标 → 源 |

表单输入常用 `TwoWay`：

```xml
<TextBox Text="{Binding Age, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
```

#### 3. INotifyPropertyChanged

ViewModel 属性变化需通知 UI 更新：

```csharp
public class UserViewModel : INotifyPropertyChanged
{
    private string _username = string.Empty;

    public string Username
    {
        get => _username;
        set
        {
            _username = value;
            OnPropertyChanged(nameof(Username));
            OnPropertyChanged(nameof(WelcomeMessage));
        }
    }

    public string WelcomeMessage => $"你好，{Username}";

    public event PropertyChangedEventHandler? PropertyChanged;
    protected void OnPropertyChanged(string name) =>
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
```

### 六、MVVM 模式

#### 1. WPF 与 MVVM

WPF 是 MVVM 最典型的桌面实现框架之一：

| MVVM | WPF |
| --- | --- |
| View | XAML |
| ViewModel | Csharp 类 + INotifyPropertyChanged |
| Model | 实体、服务、API |

详细概念可参考本博客《MVVM设计模式》。

#### 2. Command 命令绑定

按钮点击推荐用 `ICommand`，而非事件：

```csharp
public ICommand SaveCommand { get; }

public UserViewModel()
{
    SaveCommand = new RelayCommand(Save, CanSave);
}

private bool CanSave() => !string.IsNullOrWhiteSpace(Username);
private void Save() => MessageBox.Show($"保存：{Username}");
```

```xml
<Button Content="保存" Command="{Binding SaveCommand}" />
```

可使用 CommunityToolkit.Mvvm 简化 `RelayCommand` 实现。

#### 3. 使用 MVVM 框架（可选）

| 框架 | 特点 |
| --- | --- |
| CommunityToolkit.Mvvm | 轻量，微软官方 |
| Prism | 企业级，模块化、导航强 |
| MVVM Light / GalaSoft | 经典入门框架 |

### 七、样式与模板

#### 1. Style（样式）

统一控件外观：

```xml
<Window.Resources>
    <Style TargetType="Button">
        <Setter Property="Background" Value="#2196F3"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="Padding" Value="10,5"/>
    </Style>
</Window.Resources>
```

#### 2. DataTemplate（数据模板）

自定义列表项或复杂对象的显示方式：

```xml
<DataTemplate x:Key="UserTemplate">
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="{Binding Name}" FontWeight="Bold"/>
        <TextBlock Text="{Binding Email}" Margin="10,0,0,0"/>
    </StackPanel>
</DataTemplate>
```

#### 3. ControlTemplate（控件模板）

彻底改变控件视觉结构，保留原有功能，例如自定义按钮、进度条外观。

WPF 的 UI 定制能力远强于 WinForms，这也是其适合上位机、管理平台的原因之一。

### 八、资源与依赖属性

#### 1. 资源字典

将样式、颜色、模板集中管理：

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ResourceDictionary Source="Themes/Colors.xaml"/>
            <ResourceDictionary Source="Themes/Buttons.xaml"/>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

#### 2. 依赖属性

WPF 控件属性多为**依赖属性**（Dependency Property），支持：

- 数据绑定
- 样式
- 动画
- 值继承

这是 WPF 绑定和动画机制的基础。

### 九、常用功能

#### 1. 数据表格 DataGrid

```xml
<DataGrid ItemsSource="{Binding Users}" AutoGenerateColumns="False">
    <DataGrid.Columns>
        <DataGridTextColumn Header="姓名" Binding="{Binding Name}" />
        <DataGridTextColumn Header="年龄" Binding="{Binding Age}" />
    </DataGrid.Columns>
</DataGrid>
```

#### 2. 值转换器 IValueConverter

界面显示与数据格式不一致时使用：

```csharp
public class BoolToVisibilityConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return (value is bool b && b) ? Visibility.Visible : Visibility.Collapsed;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        => throw new NotImplementedException();
}
```

```xml
<Button Visibility="{Binding IsAdmin, Converter={StaticResource BoolToVisibilityConverter}}" />
```

#### 3. 对话框与消息框

```csharp
MessageBox.Show("操作成功", "提示", MessageBoxButton.OK, MessageBoxImage.Information);

var dialog = new SaveFileDialog { Filter = "文本文件|*.txt" };
if (dialog.ShowDialog() == true)
{
    File.WriteAllText(dialog.FileName, content);
}
```

#### 4. 异步操作

避免阻塞 UI 线程：

```csharp
private async Task LoadDataAsync()
{
    IsLoading = true;
    var data = await api.GetUsersAsync();
    Users = new ObservableCollection<User>(data);
    IsLoading = false;
}
```

### 十、WPF 与 .NET 演进

#### 1. 相关技术对比

| 技术 | 说明 |
| --- | --- |
| WPF | 成熟稳定的 Windows 桌面 UI |
| WinUI 3 | 现代 Windows 原生 UI |
| MAUI | 跨平台（Windows/macOS/移动端） |
| Avalonia | 跨平台 XAML 框架 |

已有大量 Windows 工控、企业项目继续使用 WPF；需要跨平台时考虑 MAUI 或 Avalonia。

#### 2. 选型建议

- 仅 Windows、企业内网、MVVM 架构 → **WPF**
- 新跨平台项目 → **MAUI / Avalonia**
- 简单工具、快速开发 → WinForms 也可

### 十一、学习路径

#### 1. 入门

1. 掌握 Csharp 基础（见《Csharp基础知识》）
2. 学习 XAML 布局与常用控件
3. 理解依赖属性与数据绑定
4. 完成按钮、表单、列表小案例

#### 2. 进阶

1. MVVM 分层与 Command
2. Style、Template、ResourceDictionary
3. ValueConverter、Behavior
4. 异步加载与异常处理
5. 日志、配置、串口/网口通讯（结合上位机场景）

#### 3. 实战项目建议

- 串口调试助手
- 设备参数配置工具
- 数据采集与曲线显示
- Modbus / TCP 通讯上位机

### 十二、常见问题

| 现象 | 可能原因 | 处理 |
| --- | --- | --- |
| 绑定不生效 | DataContext 未设置、属性未实现通知 | 检查 DataContext 与 INotifyPropertyChanged |
| 界面卡顿 | UI 线程执行耗时操作 | 使用 async/await |
| 中文乱码 | 编码或字体问题 | 统一 UTF-8，设置合适字体 |
| 样式不应用 | TargetType 不匹配 | 检查 Style 目标类型 |
| 启动失败 | 目标框架或依赖缺失 | 检查 .NET 运行时 |

### 十三、小结

WPF 是构建 Windows 桌面应用的主流框架，核心优势在于 **XAML + 数据绑定 + MVVM + 模板化 UI**。

学习路线建议：

1. Csharp 语言基础
2. XAML 与布局控件
3. 数据绑定与 MVVM
4. 样式模板与实战项目

结合本博客《Csharp基础知识》《MVVM设计模式》《网线烧录芯片与Csharp传输实现》，可以从语言、架构到上位机场景形成较完整的 WPF 开发知识链。
