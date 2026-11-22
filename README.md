# rp2040-funprog

RP2040 firmware implementation of **funprog** - an embedded programmer used by [ch32fun/minichlink](submodules/ch32fun) to program and debug microcontrollers.

This is the Raspberry Pi Pico (RP2040) port of the funprog programmer firmware, which supports programming various WCH microcontroller families (CH32V, CH33x, CH57x, CH58x, etc.). The ch32fun repository is included as a submodule and contains the host-side tools and examples that work with this firmware.

## Overview

**funprog** is a protocol-based programmer that communicates with host software to program and debug embedded systems. This RP2040 implementation provides:
- SWD (Serial Wire Debug) protocol support
- CH5xx/CH32x family chip programming
- UART and USB communication interfaces
- Flexible GPIO-based protocol implementation

See [ch32fun documentation](submodules/ch32fun/README.md) for details on the host-side software and supported chip families.
