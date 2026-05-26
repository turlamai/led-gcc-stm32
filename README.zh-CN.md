# led-gcc-stm32

[English](README.md) | [中文](README.zh-CN.md)

一个完整的 STM32F103C8T6（Blue Pill / YT32M1）LED 闪烁工程，面向 VS Code 开发，构建、烧录和调试均已开箱即配。

构建基于 GNU Make 和 ARM GCC 工具链。烧录通过 OpenOCD + CMSIS-DAP（SWD）完成。调试使用 VS Code Cortex-Debug 扩展，支持 SVD 外设寄存器视图。

## 特性

- **一条命令构建** — `make all` 生成 `.elf`、`.hex`、`.bin` 三种格式输出
- **一条命令烧录** — `make flash` 通过 OpenOCD 将固件写入 MCU
- **一键调试** — 在 VS Code 中按 `F5` 即可启动完整的 Cortex-Debug 调试会话
- **兼容 STM32CubeMX** — 包含 `.ioc` 工程文件，可随时用 CubeMX 重新生成代码
- **SVD 外设视图** — 已配置 `STM32F103xx.svd`，支持寄存器级调试
- **HAL 库** — 使用 STM32F1xx HAL 驱动（由 CubeMX 生成）

## 环境要求

| 工具 | 版本 | 用途 |
|------|------|------|
| [ARM GCC 工具链](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-rm) | 10.3+ | 交叉编译器（`arm-none-eabi-gcc`） |
| [GNU Make](http://gnuwin32.sourceforge.net/packages/make.htm) | 3.82+ | 构建自动化 |
| [OpenOCD](http://openocd.org/) | 0.11+ | 片上烧录与调试服务器 |
| [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html) | 6.0+ | （可选）重新生成 HAL 代码 |
| VS Code + [Cortex-Debug](https://marketplace.visualstudio.com/items?itemName=marus25.cortex-debug) | — | IDE 和调试前端 |
| CMSIS-DAP 调试器 | — | 硬件调试接口（SWD） |

确保 `arm-none-eabi-gcc`、`make`、`openocd` 已添加到 `PATH`。

## 快速开始

### 1. 克隆

```bash
git clone https://github.com/jrtxio/led-gcc-stm32.git
cd led-gcc-stm32
```

### 2. 构建

```bash
make all
```

输出文件生成在 `build/` 目录下：

| 文件 | 说明 |
|------|------|
| `build/LED.elf` | ELF 可执行文件（用于调试） |
| `build/LED.hex` | Intel HEX 格式（部分烧录工具使用） |
| `build/LED.bin` | 原始二进制 |
| `build/LED.map` | 链接映射文件 |

### 3. 烧录

通过 SWD 将 CMSIS-DAP 调试器连接到目标板，然后执行：

```bash
make flash
```

该命令调用 OpenOCD，使用自定义配置 `stm32yt32m1.cfg`，烧录 `build/LED.elf`、校验并复位 MCU。

### 4. 在 VS Code 中调试

按 `F5` 启动 Cortex-Debug 调试会话。调试器将：
- 使用 `stm32yt32m1.cfg` 启动 OpenOCD
- 加载 `build/LED.elf`
- 运行到 `main()` 入口处
- 启用 `STM32F103xx` SVD 外设寄存器视图

### 5. 清理

```bash
make clean
```

## Make 目标

| 目标 | 说明 |
|------|------|
| `all`（默认） | 构建 `.elf`、`.hex`、`.bin` |
| `flash` | 通过 OpenOCD 将 `.elf` 烧录到目标芯片 |
| `clean` | 删除 `build/` 目录 |

## 硬件配置

- **MCU**：STM32F103C8T6（Cortex-M3，72 MHz，64 KB Flash，20 KB SRAM）
- **开发板**：YT32M1（Blue Pill 兼容）
- **时钟**：HSE 8 MHz，PLL x9 = 72 MHz 系统时钟
- **调试器**：CMSIS-DAP，通过 SWD 连接（适配器速度 20 MHz）
- **LED GPIO**：PB0（在 `LED.ioc` / `Core/Src/gpio.c` 中配置）

## OpenOCD 配置

`stm32yt32m1.cfg` 文件为 YT32M1 开发板配置 OpenOCD：

```
source [find interface/cmsis-dap.cfg]
transport select swd
source [find target/stm32f1x.cfg]
adapter speed 20000
```

如果使用其他调试器（ST-Link、J-Link 等），请相应修改此文件。

## 项目结构

```
.
├── Makefile                          # 构建系统（CubeMX 生成，已修改）
├── LED.ioc                          # STM32CubeMX 工程文件
├── startup_stm32f103xb.s            # 启动汇编（向量表、初始化）
├── STM32F103C8Tx_FLASH.ld           # 链接脚本
├── STM32F103xx.svd                  # SVD 文件（外设寄存器视图）
├── stm32yt32m1.cfg                  # OpenOCD 配置（YT32M1 开发板）
├── Core/
│   ├── Inc/
│   │   ├── main.h                   # 主头文件
│   │   ├── gpio.h                   # GPIO 初始化头文件
│   │   ├── stm32f1xx_hal_conf.h     # HAL 配置
│   │   └── stm32f1xx_it.h          # 中断处理头文件
│   └── Src/
│       ├── main.c                   # 主程序（HAL 初始化、时钟、GPIO）
│       ├── gpio.c                   # GPIO 初始化（LED 引脚）
│       ├── stm32f1xx_hal_msp.c      # HAL MSP 回调
│       ├── stm32f1xx_it.c           # 中断处理
│       └── system_stm32f1xx.c       # 系统时钟初始化
├── Drivers/
│   ├── CMSIS/                       # CMSIS 内核及设备头文件
│   └── STM32F1xx_HAL_Driver/        # HAL 驱动源码及头文件
├── .vscode/
│   ├── launch.json                  # Cortex-Debug 调试配置
│   ├── tasks.json                   # 构建、清理、烧录任务
│   ├── c_cpp_properties.json        # IntelliSense 配置
│   └── settings.json                # 文件关联
└── build/                           # （自动生成）构建输出
```

## VS Code 集成

| 操作 | 快捷键 | 任务 |
|------|--------|------|
| 构建 | `Ctrl+Shift+B` | `make_build` |
| 清理 | 任务运行器 | `make_clean` |
| 烧录 | 任务运行器 | `make_flash` |
| 调试 | `F5` | Cortex-Debug（通过 OpenOCD） |

## 定制

### 更换目标 MCU

1. 在 STM32CubeMX 中修改 `LED.ioc` 并重新生成代码
2. 更新 Makefile：修改 `C_SOURCES`、`C_DEFS`、`C_INCLUDES`、`LDSCRIPT`、`CPU` 和 `ASM_SOURCES`
3. 替换 `startup_stm32f103xb.s` 和 `STM32F103C8Tx_FLASH.ld` 为新文件
4. 更新 `stm32yt32m1.cfg` 和 `STM32F103xx.svd` 以匹配新目标

### 添加源文件

在 `Makefile` 的 `C_SOURCES` 列表中添加新的 `.c` 文件。
