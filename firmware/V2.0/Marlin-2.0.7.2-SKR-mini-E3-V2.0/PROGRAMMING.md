# Programming the SKR Mini E3 V2.0

## Prerequisites

- STLink v2 (or clone)
- OpenOCD: `brew install openocd`

## Erasing the STM32F103 via SWD

### Wiring

Connect the STLink to the 4-pin **SW** header on the board (near the STM32):

| SW Header | STLink |
|-----------|--------|
| GND       | GND    |
| SWCLK     | SWCLK  |
| SWDIO     | SWDIO  |
| 3.3V      | 3.3V   |

> **Note:** STLink clone boards often have SWCLK and SWDIO silkscreened in the wrong order. If the erase command fails to connect, swap those two wires and retry.

### Erase command

```bash
openocd -f interface/stlink.cfg -f target/stm32f1x.cfg \
  -c "adapter speed 240; init; halt; stm32f1x mass_erase 0; exit"
```

Use `halt` rather than `reset halt` — a blank or corrupt chip often sits in a HardFault loop that causes `reset halt` to time out.

### Expected output

```
Info : [stm32f1x.cpu] Cortex-M3 r1p1 processor detected
Info : device id = 0x10036414
Info : flash size = 512 KiB
```

A clean exit (no `Error:` lines) means the erase succeeded.

## Building and flashing firmware

### Prerequisites

- PlatformIO: `pip install platformio`
- A `.tool-versions` file in this directory setting Python (required by the Marlin build scripts):

```
python 3.11.10
```

### Build

```bash
pio run
```

### Flash

PlatformIO's built-in `pio run --target upload` uses `reset halt` internally, which times out on a blank or freshly-soldered chip stuck in a HardFault loop. Use OpenOCD directly instead:

```bash
openocd -f interface/stlink.cfg -f target/stm32f1x.cfg \
  -c "adapter speed 240; init; halt; \
      flash write_image erase .pio/build/STM32F103RC_btt/firmware.elf; \
      reset run; exit"
```

### Expected output

```
Info : [stm32f1x.cpu] Cortex-M3 r1p1 processor detected
Info : flash size = 512 KiB
Warn : Adding extra erase range, ...
[stm32f1x.cpu] halted due to breakpoint, current mode: Thread
```

The final `current mode: Thread` line confirms the chip has reset and is running the new firmware.
