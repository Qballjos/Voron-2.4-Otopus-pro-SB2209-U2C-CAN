# MainsailOS Reinstall Checklist

Quick reference checklist for reinstalling MainsailOS and setting up your Voron 2.4 R2 300mm.

## 📋 Pre-Installation

- [ ] Backup current config files (if accessible)
- [ ] Download latest MainsailOS image
- [ ] Have Raspberry Pi Imager ready
- [ ] Have USB-C cable for U2C
- [ ] Have USB cable for Octopus Pro
- [ ] Have CAN bus wires ready

## 🔧 Installation Steps

### Phase 1: Software Setup
- [ ] Flash MainsailOS to Raspberry Pad 5 with CM4
- [ ] Configure network (Wi-Fi/Ethernet)
- [ ] Enable SSH access
- [ ] Complete first boot
- [ ] Update all system components via Mainsail Update Manager
- [ ] (Optional) Set custom boot screen

### Phase 2: CAN Bus Network
- [ ] Setup CAN network (`can0` interface)
- [ ] Install Katapult (for SB2209)
- [ ] Update U2C firmware (remove 120R jumper first!)
- [ ] Verify CAN bus with `canbus_query.py`

### Phase 3: Firmware
- [ ] Flash Octopus Pro firmware (USB serial)
- [ ] Flash SB2209 firmware (CAN bus)
- [ ] Verify both MCUs are detected

### Phase 4: Physical Connections
- [ ] Connect U2C to Raspberry Pad 5 via USB
- [ ] Connect Octopus Pro to Raspberry Pad 5 via USB
- [ ] Connect SB2209 to CAN bus (CANH/CANL)
- [ ] Verify 120R jumper is REMOVED from U2C v2.1
- [ ] Power up in correct sequence (U2C → Octopus Pro → SB2209)

### Phase 5: Klipper Configuration
- [ ] Upload all config files to `~/printer_data/config/`
- [ ] Upload `moonraker.conf` to `~/printer_data/`
- [ ] Update `printer.cfg` with correct Octopus Pro serial path
- [ ] Update `SB2209.cfg` with correct CAN UUID
- [ ] **CRITICAL: Enable object processing in `moonraker.conf`** ⚠️
  ```ini
  [file_manager]
  enable_object_processing: True
  ```
- [ ] **CRITICAL: Uncomment KAMP includes in `KAMP_Settings.cfg`** ⚠️
- [ ] **CRITICAL: Verify `PRINT_START` doesn't call `BED_MESH_CALIBRATE`** ⚠️
- [ ] Add input shaper config (if you have values)
- [ ] Uncomment PID values (if you have values)
- [ ] Restart Moonraker: `sudo systemctl restart moonraker`
- [ ] Restart Klipper: `sudo systemctl restart klipper`
- [ ] Verify all MCUs connect successfully

### Phase 6: Additional Features
- [ ] Configure slicer (OrcaSlicer/BambuLab Studio):
  - [ ] Set firmware type to Klipper
  - [ ] Enable firmware retraction
  - [ ] Configure start G-code (calls `PRINT_START`)
  - [ ] Verify object labeling works
- [ ] Setup KAMP (if not already done)
- [ ] (Optional) Setup KlipperScreen
- [ ] (Optional) Setup Crowsnest webcam

### Phase 7: Calibration
- [ ] PID tune hotend
- [ ] PID tune bed
- [ ] Calibrate Z-offset (Voron Tap)
- [ ] Run Quad Gantry Level (QGL)
- [ ] Run input shaper calibration (if not done)
- [ ] Test bed mesh (or verify KAMP adaptive mesh)
- [ ] Calibrate pressure advance
- [ ] Test print

## ⚠️ Critical Configuration Fixes

**These MUST be done before KAMP will work:**

1. **Moonraker Object Processing**:
   ```bash
   nano ~/printer_data/moonraker.conf
   # Change: enable_object_processing: True
   sudo systemctl restart moonraker
   ```

2. **KAMP Includes**:
   ```bash
   nano ~/printer_data/config/KAMP/KAMP_Settings.cfg
   # Uncomment: [include ./KAMP/Adaptive_Meshing.cfg]
   # Uncomment: [include ./KAMP/Voron_Purge.cfg]
   # Uncomment: [include ./KAMP/Smart_Park.cfg]
   ```

3. **PRINT_START Macro**:
   ```bash
   nano ~/printer_data/config/Macros.cfg
   # Verify it calls SETUP_KAMP_MESHING
   # Remove any direct BED_MESH_CALIBRATE calls
   ```

## 🔍 Verification Commands

```bash
# Check CAN bus devices
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0

# Check Klipper status
sudo systemctl status klipper

# Check Klipper logs
tail -f ~/printer_data/logs/klippy.log

# Check Moonraker status
sudo systemctl status moonraker

# Verify config files
ls -la ~/printer_data/config/
```

## 📝 Important Notes

- **U2C v2.1**: Remove 120R jumper before connecting to CAN bus
- **Octopus Pro**: Uses USB serial (not CAN bus) in this hybrid setup
- **SB2209**: Uses CAN bus (via U2C)
- **KAMP**: Requires object processing enabled in Moonraker
- **Firmware Retraction**: Enable in both slicer and `printer.cfg`

## 🆘 Troubleshooting

- **Can't access SSH**: Check network settings, verify hostname/IP
- **MCU not detected**: Check serial path/UUID, verify connections
- **KAMP not working**: Verify object processing enabled, check KAMP includes
- **CAN bus issues**: Verify wiring, check 120R jumper removed, verify power sequence

## 📚 Reference Links

- [Complete Build Guide](https://github.com/Qballjos/Voron-2.4-Otopus-pro-SB2209-U2C-CAN/wiki/Complete-Build-Guide)
- [KAMP Setup](https://github.com/Qballjos/Voron-2.4-Otopus-pro-SB2209-U2C-CAN/wiki/KAMP-Setup)
- [Config Review](CONFIG_REVIEW.md) - See this for detailed configuration recommendations

