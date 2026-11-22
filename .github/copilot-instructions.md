# Copilot Instructions for rp2040-funprog

## Overview

This is a Raspberry Pi Pico (RP2040) firmware project that serves as an embedded programmer/bootloader for microcontrollers. The project uses the Pico SDK, CMake for builds, and includes both C++ and PIO (Programmable I/O) code.

## Build & Test Commands

### Prerequisites
Ensure you have:
- ARM cross-compilation toolchain installed
- CMake 3.13+
- Pico SDK installed and `PICO_SDK_PATH` environment variable set
- Ninja build system (or Make as alternative)

### Building
```bash
cd /path/to/rp2040-funprog
mkdir -p build
cd build
cmake ..
cmake --build .
```

### Output Artifacts
After build, the following files are generated in `build/`:
- `rp2040-funprog.uf2` - Flashable firmware (drag-and-drop to Pico in bootloader mode)
- `rp2040-funprog.elf` - ELF binary (for debugging with debuggers)
- `rp2040-funprog.hex` - Hex format
- `rp2040-funprog.bin` - Raw binary
- `rp2040-funprog.dis` - Disassembly

### Flashing to Device
1. Connect Raspberry Pi Pico while holding BOOTSEL button
2. Wait for it to appear as a mass storage device
3. Copy `build/rp2040-funprog.uf2` to the device or use:
   ```bash
   picotool load build/rp2040-funprog.elf
   ```

## Project Architecture

### Main Components

**src/rp2040-funprog.cpp**
- Main firmware entry point (66 lines)
- Initializes Pico peripherals (stdio, PIO, UART)
- Demonstrates PIO-based LED blinking on GPIO 6 (or `PICO_DEFAULT_LED_PIN`)
- Sets up UART1 on pins 4 (TX) and 5 (RX) at 115200 baud
- Main loop prints "Hello, world!" every 1000ms

**src/blink.pio**
- Programmable I/O (PIO) state machine assembly program
- Implements a blink loop using counters and GPIO control
- Generated header (`blink.pio.h`) is created at build time by `pico_generate_pio_header()`
- PIO runs independently of CPU, allows precise timing control
- Uses `pull block` to get frequency parameters from CPU via TX FIFO

**include/funprog_config.h**
- Feature flags for the programmer (CH5xx support, USB HID, debugging, etc.)
- Bit-field capability macros for feature detection
- Programmer type selections (ESP32-S2, CH32V003, CH570, etc.)
- Configuration constants like UART timeout values
- Support for SWD protocol, UPDI, and various USB interface modes

### Hardware I/O

| Pin | Function | Details |
|-----|----------|---------|
| GPIO 4 | UART1 TX | Serial output at 115200 baud |
| GPIO 5 | UART1 RX | Serial input at 115200 baud |
| GPIO 6 (default) | LED output | Controlled via PIO state machine, 3 Hz default |

### Build System

**CMakeLists.txt**
- Configures C11/C++17 standards
- Imports Pico SDK via `pico_sdk_import.cmake`
- Sets up PIO program generation with `pico_generate_pio_header()`
- Disables stdio over UART/USB by default (can be enabled by changing `pico_enable_stdio_uart(rp2040-funprog 0)` to `1`)
- Links hardware libraries: `hardware_pio`
- Generates additional outputs via `pico_add_extra_outputs()` (elf, hex, bin, dis, uf2)

**pico_sdk_import.cmake**
- Standard Pico SDK import script
- Locates SDK from `PICO_SDK_PATH` environment variable or default installation paths

## Key Conventions

### PIO Programs
- PIO programs are defined in `.pio` files using PIO assembly syntax
- Generated C headers (via `pico_generate_pio_header()`) provide `*_program_init()` functions and program definitions
- PIO state machines run independently of the CPU
- Control via `pio_sm_set_enabled()`, `pio_sm_put()` for data transfer
- Counter calculations: `(clock_freq / (2 * desired_freq)) - 3` accounts for PIO instruction overhead

### Configuration
- Hardware pin assignments are defined in `funprog_config.h` and at the top of `.cpp` files
  - e.g., `UART_TX_PIN`, `UART_RX_PIN`, `BAUD_RATE`
- Feature toggles (debug prints, protocol support) are preprocessor defines
- Capability bits can be queried via `CAPABILITIES` macro for runtime feature detection
- `DEBUG_PRINT` enables `FUN_PRINTF_DEBUG` macro output

### UART Communication
- Uses hardware UART1 (secondary UART to avoid conflicts with stdio)
- Always pair TX and RX pins according to datasheet function select tables
- Default baud rate is 115200; configure via `BAUD_RATE` define
- Use `gpio_set_function()` to assign pins to UART function select

### Code Style
- C++ source files (.cpp)
- Pico SDK APIs for hardware abstraction
- Comments explain non-obvious timing calculations and register usage

## Debugging Tips

- Enable `DEBUG_PRINT` in `funprog_config.h` to activate `FUN_PRINTF_DEBUG` macro output
- Use `printf()` for debug output (goes to stdio, which may be UART/USB depending on configuration)
- Generated `compile_commands.json` in build/ supports LSP and debuggers
- Disassembly (`rp2040-funprog.dis`) helpful for understanding timing-critical sections
- Use `picotool` for on-device debugging and inspection

## Common Tasks

**Add a new PIO program:**
1. Create `.pio` file in `src/`
2. Add `pico_generate_pio_header()` call in CMakeLists.txt
3. Include generated header in source code
4. Call `*_program_init()` and control state machine via SDK APIs

**Change GPIO pins:**
- Update `#define` macros in `.cpp` files or `funprog_config.h`
- Verify pin function select compatibility with the RP2040 datasheet

**Enable printf over serial:**
- Change `pico_enable_stdio_uart(rp2040-funprog 0)` to `1` in CMakeLists.txt
- Rebuild and check output on the configured UART pins

**Modify capabilities:**
- Adjust feature flags in `funprog_config.h`
- Rebuild to include/exclude protocol support (CH5xx, UPDI, USB modes, etc.)
