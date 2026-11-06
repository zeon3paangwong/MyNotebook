# Qt 串口编程笔记

## 1. 串口通信基础

### 1.1 串口通信线
- 标准串口（如 RS-232）通常有 **9 根通信线**（DB9 接口）。
- **必不可少**的三根线：
  - **GND**：信号地线，提供参考电平。
  - **TXD**：发送数据线。
  - **RXD**：接收数据线。
- 其余 6 根线与 **流控制（Flow Control）** 相关（如 RTS、CTS、DSR、DTR、DCD、RI 等）。

### 1.2 物理层：UART 帧格式
串口通信在物理层采用 UART 帧格式传输数据，每一帧按时序排列包含以下部分：

1. **起始位**：1 位，低电平，表示一帧的开始。
2. **数据位**：通常为 8 位（也可配置为 5~9 位），传输时从 **最低有效位（LSB）** 开始，依次传输到 **最高有效位（MSB）**。
   > [!note] 原文中提到“6位”可能为笔误，实际数据位长度可配置，最常见为 8 位。
3. **奇偶校验位**：可选，用于简单的错误检测（如奇校验、偶校验、无校验等）。
4. **停止位**：1 位、1.5 位或 2 位，高电平，表示一帧的结束。
5. **空闲位**：线路保持高电平，表示当前无数据传输。

### 1.3 波特率（Baud Rate）
- 波特率是指 **每秒传输的二进制位数**（单位：bps，bits per second），是衡量串口通信速率的重要指标。
- 常见波特率：9600、19200、115200 等。
> [!tip] 注意：通信双方的波特率必须一致，否则会导致数据解析错误。

---

## 2. 流控制（Flow Control）

### 2.1 为什么需要流控？
串口通信两端的速度天然不对等：
- 发送方可以一口气将高速数据灌入线路（例如以 115200 bps 的速率）。
- 接收方可能因为正在处理其他任务（如写 SD 卡、刷新 UI、等待数据库响应等）而无法及时从内核缓冲区取走数据。
- 接收方的缓冲区是有限的，**当缓冲区满后，后续到达的数据会直接丢失**——没有报错、没有重传、没有任何痕迹。

### 2.2 流控的作用
流控就是为这个“生产者太快、消费者太慢”的场景装一个 **刹车**：
- 接收方能通过某种机制通知发送方“暂停发送”或“可以继续发送”。
- 从而避免数据丢失，保证通信可靠性。

### 2.3 Qt 中的流控设置
在 Qt 的 `QSerialPort` 类中，流控通过 `FlowControl` 属性（`QSerialPort::FlowControl` 枚举）设置：

| 枚举值 | 说明 |
|--------|------|
| `QSerialPort::NoFlowControl` | **不使用流控**。Qt 既不检查对方是否允许发送，也不向对方发出暂停请求，`write()` 的数据一律直接推向驱动。 |
| `QSerialPort::HardwareControl` | 硬件流控（使用 RTS/CTS 信号线）。 |
| `QSerialPort::SoftwareControl` | 软件流控（使用 XON/XOFF 特殊字符）。 |
| `QSerialPort::UnknownFlowControl` | 未知流控（通常用于错误状态）。 |

---

## 3. 应用层与物理层的关系
- **应用层（消息帧）**：用户定义的数据结构或协议帧（如 Modbus 帧、自定义数据包），通常包含多个字节。
- **物理层（UART 帧）**：每个字节在物理线路上传输时，都会被封装成一个 UART 帧（包括起始位、数据位等）。
- **组合关系**：应用层的一个消息帧由多个字节组成，每个字节通过一次 UART 帧传输，因此 **应用层消息帧由多个物理层 UART 帧组合而成**。

---

## 4. Qt 串口编程简要步骤

使用 Qt 的 `QSerialPort` 类进行串口通信的基本流程：

1. **创建 `QSerialPort` 对象**，并设置通信参数：
   - 端口名（如 `"COM1"` 或 `"/dev/ttyS0"`）
   - 波特率（如 `QSerialPort::Baud115200`）
   - 数据位（如 `QSerialPort::Data8`）
   - 奇偶校验（如 `QSerialPort::NoParity`）
   - 停止位（如 `QSerialPort::OneStop`）
   - 流控（如 `QSerialPort::NoFlowControl`）

2. **打开串口**：`open(QIODevice::ReadWrite)`。

3. **读写数据**：
   - 写入：`write(const char *data, qint64 maxSize)` 或 `write(const QByteArray &data)`
   - 读取：`read(qint64 maxSize)`、`readAll()` 等，通常配合 `readyRead()` 信号使用。

4. **关闭串口**：`close()`。
---

## 5. 总结
- 串口通信基础：3 根必需线（GND、TXD、RXD），UART 帧格式（起始位、数据位 LSB→MSB、校验位、停止位），波特率。
- 流控是关键机制，防止接收端缓冲区溢出导致数据丢失，Qt 中可通过 `setFlowControl()` 配置。
- 应用层消息由多个物理层 UART 帧组装。
- Qt 提供了 `QSerialPort` 类简化串口编程，结合信号槽机制可高效处理异步通信。

---

#Qt #串口通信 #UART #流控制 #QSerialPort
### 完整示例代码
# include <QSerialPort>

# include <QDebug>

// 创建并配置串口

QSerialPort serial;

serial.setPortName("COM1");

serial.setBaudRate(QSerialPort::Baud115200);

serial.setDataBits(QSerialPort::Data8);

serial.setParity(QSerialPort::NoParity);

serial.setStopBits(QSerialPort::OneStop);

serial.setFlowControl(QSerialPort::NoFlowControl);

// 打开串口

if (!serial.open(QIODevice::ReadWrite)) {

qDebug() << "Failed to open port:" << serial.errorString();

return;

}

// 写入数据

serial.write("Hello Serial Port");

// 连接信号槽，异步读取数据

QObject::connect(&serial, &QSerialPort::readyRead, & {

QByteArray data = serial.readAll();

qDebug() << "Received:" << data;

});

// ... 在适当时候关闭串口

// serial.close();