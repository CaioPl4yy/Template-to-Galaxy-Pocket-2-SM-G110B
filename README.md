# Template-to-Galaxy-Pocket-2-SM-G110B
# Device tree template for CWM - Samsung Galaxy Pocket 2 (SM-G110B, Duos)

This repository contains template files to start porting **CWM recovery** for the **Samsung Galaxy Pocket 2 - SM-G110B (Duos)**.

**IMPORTANT:** These files are templates. You need to:
- extract `boot.img` / `recovery.img` from the stock firmware to fill in offsets and retrieve kernel/ramdisk
- adjust `recovery.fstab` with the actual paths under `/dev/block/.../by-name/`
- test carefully in a development environment (emulator if possible) before flashing

---

Structure (this file shows each file separately):

- device/samsung/g110b/Android.mk
- device/samsung/g110b/BoardConfig.mk
- device/samsung/g110b/device.mk
- device/samsung/g110b/recovery.fstab
- device/samsung/g110b/BoardConfigCommon.mk
- device/samsung/g110b/README.md
- device/samsung/g110b/vendor/samsung/g110b/vendorsetup.sh

---

##### device/samsung/g110b/Android.mk

```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

# Minimal target declarations for CWM
LOCAL_MODULE := android.hardware.recovery
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := .so

include $(BUILD_SHARED_LIBRARY)

# Device-specific overlays (placeholder)
PRODUCT_COPY_FILES += \
    $(LOCAL_PATH)/cwm.prop:system/cwm.prop
```

---

##### device/samsung/g110b/BoardConfig.mk

```makefile
# BoardConfig.mk - template for SM-G110B

TARGET_BOARD_PLATFORM := exynos  # Adjust if needed
TARGET_KERNEL_ARCH := arm
TARGET_KERNEL_TEXT := 0x00008000

# Page size and offsets — REPLACE with values extracted from stock recovery/boot
BOARD_PAGE_SIZE := 2048        # common on some old Samsung devices; confirm
BOARD_KERNEL_CMDLINE := console=ttyHSL0,115200n8 androidboot.hardware=samsung

# Offsets (example; replace!)
BOARD_BOOTIMAGE_BASE := 0x00008000
BOARD_RAMDISK_OFFSET := 0x01000000
BOARD_TAGS_OFFSET := 0x00000100

# Pixel format for recovery (not critical for CWM)
TARGET_RECOVERY_PIXEL_FORMAT := "RGB_565"
TW_DEFAULT_LANGUAGE := en

# Tools and flags
USE_PREBUILT_KERNEL := false

# Product name
PRODUCT_NAME := g110b
PRODUCT_DEVICE := g110b
```

---

##### device/samsung/g110b/device.mk

```makefile
# device.mk - product registration
PRODUCT_COPY_FILES += \
    device/samsung/g110b/cwm.prop:system/cwm.prop

PRODUCT_NAME := g110b
PRODUCT_DEVICE := g110b
PRODUCT_BRAND := samsung
PRODUCT_MODEL := SM-G110B
PRODUCT_MANUFACTURER := Samsung
```

---

##### device/samsung/g110b/recovery.fstab

```text
# recovery.fstab - TEMPLATE
# Replace the "by-name" paths with the correct ones after inspecting /dev/block/platform/*/by-name

/dev/block/platform/*****************/by-name/BOOT   /boot   emmc    nosuid,nodev   wait
/dev/block/platform/*****************/by-name/RECOVERY /recovery emmc nosuid,nodev wait
/dev/block/platform/*****************/by-name/SYSTEM /system ext4 rw,barrier=1,data=ordered wait
/dev/block/platform/*****************/by-name/CACHE  /cache  ext4 rw,barrier=1,data=ordered wait
/dev/block/platform/*****************/by-name/USERDATA /data ext4 rw,barrier=1,data=ordered wait
/dev/block/platform/*****************/by-name/EFS    /efs    ext4 rw,barrier=1,data=ordered
```

---

##### device/samsung/g110b/BoardConfigCommon.mk

```makefile
# BoardConfigCommon.mk - common flags

targets := recovery

# CWM does not depend on DPI or touch, ideal for small screens
TW_USE_FLASHLIGHT := false
TW_BRIGHTNESS_PATH := "/sys/class/leds/lcd-backlight/brightness"

# Screen size (informative)
TW_SCREEN_WIDTH := 240
TW_SCREEN_HEIGHT := 320
```

---

##### device/samsung/g110b/vendor/samsung/g110b/vendorsetup.sh

```sh
# vendorsetup.sh - helper to add product combos to build
add_lunch_combo omni_g110b-eng
add_lunch_combo omni_g110b-userdebug
```

---

##### device/samsung/g110b/README.md

```
README - How to use the templates to build CWM for SM-G110B

1) Set up your environment (install dependencies and repo) - see the main guide.
2) Download CWM source (or TWRP adapted for CWM) for ARMv7.
3) Place this directory under device/samsung/g110b
4) Extract recovery.img and boot.img from stock firmware:
   - tar -xf G110B_XXX.tar.md5 -C stock_unpack
   - locate boot.img and recovery.img
   - run unmkbootimg to get ramdisk and offsets
5) Update BoardConfig.mk with the correct PAGE_SIZE and offsets
6) Update recovery.fstab with correct /dev/block/platform/*/by-name/ paths
7) source build/envsetup.sh
8) lunch omni_g110b-eng
9) mka recoveryimage
10) If successful, recovery.img will appear in out/target/product/g110b/, package it as .tar.md5 for Odin

WARNING: Test on a secondary device, always keep stock firmware for restoration.

NOTE: As this is CWM, there is no touch interface, making it ideal for small screens.
