# XIAO nRF54LM20A Matter 样例使用指南

本文档介绍所有已适配 XIAO nRF54LM20A 开发板的 Matter 样例，包括每个样例的功能说明、环境搭建方法及验证步骤。

---

## 目录

1. [环境搭建](#环境搭建)
2. [样例总览](#样例总览)
3. [各样例详细说明](#各样例详细说明)
4. [通用验证流程](#通用验证流程)
5. [构建产物说明](#构建产物说明)
6. [常见问题](#常见问题)

---

## 环境搭建

### 前置条件

| 项目 | 要求 |
|------|------|
| 操作系统 | Windows 10/11 + WSL2，或 Linux |
| nRF Connect SDK | v3.2.1（nCS） |
| 工具链 | nRF Connect SDK 自带工具链 |
| 开发板 | Seeed XIAO nRF54LM20A |
| USB 线缆 | Type-C（用于供电和烧录） |
| 串口工具 | minicom / picocom / PuTTY（波特率 115200） |

### 方式一：使用 nRF Connect for VS Code（推荐新手）

1. 安装 [VS Code](https://code.visualstudio.com/)
2. 安装 [nRF Connect for VS Code 扩展](https://marketplace.visualstudio.com/items?itemName=nordic-semiconductor.nrf-connect)
3. 使用扩展安装 nRF Connect SDK v3.2.1
4. 将 XIAO nRF54LM20A 板级定义添加到 SDK（参考 [CLAUDE.md](CLAUDE.md) 中 Board 定义路径）
5. 在 VS Code 中打开本仓库目录
6. 使用 nRF Connect 侧边栏选择样例、选择板子 `xiao_nrf54lm20a/nrf54lm20a/cpuapp`，点击 Build

### 方式二：命令行构建（WSL/Linux）

#### 1. 安装 nRF Connect SDK v3.2.1

```bash
# 安装 west
pip3 install west

# 初始化 nCS 工作区
west init -m https://github.com/nrfconnect/sdk-nrf --mr v3.2.1 ~/ncs/v3.2.1
cd ~/ncs/v3.2.1
west update
```

#### 2. 添加 XIAO 板级定义

```bash
# 将板级定义复制到 Zephyr boards 目录
cp -r <board-definition-path>/xiao_nrf54lm20a \
      ~/ncs/v3.2.1/zephyr/boards/seeed/xiao_nrf54lm20a
```

#### 3. 构建任意样例

```bash
cd ~/ncs/v3.2.1

# 构建指定样例
west build -p -b xiao_nrf54lm20a/nrf54lm20a/cpuapp \
    /mnt/d/workspace/matter_20a/<sample_name>
```

#### 4. 烧录

```bash
west flash --erase
```

### 方式三：一键构建脚本

在 WSL 中可使用以下脚本批量构建：

```bash
#!/bin/bash
set -e
TOOLCHAIN_ROOT=$HOME/ncs/toolchains/<your-toolchain-hash>
export ZEPHYR_BASE=$HOME/ncs/v3.2.1/zephyr
export ZEPHYR_TOOLCHAIN_VARIANT=zephyr
export ZEPHYR_SDK_INSTALL_DIR=$TOOLCHAIN_ROOT/opt/zephyr-sdk
export PATH=$TOOLCHAIN_ROOT/bin:$TOOLCHAIN_ROOT/usr/bin:$TOOLCHAIN_ROOT/usr/local/bin:$PATH
export PYTHONHOME=$TOOLCHAIN_ROOT/usr/local

SAMPLE=$1
cd $HOME/ncs/v3.2.1
west build -d /tmp/build_xiao_${SAMPLE} \
    -b xiao_nrf54lm20a/nrf54lm20a/cpuapp \
    /mnt/d/workspace/matter_20a/${SAMPLE} --pristine
```

使用方式：

```bash
bash build_xiao.sh contact_sensor
bash build_xiao.sh light_bulb
# ... 依此类推
```

---

## 样例总览

| 样例 | Matter 设备类型 | 功能概述 | 需要外设 | 复杂度 |
|------|----------------|----------|----------|--------|
| **template** | 最小 Matter 设备 | 入门模板，仅配网功能 | 无 | 入门 |
| **light_bulb** | 可调光灯泡 | PWM 控制灯的开关和亮度 | LED（板载） | 基础 |
| **light_switch** | 灯开关 | 绑定灯泡设备并远程控制 | 需配合 light_bulb | 基础 |
| **lock** | 门锁 | 模拟门锁的开/关操作 | 无 | 基础 |
| **contact_sensor** | 接触传感器 | 检测接触状态（按钮模拟） | 无 | 基础 |
| **temperature_sensor** | 温度传感器 | 模拟温度数据上报 | 无 | 基础 |
| **thermostat** | 恒温器 | 温度监控与控制 | 可选外接温度传感器 | 中等 |
| **window_covering** | 窗帘/百叶窗 | 升降和倾斜控制（2x PWM） | LED（板载） | 中等 |
| **closure** | 闭合装置（如车库门） | 位置和速度控制（1x PWM） | LED（板载） | 中等 |
| **smoke_co_alarm** | 烟雾/CO 报警器 | 多级报警演示 | 无 | 中等 |
| **manufacturer_specific** | 自定义厂商集群 | 演示自定义 Matter 集群开发 | 无 | 进阶 |

---

## 各样例详细说明

### 1. template（模板）

**功能**：最小的 Matter 设备实现，仅包含配网（commissioning）能力。可作为开发自己 Matter 应用的起点。

**用户界面**：
- 状态 LED 闪烁表示设备广播状态
- 按住 Button 0 超过 6 秒恢复出厂设置

**验证方式**：
1. 烧录后设备自动开始 BLE 广播（LED 短闪）
2. 使用 chip-tool 配网后，LED 变为短闪灭（Short Flash Off），表示已配网但尚无 IPv6 连接

**chip-tool 示例**：
```bash
# 配网
./chip-tool pairing onnetwork-long <node_id> <pin_code>
# 配网成功后 LED 状态变化
```

---

### 2. light_bulb（灯泡）

**功能**：实现一个可调光（Dimmable）的白色灯泡。通过 PWM 控制 LED 模拟灯泡的开关和亮度调节。该样例也是 light_switch 样例的配套设备。

**用户界面**：
- LED 1：显示灯泡状态（亮/灭）
- Button 1：切换灯泡开关

**验证方式**：
1. 烧录后 LED 默认关闭
2. 按 Button 1 开灯，再按关灯
3. 使用 chip-tool 远程控制：

```bash
# 配网
./chip-tool pairing onnetwork-long 1 34970112332

# 开灯
./chip-tool onoff on 1 1

# 关灯
./chip-tool onoff off 1 1

# 调节亮度（0-254）
./chip-tool levelcontrol move-to-level 128 1 1
```

---

### 3. light_switch（灯开关）

**功能**：实现一个灯开关设备，可以绑定一个或多个灯泡设备并远程控制其状态。支持单播（控制单个灯）和组播（控制一组灯）两种模式。

**用户界面**：
- Button 1：短按切换绑定灯泡的开关，长按调节亮度

**验证方式**：

需要同时准备一个烧录了 light_bulb 的设备。

1. 分别配网两个设备，记录各自的 node ID
2. 设置 ACL 和绑定表：
```bash
# 写入 ACL（允许 light_switch 控制 light_bulb）
./chip-tool accesscontrol write acl '[{"fabricIndex":1,"privilege":5,"authMode":2,"subjects":[112233],"targets":null},{"fabricIndex":1,"privilege":3,"authMode":2,"subjects":[<switch_node_id>],"targets":[{"cluster":6,"endpoint":1,"deviceType":null},{"cluster":8,"endpoint":1,"deviceType":null}]}]' <bulb_node_id> 0

# 写入绑定表
./chip-tool binding write binding '[{"fabricIndex":1,"node":<bulb_node_id>,"endpoint":1,"cluster":6},{"fabricIndex":1,"node":<bulb_node_id>,"endpoint":1,"cluster":8}]' <switch_node_id> 1
```
3. 在 light_switch 设备上按 Button 1，观察 light_bulb 设备的 LED 变化

---

### 4. lock（门锁）

**功能**：模拟一个带基本锁舌的门锁设备。支持 PIN 码认证、定时访问计划等高级功能。

**用户界面**：
- LED 1：显示锁状态（亮=锁定，灭=解锁）
- Button 1：切换锁的开/关状态
- 锁定时 LED 快闪约 2 秒模拟锁舌运动

**验证方式**：
1. 烧录后 LED 1 亮（锁定状态）
2. 按 Button 1 解锁，LED 闪烁 2 秒后熄灭
3. 再按 Button 1 锁定，LED 闪烁 2 秒后常亮

使用 chip-tool：
```bash
# 解锁
./chip-tool doorlock unlock-door <node_id> 1

# 锁定
./chip-tool doorlock lock-door <node_id> 1

# 设置 PIN 码后解锁
./chip-tool doorlock unlock-door <node_id> 1 --PINCode 12345678
```

---

### 5. contact_sensor（接触传感器）

**功能**：模拟一个接触传感器设备（如门窗传感器）。由于没有真实传感器硬件，通过按钮模拟接触/分离检测。设备类型为 ICD（间歇连接设备），支持低功耗 LIT 模式。

**用户界面**：
- LED 1：显示接触检测状态（亮=检测到接触，灭=未检测到）
- Button 1（按住/释放）：模拟接触检测

**验证方式**：
```bash
# 读取接触状态
./chip-tool booleanstate read state-value <node_id> 1
# 返回 StateValue: FALSE（未接触）

# 按住 Button 1 后再次读取
./chip-tool booleanstate read state-value <node_id> 1
# 返回 StateValue: TRUE（已接触）

# 订阅状态变化（实时推送）
./chip-tool booleanstate subscribe state-value 0 300 <node_id> 1
```

---

### 6. temperature_sensor（温度传感器）

**功能**：模拟一个温度传感器设备。温度值从 -20°C 线性递增到 +20°C，每 10 秒更新一次，到达上限后重新从 -20°C 开始。设备类型为 ICD LIT。

**用户界面**：
- LED 0：显示设备连接状态
- Button 0：恢复出厂设置（长按 6 秒）

**验证方式**：
```bash
# 读取当前温度（值 x100，即 1252 表示 12.52°C）
./chip-tool temperaturemeasurement read measured-value <node_id> 1

# 等待 30 秒后再次读取，观察数值变化
./chip-tool temperaturemeasurement read measured-value <node_id> 1

# 订阅温度变化
./chip-tool temperaturemeasurement subscribe measured-value 0 300 <node_id> 1
```

---

### 7. thermostat（恒温器）

**功能**：实现一个恒温器设备，支持本地模拟温度监控和远程温度控制。可绑定外部真实温度传感器获取实际温度数据。

**用户界面**：
- Button 1：打印最新恒温器数据到串口终端
- 设备每 30 秒自动输出模拟温度数据

**验证方式**：
```bash
# 配网后读取系统模式
./chip-tool thermostat read system-mode <node_id> 1

# 设置目标温度（单位 0.01°C）
./chip-tool thermostat write occupied-heating-setpoint 2000 <node_id> 1

# 读取当前温度
./chip-tool thermostat read local-temperature <node_id> 1
```

---

### 8. window_covering（窗帘/百叶窗）

**功能**：实现一个窗帘/百叶窗设备，支持升降（Lift）和倾斜（Tilt）两种运动模式。设备类型为 SSED（同步休眠终端设备），优化了功耗。使用 2 个 PWM LED 分别表示升降位置和倾斜角度。

**用户界面**：
- LED 1（亮度 0-255）：表示升降位置（0=全开，255=全关）
- LED 3（亮度 0-255）：表示倾斜角度（0=水平全开，255=竖直全关）
- Button 1：向开的方向移动一步
- Button 2：向关的方向移动一步
- Button 1 + Button 2 同时按：切换升降/倾斜模式

**验证方式**：
```bash
# 升降控制（0=全开，100=全关）
./chip-tool windowcovering go-to-lift-percentage 50 <node_id> 1

# 倾斜控制
./chip-tool windowcovering go-to-tilt-percentage 75 <node_id> 1

# 停止运动
./chip-tool windowcovering stop-motion <node_id> 1
```

手动验证：按 Button 2 共 20 次将 LED 1 从灭逐渐调到最亮（全关），再按 Button 1 共 20 次调回全灭（全开）。

---

### 9. closure（闭合装置）

**功能**：实现一个闭合装置设备（如车库门），支持预设位置定位和速度控制。使用 1 个 PWM LED 模拟闭合装置的位置状态。

**用户界面**：
- LED 1（亮度）：表示闭合装置位置（灭=全开，亮=全关）
- Button 0：恢复出厂设置（长按 6 秒）

**验证方式**：
```bash
# 订阅运动完成事件
./chip-tool closurecontrol subscribe-event movement-completed 1 5 <node_id> 1

# 移动到指定位置
# Position: 0=FullyClosed, 1=FullyOpened
# Speed: 0=Auto, 1=Low, 2=Medium, 3=High
./chip-tool closurecontrol move-to <node_id> 1 \
    --Position 0 --Speed 1 --timedInteractionTimeoutMs 5000

# 停止运动
./chip-tool closurecontrol stop <node_id> 1 --timedInteractionTimeoutMs 5000
```

---

### 10. smoke_co_alarm（烟雾/CO 报警器）

**功能**：模拟一个烟雾和一氧化碳报警设备。通过 LED 模式展示不同级别的报警。支持以下报警类型（按优先级从高到低）：
1. 烟雾报警
2. CO 报警
3. 硬件故障
4. 自检
5. 寿命终止
6. 电池低电量

**用户界面**：
- LED 1 闪烁（300ms）：烟雾报警
- LED 2 闪烁（300ms）：CO 报警
- LED 3 闪烁（300ms）：电池低电量
- LED 1/2/3 同时短闪：硬件故障
- LED 1/2/3 同时长闪：寿命终止

**验证方式**：
```bash
# 触发自检
./chip-tool smokecoalarm self-test-request <node_id> 1

# 触发烟雾报警（需要 test_event_enable_key）
./chip-tool generaldiagnostics test-event-trigger \
    hex:<test_event_enable_key> 0x005c00000000009c <node_id> 0

# 停止烟雾报警
./chip-tool generaldiagnostics test-event-trigger \
    hex:<test_event_enable_key> 0x005c0000000000a0 <node_id> 0
```

---

### 11. manufacturer_specific（厂商自定义集群）

**功能**：演示如何创建自定义 Matter 集群。包含一个自定义的 `NordicDevkit` 集群，具有以下特性：
- `DevKitName` 属性：可读写的字符串，持久化存储
- `UserLED` 属性：控制 LED 状态
- `UserButton` 属性：反映按钮状态，变化时触发事件
- `SetLED` 命令：控制 LED（0=关，1=开，2=切换）
- `UserButtonChanged` 事件：按钮状态变化通知

此外还扩展了 `Basic Information` 集群：
- `RandomNumber` 属性：随机数
- `GenerateRandom` 命令：生成新随机数

**验证方式**：
```bash
# 使用 chip-tool 交互模式
chip-tool interactive start

# 读取 DevKitName 属性
any read-by-id 0xFFF1FC01 0xFFF10000 1 1

# 写入 DevKitName
any write-by-id 0xFFF1FC01 0xFFF10000 "MyXIAO" 1 1

# 控制LED（1=开）
any command-by-id 0xFFF1FC01 0xFFF10000 '{ "0x0": "u:1" }' 1 1

# 订阅按钮状态
any subscribe-by-id 0xFFF1FC01 0xFFF10002 0 120 1 1
```

---

## 通用验证流程

### 烧录步骤

```bash
# 1. 构建样例
cd ~/ncs/v3.2.1
west build -p -b xiao_nrf54lm20a/nrf54lm20a/cpuapp \
    /path/to/matter_20a/<sample_name>

# 2. 连接 XIAO 开发板（Type-C USB）

# 3. 烧录固件
west flash --erase

# 4. 打开串口终端观察日志
minicom -D /dev/ttyACM0 -b 115200
```

### 配网步骤（以 chip-tool 为例）

```bash
# 1. 构建 chip-tool（如果尚未构建）
cd ~/ncs/v3.2.1/modules/lib/matter
./scripts/examples/gn_build_example.sh examples/chip-tool

# 2. 配网设备（使用默认 PIN 码 34970112332）
./chip-tool pairing onnetwork-long 1 34970112332

# 3. 验证配网成功
./chip-tool descriptor read device-type-list 1 0
```

### 恢复出厂设置

在任何样例中，长按 Button 0 超过 6 秒即可恢复出厂设置。设备重启后重新开始 BLE 广播。

---

## 构建产物说明

构建完成后，产物位于构建目录下：

| 文件 | 说明 |
|------|------|
| `zephyr/zephyr.elf` | ELF 固件文件 |
| `merged.hex` | 合并的 HEX 文件（包含 MCUboot + 应用） |
| `dfu_application.zip` | DFU 升级包（用于 nRF Connect Device Programmer） |
| `matter.ota` | Matter OTA 升级包 |
| `dfu_multi_image.bin` | 多镜像 DFU 二进制文件 |

### 各样例资源占用

| 样例 | FLASH 占用 | RAM 占用 | FLASH 使用率 | RAM 使用率 |
|------|-----------|---------|-------------|-----------|
| template | 742 KB | 169 KB | 37.78% | 32.23% |
| manufacturer_specific | 745 KB | 170 KB | 37.93% | 32.51% |
| contact_sensor | 775 KB | 172 KB | 39.48% | 32.85% |
| temperature_sensor | 775 KB | 172 KB | 39.46% | 32.85% |
| light_switch | 793 KB | 172 KB | 40.39% | 32.80% |
| smoke_co_alarm | 783 KB | 172 KB | 39.89% | 32.96% |
| window_covering | 777 KB | 170 KB | 39.56% | 32.45% |
| thermostat | 803 KB | 170 KB | 40.86% | 32.53% |
| closure | 804 KB | 184 KB | 40.93% | 35.10% |
| light_bulb | 828 KB | 184 KB | 42.17% | 35.17% |
| lock | 808 KB | 175 KB | 41.14% | 33.36% |

> 总可用资源：FLASH 1918 KB，RAM 511 KB

---

## 常见问题

### Q: 构建报错 `chip_codegen.cmake not found`
A: 这是 Matter SDK 预构建依赖缺失的问题。确保使用 nRF Connect SDK v3.2.1 的完整工作区（`west update` 已执行完成），并且 `SB_CONFIG_MATTER=y` 已在 sysbuild 中启用。

### Q: 开发板无法识别
A: 确保板级定义已正确复制到 `zephyr/boards/seeed/xiao_nrf54lm20a` 目录（不能使用符号链接，因为 Python 的 `rglob` 不会跟随符号链接到目录）。

### Q: 板子目标格式是什么？
A: Zephyr v4（nCS v3.2.1）使用斜杠分隔格式：`xiao_nrf54lm20a/nrf54lm20a/cpuapp`。但 overlay 和 conf 文件名仍使用下划线格式：`xiao_nrf54lm20a_nrf54lm20a_cpuapp`。

### Q: Factory Data 被禁用了有影响吗？
A: 当前已临时禁用 Factory Data（`CONFIG_CHIP_FACTORY_DATA=n`），这是为了简化适配验证。配网仍可正常使用默认凭据。正式产品中需要启用并写入真实 Factory Data。

### Q: XIAO 板没有板载 LED，PWM 样例如何验证？
A: XIAO nRF54LM20A 的板级 DTS 中已定义 PWM 引脚（Port 1.22/1.23/1.24），可通过外接 LED 或使用逻辑分析仪/示波器观察 PWM 输出来验证。对于纯功能验证，可通过 chip-tool 命令读取设备属性确认逻辑正确。

### Q: 如何选择样例进行学习？
A: 推荐顺序：
1. **template** - 理解 Matter 最小设备架构
2. **light_bulb** - 学习 PWM 控制和基本交互
3. **light_switch** - 学习设备绑定和 ACL
4. **lock** / **contact_sensor** - 学习不同设备类型
5. **window_covering** / **closure** - 学习多 PWM 控制
6. **manufacturer_specific** - 学习自定义集群开发
