# Configuration Review & Recommendations

## 🔴 Critical Issues (Must Fix)

### 1. Moonraker Object Processing Disabled
**File**: `backup/moonraker.conf`  
**Line**: 11  
**Issue**: `enable_object_processing: False`  
**Fix**: Change to `True` - Required for KAMP to work!

```ini
[file_manager]
enable_object_processing: True  # Changed from False
```

### 2. KAMP Includes Commented Out
**File**: `backup/KAMP/KAMP_Settings.cfg`  
**Lines**: 3-6  
**Issue**: All KAMP feature includes are commented out  
**Fix**: Uncomment the features you want to use

```ini
[include ./KAMP/Adaptive_Meshing.cfg]       # Enable adaptive meshing
[include ./KAMP/Voron_Purge.cfg]            # Enable adaptive Voron logo purging
# [include ./KAMP/Line_Purge.cfg]           # Alternative: line purge (choose one)
[include ./KAMP/Smart_Park.cfg]             # Enable Smart Park feature
```

### 3. PRINT_START Conflicts with KAMP
**File**: `backup/Macros.cfg`  
**Lines**: 163-164  
**Issue**: `BED_MESH_CLEAR` and `BED_MESH_CALIBRATE` called directly, bypassing KAMP  
**Fix**: Remove these lines - KAMP handles mesh creation automatically

**Current (WRONG)**:
```gcode
SETUP_KAMP_MESHING DISPLAY_PARAMETERS=1 LED_ENABLE=1 FUZZ_ENABLE=1
BED_MESH_CLEAR
BED_MESH_CALIBRATE
```

**Should be**:
```gcode
SETUP_KAMP_MESHING DISPLAY_PARAMETERS=1 LED_ENABLE=1 FUZZ_ENABLE=1
# KAMP will automatically create adaptive mesh - don't call BED_MESH_CALIBRATE here
```

## ⚠️ Important Improvements

### 4. Input Shaper Not Active
**File**: `backup/printer.cfg`  
**Issue**: Input shaper values are saved but not in active config  
**Fix**: Add input shaper section (values from SAVE_CONFIG):

```ini
[input_shaper]
shaper_type_x = mzv
shaper_freq_x = 58.8
shaper_type_y = mzv
shaper_freq_y = 42.0
```

**Note**: These values are from your saved config. If you haven't run input shaper calibration recently, consider re-running it.

### 5. PID Values Commented Out
**File**: `backup/printer.cfg`  
**Lines**: 82-85, 128-131  
**Issue**: PID values are saved but commented out  
**Fix**: Uncomment and use saved values:

```ini
[extruder]
control: pid
pid_Kp: 39.833
pid_Ki: 13.976
pid_Kd: 28.382

[heater_bed]
control: pid
pid_kp: 38.385
pid_ki: 1.662
pid_kd: 221.675
```

### 6. Bed Mesh Configuration
**File**: `backup/printer.cfg`  
**Lines**: 266-287  
**Issue**: Static bed mesh may conflict with KAMP adaptive meshing  
**Recommendation**: Keep the bed_mesh section for fallback, but KAMP will override it with adaptive meshing. The current config is fine as-is.

### 7. KAMP Verbose Mode Enabled
**File**: `backup/KAMP/KAMP_Settings.cfg`  
**Line**: 12  
**Issue**: `variable_verbose_enable: True` - good for debugging but verbose  
**Recommendation**: Set to `False` once KAMP is working correctly to reduce console spam

## ✅ Good Practices (Keep These)

1. ✅ Firmware retraction configured correctly
2. ✅ CM4 temperature fan configured
3. ✅ Exclude object enabled
4. ✅ G-code arcs enabled
5. ✅ Proper MCU configuration
6. ✅ Good stepper motor settings
7. ✅ Proper probe configuration for Voron Tap
8. ✅ Good safety settings (idle timeout, min/max temps)

## 🔧 Optimization Suggestions

### 8. Extruder Rotation Distance
**File**: `backup/printer.cfg`  
**Line**: 72  
**Current**: `rotation_distance: 23.135`  
**Note**: There's a commented value `#22.67895` - verify which is correct for your extruder

### 9. Extruder Current
**File**: `backup/printer.cfg`  
**Line**: 106  
**Current**: `run_current: 0.650`  
**Note**: This seems low for a direct drive extruder. Typical range is 0.8-1.2A. Consider testing higher values if you experience skipping.

### 10. Max Acceleration
**File**: `backup/printer.cfg`  
**Line**: 60  
**Current**: `max_accel: 4000`  
**Note**: With input shaper configured, you might be able to increase this. Test incrementally.

### 11. Z Velocity/Accel
**File**: `backup/printer.cfg`  
**Lines**: 62-63  
**Current**: `max_z_velocity: 15`, `max_z_accel: 350`  
**Note**: These are conservative values. With 12V TMC drivers, you could potentially increase these slightly if needed.

### 12. Bed Mesh Probe Count
**File**: `backup/printer.cfg`  
**Line**: 285  
**Current**: `probe_count: 5,5`  
**Note**: This is fine for static mesh. KAMP will use adaptive probe counts based on print area.

### 13. QGL Probe Points
**File**: `backup/printer.cfg`  
**Lines**: 209-213  
**Current**: 4 points  
**Note**: Consider adding a center point for better accuracy:
```ini
points:
   50,25
   50,225
   150,150    # Add center point
   250,225
   250,25
```

## 📝 Additional Recommendations

### 14. Add Pressure Advance
**File**: `backup/printer.cfg`  
**Missing**: Pressure advance configuration  
**Recommendation**: Add after extruder section:
```ini
[extruder]
# ... existing config ...
pressure_advance: 0.05  # Calibrate this value for your setup
pressure_advance_smooth_time: 0.040
```

### 15. Add Skew Correction (Optional)
If you notice prints are not perfectly square, consider adding skew correction after calibration.

### 16. Consider Adding Nevermore Support
**File**: `backup/Macros.cfg`  
**Line**: 124  
**Note**: Nevermore is commented out. If you have one, uncomment and configure.

### 17. Timelapse Configuration
**File**: `backup/timelaps.cfg`  
**Note**: Timelapse is included but verify it's configured correctly for your setup.

## 🎯 Priority Fix Order

1. **IMMEDIATE**: Fix `enable_object_processing: False` → `True` in moonraker.conf
2. **IMMEDIATE**: Uncomment KAMP includes in KAMP_Settings.cfg
3. **IMMEDIATE**: Remove `BED_MESH_CALIBRATE` from PRINT_START macro
4. **HIGH**: Add input shaper configuration
5. **HIGH**: Uncomment PID values
6. **MEDIUM**: Verify extruder rotation_distance
7. **MEDIUM**: Test higher extruder current if needed
8. **LOW**: Optimize acceleration/velocity settings
9. **LOW**: Add pressure advance calibration

## 📋 Verification Checklist

After making changes:
- [ ] Restart Moonraker: `sudo systemctl restart moonraker`
- [ ] Restart Klipper: `sudo systemctl restart klipper`
- [ ] Verify KAMP creates adaptive meshes
- [ ] Test a print to ensure everything works
- [ ] Check Klipper logs for errors: `tail -f ~/printer_data/logs/klippy.log`

