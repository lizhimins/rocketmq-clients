# RocketMQ 多语言客户端测试指南

> 整合了测试实例信息、功能对比、测试记录、Docker 测试结果、国内镜像、环境搭建计划及 C++ 客户端说明。

---

## 一、测试实例连接信息

| 字段 | 值 |
|------|------|
| 实例 ID | `rmq-cn-u7c3giqmw0s` |
| 接入点 | 见 `.env.example` 中的 `ROCKETMQ_ENDPOINT` |
| 地域 | 杭州（cn-hangzhou） |
| 协议 | gRPC（8080 端口） |

### Topic 信息

| 消息类型 | Topic | 说明 |
|----------|-------|------|
| 普通消息 | NormalTest | 基础消息收发 |
| 顺序消息 | OrderTest | FIFO 顺序消息，需指定 message_group |
| 定时消息 | TimerTest | 延时/定时消息，需设置 delivery_timestamp |
| 事务消息 | TransTest | 事务消息，需实现 TransactionChecker |

### 消费组信息

| 消费组名称 | 消费者类别 | 是否顺序 |
|------------|------------|----------|
| PushConsumer | Push Consumer | 否 |
| PushOrderConsumer | Push Consumer | 是 |
| SimpleConsumer | Simple Consumer | 否 |
| SimpleOrderConsumer | Simple Consumer | 是 |

### 环境变量配置

所有语言统一使用以下环境变量：

| 环境变量 | 说明 |
|----------|------|
| `ROCKETMQ_ENDPOINT` | 接入点地址（如 `xxx.rmq.aliyuncs.com:8080`） |
| `ROCKETMQ_ACCESS_KEY` | 访问密钥 |
| `ROCKETMQ_SECRET_KEY` | 密钥 |

> **注意**：8080 端口为 VPC 内网接入点，仅支持在阿里云 VPC 网络内访问。

---

## 二、各语言客户端功能对比

> 评估时间：2026-05-09 | 数据来源：仓库代码静态分析

### 客户端类型覆盖

| 客户端类型 | Java | Go | C++ | C# | Rust | Python | Node.js | PHP |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Producer（同步） | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ |
| Producer（异步） | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Producer（事务） | ✅ | ✅ | ✅ | ✅ | ⚠️ | ✅ | ✅ | ❌ |
| Push Consumer | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Simple Consumer | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ |
| Lite Push Consumer | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ | ❌ |
| Lite Simple Consumer | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |

### 消息类型支持

| 消息类型 | Java | Go | C++ | C# | Rust | Python | Node.js | PHP |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| NORMAL | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| FIFO | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| DELAY/TIMED | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| TRANSACTION | ✅ | ✅ | ✅ | ✅ | ⚠️ | ✅ | ✅ | ❌ |
| PRIORITY | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ | ❌ |

### 高级功能

| 功能 | Java | Go | C++ | C# | Rust | Python | Node.js | PHP |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 消息撤回 | ✅ | ✅ | ✅ | ✅ | ❌ | ⚠️ | ✅ | ❌ |
| 批量发送 | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ |
| Tag 过滤 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| 可观测性 | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| 端点隔离 | ✅ | ✅ | ⚠️ | ⚠️ | ❌ | ❌ | ⚠️ | ❌ |

### 实现质量

| 指标 | Java | Go | C++ | C# | Rust | Python | Node.js | PHP |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 源码文件数 | 127 | 53 | 84+29h | 103 | 15 | 43 | 71 | 3 |
| 测试文件数 | 34 | 10 | 25 | 36 | 3 | 3 | 14 | 0 |
| 整体成熟度 | **完备** | **成熟** | **成熟** | **成熟** | **成长中** | **可用** | **成熟** | **原型** |

### 共性结论

1. **Pull Consumer** 和 **SQL 过滤** 在所有语言中均未实现
2. **Java** 是功能最全面的参考实现
3. **Node.js** 和 **C#** 成熟度超出预期
4. **Rust** 和 **Python** 是主要差距所在
5. **PHP** 需要大量工作才能达到生产可用

---

## 三、多语言测试记录

### 测试环境

- **操作系统**: Alibaba Cloud Linux 3 (内核 5.10.134-19.2.al8.x86_64)
- **RocketMQ 实例**: rmq-cn-u7c3giqmw0s
- **代码仓库**: https://github.com/apache/rocketmq-clients (master 分支)

### 测试结果总览

| 语言 | 普通消息 | FIFO | 延时消息 | 事务消息 | 状态 |
|------|----------|------|----------|----------|------|
| Java | ✅ | ✅ | ✅ | ✅ | 通过 |
| Golang | ✅ | ✅ | ✅ | ✅ | 通过 |
| Rust | ✅ | ✅ | ✅ | ✅ | 通过 |
| Python | ✅ | ✅ | ✅ | ✅ | 通过 |
| Node.js | ✅ | ✅ | ✅ | ✅ | 通过 |
| C# | ✅ | ✅ | ✅ | ✅ | 通过 (重试成功) |
| C++ | ✅ | ✅ | ✅ | ✅ | 通过 (修复后) |
| PHP | ⚠️ | - | - | - | 仅验证依赖 |

### 各语言踩坑详情

**Java**: `ProducerSingleton.java` 中 AK/SK 为空字符串导致 NPE；CheckStyle 检查需用 `-Dcheckstyle.skip=true` 跳过；`producer.close()` 需 try-catch 包裹。Maven 镜像用阿里云。

**Golang**: `golang.google.cn` 不可达，改用阿里云镜像 `mirrors.aliyun.com/golang/` 下载。GOPROXY 设为 `goproxy.cn,direct`。4 个 Producer 均通过。

**Rust**: 阿里云/清华 crates 镜像均失败（`config.json not found` / 超时），最终直连 crates.io 成功。首次编译约 64 分钟。编译通过后 4 个 Producer 均通过。

**Python**: 系统 `python3` 指向 3.6，需用 `python3.8`。手动安装 `opentelemetry`/`grpcio`/`protobuf` 后通过。

**Node.js**: `ProducerTransactionMessageExample.ts` 导入路径错误（改相对路径修复）；多 Producer 顺序运行有 log stream 关闭报错（不影响功能）。

**C#**: 首次因 proto 子模块未初始化失败（GitHub 不可达），网络恢复后重试成功。

**C++**（坑最多）:
1. GitHub 不可达导致 proto 子模块无法克隆
2. gRPC 未预装，需从源码编译（v1.54.3，约 30 分钟）
3. gflags 未预装，需从源码编译
4. C++11 不支持 `std::make_unique`，改为 C++14
5. `absl::make_unique` 不兼容，替换为 `std::make_unique`
6. 线程注解宏需改为 `ABSL_` 前缀
7. **Telemetry 握手死循环**：`awaitApplyingSettings()` 超时后改为返回 true；移除 `OnDone()` 自动重连
8. 认证凭证需通过命令行参数显式传入
9. Transaction 示例末尾 sleep 5 分钟，改为 10 秒
10. FIFO 示例成功输出被注释，取消注释

**PHP**: 仅验证依赖，Composer 不可用且无法下载。PHP 示例不完整。

### 通用踩坑

- **GitHub 连接不稳定**: `git submodule update` 多次失败
- **凭证管理**: 所有语言初始使用占位符，需逐一替换。C++ 是唯一通过命令行参数传入凭证的语言
- **协议子模块**: 首次克隆务必 `git clone --recursive`

---

## 四、Docker 测试结果（2026-05-15）

| 语言 | 镜像大小 | 构建 | 运行 | 状态 |
|------|----------|------|------|------|
| Java | 328MB | ✅ | ✅ | PASSED |
| Golang | 199MB | ✅ | ✅ | PASSED |
| C++ | 786MB | ✅ | ✅ | PASSED |
| C# | 197MB | ✅ | ✅ | PASSED |
| Rust | 125MB | ✅ | ✅ | PASSED |
| Python | 166MB | ✅ | ✅ | PASSED |
| Node.js | 327MB | ✅ | ⚠️ | PASSED (close 超时但消息成功) |
| PHP | 1.07GB | ✅ | ✅ | PASSED |

### Docker 关键修复

- **Java**: 修复 `ProducerSingleton.java` 的 `setCredentialProvider(null)` NPE
- **C++**: 创建 `gRPCPkgConfigShim.cmake` 桥接 pkg-config → cmake targets；切换 Ubuntu 24.04
- **Rust**: 移除 cargo mirror，直连 crates.io；edition 2024 需 `rust:1.88-slim`
- **Node.js**: 添加 Aliyun apk 镜像；修复 example 路径
- **PHP**: 放弃 composer，改用 PECL 扩展 + `Grpc\Call` 直接调用
- **C#**: COPY source 改为 `all-demo/csharp/examples/`

### 发现的环境变量问题

| 变量 | 统一前 | 统一后 |
|------|--------|--------|
| Endpoint | 各语言不一致（`ENDPOINTS`/`ENDPOINT`/CLI 参数/hardcoded） | 统一为 `ROCKETMQ_ENDPOINT` |
| Access Key | 基本一致 | `ROCKETMQ_ACCESS_KEY` |

---

## 五、国内镜像源参考

> 基于 2025-2026 年实测

### Docker Hub 加速

| 镜像源 | 地址 | 状态 |
|--------|------|------|
| 轩辕 | `https://docker.xuanyuan.me` | 推荐 |
| 毫秒 | `https://docker.1ms.run` | 可用 |
| DaoCloud | `https://docker.m.daocloud.io` | 可用 |

### 语言运行时 & 包管理

| 语言 | 推荐源 | 不可用 |
|------|--------|--------|
| Go | `goproxy.cn`；SDK: `mirrors.aliyun.com/golang/` | `golang.google.cn` |
| Rust | `mirrors.rustcc.cn/crates.io-index.git`；rustup: `rsproxy.cn` | 阿里/中科大 crates |
| Node.js | npm: `registry.npmmirror.com`；二进制: `npmmirror.com/mirrors/node/` | `registry.npm.taobao.org` |
| Python | `mirrors.aliyun.com/pypi/simple/` | - |
| Java | Maven: `maven.aliyun.com/repository/public` | - |
| .NET | NuGet: `nuget.cdn.azure.cn`；SDK: `mirrors.tuna.tsinghua.edu.cn/dotnet/` | - |
| PHP | 阿里云 Composer 源 | - |
| C++ | 无可靠国内 gRPC 二进制镜像，需从源码编译 | - |

---

## 六、环境搭建与测试执行计划

### 前置条件

```bash
sudo dnf install -y \
  wget curl git unzip tar gcc gcc-c++ make autoconf automake \
  openssl-devel zlib-devel bzip2-devel libffi-devel readline-devel \
  protobuf-compiler protobuf-devel cmake
```

### 语言环境安装

| Step | 语言 | 安装方式 | 验证 |
|------|------|----------|------|
| 1 | Go | 阿里云镜像下载 1.22.5；`GOPROXY=goproxy.cn,direct` | `go version` |
| 2 | .NET | 清华源下载 8.0 SDK | `dotnet --version` |
| 3 | Rust | `rsproxy.cn/rustup-init.sh`；需 >= 1.74.0 | `rustc --version` |
| 4 | Node.js | npmmirror 下载 18.x；npm 设 `registry.npmmirror.com` | `node --version` |
| 5 | Python | `dnf install python39`；pip 装 `grpcio`/`protobuf` 等 | `python3.9 --version` |
| 6 | PHP | `dnf install php`；Composer 阿里源 | `php --version` |
| 7 | Maven | `dnf install maven`；配阿里云 mirror | `mvn --version` |

### C++ 环境搭建（详细步骤）

1. **GCC 11.5.0**: 从 `ftp.gnu.org` 下载源码编译，`--enable-languages=c,c++`
2. **JDK 11**: `yum install java-11-openjdk-devel`（Bazel 需要）
3. **CMake 3.30.2**: 从 GitHub releases 下载源码编译
4. **Bazel 7.2.1**: 从 GitHub releases 下载二进制
5. **protobuf 3.20.1**: 源码编译，`./configure --prefix=/usr/local/protobuf`
6. **gflags 2.2.2**: CMake 编译安装
7. **gRPC 1.46.3**: `git clone --recurse-submodules -b v1.46.3 --depth 1`，CMake 编译到 `$HOME/grpc`（约 30 分钟）

> 自动化脚本：`bash all-demo/cpp/setup.sh`

### C++ 客户端运行

```bash
cd cpp && mkdir -p build && cd build && cmake .. && make -j $(nproc)

# 运行示例
./ExampleProducer --access_point="<endpoint>" --topic=NormalTest
./ExampleProducerWithFifoMessage --access_point="<endpoint>" --topic=OrderTest
./ExampleProducerWithTimedMessage --access_point="<endpoint>" --topic=TimerTest
./ExampleProducerWithTransactionalMessage --access_point="<endpoint>" --topic=TransTest
```

#### C++ 技术要点

- **ABI 兼容性**: GCC 4.9.2 和 GCC 5.x+ 使用不同 C++ ABI，链接错误时需确保版本匹配
- **端口偏移**: gRPC 通信时客户端端口偏移 +10（NameServer: `9876` → `9886`）
- **链接方式**: 支持静态和动态链接；部署建议静态链接以避免依赖版本冲突
- **延迟抖动**: 使用 unary-rpc 传大数据时可能出现，参考 protobuf 大数据技巧

### 各语言测试配置策略

| 语言 | 配置方式 |
|------|----------|
| Java | 修改 `ProducerSingleton.java` 常量 + 各 example 中 topic |
| Golang | 修改各 `main.go` 中的 `const` 块 |
| C++ | 通过 `--access_point` 和 `--topic` 命令行参数传入 |
| C# | 设置环境变量 + 修改 topic 常量 |
| Rust | 修改 example 中 `set_access_url` 和 topic |
| Python | 修改 example 中 `endpoints` 和 `topic` 变量 |
| Node.js | 设置环境变量 `ROCKETMQ_NODEJS_CLIENT_ENDPOINTS` |
| PHP | 仅 WIP，跳过消息收发测试 |

### 测试执行规则

1. **发送先行**：先运行 Producer，再运行 Consumer
2. **Topic 严格匹配**：普通→`NormalTest`，顺序→`OrderTest`，定时→`TimerTest`，事务→`TransTest`
3. **Consumer 类型覆盖**：每语言至少测试 Push Consumer 和 Simple Consumer
4. **超时控制**：Consumer 运行 15-30 秒后停止，Producer 发送 3-10 条消息
