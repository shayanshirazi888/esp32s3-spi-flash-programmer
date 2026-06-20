# ESP32-S3 SPI Flash Programmer (Serprog)

A lightweight, high-speed SPI Flash Programmer utilizing the native USB-OTG/JTAG peripheral of the **ESP32-S3-WROOM-1** dev kit. This project implements the **Serprog** protocol, enabling seamless integration with the open-source **Flashrom** utility on Linux.

This specific guide and pinout are tailored for unbricking and flashing bootloaders (such as U-Boot) onto the **HLK-7628N** (MT7628AN-based) Wi-Fi module, but it can be adapted for any standard 3.3V SPI flash memory (e.g., W25Qxx series).

---

## Features
- **High Speed**: Leverages the ESP32-S3 HW SPI peripheral (up to 80MHz) and native USB-CDC speed.
- **In-Circuit Programming**: Program the flash chip without desoldering, by holding the target CPU in reset state.
- **Standard Toolchain**: Built using the native Espressif IoT Development Framework (ESP-IDF v5.x).

---

## Hardware Wiring Guide (ESP32-S3 to HLK-7628N)

> **IMPORTANT**: The ESP32-S3 GPIO operates at **3.3V**. Ensure your target chip/module is also 3.3V compatible. If you are flashing a 1.8V chip, you must use a proper bi-directional level shifter.

To program the onboard SPI Flash on the **HLK-7628N**, you must pull the **PORST_N** (Power-On Reset) pin to **GND**. This holds the MT7628 SoC in reset and prevents bus contention on the SPI lines.

| HLK-7628N Pin | ESP32-S3 Pin | Description |
| :--- | :--- | :--- |
| **3.3VD** | **3V3** | Power Supply |
| **GND** | **GND** | Common Ground |
| **PORST_N** (Reset) | **GND** | Hold SoC in Reset |
| **SPI_CS0** | **GPIO 10** | Chip Select (Main Boot Flash) |
| **SPI_CLK** | **GPIO 12** | SPI Clock |
| **SPI_MISO** | **GPIO 13** | SPI Master In Slave Out |
| **SPI_MOSI** | **GPIO 11** | SPI Master Out Slave In |

---

## Software Installation & Setup (Linux)

### 1. Install System Prerequisites
Install the required packages for building ESP-IDF on Ubuntu/Debian:
```bash
sudo apt update
sudo apt install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0 -y
