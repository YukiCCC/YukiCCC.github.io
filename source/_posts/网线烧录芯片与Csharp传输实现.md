---
title: 网线烧录芯片与Csharp传输实现
date: 2026-05-30 17:00:00
tags: Csharp
category: Csharp
---

### 一、概述

#### 1. 为什么用网线烧录芯片

传统烧录方式多依赖 JTAG、SWD、UART 或 USB 下载器，需要物理靠近设备、逐台连接。当芯片或产品板已具备**以太网接口**（或通过串口转以太网模块联网）时，可以通过网线完成：

- **固件传输**：把 `.bin` / `.hex` 文件从 PC 发到设备
- **在线烧录**：设备 Bootloader 接收数据并写入 Flash
- **批量升级**：产线、现场设备远程维护

典型场景：工业网关、STM32+W5500 方案、带网口的 PLC、物联网网关批量升级。

#### 2. 整体架构

```
┌─────────────┐      网线(TCP/IP)      ┌──────────────────┐
│  PC 上位机   │  ──────────────────►  │  目标芯片/设备    │
│ Csharp 工具  │  ◄──────────────────  │  Bootloader      │
└─────────────┘      应答/进度/校验      └────────┬─────────┘
                                                  │ 写 Flash
                                                  ▼
                                           ┌──────────────┐
                                           │  程序存储区   │
                                           └──────────────┘
```

PC 端负责读固件、分包、校验、重传；设备端 Bootloader 负责擦除、写入、校验、重启。

### 二、常见实现方式

#### 1. 自定义 TCP 协议（本文重点）

在应用层定义简单命令帧，通过 TCP 长连接传输固件块。优点是协议可控、Csharp 实现直观，适合自有 Bootloader。

#### 2. TFTP / FTP

部分 Bootloader（如 U-Boot）支持 TFTP 拉取固件。PC 端可搭建 TFTP 服务器，设备主动下载。Csharp 可调用现有 TFTP 库或配合第三方工具。

#### 3. HTTP OTA

设备作为 HTTP Client 从服务器下载固件，或通过 HTTP POST 上传。适合已有 Web 能力的模块（如 ESP32、Linux 嵌入式）。

#### 4. 与 JTAG 远程调试的区别

JTAG over IP 多用于**调试**，烧录仍常走专用工具链。本文侧重**产品化固件升级**：业务协议 + Bootloader + 上位机工具。

### 三、协议设计示例

#### 1. 设计原则

- 固定帧头，便于 Bootloader 识别
- 每条命令带序号，支持 ACK/NACK
- 数据块带 CRC32，防止传输错误写入 Flash
- 分阶段：握手 → 擦除 → 传输 → 校验 → 复位

#### 2. 命令定义

| 命令 | 值 | 说明 |
| --- | --- | --- |
| HELLO | 0x01 | 握手，交换版本与 Flash 容量 |
| ERASE | 0x02 | 擦除目标区域 |
| DATA | 0x03 | 传输固件数据块 |
| VERIFY | 0x04 | 整包 CRC 校验 |
| RESET | 0x05 | 复位并跳转 App |
| ACK | 0x80 | 成功应答 |
| NACK | 0x81 | 失败应答 |

#### 3. DATA 帧结构（示例）

```
+--------+--------+----------+----------+---------+----------+
| Magic  | Cmd    | Seq      | Offset   | Length  | Payload  | CRC32 |
| 2 byte | 1 byte | 2 byte   | 4 byte   | 2 byte  | N byte   | 4 byte|
+--------+--------+----------+----------+---------+----------+
Magic: 0x55AA
```

块大小常用 512 / 1024 / 4096 字节，需与 MCU Flash 页大小对齐。

### 四、Csharp 上位机实现

#### 1. 环境准备

- .NET 6 或 .NET 8 SDK
- 创建控制台项目：

```bash
dotnet new console -n FirmwareNetFlasher
cd FirmwareNetFlasher
```

#### 2. 固件读取与 CRC 计算

```csharp
using System;
using System.IO;
using System.IO.Hashing;

public static class FirmwareFileHelper
{
    public static byte[] ReadFirmware(string path)
    {
        if (!File.Exists(path))
            throw new FileNotFoundException("固件文件不存在", path);

        return File.ReadAllBytes(path);
    }

    public static uint CalcCrc32(byte[] data)
    {
        return BitConverter.ToUInt32(Crc32.Hash(data), 0);
    }
}
```

#### 3. 协议帧封装

```csharp
using System;
using System.Buffers.Binary;

public enum BootCommand : byte
{
    Hello = 0x01,
    Erase = 0x02,
    Data = 0x03,
    Verify = 0x04,
    Reset = 0x05,
    Ack = 0x80,
    Nack = 0x81
}

public static class BootFrameBuilder
{
    private const ushort Magic = 0x55AA;

    public static byte[] BuildDataFrame(ushort seq, uint offset, ReadOnlySpan<byte> payload)
    {
        int frameLen = 2 + 1 + 2 + 4 + 2 + payload.Length + 4;
        byte[] frame = new byte[frameLen];
        int index = 0;

        BinaryPrimitives.WriteUInt16BigEndian(frame.AsSpan(index, 2), Magic);
        index += 2;
        frame[index++] = (byte)BootCommand.Data;
        BinaryPrimitives.WriteUInt16BigEndian(frame.AsSpan(index, 2), seq);
        index += 2;
        BinaryPrimitives.WriteUInt32BigEndian(frame.AsSpan(index, 4), offset);
        index += 4;
        BinaryPrimitives.WriteUInt16BigEndian(frame.AsSpan(index, 2), (ushort)payload.Length);
        index += 2;
        payload.CopyTo(frame.AsSpan(index, payload.Length));
        index += payload.Length;

        uint crc = BitConverter.ToUInt32(System.IO.Hashing.Crc32.Hash(frame.AsSpan(0, index)), 0);
        BinaryPrimitives.WriteUInt32BigEndian(frame.AsSpan(index, 4), crc);
        return frame;
    }
}
```

#### 4. TCP 连接与发送

```csharp
using System;
using System.Net.Sockets;
using System.Threading;
using System.Threading.Tasks;

public class NetFlasherClient : IAsyncDisposable
{
    private readonly TcpClient _client = new();
    private NetworkStream? _stream;

    public async Task ConnectAsync(string ip, int port, CancellationToken token = default)
    {
        await _client.ConnectAsync(ip, port, token);
        _stream = _client.GetStream();
        _client.ReceiveTimeout = 5000;
        _client.SendTimeout = 5000;
    }

    public async Task SendFrameAsync(byte[] frame, CancellationToken token = default)
    {
        if (_stream == null)
            throw new InvalidOperationException("尚未连接设备");

        await _stream.WriteAsync(frame, token);
        await _stream.FlushAsync(token);
    }

    public async Task<byte[]> ReceiveFrameAsync(int maxLength = 256, CancellationToken token = default)
    {
        if (_stream == null)
            throw new InvalidOperationException("尚未连接设备");

        byte[] buffer = new byte[maxLength];
        int read = await _stream.ReadAsync(buffer, token);
        if (read <= 0)
            throw new IOException("设备连接已断开");

        byte[] result = new byte[read];
        Array.Copy(buffer, result, read);
        return result;
    }

    public async ValueTask DisposeAsync()
    {
        _stream?.Dispose();
        _client.Close();
        await Task.CompletedTask;
    }
}
```

#### 5. 完整烧录流程

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class FirmwareFlasher
{
    private const int ChunkSize = 1024;
    private readonly NetFlasherClient _client = new();

    public async Task FlashAsync(string ip, int port, string firmwarePath, IProgress<int>? progress = null)
    {
        byte[] firmware = FirmwareFileHelper.ReadFirmware(firmwarePath);
        uint totalCrc = FirmwareFileHelper.CalcCrc32(firmware);

        await _client.ConnectAsync(ip, port);

        // 1. 握手
        await SendHelloAsync();

        // 2. 擦除
        await SendEraseAsync((uint)firmware.Length);

        // 3. 分包写入
        ushort seq = 0;
        for (int offset = 0; offset < firmware.Length; offset += ChunkSize)
        {
            int len = Math.Min(ChunkSize, firmware.Length - offset);
            byte[] chunk = new byte[len];
            Array.Copy(firmware, offset, chunk, 0, len);

            byte[] frame = BootFrameBuilder.BuildDataFrame(seq, (uint)offset, chunk);
            await SendWithRetryAsync(frame);

            seq++;
            progress?.Report((offset + len) * 100 / firmware.Length);
        }

        // 4. 校验
        await SendVerifyAsync(totalCrc);

        // 5. 复位运行
        await SendResetAsync();

        await _client.DisposeAsync();
        Console.WriteLine("烧录完成");
    }

    private async Task SendWithRetryAsync(byte[] frame, int maxRetry = 3)
    {
        for (int i = 0; i < maxRetry; i++)
        {
            await _client.SendFrameAsync(frame);
            byte[] resp = await _client.ReceiveFrameAsync();
            if (resp.Length > 0 && resp[0] == (byte)BootCommand.Ack)
                return;
        }
        throw new Exception("设备应答失败，烧录中断");
    }

    private Task SendHelloAsync() => Task.CompletedTask;
    private Task SendEraseAsync(uint size) => Task.CompletedTask;
    private Task SendVerifyAsync(uint crc) => Task.CompletedTask;
    private Task SendResetAsync() => Task.CompletedTask;
}
```

> 上述 `SendHelloAsync` 等方法需按实际协议补全帧内容；核心流程为：**连接 → 擦除 → 分包发送 → 校验 → 复位**。

#### 6. 程序入口

```csharp
class Program
{
    static async Task Main(string[] args)
    {
        string ip = args.Length > 0 ? args[0] : "192.168.1.100";
        int port = args.Length > 1 ? int.Parse(args[1]) : 9000;
        string firmware = args.Length > 2 ? args[2] : "app.bin";

        var progress = new Progress<int>(p => Console.Write($"\r进度: {p}%"));
        var flasher = new FirmwareFlasher();

        try
        {
            await flasher.FlashAsync(ip, port, firmware, progress);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"烧录失败: {ex.Message}");
        }
    }
}
```

运行示例：

```bash
dotnet run -- 192.168.1.100 9000 app.bin
```

### 五、设备端 Bootloader 要点

设备端通常用 C 语言编写（与芯片 SDK 配套），PC 端用 Csharp。Bootloader 需实现：

1. **上电判断**：按键/GPIO/标志位进入升级模式
2. **TCP 监听**：绑定固定端口，等待 PC 连接
3. **Flash 操作**：按页擦除、按地址写入
4. **应答机制**：每收到 DATA 帧校验 CRC 后回 ACK/NACK
5. **跳转 App**：烧录完成后关闭中断，设置栈指针，跳转到应用程序入口

Bootloader 占用空间要小，常放在 Flash 起始扇区；App 从偏移地址运行，链接脚本需与 PC 端固件地址一致。

### 六、硬件与网络配置

#### 1. 典型硬件组合

| 方案 | 说明 |
| --- | --- |
| MCU + 以太网 PHY | 如 STM32 + LAN8720，Bootloader 内实现 TCP/IP 协议栈（lwIP） |
| MCU + W5500 | 硬件 TCP/IP，减轻 MCU 负担 |
| 串口转以太网模块 | 模块网口接 PC，串口接目标 MCU，PC 仍用 TCP 连接模块 |
| 带网口的 Linux 板 | 可用 Csharp 工具 + SSH/脚本配合，或直接 socket 写 MCU |

#### 2. 网络设置

- PC 与设备处于同一网段，如 PC `192.168.1.10`，设备 `192.168.1.100`
- 先用 `ping` 确认连通
- 确认防火墙未拦截目标端口（如 9000）
- 产线环境建议固定 IP，避免 DHCP 变化导致烧录失败

### 七、调试与排错

#### 1. Wireshark 抓包

过滤条件：

```
tcp.port == 9000 && ip.addr == 192.168.1.100
```

可观察：TCP 三次握手是否成功、每包 DATA 是否有 ACK、是否存在重传。

#### 2. 常见问题

| 现象 | 可能原因 | 处理 |
| --- | --- | --- |
| 连接超时 | IP/端口错误、设备未进 Bootloader | Ping + 确认监听端口 |
| 写入后无法启动 | 链接地址错误、向量表偏移不对 | 核对 Flash 布局与 hex/bin 起始地址 |
| CRC 校验失败 | 大小端不一致、块长度不对齐 | 统一 BigEndian/LittleEndian |
| 中途断开 | 包过大、看门狗复位 | 缩小 ChunkSize、喂狗 |

#### 3. 日志建议

Csharp 上位机建议记录：时间戳、序号、偏移、重试次数、每阶段耗时。产线问题可追溯具体哪一台、哪一包出错。

### 八、安全与量产建议

- **版本校验**：握手阶段比对硬件版本号，防止错刷固件
- **数字签名**：Bootloader 验证固件签名后再写入
- **断电保护**：双区备份（A/B 分区），写入完成并校验通过再切换启动区
- **权限控制**：限制仅内网网段可连接烧录端口
- **批量脚本**：Csharp 工具配合 CSV 设备列表，循环 IP 批量升级

### 九、小结

通过网线烧录芯片的本质，是 **TCP/IP 传输 + Bootloader 写 Flash + PC 上位机工具** 三者配合。Csharp 适合编写 Windows 工控机上的烧录软件：文件读取、Socket 通讯、进度界面、日志记录都很方便。

学习与实践路径：

1. 先用 Netcat 或简易 TCP 工具验证设备端口可达
2. 定义并文档化 Bootloader 协议
3. 用 Csharp 实现分包发送与 CRC 校验
4. 配合 Wireshark 联调
5. 再在 WinForms / WPF 中封装为可视化烧录工具

与《网络通讯协议与抓包工具学习》结合，可以从协议分层到实际上位机开发形成完整链路。
