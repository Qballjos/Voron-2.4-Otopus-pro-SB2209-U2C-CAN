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

#### Flashing Process:
1. Flash the Octopus Pro firmware in DFU mode
2. **IMPORTANT**: After flashing, connect the Octopus Pro via CAN bus (not USB)
3. The CAN network will only work when connected via CAN bus after initial flash
4. Verify CAN bus connection before proceeding with configuration

#### U2C Configuration:
- Remove the 120R termination resistor jumper on the U2C board
- This is required for proper CAN bus termination

> **Note**: Many installation points are not obvious. If you encounter issues, ensure you've followed the CAN bus connection step after flashing.

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
