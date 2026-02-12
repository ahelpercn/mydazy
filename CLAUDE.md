# CLAUDE.md - XiaoZhi AI Chatbot (xiaozhi-esp32)

## Project Overview

XiaoZhi is an open-source ESP32-based IoT voice AI chatbot firmware (v2.2.2). It enables voice interaction with large language models (Qwen, DeepSeek, etc.) on ESP32 microcontrollers through WiFi or 4G cellular connectivity. The system uses MCP (Model Context Protocol) for extensible device and cloud integration.

**Language:** C++ (primary), C, Python (build/utility scripts)
**Framework:** ESP-IDF v5.5.2+
**License:** MIT

## Repository Structure

```
├── main/                          # Core firmware source code
│   ├── main.cc                    # Entry point (app_main)
│   ├── application.cc/h           # Singleton Application class, main event loop
│   ├── mcp_server.cc/h            # MCP protocol server implementation
│   ├── ota.cc/h                   # Over-the-air firmware updates
│   ├── settings.cc/h              # NVS-based settings management
│   ├── system_info.cc/h           # System information utilities
│   ├── device_state_machine.cc/h  # Device state management
│   ├── audio/                     # Audio pipeline (capture, codecs, wake words, processing)
│   │   ├── audio_service.cc/h     # Core audio service
│   │   ├── codecs/                # Hardware codec drivers (ES8311, ES8388, etc.)
│   │   ├── processors/            # Audio processing (AFE, noise reduction)
│   │   └── wake_words/            # Wake word detection implementations
│   ├── display/                   # Display abstraction (OLED, LCD, LVGL)
│   │   ├── display.cc/h           # Base display class
│   │   └── lvgl_display/          # LVGL-based advanced UI with emoji/GIF support
│   ├── protocols/                 # Network communication
│   │   ├── protocol.h             # Abstract Protocol base class
│   │   ├── websocket_protocol.cc/h
│   │   └── mqtt_protocol.cc/h     # MQTT + UDP hybrid
│   ├── led/                       # LED control (single, strip, GPIO)
│   ├── boards/                    # 93+ hardware board configurations
│   │   ├── common/                # Shared board utilities (wifi_board, ml307_board, etc.)
│   │   └── <board-name>/          # Each board has its own directory with config.json
│   ├── CMakeLists.txt             # Main build configuration (~1100 lines)
│   ├── Kconfig.projbuild          # ESP-IDF menuconfig options
│   └── idf_component.yml          # 60+ external component dependencies
├── docs/                          # Documentation (code style, protocols, board customization)
├── scripts/                       # Python build/utility scripts
│   ├── release.py                 # Main build orchestration
│   ├── build_default_assets.py    # Asset bundle building
│   └── gen_lang.py                # Language configuration generation
├── partitions/v2/                 # Flash partition tables (8m, 16m, 32m)
├── .github/workflows/build.yml    # CI/CD pipeline
├── CMakeLists.txt                 # Root CMake (project version, IDF integration)
├── sdkconfig.defaults             # Global ESP-IDF defaults
├── sdkconfig.defaults.esp32*      # Chip-specific defaults (esp32, esp32s3, esp32c3, etc.)
└── .clang-format                  # Code formatting rules
```

## Build System

### Prerequisites

- **ESP-IDF v5.5.2+** (required)
- **Docker alternative:** `espressif/idf:v5.5.2` container image
- **Python 3** for build scripts

### Building Firmware

```bash
# Set up ESP-IDF environment
source $IDF_PATH/export.sh

# Build a specific board variant
python scripts/release.py <board-name> --name <variant-name>

# Example:
python scripts/release.py bread-compact-wifi --name bread-compact-wifi

# List all available board variants
python scripts/release.py --list-boards --json

# Standard ESP-IDF build (after menuconfig)
idf.py build
```

**Output:** `build/merged-binary.bin` (combined bootloader + app + partition table)

### Board Configuration

Each board directory (`main/boards/<board-name>/`) contains a `config.json`:
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

The `target` field specifies the ESP32 chip family. The `builds` array defines one or more build variants with optional sdkconfig overrides.

### Supported Chip Families

- ESP32 (original)
- ESP32-S3 (most common, required for AFE wake word)
- ESP32-C3, ESP32-C5, ESP32-C6 (RISC-V based)
- ESP32-P4 (high-performance)

## Architecture

### Core Design Patterns

- **Singleton:** `Application::GetInstance()`, `Board::GetInstance()`, `McpServer`
- **Event-driven:** FreeRTOS event groups drive the main loop (`Application::Run()`)
- **Strategy:** Pluggable `Protocol`, `Display`, `AudioCodec`, `Led` implementations
- **Factory:** Board creation via `DECLARE_BOARD(ClassName)` macro and `create_board()`

### Application Lifecycle

```
app_main() → NVS init → Application::GetInstance().Initialize() → Application::Run()
```

`Run()` is an infinite event loop processing events via FreeRTOS event groups:
- `MAIN_EVENT_NETWORK_CONNECTED/DISCONNECTED` — network state
- `MAIN_EVENT_WAKE_WORD_DETECTED` — voice activation
- `MAIN_EVENT_TOGGLE_CHAT` — user interaction
- `MAIN_EVENT_SEND_AUDIO` — audio streaming
- `MAIN_EVENT_SCHEDULE` — deferred task execution
- `MAIN_EVENT_STATE_CHANGED` — device state transitions

### Key Abstractions

| Class | Purpose |
|-------|---------|
| `Application` | Main singleton, event loop, state coordination |
| `Board` | Hardware abstraction (audio codec, display, LED, network) |
| `Protocol` | Network communication (WebSocket or MQTT+UDP) |
| `AudioService` | Audio capture, processing, codec management |
| `Display` | UI rendering (OLED, LCD, LVGL variants) |
| `McpServer` | MCP protocol for device/cloud control |
| `Ota` | Over-the-air firmware and asset updates |

### Thread Model

- **Main task** (priority 10): Event loop in `Application::Run()`
- **Audio task:** Real-time audio capture and processing
- **Network tasks:** WiFi/cellular connectivity management
- **Background tasks:** OTA updates, asset management, MCP processing

### Adding a New Board

1. Create directory `main/boards/<board-name>/`
2. Add `config.json` with target chip and build variants
3. Create `<board-name>.cc` implementing `Board` interface
4. Use `DECLARE_BOARD(YourBoardClass)` macro to register
5. Add `BOARD_TYPE_<NAME>` entry in `main/Kconfig.projbuild`
6. Add source file mapping in `main/CMakeLists.txt`

See `docs/custom-board.md` for detailed instructions.

## Code Style and Formatting

### clang-format Configuration

The project uses clang-format based on Google C++ style with customizations:

- **Indentation:** 4 spaces (no tabs)
- **Line width:** 100 characters max
- **Braces:** Attach style (K&R variant — opening brace on same line)
- **Pointers:** Left-aligned (`int* ptr`, not `int *ptr`)
- **Includes:** Sorted, with ESP-IDF headers prioritized
- **Access modifiers:** -4 offset (aligned with class keyword)

### Format Commands

```bash
# Format a single file
clang-format -i path/to/file.cc

# Format the entire project
find main -iname *.h -o -iname *.cc | xargs clang-format -i

# Check formatting without modifying
clang-format --dry-run -Werror path/to/file.cc
```

### Code Conventions

- C++ source files use `.cc` extension, headers use `.h`
- Google C++ Style Guide as baseline (see `docs/code_style.md`)
- ESP-IDF logging macros: `ESP_LOGI`, `ESP_LOGW`, `ESP_LOGE` with `TAG` constants
- FreeRTOS primitives for concurrency (event groups, mutexes, tasks)
- `cJSON` library for JSON parsing
- `#define TAG "module_name"` at top of each `.cc` file for logging
- Use `ESP_ERROR_CHECK()` for critical ESP-IDF return values
- Comments in both English and Chinese are acceptable

### Naming Conventions

- Classes: `PascalCase` (e.g., `AudioService`, `McpServer`)
- Methods: `PascalCase` (e.g., `GetInstance()`, `HandleEvent()`)
- Member variables: `snake_case_` with trailing underscore (e.g., `protocol_`, `event_group_`)
- Constants/Macros: `UPPER_SNAKE_CASE` (e.g., `MAIN_EVENT_SEND_AUDIO`)
- Enum values: `kPascalCase` (e.g., `kAbortReasonNone`, `kAecOff`)
- File names: `snake_case.cc` / `snake_case.h`
- Board directories: `kebab-case` (e.g., `bread-compact-wifi`, `esp-box-3`)

## Configuration System

### Layers (evaluated in order)

1. **sdkconfig.defaults** — Global defaults for all chips
2. **sdkconfig.defaults.esp32\<variant\>** — Chip-specific defaults
3. **Board config.json → sdkconfig_append** — Board-specific overrides
4. **Kconfig.projbuild** — Interactive menuconfig options (board type, language, wake word, display, etc.)
5. **NVS (runtime)** — Persistent key-value storage for WiFi credentials and device settings

### Key Kconfig Options

- `BOARD_TYPE_*` — Select hardware board
- `LANGUAGE_*` — Display language (20+ supported)
- `WAKE_WORD_TYPE` — Wake word detection (disabled, ESP, AFE, custom)
- `DISPLAY_STYLE` — UI style (default, WeChat, emote)
- `OTA_URL` — Firmware update server
- `FLASH_*_ASSETS` — Asset flashing strategy

## CI/CD Pipeline

**File:** `.github/workflows/build.yml`

- **Triggers:** Push to `main`, pull requests to `main`
- **Container:** `espressif/idf:v5.5.2`
- **Strategy:** Smart variant selection:
  - Push to main: builds all 70+ variants
  - Pull requests: only builds variants affected by changed files
  - Changes in `main/` (non-board files) or `main/boards/common/` trigger full rebuild
  - Changes only in `main/boards/<board>/` trigger that board only

## Testing

There is no formal unit test suite. Quality assurance relies on:

1. **CI compilation testing** — All 70+ board variants must compile successfully
2. **Hardware testing** — Manual testing on physical boards
3. **Audio debugging** — `scripts/audio_debug_server.py` for audio pipeline testing
4. **Acoustic validation** — `scripts/acoustic_check/` tools

## Key Dependencies

| Category | Component | Purpose |
|----------|-----------|---------|
| Audio | `esp-sr` v2.3.0 | Speech recognition and wake word |
| Audio | `esp_audio_codec`, `esp_codec_dev` | Audio codec abstraction |
| Display | `lvgl` v9.4.0, `esp_lvgl_port` | LVGL GUI framework |
| Display | Various `esp_lcd_*` | LCD panel drivers |
| Network | `esp-wifi-connect` v3.0.2 | WiFi provisioning |
| Network | `esp-ml307` v3.6.4 | 4G cellular modem |
| LED | `led_strip` v3.0.2 | LED strip control |
| Input | `button` v4.1.5, `knob` | User input |
| Camera | `esp32-camera` v2.1.4 | Camera support (ESP32-S3) |

## Data Storage

- **NVS (Non-Volatile Storage):** Key-value store for WiFi config, device settings, asset versions
- **SPIFFS/Assets partition:** Dynamic assets (wake words, fonts, emoji, backgrounds) — up to 8MB on 16MB flash
- **OTA partitions:** Dual firmware slots (ota_0, ota_1) for safe firmware updates with rollback

## Communication Protocols

- **WebSocket:** Real-time bidirectional communication with cloud (audio + JSON)
- **MQTT + UDP:** MQTT for control messages, UDP for low-latency audio streaming
- **MCP (Model Context Protocol):** Device-side tools (speaker, LED, GPIO, servo) and cloud-side tools (smart home, email, weather)

## Common Development Tasks

### Adding a new display type

1. Create class inheriting from `Display` in `main/display/`
2. Implement virtual methods for rendering
3. Wire it up in the board's constructor

### Adding a new audio codec

1. Create class inheriting from `AudioCodec` in `main/audio/codecs/`
2. Implement I2S configuration and codec initialization
3. Reference it from the board implementation

### Adding a new MCP tool

1. Add tool definition in `McpServer` (`main/mcp_server.cc`)
2. Implement the tool handler function
3. Register it in the tool list

### Modifying communication protocol

1. `Protocol` base class in `main/protocols/protocol.h`
2. Implement `WebSocketProtocol` or `MqttProtocol` interface
3. Binary protocol formats: `BinaryProtocol2` (v2) and `BinaryProtocol3` (v3) defined in `protocol.h`
