# GRUB Debug Mode - BIOS Boot Troubleshooting

## Quick Debug for Black Screen on USB Boot

### Option 1: Edit GRUB Menu at Boot Time (Fastest)

1. **Boot USB and see GRUB menu**
2. **Press `e`** to edit the default entry
3. **Add these lines** after the `linux` command:
   ```
   # For BIOS (i386-pc) boot:
   linux /vmlinuz quiet debug=all verbose=1
   ```
4. **Press Ctrl+X** to boot with debug enabled
5. **Watch serial console** for detailed output

### Option 2: Build with Debug Mode

```bash
cd /home/gaspare/Documents/code/c/onie
make clean
make GRUB_DEBUG=yes -j4
```

This will:
- Compile GRUB with emulation support (better error detection)
- Both i386-pc and x86_64-efi will have debug symbols
- Create recovery image with debug-enabled bootloader

### Option 3: Add Debug to Recovery ISO Config

Edit `build-config/recovery/grub-iso.cfg` and add after `set timeout=3`:

```
# Debug output
set debug=all
set pager=1
```

Then rebuild:
```bash
make image-complete
```

## Debug Output on Serial Console

Your machine.make shows:
```
CONSOLE_SPEED = 115200
CONSOLE_DEV = 0
```

**Make sure:**
1. BIOS serial port is **enabled** and set to **COM1 (Same as ttyS0)**
2. Connect serial cable to port COM1
3. Terminal/minicom configured for **115200 baud, 8N1**

Run minicom to watch debug output:
```bash
minicom -D /dev/ttyUSB0 -b 115200 -8
```

## BIOS Boot Checklist

- [ ] UEFI Enabled in BIOS
- [ ] CSM/Legacy Boot Enabled
- [ ] USB Boot Order Set Correctly (before hard disk)
- [ ] Serial Port Enabled (if using serial console)
- [ ] Console Output matches machine.make settings:
  - Speed: 115200
  - Port: ttyS0 (CONSOLE_DEV=0)

## What Debug Output Shows

With `debug=all`, you should see:
1. **GRUB loading message**
2. **Module loading** (USB, filesystem modules)
3. **Partition search** (ONIE-BOOT label)
4. **Menu entry execution**
5. **Kernel loading** and **initrd**
6. **Kernel boot parameters**

If you see nothing, the issue is **before GRUB** (firmware/USB not initializing).

## Common Issues

### Issue: Black screen, no output at all
- [ ] Check USB boot is working (try UEFI boot first)
- [ ] Verify serial cable is connected
- [ ] Try different USB ports
- [ ] Try `loadfont` issue: rebuild GRUB without theme options

### Issue: GRUB menu shows but no vmlinuz found
- [ ] Check recovery ISO has correct files: `ls recovery/iso-sysroot/`
- [ ] Verify grub.cfg points to correct paths: `cat recovery/iso-sysroot/boot/grub/grub.cfg`
- [ ] Look for template placeholder substitution errors

### Issue: "file not found" after boot
- [ ] Check ONIE-BOOT partition exists: `blkid` in boot shell
- [ ] Verify kernel path in grub.cfg matches actual file

## Next Steps

1. Build with: `make GRUB_DEBUG=yes -j4`
2. Copy `onie-recovery-x86_64-dell_s4000_c2338.iso` to USB
3. Boot and watch serial output for detailed messages
4. Share the first 50 lines of debug output for analysis
