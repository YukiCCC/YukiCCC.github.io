---
title: Csharp串口通信
date: 2026-06-19 23:02:57
tags: Csharp
category: Csharp
---

### 一、串口通信概述

#### 1. 什么是串口通信

串口通信（Serial Communication）是一种**按位顺序传输数据**的通信方式。在嵌入式与工控领域，最常见的实现是 **UART**（Universal Asynchronous Receiver/Transmitter，通用异步收发器）。

PC 端 Csharp 程序通过串口与 MCU、传感器、蓝牙模块、485 转换器、PLC 等设备交换数据，是上位机开发中最基础、最常用的通信手段之一。

#### 2. 典型应用场景

| 场景 | 说明 |
| --- | --- |
| 串口调试 | 打印 MCU 日志、收发调试命令 |
| 固件烧录 | 通过 UART Bootloader 下载程序 |
| 传感器采集 | 读取温湿度、称重、条码枪等数据 |
| 设备配置 | 下发参数、读取设备状态 |
| 工业协议 | Modbus RTU、自定义帧协议 |
| 模块联网 | 串口转以太网/4G 模块透传 |

本博客《MCU基本知识》介绍了 MCU 侧 UART 原理，《网线烧录芯片与Csharp传输实现》讲解了网口传输；本文聚焦 **PC 端 Csharp 串口编程**。

#### 3. 整体架构

```
┌─────────────┐    USB/UART/RS232/RS485    ┌──────────────┐
│  PC 上位机   │  ◄──────────────────────► │  嵌入式设备   │
│ Csharp 程序  │      TX / RX / GND        │  MCU / 模块   │
└─────────────┘                            └──────────────┘
```

### 二、串口参数说明

#### 1. 常用参数

| 参数 | 说明 | 常见值 |
| --- | --- | --- |
| 端口名 | PC 上的串口标识 | `COM1`、`COM3` |
| 波特率（BaudRate） | 每秒传输位数 | 9600、115200、921600 |
| 数据位（DataBits） | 一帧中的数据位数 | 8 |
| 停止位（StopBits） | 帧结束标志 | One（1 位） |
| 校验位（Parity） | 奇偶校验 | None、Even、Odd |
| 流控（Handshake） | 硬件/软件流控 | None |

#### 2. 参数必须一致

通信双方（PC 与设备）的波特率、数据位、停止位、校验位必须**完全一致**，否则会出现乱码、丢字节或完全无法通信。

#### 3. RS232 / RS485 区别

| 类型 | 特点 |
| --- | --- |
| RS232 | 点对点，距离较短，PC 直连常见 |
| RS485 | 差分信号，支持多机总线，需 485 转换器 |

Csharp 侧仍使用 `SerialPort` 操作 COM 口，物理层差异由硬件转换器处理。

### 三、开发环境

#### 1. 命名空间

串口类位于 `System.IO.Ports` 命名空间：

```csharp
using System.IO.Ports;
```

#### 2. 项目引用

- **.NET Framework**：内置 `System.IO.Ports`
- **.NET 6 / .NET 8 桌面项目**：Windows 上通常可直接使用；跨平台或非 Windows 环境需安装 NuGet 包：

```bash
dotnet add package System.IO.Ports
```

#### 3. 创建项目

```bash
dotnet new winforms -n SerialPortDemo
cd SerialPortDemo
```

WinForms 适合快速搭建串口调试助手；控制台项目也可用于协议测试。

### 四、SerialPort 基本用法

#### 1. 创建与配置

```csharp
var serialPort = new SerialPort
{
    PortName = "COM3",
    BaudRate = 115200,
    DataBits = 8,
    StopBits = StopBits.One,
    Parity = Parity.None,
    Handshake = Handshake.None,
    ReadTimeout = 1000,
    WriteTimeout = 1000,
    Encoding = Encoding.UTF8
};
```

#### 2. 打开与关闭

```csharp
try
{
    if (!serialPort.IsOpen)
        serialPort.Open();
}
catch (UnauthorizedAccessException)
{
    // 端口被其他程序占用
}
catch (IOException)
{
    // 端口不存在或设备未连接
}

// 使用完毕
if (serialPort.IsOpen)
    serialPort.Close();

serialPort.Dispose();
```

推荐使用 `using` 或在窗体关闭时确保释放端口。

#### 3. 获取可用串口列表

```csharp
string[] ports = SerialPort.GetPortNames();
Array.Sort(ports);
foreach (var port in ports)
{
    Console.WriteLine(port);
}
```

WinForms 中通常绑定到 `ComboBox` 供用户选择。

### 五、数据发送

#### 1. 发送字符串

```csharp
serialPort.WriteLine("AT\r\n");
```

`WriteLine` 会自动追加换行符（默认 `\n`，可通过 `NewLine` 属性修改）。

#### 2. 发送字节数组

协议通信用字节数组更常见：

```csharp
byte[] frame = { 0x55, 0xAA, 0x01, 0x00, 0x01, 0xFE };
serialPort.Write(frame, 0, frame.Length);
```

#### 3. 发送十六进制字符串（调试常用）

```csharp
public static byte[] HexStringToBytes(string hex)
{
    hex = hex.Replace(" ", "").Replace("-", "");
    var bytes = new byte[hex.Length / 2];
    for (int i = 0; i < bytes.Length; i++)
        bytes[i] = Convert.ToByte(hex.Substring(i * 2, 2), 16);
    return bytes;
}

// 发送 "55 AA 01 00"
serialPort.Write(HexStringToBytes("55 AA 01 00"), 0, 4);
```

### 六、数据接收

#### 1. 同步读取

```csharp
// 读取指定字节数（阻塞，受 ReadTimeout 限制）
byte[] buffer = new byte[64];
int bytesRead = serialPort.Read(buffer, 0, buffer.Length);

// 读取到指定字符（如换行）
string line = serialPort.ReadLine();
```

同步方式适合控制台测试，**不适合 WinForms 主线程**，会造成界面卡死。

#### 2. 事件驱动接收（推荐）

`DataReceived` 在串口缓冲区有数据时触发，运行在**线程池线程**：

```csharp
serialPort.DataReceived += SerialPort_DataReceived;

private void SerialPort_DataReceived(object sender, SerialDataReceivedEventArgs e)
{
    try
    {
        int count = serialPort.BytesToRead;
        byte[] buffer = new byte[count];
        serialPort.Read(buffer, 0, count);
        OnDataReceived(buffer);
    }
    catch (Exception ex)
    {
        // 记录异常，避免事件线程崩溃
    }
}
```

#### 3. 跨线程更新 UI

`DataReceived` 不在 UI 线程，更新界面需 `Invoke`：

```csharp
private void OnDataReceived(byte[] data)
{
    if (txtLog.InvokeRequired)
    {
        txtLog.BeginInvoke(() => OnDataReceived(data));
        return;
    }

    string hex = BitConverter.ToString(data).Replace("-", " ");
    txtLog.AppendText($"[{DateTime.Now:HH:mm:ss}] RX: {hex}{Environment.NewLine}");
}
```

详见本博客《Csharp控件基础知识》中的跨线程章节。

### 七、协议帧处理

#### 1. 粘包与拆包

串口是**字节流**，没有天然消息边界。一次 `DataReceived` 可能收到：

- 半帧数据（拆包）
- 多帧合并（粘包）
- 恰好一帧

因此需要在接收端维护**缓冲区**，按协议解析完整帧。

#### 2. 固定长度帧示例

```csharp
private readonly List<byte> _buffer = new();
private const int FrameLength = 8;

private void AppendAndParse(byte[] incoming)
{
    _buffer.AddRange(incoming);

    while (_buffer.Count >= FrameLength)
    {
        byte[] frame = _buffer.GetRange(0, FrameLength).ToArray();
        _buffer.RemoveRange(0, FrameLength);
        ProcessFrame(frame);
    }
}
```

#### 3. 帧头 + 长度 + 校验示例

```
+--------+--------+----------+---------+-------+
| Magic  | Cmd    | Length   | Payload | CRC   |
| 2 byte | 1 byte | 2 byte   | N byte  | 2 byte|
+--------+--------+----------+---------+-------+
Magic: 0x55AA
```

```csharp
private void TryParseFrames()
{
    int i = 0;
    while (i + 5 <= _buffer.Count)
    {
        if (_buffer[i] != 0x55 || _buffer[i + 1] != 0xAA)
        {
            i++;
            continue;
        }

        int payloadLen = _buffer[i + 3] | (_buffer[i + 4] << 8);
        int totalLen = 5 + payloadLen + 2;
        if (i + totalLen > _buffer.Count)
            break;

        byte[] frame = _buffer.GetRange(i, totalLen).ToArray();
        _buffer.RemoveRange(0, i + totalLen);
        ProcessFrame(frame);
        i = 0;
    }

    if (i > 0)
        _buffer.RemoveRange(0, i);
}
```

#### 4. 文本行协议

调试场景常用 `\r\n` 结尾的文本命令：

```csharp
private readonly StringBuilder _textBuffer = new();

private void AppendTextData(byte[] data)
{
    _textBuffer.Append(Encoding.UTF8.GetString(data));
    string content = _textBuffer.ToString();
    int index;
    while ((index = content.IndexOf("\r\n")) >= 0)
    {
        string line = content[..index];
        ProcessLine(line);
        content = content[(index + 2)..];
    }
    _textBuffer.Clear();
    _textBuffer.Append(content);
}
```

### 八、异步编程方式

#### 1. async/await 读取

```csharp
public async Task<byte[]> ReadAsync(int length, CancellationToken token)
{
    byte[] buffer = new byte[length];
    int offset = 0;

    while (offset < length)
    {
        int read = await serialPort.BaseStream.ReadAsync(
            buffer, offset, length - offset, token);
        if (read == 0)
            throw new EndOfStreamException("串口已关闭");
        offset += read;
    }
    return buffer;
}
```

#### 2. 取消长时间操作

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(3));
try
{
    byte[] response = await ReadAsync(64, cts.Token);
}
catch (OperationCanceledException)
{
    // 超时处理
}
```

异步方式适合控制台工具或后台服务；WinForms 事件驱动 + `BeginInvoke` 同样常用。

### 九、WinForms 串口助手结构

#### 1. 界面元素

| 控件 | 用途 |
| --- | --- |
| ComboBox | 串口、波特率选择 |
| Button | 打开/关闭、发送 |
| TextBox / RichTextBox | 日志显示 |
| CheckBox | 十六进制显示、自动换行 |

可参考《Csharp控件基础知识》中的上位机控件选型。

#### 2. 封装串口服务类

```csharp
public class SerialPortService : IDisposable
{
    private SerialPort? _port;

    public event Action<byte[]>? DataReceived;
    public event Action<string>? ErrorOccurred;

    public bool IsOpen => _port?.IsOpen == true;

    public void Open(string portName, int baudRate)
    {
        Close();
        _port = new SerialPort(portName, baudRate, Parity.None, 8, StopBits.One)
        {
            ReadTimeout = 500,
            WriteTimeout = 500
        };
        _port.DataReceived += OnDataReceived;
        _port.Open();
    }

    public void Send(byte[] data) => _port?.Write(data, 0, data.Length);

    private void OnDataReceived(object sender, SerialDataReceivedEventArgs e)
    {
        try
        {
            int count = _port!.BytesToRead;
            byte[] buf = new byte[count];
            _port.Read(buf, 0, count);
            DataReceived?.Invoke(buf);
        }
        catch (Exception ex)
        {
            ErrorOccurred?.Invoke(ex.Message);
        }
    }

    public void Close()
    {
        if (_port == null) return;
        _port.DataReceived -= OnDataReceived;
        if (_port.IsOpen) _port.Close();
        _port.Dispose();
        _port = null;
    }

    public void Dispose() => Close();
}
```

将 UI 与通信逻辑分离，便于测试和维护。

### 十、Modbus RTU 简介

#### 1. 什么是 Modbus RTU

Modbus RTU 是工业领域常见的串口协议，主站（PC/PLC）轮询从站（传感器/仪表），采用 CRC16 校验。

#### 2. Csharp 实现方式

| 方式 | 说明 |
| --- | --- |
| 手动组帧 | 理解协议，自己拼装功能码、地址、CRC |
| 开源库 | 如 NModbus 等，封装读写寄存器 |

手动发送读保持寄存器（功能码 0x03）示例思路：

1. 构造：地址 + 功能码 + 起始地址 + 寄存器数量
2. 计算 CRC16（低字节在前）
3. `serialPort.Write` 发送
4. 等待并解析应答帧

工业项目建议直接使用成熟库，减少 CRC 与边界处理错误。

### 十一、常见问题与排查

| 现象 | 可能原因 | 处理 |
| --- | --- | --- |
| 打开失败 / 拒绝访问 | 端口被占用 | 关闭其他串口工具、重启设备 |
| 完全无数据 | 线序错、未共地、端口选错 | 检查 TX/RX 交叉、GND 连接 |
| 乱码 | 波特率不一致 | 统一双方参数 |
| 数据不完整 | 缓冲区未及时读取 | 使用 DataReceived，增大读取频率 |
| UI 卡死 | 主线程阻塞 Read | 改用事件或 async |
| 间歇丢包 | 无流控、处理过慢 | 优化解析逻辑、降低发送速率 |
| 发送后无响应 | 时序不对、设备未就绪 | 发送后 Delay，检查设备协议 |

#### 调试建议

1. 用 USB 转串口 + 串口助手先验证硬件
2. PC 端与设备端**分别**打印十六进制收发日志
3. 示波器或逻辑分析仪查看波形（疑难硬件问题）
4. 本博客《网络通讯协议与抓包工具学习》中的分析思路同样适用于帧格式排查

### 十二、最佳实践

#### 1. 资源管理

- 窗体关闭时关闭串口
- 使用 `Dispose` 释放资源
- 避免多处重复 Open 同一端口

#### 2. 线程安全

- `DataReceived` 中不做耗时操作
- UI 更新走 `Invoke` / `BeginInvoke`
- 共享缓冲区加锁或使用专用接收线程

#### 3. 协议设计

- 固定帧头，便于同步
- 带长度字段或固定长度
- 加校验（CRC / 校验和）
- 超时与重试机制

#### 4. 日志

- 记录 TX/RX 十六进制与时间戳
- 异常写入日志文件，便于现场排查

#### 5. 配置持久化

- 保存上次使用的 COM 口、波特率到配置文件
- 启动时自动加载，提升调试效率

### 十三、完整控制台示例

```csharp
using System.IO.Ports;
using System.Text;

class Program
{
    static void Main()
    {
        var ports = SerialPort.GetPortNames();
        if (ports.Length == 0)
        {
            Console.WriteLine("未找到串口");
            return;
        }

        using var sp = new SerialPort(ports[0], 115200)
        {
            Encoding = Encoding.UTF8,
            NewLine = "\r\n"
        };

        sp.DataReceived += (_, _) =>
        {
            string data = sp.ReadExisting();
            Console.WriteLine($"RX: {data}");
        };

        sp.Open();
        Console.WriteLine($"已打开 {sp.PortName}，输入内容回车发送，输入 exit 退出");

        while (true)
        {
            string? input = Console.ReadLine();
            if (input == "exit") break;
            sp.WriteLine(input);
        }
    }
}
```

### 十四、与其他文章的关系

| 文章 | 关联 |
| --- | --- |
| 《MCU基本知识》 | 设备端 UART 原理 |
| 《Csharp控件基础知识》 | 串口助手 UI 与跨线程 |
| 《网线烧录芯片与Csharp传输实现》 | 网口升级；串口常用于本地烧录 |
| 《网络通讯协议与抓包工具学习》 | 协议分析与调试思路 |

串口负责**本地、简单、可靠**的通信；当设备具备网口后，可在此基础上扩展 TCP/OTA 等能力。

### 十五、小结

Csharp 串口通信的核心是 `System.IO.Ports.SerialPort`：

1. 正确配置端口参数，与设备保持一致
2. 用 `DataReceived` 或 async 接收，避免阻塞 UI
3. 维护接收缓冲区，处理粘包拆包
4. 封装服务类，分离 UI 与协议逻辑
5. 重视异常、超时、日志与资源释放

掌握串口编程，是开发调试助手、产线工具、设备配置软件的基础。结合 WinForms 或 WPF 界面，即可快速搭建实用的上位机工具。
