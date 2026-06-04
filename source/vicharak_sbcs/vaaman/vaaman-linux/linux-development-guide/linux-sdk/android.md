### Building Android 12

After successful syncing of the source, you are now ready to build the Linux SDK.

Enter the SDK source directory, and confirm that you have a `build.sh` file in the current directory.

#### 1. Source the Android 12 environment setup

```bash
source build/envsetup.sh
```

#### 2. Copy Android Specific configs into Kernel Repository

```bash
cp -vr mkcombinedroot/configs/android-1* kernel-5.10/arch/arm64/configs
```

The device specific configuration file for Android 12 is located at `device/rockchip/rk3399/vaaman.mk`

#### 3. Launch the device configuration

```bash
lunch vaaman-userdebug
```

#### 4. Building the Android 12 firmware

```bash
./build.sh -UACKup
```

```{note}
`build.sh` is the firmware build script that can be used to interactively build the firmware.
It can accept command line arguments to customize the build process.

- -U -> Build U-Boot
- -A -> Build Android
- -C -> Build kernel using clang compiler
- -K -> Build kernel
- -u -> Build rockchip update image
- -p -> Pack the firmware
```

#### 5. After the firmware is compiled, you can find an `update.img` file inside `rockdev/pack`


(firmware-file-formats)=

:::{important}

## Understanding Firmware Formats

There are two primary types of firmware formats you'll encounter:

1. **Raw Firmware:**
   - **Description:** Raw Firmware is essentially a direct copy of your storage device, capturing every bit of data as is.
   - **Flashing Tools:**
     - _For SD Card:_
       - **GUI Tools:**
         - `SDCard Installer` (Linux/Windows/MacOS)
         - `Balena Etcher` (Linux/Windows/MacOS)
       - **CLI Tool:**
         - `dd` (Linux)
     - _For eMMC:_
       - **GUI Tool:**
         - `AndroidTool/RKDevTool` (Windows)
       - **CLI Tools:**
         - `upgrade_tool` (Linux/MacOS)
         - `rkdeveloptool` (Linux)

2. **RK Firmware:**
   - **Description:** RK Firmware comes in Rockchip's proprietary format. Specific tools provided by Rockchip are used to flash this firmware onto eMMC or SD Card.
   - **Flashing Tools:**
     - _For SD Card:_
       - **GUI Tool:**
         - `SD Firmware Tool` (Windows)
     - _For eMMC:_
       - **GUI Tool:**
         - `AndroidTool/RKDevTool` (Windows)
       - **CLI Tool:**
         - `upgrade_tool` (Linux)

**Partition Image:**

- **Description:** Partition Image refers to segmented images of different
  parts of the firmware, such as boot.img, kernel.img, and system.img.
  These components are assembled like pieces of a puzzle to form a complete Android system.

- **Usage:**
  - Place each specific image (e.g., kernel.img) into its corresponding
    partition on the SD card or eMMC to construct the complete Android system.

:::

---

```{seealso}
[How to use Balena Etcher](../../vaaman-sdcard-boot.rst)
```

```{admonition} More information
For more information about the `Linux_Pack_Firmware` tool or how to flash
firmware onto your board, please refer to the [Rockchip Development Guide](../linux-usage-guide/rockchip-develop-guide.md)
```
