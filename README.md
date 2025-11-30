# Voron 2.4 R2 with Octopus pro running klipper

## **:warning: Still a work in progress not finished yet ! :warning:**

Voron 2.4 R2 with Stealthburner, Voron Tap, Bambu hotend.
Octopus pro with TMC2209 UART, EBB SB2209 v1.0 and U2C v2.1 running klipper

> ## For installation steps check the [Wikipage](https://github.com/Qballjos/Voron-2.4-Otopus-pro-SB2209-U2C-CAN/wiki)

## ⚠️ Important Installation Notes

### Setup Overview

**This configuration uses a Hybrid approach:**
- **Octopus Pro**: Connected via USB serial (mainboard communication)
- **SB2209 Toolhead**: Connected via CAN bus (toolhead communication)
- **U2C**: USB-to-CAN bridge for the toolhead

#### Setup Requirements:
- **Octopus Pro v1.1** (or compatible version) - Connected via USB
- **U2C v2.1** (or compatible version) - USB-to-CAN bridge
- **EBB SB2209 v1.0** - Connected via CAN bus
- **U2C Configuration**: Remove the 120R jumper on the U2C board

---

### Complete Installation Guide

#### Step 1: U2C Configuration
1. **Remove the 120R termination resistor jumper** on the U2C board
   - This is required for proper CAN bus termination
   - The jumper is typically located near the CAN bus connector
2. Flash U2C firmware if needed:
   - Use `U2C_V2_STM32G0B1.bin` from the `u2c/` folder
   - Follow U2C manual for flashing instructions

#### Step 2: Octopus Pro Firmware Setup

**Note**: For USB serial setup, Katapult is **NOT required**. You can flash directly via DFU or SD card.

1. **Compile Klipper firmware** for Octopus Pro:
   - Micro-controller: `STM32F446`
   - Bootloader offset: `32KiB bootloader`
   - Clock Reference: `12 MHz crystal` (enable "extra low-level configuration options")
   - Communication interface: `USB (on PA11/PA12)` - This setup uses USB serial for the mainboard

2. **Flash the Octopus Pro** - Choose one method:

   **Method A: SD Card (Easiest)**
   - Copy the generated `klipper.bin` file to an SD card, rename it to `firmware.bin`
   - Insert SD card into Octopus Pro
   - Power on or restart the board
   - The firmware will flash automatically

   **Method B: DFU Mode**
   - Put the board in DFU mode (usually BOOT0 button + reset)
   - Flash using `make flash FLASH_DEVICE=0483:df11` or use STM32CubeProgrammer
   - Verify device ID with `lsusb` first

3. **Connection**: After flashing, connect the Octopus Pro via **USB** to your Raspberry Pi
   - This setup uses a hybrid approach: USB for mainboard, CAN bus for toolhead
   - The Octopus Pro will communicate via USB serial
   - The SB2209 toolhead will communicate via CAN bus

#### Step 3: EBB SB2209 Firmware Setup

**Note**: For CAN bus toolhead, **Katapult IS recommended** to enable easy firmware updates over CAN bus.

1. **Install Katapult** (if not already installed):
   ```bash
   cd ~
   git clone https://github.com/Arksine/katapult
   ```

2. **Flash Katapult bootloader to SB2209** (via USB/DFU first):
   - Configure Katapult for STM32G0B1, 8MHz crystal, CAN bus
   - Put SB2209 in DFU mode
   - Flash Katapult using `dfu-util` (see wiki for details)

3. **Compile Klipper firmware** for EBB SB2209:
   - Micro-controller: `STM32G0B1`
   - Clock Reference: `8 MHz crystal`
   - Communication interface: `CAN bus (on PB0/PB1)`

4. **Flash Klipper to SB2209**:
   - **Option A**: Use pre-compiled firmware from `ebb/SB2209/` folder via USB/DFU
   - **Option B**: After connecting via CAN bus, use Katapult to flash over CAN:
     ```bash
     python3 ~/katapult/scripts/flashtool.py -i can0 -u YOUR_UUID -f ~/klipper/out/klipper.bin
     ```
   - Connect SB2209 to CAN bus network after initial flash

#### Step 4: CAN Bus Connection Setup
1. **Physical connections**:
   - Connect U2C to Raspberry Pi via USB
   - Connect Octopus Pro to Raspberry Pi via USB (for mainboard communication)
   - Connect EBB SB2209 to CAN bus network via U2C (CANH/CANL wires)
   - Ensure proper CAN bus termination (120R at each end of the bus)
   - **Note**: This setup uses USB for the mainboard and CAN bus only for the toolhead
2. **Power up sequence**:
   - Power on U2C first
   - Power on Octopus Pro (via USB connection)
   - Power on EBB SB2209
3. **Verify CAN bus connection**:
   ```bash
   ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
   ```
   - You should see the CAN UUIDs of connected devices
   - Note the UUID for your EBB SB2209 (example: `a863bc1811c2`)

#### Step 5: Klipper Configuration

**This setup uses a Hybrid approach:**
- **Octopus Pro**: Connected via USB serial (mainboard)
- **SB2209**: Connected via CAN bus (toolhead)

1. **Update `printer.cfg` main `[mcu]` section**:
   - Find your Octopus Pro serial path: `ls -l /dev/serial/by-id/`
   - Update the `serial:` line with your device path
   - Example: `serial: /dev/serial/by-id/usb-Klipper_stm32f446xx_XXXXXXXX-if00`
   - Keep `restart_method: command`

2. **Update `SB2209.cfg`**:
   - Find your SB2209 UUID using: `~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`
   - Update `canbus_uuid` in `SB2209.cfg` with your EBB's UUID
   - Example: `canbus_uuid: a863bc1811c2`

3. **Verify all include paths are correct** in `printer.cfg`

4. **Restart Klipper** and verify all MCUs connect:
   ```bash
   sudo systemctl restart klipper
   ```

5. **Check Klipper logs** for any connection errors:
   ```bash
   tail -f ~/printer_data/logs/klippy.log
   ```

#### Troubleshooting
- **Octopus Pro not detected**: Verify USB connection and check serial path with `ls -l /dev/serial/by-id/`
- **EBB not detected**: Verify CAN UUID is correct in `SB2209.cfg` and check with `canbus_query.py can0`
- **U2C issues**: Check that 120R jumper is removed on U2C
- **Connection errors**: Verify CAN bus wiring (CANH/CANL) and termination resistors
- **CAN bus not working**: Ensure SB2209 is properly connected to CAN bus network via U2C

> **Note**: This setup uses USB for the mainboard (Octopus Pro) and CAN bus only for the toolhead (SB2209). This hybrid approach is simpler and works well for most setups.

![voron 2 4 R2](https://user-images.githubusercontent.com/1911646/210006979-284c8834-5c52-45b6-8e7a-231ccd9c2b9e.jpeg)

![U2C-Octopus-SB2209](https://user-images.githubusercontent.com/1911646/211887538-b0239d62-0468-4e26-8723-45eb765b60e9.jpg)

links to original sites:

[Bigtreetech EBB](https://github.com/bigtreetech/EBB)

[Bigtreetech U2C](https://github.com/bigtreetech/U2C)

[Octopus](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-V1.0) & [Octopus Pro](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-Pro)

[Grabcat](https://grabcad.com/library)

[Voron](https://docs.vorondesign.com)

[Maz0r](https://github.com/maz0r)

[Lab4450](https://lab4450.com)

And the ones I forgot to mention here :smile:
