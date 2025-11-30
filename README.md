# Voron 2.4 R2 with Octopus pro running klipper

## **:warning: Still a work in progress not finished yet ! :warning:**

Voron 2.4 R2 with Stealthburner, Voron Tap, Bambu hotend.
Octopus pro with TMC2209 UART, EBB SB2209 v1.0 and U2C v2.1 running klipper

> ## For installation steps check the [Wikipage](https://github.com/Qballjos/Voron-2.4-Otopus-pro-SB2209-U2C-CAN/wiki)

## ⚠️ Important Installation Notes

### CAN Bus Setup - Critical Steps

**After flashing the Octopus Pro in DFU mode, you MUST connect it via CAN bus. This is the only way the CAN network will work properly.**

#### Setup Requirements:
- **Octopus Pro v1.1** (or compatible version)
- **U2C v2.1** (or compatible version)  
- **EBB SB2209 v1.0** (or EBB36)
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
1. **Compile Klipper firmware** for Octopus Pro:
   - Micro-controller: `STM32F446`
   - Bootloader offset: `32KiB bootloader`
   - Clock Reference: `12 MHz crystal` (enable "extra low-level configuration options")
   - Communication interface: Configure for CAN bus operation
2. **Flash the Octopus Pro** in DFU mode:
   - Put the board in DFU mode (usually BOOT0 button + reset)
   - Flash using `make flash FLASH_DEVICE=0483:df11` or use STM32CubeProgrammer
   - Or copy `firmware.bin` to SD card and restart
3. **⚠️ CRITICAL**: After flashing in DFU mode, **connect via CAN bus (not USB)**
   - Do NOT connect via USB after flashing
   - The CAN network will ONLY work when connected via CAN bus
   - This is the most common mistake - USB connection will prevent CAN bus from working

#### Step 3: EBB SB2209 Firmware Setup
1. **Compile Klipper firmware** for EBB SB2209:
   - Micro-controller: `STM32G0B1`
   - Clock Reference: `8 MHz crystal`
   - Communication interface: `CAN bus (on PB0/PB1)`
2. **Flash EBB SB2209 firmware**:
   - Use pre-compiled firmware from `ebb/SB2209/` folder:
     - `firmware_canbus.bin` - Standard CAN bus firmware
     - `firmware_canbus_8k_bootloader.bin` - For 8KiB bootloader
   - Follow EBB SB2209 manual for flashing instructions
   - Can be flashed via USB first, then switch to CAN bus

#### Step 4: CAN Bus Connection Setup
1. **Physical connections**:
   - Connect U2C to Raspberry Pi via USB
   - Connect Octopus Pro to U2C via CAN bus (CANH/CANL)
   - Connect EBB SB2209 to CAN bus network (daisy chain or via U2C)
   - Ensure proper CAN bus termination (120R at each end of the bus)
2. **Power up sequence**:
   - Power on U2C first
   - Power on Octopus Pro (via CAN bus, not USB)
   - Power on EBB SB2209
3. **Verify CAN bus connection**:
   ```bash
   ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
   ```
   - You should see the CAN UUIDs of connected devices
   - Note the UUID for your EBB SB2209 (example: `a863bc1811c2`)

#### Step 5: Klipper Configuration

**Choose your connection method:**

**Option A: Full CAN Bus Setup (Recommended)**
- Octopus Pro configured as USB-CAN-Bridge (on CAN bus)
- SB2209 on CAN bus
- Both use `canbus_uuid` in config
- Update `printer.cfg` main `[mcu]` section to use `canbus_uuid: YOUR_OCTOPUS_UUID` (no `serial:` or `restart_method:`)
- Update `canbus_uuid` in `SB2209.cfg` with your EBB's UUID

**Option B: Hybrid Setup (Current config)**
- Octopus Pro on USB serial connection
- SB2209 on CAN bus
- Update `printer.cfg` main `[mcu]` section with correct serial path (check with `ls -l /dev/serial/by-id/`)
- Update `canbus_uuid` in `SB2209.cfg` with your EBB's UUID

**Note**: The current `backup/printer.cfg` uses Option B (hybrid). If you want Option A (full CAN), you'll need to:
1. Find the Octopus Pro UUID using `canbus_query.py can0`
2. Change the `[mcu]` section from `serial:` to `canbus_uuid:`
2. **Restart Klipper** and verify all MCUs connect:
   ```bash
   sudo systemctl restart klipper
   ```
3. **Check Klipper logs** for any connection errors:
   ```bash
   tail -f ~/printer_data/logs/klippy.log
   ```

#### Troubleshooting
- **CAN bus not working**: Ensure Octopus Pro is connected via CAN bus (not USB) after flashing
- **EBB not detected**: Verify CAN UUID is correct in `SB2209.cfg`
- **U2C issues**: Check that 120R jumper is removed
- **Connection errors**: Verify CAN bus wiring (CANH/CANL) and termination resistors

> **Note**: Many installation points are not obvious. The critical step is connecting Octopus Pro via CAN bus (not USB) after initial firmware flash. This is the only way the CAN network will work properly.

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
