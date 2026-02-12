# CLAUDE.md - 小智 AI 语音聊天机器人 (xiaozhi-esp32)

## 项目概述

小智是一个基于 ESP32 的开源物联网语音 AI 聊天机器人固件（v2.2.2）。它通过 WiFi 或 4G 蜂窝网络在 ESP32 微控制器上实现与大语言模型（通义千问、DeepSeek 等）的语音交互。系统使用 MCP（Model Context Protocol，模型上下文协议）实现可扩展的设备与云端集成。

- **编程语言：** C++（主要）、C、Python（构建/工具脚本）
- **开发框架：** ESP-IDF v5.5.2+
- **开源协议：** MIT

## 仓库结构

```
├── main/                          # 核心固件源代码
│   ├── main.cc                    # 入口点 (app_main)
│   ├── application.cc/h           # 单例 Application 类，主事件循环
│   ├── mcp_server.cc/h            # MCP 协议服务器实现
│   ├── ota.cc/h                   # 空中固件升级 (OTA)
│   ├── settings.cc/h              # 基于 NVS 的设置管理
│   ├── system_info.cc/h           # 系统信息工具
│   ├── device_state_machine.cc/h  # 设备状态机管理
│   ├── assets.cc/h                # 资源管理
│   ├── audio/                     # 音频流水线（采集、编解码、唤醒词、处理）
│   │   ├── audio_service.cc/h     # 核心音频服务
│   │   ├── audio_codec.cc/h       # 音频编解码器基类
│   │   ├── codecs/                # 硬件编解码驱动（ES8311、ES8388 等）
│   │   ├── processors/            # 音频处理（AFE、降噪）
│   │   ├── wake_words/            # 唤醒词检测实现
│   │   └── demuxer/               # OGG 解复用器
│   ├── display/                   # 显示抽象层（OLED、LCD、LVGL）
│   │   ├── display.cc/h           # Display 基类
│   │   ├── lcd_display.cc/h       # LCD 显示实现
│   │   ├── oled_display.cc/h      # OLED 显示实现
│   │   ├── emote_display.cc/h     # 表情显示
│   │   └── lvgl_display/          # 基于 LVGL 的高级 UI（支持表情/GIF）
│   ├── protocols/                 # 网络通信协议
│   │   ├── protocol.h             # Protocol 抽象基类
│   │   ├── websocket_protocol.cc/h# WebSocket 协议实现
│   │   └── mqtt_protocol.cc/h     # MQTT + UDP 混合协议
│   ├── led/                       # LED 控制（单个、灯带、GPIO）
│   │   ├── led.h                  # LED 基类
│   │   ├── single_led.cc/h        # 单 LED 控制
│   │   ├── circular_strip.cc/h    # 环形灯带
│   │   └── gpio_led.cc/h          # GPIO LED
│   ├── boards/                    # 93+ 硬件开发板配置
│   │   ├── common/                # 公共板级工具
│   │   │   ├── board.cc/h         # Board 基类（纯虚接口）
│   │   │   ├── wifi_board.cc/h    # WiFi 连接板
│   │   │   ├── ml307_board.cc/h   # 4G 模块板
│   │   │   ├── dual_network_board.cc/h # WiFi + 4G 双网络板
│   │   │   ├── button.cc/h        # 按钮输入
│   │   │   ├── knob.cc/h          # 旋钮输入
│   │   │   ├── backlight.cc/h     # 背光控制
│   │   │   ├── esp32_camera.cc/h  # 摄像头支持
│   │   │   └── power_save_timer.cc/h # 节能定时器
│   │   └── <board-name>/          # 每个开发板有独立目录（含 config.json）
│   ├── CMakeLists.txt             # 主构建配置（约 1100 行）
│   ├── Kconfig.projbuild          # ESP-IDF menuconfig 选项
│   └── idf_component.yml          # 60+ 外部组件依赖
├── docs/                          # 文档（代码风格、协议、开发板定制）
│   ├── code_style.md              # 代码风格指南
│   ├── custom-board.md            # 自定义开发板指南
│   ├── mcp-protocol.md            # MCP 协议文档
│   ├── websocket.md               # WebSocket 协议规范
│   └── mqtt-udp.md                # MQTT+UDP 协议规范
├── scripts/                       # Python 构建/工具脚本
│   ├── release.py                 # 主构建编排脚本
│   ├── build_default_assets.py    # 资源打包构建
│   ├── gen_lang.py                # 语言配置生成
│   ├── audio_debug_server.py      # 音频调试服务器
│   ├── spiffs_assets/             # SPIFFS 资源生成工具
│   ├── acoustic_check/            # 声学校验工具
│   └── Image_Converter/           # 图片转换工具
├── partitions/v2/                 # Flash 分区表（8m、16m、32m）
├── .github/workflows/build.yml    # CI/CD 流水线
├── CMakeLists.txt                 # 根 CMake 配置（项目版本 v2.2.2）
├── sdkconfig.defaults             # 全局 ESP-IDF 默认配置
├── sdkconfig.defaults.esp32*      # 各芯片特定默认配置
└── .clang-format                  # 代码格式化规则
```

## 构建系统

### 环境要求

- **ESP-IDF v5.5.2+**（必需）
- **Docker 替代方案：** `espressif/idf:v5.5.2` 容器镜像
- **Python 3** 用于构建脚本

### 固件编译

```bash
# 配置 ESP-IDF 环境
source $IDF_PATH/export.sh

# 编译指定开发板变体
python scripts/release.py <开发板名称> --name <变体名称>

# 示例：
python scripts/release.py bread-compact-wifi --name bread-compact-wifi

# 列出所有可用的开发板变体
python scripts/release.py --list-boards --json

# 标准 ESP-IDF 构建（配置完 menuconfig 后）
idf.py build
```

**输出：** `build/merged-binary.bin`（合并的引导加载程序 + 应用 + 分区表）

### 开发板配置

每个开发板目录（`main/boards/<board-name>/`）包含一个 `config.json`：
```json
{
    "target": "esp32s3",
    "builds": [
        {
            "name": "variant-name",
            "sdkconfig_append": ["CONFIG_OPTION=y"]
        }
    ]
}
```

- `target`：指定 ESP32 芯片系列
- `builds`：定义一个或多个构建变体，可带可选的 sdkconfig 覆盖配置

### 支持的芯片系列

- **ESP32**（经典版本）
- **ESP32-S3**（最常用，AFE 唤醒词必需）
- **ESP32-C3、ESP32-C5、ESP32-C6**（基于 RISC-V 架构）
- **ESP32-P4**（高性能版本）

## 架构设计

### 核心设计模式

- **单例模式：** `Application::GetInstance()`、`Board::GetInstance()`、`McpServer::GetInstance()`
- **事件驱动：** FreeRTOS 事件组驱动主循环（`Application::Run()`）
- **策略模式：** 可插拔的 `Protocol`、`Display`、`AudioCodec`、`Led` 实现
- **工厂模式：** 通过 `DECLARE_BOARD(ClassName)` 宏和 `create_board()` 创建开发板实例
- **观察者模式：** `DeviceStateMachine` 状态变更监听器

### 应用生命周期

```
app_main() → NVS 初始化 → Application::GetInstance().Initialize() → Application::Run()
```

`Run()` 是一个无限事件循环，通过 FreeRTOS 事件组处理事件：
- `MAIN_EVENT_NETWORK_CONNECTED/DISCONNECTED` — 网络连接/断开
- `MAIN_EVENT_WAKE_WORD_DETECTED` — 语音唤醒
- `MAIN_EVENT_TOGGLE_CHAT` — 用户交互（切换聊天状态）
- `MAIN_EVENT_SEND_AUDIO` — 音频流传输
- `MAIN_EVENT_SCHEDULE` — 延迟任务执行
- `MAIN_EVENT_STATE_CHANGED` — 设备状态转换
- `MAIN_EVENT_START_LISTENING/STOP_LISTENING` — 开始/停止监听
- `MAIN_EVENT_ACTIVATION_DONE` — 激活完成
- `MAIN_EVENT_CLOCK_TICK` — 时钟节拍

### 设备状态机

设备通过 `DeviceStateMachine` 管理状态转换，定义在 `device_state.h` 中：

| 状态 | 说明 |
|------|------|
| `kDeviceStateUnknown` | 未知状态（初始） |
| `kDeviceStateStarting` | 启动中 |
| `kDeviceStateWifiConfiguring` | WiFi 配网中 |
| `kDeviceStateIdle` | 空闲待机 |
| `kDeviceStateConnecting` | 连接服务器中 |
| `kDeviceStateListening` | 正在聆听语音 |
| `kDeviceStateSpeaking` | 正在播放语音 |
| `kDeviceStateUpgrading` | 固件升级中 |
| `kDeviceStateActivating` | 设备激活中 |
| `kDeviceStateAudioTesting` | 音频测试中 |
| `kDeviceStateFatalError` | 致命错误 |

### 核心类抽象

| 类 | 职责 |
|----|------|
| `Application` | 主单例，事件循环，状态协调 |
| `Board` | 硬件抽象层（音频编解码、显示、LED、网络） |
| `Protocol` | 网络通信（WebSocket 或 MQTT+UDP） |
| `AudioService` | 音频采集、处理、编解码管理 |
| `AudioCodec` | 音频编解码器抽象（I2S 配置，输入/输出控制） |
| `Display` | UI 渲染（OLED、LCD、LVGL 变体） |
| `McpServer` | MCP 协议，设备端/云端工具管理 |
| `Ota` | 空中固件和资源升级 |
| `DeviceStateMachine` | 设备状态管理与验证 |

### 开发板继承关系

```
Board（基类，纯虚接口）
├── WifiBoard — WiFi 连接的开发板
├── Ml307Board — 使用 4G 模块的开发板
└── DualNetworkBoard — WiFi + 4G 双网络开发板
```

### 线程模型

- **主任务**（优先级 10）：`Application::Run()` 中的事件循环
- **音频任务：** 实时音频采集和处理
- **网络任务：** WiFi/蜂窝网络连接管理
- **后台任务：** OTA 升级、资源管理、MCP 处理

## 代码风格和格式化

### clang-format 配置

项目使用 clang-format，基于 Google C++ 风格并做了自定义调整：

- **缩进：** 4 个空格（禁止使用 Tab）
- **行宽：** 最大 100 个字符
- **大括号：** Attach 风格（K&R 变体 — 左大括号与语句同行）
- **指针：** 左对齐（`int* ptr`，而非 `int *ptr`）
- **头文件包含：** 排序，ESP-IDF 头文件优先
- **访问修饰符：** -4 偏移量（与 class 关键字对齐）
- **模板：** 模板声明前总是换行
- **尾部注释：** 前面留 2 个空格

### 格式化命令

```bash
# 格式化单个文件
clang-format -i path/to/file.cc

# 格式化整个项目
find main -iname *.h -o -iname *.cc | xargs clang-format -i

# 检查格式化（不修改文件）
clang-format --dry-run -Werror path/to/file.cc
```

### 代码规范

- C++ 源文件使用 `.cc` 扩展名，头文件使用 `.h`
- 基于 Google C++ 风格指南（详见 `docs/code_style.md`）
- 使用 ESP-IDF 日志宏：`ESP_LOGI`、`ESP_LOGW`、`ESP_LOGE`，配合 `TAG` 常量
- 使用 FreeRTOS 原语进行并发（事件组、互斥锁、任务）
- 使用 `cJSON` 库进行 JSON 解析
- 每个 `.cc` 文件顶部定义 `#define TAG "module_name"` 用于日志
- 关键 ESP-IDF 返回值使用 `ESP_ERROR_CHECK()`
- 注释可以使用中文和英文

### 命名规范

| 类别 | 规则 | 示例 |
|------|------|------|
| 类名 | `PascalCase` | `AudioService`、`McpServer` |
| 方法名 | `PascalCase` | `GetInstance()`、`HandleEvent()` |
| 成员变量 | `snake_case_`（尾部下划线） | `protocol_`、`event_group_` |
| 常量/宏 | `UPPER_SNAKE_CASE` | `MAIN_EVENT_SEND_AUDIO` |
| 枚举值 | `kPascalCase` | `kAbortReasonNone`、`kAecOff` |
| 文件名 | `snake_case.cc` / `snake_case.h` | `audio_service.cc` |
| 开发板目录 | `kebab-case` | `bread-compact-wifi`、`esp-box-3` |

## 配置系统

### 配置层次（按优先级从低到高）

1. **sdkconfig.defaults** — 全局默认配置（所有芯片）
2. **sdkconfig.defaults.esp32\<variant\>** — 芯片特定默认配置
3. **Board config.json → sdkconfig_append** — 开发板特定覆盖
4. **Kconfig.projbuild** — 交互式 menuconfig 选项（开发板类型、语言、唤醒词、显示等）
5. **NVS（运行时）** — 持久化键值存储（WiFi 凭证、设备设置）

### 关键 Kconfig 选项

| 配置项 | 说明 |
|--------|------|
| `BOARD_TYPE_*` | 选择硬件开发板 |
| `LANGUAGE_*` | 显示语言（支持 20+ 种语言） |
| `WAKE_WORD_TYPE` | 唤醒词检测（禁用、ESP、AFE、自定义） |
| `DISPLAY_STYLE` | UI 风格（默认、微信、表情） |
| `OTA_URL` | 固件更新服务器地址 |
| `FLASH_*_ASSETS` | 资源烧录策略 |

### 全局默认配置要点

- 编译器大小优化 (`CONFIG_COMPILER_OPTIMIZATION_SIZE`)
- 启用 C++ 异常和 RTTI
- 自定义分区表（`partitions/v2/16m.csv`）
- 看门狗超时 10 秒
- LVGL 9.x 配置（精简不必要的组件以节省 Flash）
- mBedTLS 动态缓冲区以节省内存

## CI/CD 流水线

**文件：** `.github/workflows/build.yml`

- **触发条件：** 推送到 `main` 分支、向 `main` 分支提交 PR
- **运行容器：** `espressif/idf:v5.5.2`
- **智能构建策略：**
  - 推送到 main：构建所有 70+ 个变体
  - Pull Request：仅构建受文件变更影响的变体
  - `main/`（非 boards 文件）或 `main/boards/common/` 的变更触发全量构建
  - 仅 `main/boards/<board>/` 的变更只触发该开发板的构建
- **构建产物：** 以 `xiaozhi_<变体名>_<commit-sha>.bin` 形式上传

## 测试

项目没有正式的单元测试套件。质量保障依赖于：

1. **CI 编译测试** — 所有 70+ 个开发板变体必须编译成功
2. **硬件测试** — 在物理开发板上进行手动测试
3. **音频调试** — `scripts/audio_debug_server.py` 用于音频流水线测试
4. **声学校验** — `scripts/acoustic_check/` 工具

## 关键依赖

| 分类 | 组件 | 用途 |
|------|------|------|
| 音频 | `esp-sr` v2.3.0 | 语音识别和唤醒词 |
| 音频 | `esp_audio_codec`、`esp_codec_dev` | 音频编解码抽象 |
| 显示 | `lvgl` v9.4.0、`esp_lvgl_port` | LVGL GUI 框架 |
| 显示 | 各种 `esp_lcd_*` | LCD 面板驱动 |
| 网络 | `esp-wifi-connect` v3.0.2 | WiFi 配网 |
| 网络 | `esp-ml307` v3.6.4 | 4G 蜂窝模块 |
| 网络 | `esp_hosted` v2.0.17 | WiFi 卸载（ESP32-P4） |
| LED | `led_strip` v3.0.2 | LED 灯带控制 |
| 输入 | `button` v4.1.5、`knob` | 用户输入 |
| 摄像头 | `esp32-camera` v2.1.4 | 摄像头支持（ESP32-S3） |
| 图片 | `esp_new_jpeg`、`esp_mmap_assets` | 图片处理和内存映射 |
| 字体 | `xiaozhi-fonts` v1.6.0 | 自定义字体库 |
| 表情 | `otto-emoji-gif-component` | 动画表情 |
| 传感器 | `bmi270_sensor` | IMU 传感器 |

## 数据存储

- **NVS（非易失性存储）：** 键值存储，保存 WiFi 配置、设备设置、资源版本
- **SPIFFS/资源分区：** 动态资源（唤醒词、字体、表情、背景图）— 16MB Flash 上最大 8MB
- **OTA 分区：** 双固件槽（ota_0、ota_1），支持安全的固件升级和回滚

### 分区布局（16MB 标准）

| 分区 | 大小 | 用途 |
|------|------|------|
| nvs | 16KB | WiFi/设备配置 |
| otadata | 8KB | OTA 元数据 |
| phy_init | 4KB | PHY 初始化 |
| ota_0 | 4MB | 固件槽 0 |
| ota_1 | 4MB | 固件槽 1 |
| assets | 8MB | 动态资源 |

## 通信协议

### WebSocket 协议
- **位置：** `main/protocols/websocket_protocol.cc/h`
- 与云端的实时双向通信（音频 + JSON）
- 支持二进制协议格式 `BinaryProtocol2`（v2）和 `BinaryProtocol3`（v3）

### MQTT + UDP 协议
- **位置：** `main/protocols/mqtt_protocol.cc/h`
- MQTT 用于控制消息，UDP 用于低延迟音频流传输

### MCP（模型上下文协议）
- **位置：** `main/mcp_server.cc/h`
- **设备端工具：** 扬声器控制、LED 控制、GPIO 操作、舵机控制、传感器读取
- **云端工具：** 智能家居、PC 桌面控制、知识搜索、邮件、天气
- MCP 工具通过 `McpTool` 类定义，支持属性类型（布尔、整数、字符串）和值范围验证
- 支持 `user_only` 标记（仅对用户可见，对 AI 不可见的工具）

## 常见开发任务

### 新增开发板

1. 创建目录 `main/boards/<board-name>/`
2. 添加 `config.json`，指定目标芯片和构建变体
3. 创建 `config.h`，定义硬件管脚映射
4. 创建 `<board-name>.cc`，继承 `WifiBoard`/`Ml307Board`/`DualNetworkBoard`
5. 使用 `DECLARE_BOARD(YourBoardClass)` 宏注册
6. 在 `main/Kconfig.projbuild` 添加 `BOARD_TYPE_<NAME>` 条目
7. 在 `main/CMakeLists.txt` 添加源文件映射和字体/表情配置

详见 `docs/custom-board.md`。

### 新增显示类型

1. 在 `main/display/` 创建继承自 `Display` 的类
2. 实现虚方法（`SetStatus`、`SetEmotion`、`SetChatMessage` 等）
3. 实现 `Lock`/`Unlock` 方法用于线程安全
4. 在开发板构造函数中关联

### 新增音频编解码器

1. 在 `main/audio/codecs/` 创建继承自 `AudioCodec` 的类
2. 实现 I2S 配置和编解码器初始化
3. 实现 `Read`/`Write` 纯虚方法
4. 在开发板实现中引用

### 新增 MCP 工具

1. 在 `McpServer`（`main/mcp_server.cc`）中添加工具定义
2. 使用 `McpTool` 类定义工具名称、描述、属性和回调
3. 通过 `AddTool()` 或 `AddUserOnlyTool()` 注册
4. 回调返回值支持 `bool`、`int`、`std::string`、`cJSON*`、`ImageContent*`

### 修改通信协议

1. `Protocol` 基类在 `main/protocols/protocol.h`
2. 实现 `WebSocketProtocol` 或 `MqttProtocol` 接口
3. 必须实现的纯虚方法：`Start()`、`OpenAudioChannel()`、`CloseAudioChannel()`、`SendAudio()`、`SendText()`
4. 二进制协议格式：`BinaryProtocol2`（v2）和 `BinaryProtocol3`（v3）
