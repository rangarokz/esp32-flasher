# Lazy Idli - ESP32 Train Simulator Firmware Flasher

A browser-based firmware deployment platform developed for **Lazy Idli** custom train simulator hardware controllers.

This platform allows clients and users to flash ESP32 microcontrollers directly from **Google Chrome** or **Microsoft Edge** via the Web Serial API—no need to install Python, drivers, or the ESP-IDF toolchain. It distributes pre-compiled binary packages without exposing underlying C/C++ source code.

Live Web Flasher: **[https://rangarokz.github.io/esp32-flasher/](https://rangarokz.github.io/esp32-flasher/)**

---

## Supported Controllers

### 1. Accelerator Controller
Firmware for train accelerator and reverser throttle control:
- **Target Hardware**: ESP32 / ESP32-S3 (2MB+ Flash)
- **Throttle Input**: Potentiometer connected to GPIO 34 (ADC1 Channel 6) with moving-average filtering
- **Direction Switches**: Reverser Forward (GPIO 4), Reverser Reverse (GPIO 16)
- **Calibration**: Onboard NVS (Non-Volatile Storage) flash memory for notch position lookup tables
- **Manifest**: `manifest_accel.json` &rarr; `accelerator_merged.bin`

### 2. Brake Controller
Firmware for locomotive pneumatic brake handle control:
- **Target Hardware**: ESP32 / ESP32-S3 (2MB+ Flash)
- **Brake Input**: Multi-stage brake potentiometer on GPIO 32 (ADC1 Channel 4)
- **Detent Positions**: Calibrated for Release, Run, Service Notches (25%, 50%, 75%, Full Service), and Emergency
- **Calibration Trigger**: BOOT button (GPIO 0) with step-by-step detent recording saved to NVS
- **Status Indicator**: Onboard LED (GPIO 2)
- **Manifest**: `manifest_brake.json` &rarr; `brake_merged.bin`

---

## How to Flash (Client Guide)

1. Open **[https://rangarokz.github.io/esp32-flasher/](https://rangarokz.github.io/esp32-flasher/)** in **Google Chrome** or **Microsoft Edge** on Desktop.
2. Select the tab corresponding to the device you want to flash:
   - **Accelerator**
   - **Brake**
3. Connect the ESP32 board to your computer using a data-capable USB cable.
4. Click **Connect & Flash [Device]**.
5. Select the ESP32 COM port from the browser pop-up and click **Connect**.
6. The browser will automatically erase and write the firmware directly to the board.

> **Troubleshooting**: If the browser gets stuck on *"Connecting..."*, press and hold the **BOOT** button on the ESP32 board while clicking Connect, then release it once flashing starts.

---

## Architecture & Binary Merging

Each controller project is compiled using the ESP-IDF framework. To enable one-click browser flashing at offset `0x0`, the bootloader, partition table, and application binaries are merged into a single image using `esptool.py`:

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
This repository is configured via `.gitignore` to explicitly ignore `main/`, `build/`, `CMakeLists.txt`, and `sdkconfig`. Only the pre-compiled binaries (`.bin`), manifests (`.json`), and web interface files are tracked and deployed, ensuring complete source code privacy.

---

## Project Structure

```
upload_code/
├── README.md                 # Project documentation & guide
├── index.html                # Entry point redirecting to flash.html
├── flash.html                # Lazy Idli dual-controller web UI
├── manifest_accel.json       # Web flasher configuration for Accelerator
├── manifest_brake.json       # Web flasher configuration for Brake
├── manifest.json             # Legacy manifest (backwards compatibility)
├── accelerator_merged.bin    # Merged firmware binary for Accelerator
├── brake_merged.bin          # Merged firmware binary for Brake
├── merged.bin                # Original binary copy (backwards compatibility)
└── .nojekyll                 # GitHub Pages configuration
```

---

&copy; Lazy Idli. Developed for Lazy Idli Train Simulator Controllers.
