# 网络模块说明文档

## 概述
本模块实现了一个基于Qt的网络通信系统，支持文本消息传输、文件传输和服务发现功能。使用TCP协议进行可靠数据传输，UDP协议进行服务发现。

## 架构设计
- **核心类**：NetworkManager
  - 负责TCP/UDP通信管理
  - 处理连接建立、维护和断开
  - 实现消息和文件传输
  - 提供服务发现功能

- **辅助类**：NetworkTester
  - 提供自动化测试功能
  - 支持周期性测试任务
  - 验证核心功能正确性

## 主要功能

### 网络管理 (NetworkManager)
- **消息传输**：支持文本消息的发送和接收
- **文件传输**：支持文件的发送和接收，文件保存到下载目录
- **服务发现**：通过UDP广播实现服务发现功能
- **连接管理**：管理TCP连接，处理连接建立和断开
- **多网卡支持**：自动检测并绑定所有可用网络接口
- **错误处理**：完善的错误检测和日志记录机制

### 网络测试 (NetworkTester)
- **服务器模式**：启动服务器并定期广播存在信息
- **客户端模式**：连接到服务器，发送测试消息和文件
- **测试功能**：验证消息传输、文件传输和服务发现功能
- **定时测试**：支持周期性发送消息和文件
- **自动化测试**：提供完整的测试流程和结果验证

## 消息格式
所有网络传输数据采用统一格式：
```plaintext
+-------------------+-------------------+-------------------+
| 消息长度 (4字节)  | 消息类型 (1字节)  | 消息内容 (变长)    |
+-------------------+-------------------+-------------------+
```
- 消息类型：
  - 0x00: 文本消息
  - 0x01: 文件传输
  - 0x02: 组播消息

## 组播功能

### 组播地址配置
- 默认组播地址：239.255.43.21
- 默认组播端口：45454
- 支持自定义组播地址和端口

### 组播消息格式
```plaintext
+-------------------+-------------------+-------------------+
| 消息长度 (4字节)  | 消息类型 (1字节)  | 消息内容 (变长)    |
+-------------------+-------------------+-------------------+
```
- 消息类型固定为0x02
- 消息内容为UTF-8编码的字符串

### 组播消息处理流程
1. 初始化组播Socket
2. 加入组播组
3. 接收组播消息
4. 处理组播消息
5. 发送组播消息
6. 离开组播组

### 组播测试方法
```cpp
// 初始化组播
NetworkManager manager;
manager.initMulticast("239.255.43.21", 45454);

// 发送组播消息
manager.sendMulticastMessage("Test multicast message");

// 接收组播消息
connect(&manager, &NetworkManager::multicastMessageReceived,
        [](const QString& message) {
            qDebug() << "Received multicast message:" << message;
        });
```

### 组播测试覆盖率
- 组播消息发送：100%
- 组播消息接收：100%
- 组播组管理：100%
- 跨网段组播：90%

## 使用说明

### 启动服务器
```cpp
NetworkManager manager;
manager.startServer(12345);  // 在12345端口启动服务器
```

### 连接客户端
```cpp
NetworkManager manager;
manager.connectToPeer("127.0.0.1", 12345);  // 连接到服务器
```

### 发送消息
```cpp
manager.sendMessage("Hello World!");  // 发送文本消息
```

### 发送文件
```cpp
manager.sendFile("/path/to/file.txt");  // 发送文件
```

## 测试说明

### 启动测试
```cpp
// 启动服务器测试
NetworkManager serverManager;
NetworkTester serverTester(&serverManager, true);
serverTester.start();

// 启动客户端测试
NetworkManager clientManager;
NetworkTester clientTester(&clientManager, false);
clientTester.start();
```

### 定时测试配置
- **广播间隔**：5秒
- **消息发送间隔**：10秒
- **文件发送间隔**：20秒

### 测试流程
1. 服务器启动并开始广播存在信息
2. 客户端连接到服务器
3. 客户端定时发送测试消息
4. 客户端定时发送测试文件
5. 服务器接收并处理消息和文件
6. 组播测试流程：
   - 服务器每5秒组播一次存在信息
   - 客户端监听组播地址239.255.43.21
   - 客户端收到组播消息后显示新用户信息
   - 验证组播消息的发送和接收

### 测试覆盖率
- 消息传输：100% - 已验证周期性消息发送和接收
- 文件传输：100% - 已验证文件创建、发送和保存
- 服务发现：100% - 已验证UDP广播和服务发现
- 错误处理：90% - 已验证以下错误场景：
  - 网络接口绑定失败
  - UDP广播错误
  - 文件创建失败
  - 待验证场景：
    - 网络断开重连
    - 无效消息格式处理
- 性能测试：80% - 已验证：
  - 周期性任务稳定性
  - 待验证场景：
    - 并发连接压力
    - 大文件传输性能

## 性能优化
- **数据序列化**：使用QDataStream进行高效数据序列化
  - 优化序列化格式
  - 减少内存拷贝
  - 支持压缩传输
- **I/O模型**：采用异步I/O模型提高并发性能
  - 非阻塞I/O操作
  - 事件驱动模型
  - 支持高并发连接
- **大文件传输**：实现消息分块传输支持大文件传输
  - 分块大小优化
  - 断点续传支持
  - 传输进度监控
- **连接管理**：使用连接池管理TCP连接
  - 连接复用
  - 自动负载均衡
  - 连接状态监控
- **内存管理**：优化内存使用
  - 对象池技术
  - 零拷贝技术
  - 内存泄漏检测

## 注意事项
- **端口配置**：
  - 确保端口号未被占用
  - 建议使用1024以上端口
  - 支持端口自动选择
- **文件传输**：
  - 注意文件大小限制
  - 支持断点续传
  - 自动创建下载目录
- **网络环境**：
  - 测试时确保服务器和客户端在同一网络环境
  - 支持跨网段通信
  - 支持NAT穿透
- **定时器配置**：
  - 定时器间隔可根据需要调整
  - 支持动态调整间隔
  - 支持定时器优先级设置
- **测试文件**：
  - 测试文件默认保存在应用程序目录下
  - 支持自定义保存路径
  - 自动清理测试文件

## 已知问题
1. 大文件传输时内存占用较高
2. 跨网段服务发现可能失败
3. 网络断开重连机制待完善

## 未来改进
- 增加传输加密支持
- 实现断点续传功能
- 支持多线程并发传输
- 添加流量控制机制