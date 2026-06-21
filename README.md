# ESP32 Family SPI Flash Programmer (Serprog)

A lightweight, high-speed SPI Flash Programmer utilizing the native HW SPI peripheral of the **Espressif ESP32 MCU family** (including ESP32, ESP32-S3, ESP32-C3, etc.). This project implements the **Serprog** protocol, enabling seamless integration with the open-source **Flashrom** utility on Linux.

This specific guide is tailored for unbricking and flashing bootloaders (such as U-Boot) onto the **HLK-7628N** (MT7628AN-based) Wi-Fi module, but it is compatible with any standard 3.3V SPI flash memory (e.g., W25Qxx series).

---

## Features
- **Multi-Target Support**: Compatible with the entire ESP32 family (Classic ESP32, S3, C3, S2).
- **High Speed**: Leverages the ESP32 HW SPI peripheral capable of up to 80MHz clock rate.
- **In-Circuit Programming**: Program the flash chip without desoldering by holding the target CPU in a reset state.
- **Native CDC/UART**: Automatically chooses between Native USB-CDC or standard USB-to-UART based on the chip.

---

## Hardware Configuration & MCU Family Reference

Depending on the ESP32 chip model you are using, the default SPI pins and Linux serial ports differ. Refer to the table below to configure your wiring and toolchain:

| MCU Target | Port on Linux | Install Command | Set-Target Command | SPI Pin Configuration (MISO / MOSI / CLK / CS) |
| :--- | :--- | :--- | :--- | :--- |
| **ESP32-S3** (e.g., S3-WROOM-1) | `/dev/ttyACM0` | `./install.sh esp32s3` | `idf.py set-target esp32s3` | MISO: **13** / MOSI: **11** / CLK: **12** / CS: **10** |
| **ESP32** (Classic, e.g., WROOM-32) | `/dev/ttyUSB0` | `./install.sh esp32` | `idf.py set-target esp32` | MISO: **18** / MOSI: **23** / CLK: **19** / CS: **13** |
| **ESP32-C3** | `/dev/ttyACM0` | `./install.sh esp32c3` | `idf.py set-target esp32c3` | MISO: **2** / MOSI: **7** / CLK: **6** / CS: **10** |

> **IMPORTANT**: ESP32 GPIOs operate at **3.3V**. Ensure your target module is also 3.3V compatible. If you are flashing a 1.8V chip, you must use a bi-directional level shifter.

### HLK-7628N Connection Layout
To program the onboard SPI Flash on the **HLK-7628N**, you must pull the **PORST_N** (Power-On Reset) pin of the HLK-7628N to **GND** to hold its SoC in a reset state.

---

## Software Installation & Setup (Linux)

### 1. Install System Prerequisites
Install the required packages for building ESP-IDF on Ubuntu/Debian:
```bash
sudo apt update
sudo apt install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0 -y
```
### 2. Set Up ESP-IDF
Clone the Espressif SDK and install the compiler tools.
(Replace esp32s3 in ./install.sh with esp32 or esp32c3 if you are using another chip).
```Bash
mkdir -p ~/esp
cd ~/esp
git clone --recursive -b v5.2.2 https://github.com/espressif/esp-idf.git
cd esp-idf

# Set up tools for your target (e.g., esp32s3, esp32, esp32c3)
./install.sh esp32s3
```
### 3. Build & Flash the Programmer Firmware
Clone the programmer repository, configure the environment, and flash your target MCU:
```Bash
# 1. Activate the ESP-IDF environment variables
. ~/esp/esp-idf/export.sh

# 2. Clone this programmer repository
cd ~/esp
git clone https://github.com/thisiseth/esp32-serprog.git
cd esp32-serprog/src/serprog

# 3. Set the target to match your board (e.g., esp32s3, esp32, esp32c3)
idf.py set-target esp32s3

# 4. Build the project
idf.py build

# 5. Flash the binary to your MCU (adjust the port /dev/ttyACM0 or /dev/ttyUSB0)
sudo chmod 666 /dev/ttyACM0
idf.py flash -p /dev/ttyACM0
```
# Flashing the Target Module (e.g., HLK-7628N)
Because standard bootloader binaries (e.g., U-Boot) are smaller than the expected full flash size, we pad the binary to 32MB and flash only the bootloader region to protect other partitions.
Install Flashrom
```Bash
sudo apt install flashrom -y
```
### 1. Prepare the 32MB Padded Bootloader File
Execute these commands in the terminal where your bootloader binary is located (replace uboot.bin with your actual filename):
code
```Bash
# Create a 32MB file filled with 0xFF (flash empty state)
python3 -c "open('full_32m_uboot.bin', 'wb').write(b'\xff' * 33554432)"

# Copy your bootloader to the beginning of the 32MB file
dd if=uboot.bin of=full_32m_uboot.bin conv=notrunc

# Create the layout configuration file
echo -e "0x00000000:0x0001ffff bootloader\n0x00020000:0x01ffffff remaining" > layout.txt
```
### 2. Identify the Chip & Flash the Bootloader
Make sure your target CPU is in the reset state (PORST_N connected to GND).
(Adjust the port /dev/ttyACM0 or /dev/ttyUSB0 according to your ESP32 model).
```Bash
# Verify the SPI flash connection
sudo flashrom -p serprog:dev=/dev/ttyACM0:115200,spispeed=8M

# Flash only the bootloader partition (takes ~2 seconds)
sudo flashrom -p serprog:dev=/dev/ttyACM0:115200,spispeed=8M -c "W25Q256FV" -l layout.txt -i bootloader -w full_32m_uboot.bin
```
License & Credits
This implementation is based upon the upstream open-source serprog protocol. Special thanks to the creators of flashrom and the esp32-serprog firmware community.
License: GPL-3.0
