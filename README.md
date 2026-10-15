# Lazy Idli - ESP32 Train Simulator Firmware Flasher

A browser-based firmware deployment platform developed for **Lazy Idli** train simulator controllers.

This platform allows clients and users to flash ESP32 microcontrollers directly from **Google Chrome** or **Microsoft Edge** via the Web Serial API without needing to install Python, drivers, or development tools. It distributes pre-compiled binary packages safely without exposing underlying C/C++ source code.

Live Web Flasher: **[https://rangarokz.github.io/esp32-flasher/](https://rangarokz.github.io/esp32-flasher/)**

---

## Supported Controllers

### 1. Accelerator Controller
Firmware for train accelerator and reverser throttle control:
- Supported Hardware: ESP32 (2MB+ Flash)
- Throttle potentiometer input with moving-average filtering
- Direction control for Forward and Reverse
- Onboard calibration stored in non-volatile flash memory
- Deployed Binary: `accelerator_merged.bin` (`manifest_accel.json`)

### 2. Brake Controller
Firmware for locomotive pneumatic brake handle control:
- Supported Hardware: ESP32 (2MB+ Flash)
- Multi-stage pneumatic brake detents (Release, Run, Service notches, and Emergency)
- Calibration routine stored in non-volatile flash memory
- Status indicator support
- Deployed Binary: `brake_merged.bin` (`manifest_brake.json`)

---

## How to Flash (User Guide)

1. Open **[https://rangarokz.github.io/esp32-flasher/](https://rangarokz.github.io/esp32-flasher/)** in **Google Chrome** or **Microsoft Edge** on Desktop.
2. Select the controller tab (**Accelerator** or **Brake**).
3. Connect the ESP32 board to your computer using a data-capable USB cable.
4. Click **Connect & Flash**.
5. Select the ESP32 serial COM port from the browser prompt and confirm.
6. The browser will flash the firmware directly to the board.

> **Note**: If the connection gets stuck on *"Connecting..."*, press and hold the **BOOT** button on the ESP32 board while clicking Connect, then release it once flashing begins.

---

## Binary Merging & Deployment

Each controller firmware is compiled and merged into a single binary image ready to flash at offset `0x0`:

```powershell
# Accelerator binary merge:
esptool.py --chip esp32 merge_bin -o accelerator_merged.bin `
  --flash_mode dio --flash_size 2MB --flash_freq 40m `
  0x1000 build/bootloader/bootloader.bin `
  0x8000 build/partition_table/partition-table.bin `
  0x10000 build/upload_code.bin

# Brake binary merge:
esptool.py --chip esp32 merge_bin -o brake_merged.bin `
  --flash_mode dio --flash_size 2MB --flash_freq 40m `
  0x1000 build/bootloader/bootloader.bin `
  0x8000 build/partition_table/partition-table.bin `
  0x10000 build/loco_brake_adc.bin
```

### Source Code Privacy
All raw source code (`main/`), build outputs (`build/`), and CMake files are protected by `.gitignore`. Only pre-compiled binaries and web deployment files are tracked in this repository.

---

&copy; Lazy Idli. Developed for Lazy Idli Train Simulator Controllers.
