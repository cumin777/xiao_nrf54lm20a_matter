# Claude.md - XIAO nRF54LM20A Matter Sample Adaptation

## Project Overview

This project adapts all Matter application samples in this repository to support the **Seeed XIAO nRF54LM20A** development board. The board definition is located at:

```
D:\workspace\platformio\platform-seeedboards\zephyr\boards\arm\xiao_nrf54lm20a
```

Remote repository: `https://github.com/cumin777/xiao_nrf54lm20a_matter`

---

## Board Hardware Key Differences (XIAO vs nRF54LM20DK)

| Item | nRF54LM20DK | XIAO nRF54LM20A |
|------|-------------|-----------------|
| Board name | `nrf54lm20dk_nrf54lm20a_cpuapp` | `xiao_nrf54lm20a_nrf54lm20a_cpuapp` |
| External flash chip | MX25R64 | **PY25Q64HA** (jedec-id: 85 20 17) |
| External flash size | 64Mb (8MB) | 64Mb (8MB / 67108864 bytes / 0x8000000) |
| External flash SPI bus | spi00 | spi00 (same) |
| Factory data | Enabled by default | **Temporarily disabled** |
| LEDs | Board-level LEDs + PWM | No on-board LEDs suitable for Matter indications |
| PWM pins | Port 1.25 | Port 1.22/1.23/1.24 (pinctrl defined in board DTS) |
| Watchdog | wdt31 | wdt31 (same) |

---

## Samples Requiring Adaptation (11 total)

Only **light_switch** already has XIAO support. The remaining 10 samples need adaptation:

1. **closure** - PWM LED support for closure control
2. **contact_sensor** - Basic sensor, no special peripherals
3. **light_bulb** - PWM LED support required
4. **lock** - Basic lock actuator
5. **manufacturer_specific** - Basic sample
6. **smoke_co_alarm** - Basic alarm
7. **temperature_sensor** - Basic sensor
8. **template** - Template/sample, minimal config
9. **thermostat** - Basic thermostat
10. **window_covering** - 2x PWM LED support required

> **common/** is shared source code only, no board-specific files needed.

---

## Files Required Per Sample

For each of the 10 samples, the following files must be created (modeled after `light_switch`):

### 1. Board Overlay
**Path:** `<sample>/boards/xiao_nrf54lm20a_nrf54lm20a_cpuapp.overlay`

```dts
/ {
    chosen {
        nordic,pm-ext-flash = &py25q64;
    };
    aliases {
        watchdog0 = &wdt31;
    };
};
&py25q64 {
    status = "okay";
    size = <0x8000000>;
};
&wdt31 {
    status = "okay";
};
```

**Special cases (PWM LED):**
- `closure` and `light_bulb`: Add `pwm-led1` alias + `pwmleds` node (use XIAO pinctrl `pwm20` pins: 1.22/1.23/1.24)
- `window_covering`: Add `pwm-led1` and `pwm-led2` aliases + `pwmleds` nodes

### 2. Board Config
**Path:** `<sample>/boards/xiao_nrf54lm20a_nrf54lm20a_cpuapp.conf`

```ini
# Temporary bring-up: disable factory data until board adaptation is confirmed stable
CONFIG_CHIP_FACTORY_DATA=n
CONFIG_CHIP_FACTORY_DATA_BUILD=n
```

### 3. Partition Manager Static File
**Path:** `<sample>/pm_static_xiao_nrf54lm20a_nrf54lm20a_cpuapp.yml`

Copy from `light_switch/pm_static_xiao_nrf54lm20a_nrf54lm20a_cpuapp.yml` (identical content for all samples).

### 4. MCUboot Board Config
**Path:** `<sample>/sysbuild/mcuboot/boards/xiao_nrf54lm20a_nrf54lm20a_cpuapp.conf`

Copy from `light_switch/sysbuild/mcuboot/boards/xiao_nrf54lm20a_nrf54lm20a_cpuapp.conf` (identical content - SPI NOR, tickless kernel, etc.).

### 5. MCUboot Board Overlay
**Path:** `<sample>/sysbuild/mcuboot/boards/xiao_nrf54lm20a_nrf54lm20a_cpuapp.overlay`

```dts
/ {
    chosen {
        nordic,pm-ext-flash = &py25q64;
    };
};
&py25q64 {
    status = "okay";
    size = <67108864>;
};
```

---

## Implementation Steps

### Step 1: Basic samples (no PWM needed)
Create the 5 files listed above for these 7 samples:
- [x] contact_sensor
- [x] lock
- [x] manufacturer_specific
- [x] smoke_co_alarm
- [x] temperature_sensor
- [x] template
- [x] thermostat

### Step 2: PWM samples
Create the 5 files (with PWM overlay additions) for these 3 samples:
- [x] closure (1x PWM LED)
- [x] light_bulb (1x PWM LED)
- [x] window_covering (2x PWM LED)

### Step 3: Verify light_switch Kconfig.sysbuild already covers XIAO
- [x] Confirm `BOARD_XIAO_NRF54LM20A` is present in all samples' Kconfig.sysbuild (currently only `light_switch/Kconfig.sysbuild:64` has it)
- [x] Added `BOARD_XIAO_NRF54LM20A` to all 10 remaining samples' Kconfig.sysbuild

### Step 4: Build verification
- [ ] Attempt build for each sample targeting `xiao_nrf54lm20a_nrf54lm20a_cpuapp`

---

## Development Conventions (SuperMCP)

All development in this project **must** follow the SuperMCP development specification:

1. **One task at a time** - Complete and verify each sample adaptation before moving to the next.
2. **Commit after each verified change** - After creating the board files for a sample AND confirming they are correct, immediately commit and push to the remote repository.
3. **Incremental progress tracking** - Update the progress checklist in this file after each push.
4. **No speculative changes** - Only create files that are directly needed. Do not refactor, clean up, or modify existing code unless necessary for the adaptation.
5. **Push discipline** - Every effective and verified modification must be pushed to `https://github.com/cumin777/xiao_nrf54lm20a_matter` immediately.
6. **Progress log** - Record each completed step in the Progress Log section below.

---

## Progress Log

| Date | Sample | Action | Status |
|------|--------|--------|--------|
| 2025-05-25 | light_switch | Already adapted (reference implementation) | Done |
| 2025-05-25 | contact_sensor | Board files created (5 files), committed | Done |
| 2025-05-25 | lock | Board files created (5 files), committed | Done |
| 2025-05-25 | manufacturer_specific | Board files created (5 files), committed | Done |
| 2025-05-25 | smoke_co_alarm | Board files created (5 files), committed | Done |
| 2025-05-25 | temperature_sensor | Board files created (5 files), committed | Done |
| 2025-05-25 | template | Board files created (5 files), committed | Done |
| 2025-05-25 | thermostat | Board files created (5 files), committed | Done |
| 2025-05-25 | closure | Board files created (5 files), committed | Done |
| 2025-05-25 | light_bulb | Board files created (5 files), committed | Done |
| 2025-05-25 | window_covering | Board files created (5 files), committed | Done |
| 2025-05-25 | All samples | Added BOARD_XIAO_NRF54LM20A to Kconfig.sysbuild (10 files) | Done |

---

## Reference: light_switch File Structure (Template)

```
light_switch/
  boards/
    xiao_nrf54lm20a_nrf54lm20a_cpuapp.conf          # Factory data disabled
    xiao_nrf54lm20a_nrf54lm20a_cpuapp.overlay        # py25q64 ext-flash, wdt31
  pm_static_xiao_nrf54lm20a_nrf54lm20a_cpuapp.yml   # Partition layout
  pm_static_xiao_nrf54lm20a_nrf54lm20a_cpuapp_debug.yml  # Same partition layout
  sysbuild/mcuboot/boards/
    xiao_nrf54lm20a_nrf54lm20a_cpuapp.conf           # SPI NOR, tickless kernel
    xiao_nrf54lm20a_nrf54lm20a_cpuapp.overlay        # py25q64 ext-flash for MCUboot
```
