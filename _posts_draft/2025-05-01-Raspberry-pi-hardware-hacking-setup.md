---
layout: post
title: "Setting up Raspberry Pi for Hardware Hacking""
author: "TheDyingYAK"
catalog: true
header-img: "img/memory/output.jpg"
header-mask: 0.4
tags:
  - Hardware Hacking
  - Raspberry Pi
  - Hacking
  - Cybersecurity
  - Red Team
---


- download pi imager 
  link: https://www.raspberrypi.com/software/

- install kali from pi imager onto sd card
  - select os
    - kali linux
  - select storage
    - sd card
  - write
    - wait for it to finish


- insert sd card into pi
- connect pi to monitor

- update and upgrade the system
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

- install python3-gpiozero
  ```bash
  sudo apt install python3-gpiozero -y
  ```
  this library is used to control the GPIO pins on the Raspberry Pi. It provides a simple interface for controlling LEDs, buttons, and other hardware components.

### Flashrom
- flashrom already comes installed on kali
 but if you want to install it manually, you can do so with the following command:
 
  ```bash
  sudo apt install flashrom -y
  ```
 flashrom is a utility for reading, writing, and verifying flash memory chips. It supports a wide range of chips and is commonly used for BIOS flashing and recovery.
 
 ![alt text](../img/pihat-hardware/pinout.webp)

  - 3v3
  - MOSI
  - MISO
  - SCLK
  - CS (SPI0 CE0)
  - GND

<!-- > [!NOTE] -->
GPIO Pins operate at 3.3V for other voltage levels, use a level shifter.

### Enable SPI interface

```bash
sudo vim /boot/firmware/config.txt
dtparam=spi=on  # uncomment
```

### Reading Data from SPI Flash
Once connected, use __flashrom__ to read data from the flash chip:

```bash
sudo flashrom -p linux_spi:dev=/dev/spidev0.0 -r backup.bin
```
this will read the contents of a flashchip and put it in backup.bin


### Configure the Raspberry Pi for JTAG/SWD
 - JTAG (Joint Test Action Group) 
 - SWD (Serial Wire Debug)

can both be used with OpenOCD (Open On-Chip Debugger)

- to install
```bash
sudo apt install openocd
```
- To configure openocd go to the following directory
```bash
/usr/share/openocd/
```
inside you will find the following
```bash
interface/  # config files for hardware interfaces (e.g. J-Link, FTDI, Raspberry Pi GPIO)
board/      # predefined configs for common dev boards (e.g. STM32 discovery boards)
target/     # config files for specific MCUs/SoCs (e.g. stm32f1x.cfg, nrf52.cfg)
chip/       # lower-level chip-specific scripts
flash/      # lower-level chip-specific scripts
cpu/        # lower-level chip-specific scripts
```

#### Customize the configuration
- We need to modify the configuration file so it knows what pins to use for JTAG in raspberrypi-native.cfg
```bash
# Each of the JTAG lines needs a GPIO number set: tck tms tdi tdo
bcm2835gpio_jtag_nums 4 22 23 24
```


#### Example on how to use
```bash
sudo openocd -f interface/raspberrypi-swd.cfg -f target/stm32f1x.cfg
```


### Raspberry Pi for UART communication
The Raspberry Pi's UART interface can be used as a serial console for debugging

#### Connecting UART
- Use the following pins to connect to a UART-enabled device
  
![alt text](../img/pihat-hardware/pinout.webp)

- TXD
- RXD
- GND

Minicom is already installed on kali but if you need it
```bash
sudo apt install minicom
```

#### Usage
```bash
minicom -b 115200 -o -D /dev/serial0
```

### Connecting to I2c

<!--  > [!NOTE] -->
 
The Raspberry Pi can only function as an I2C master




#### Connect I2c Devices
Use the following I2C pins on the pi

![alt text](../img/pihat-hardware/pinout.webp)

- SDA
- SCL
- GND

#### Test I2C devices
```bash
sudo i2cdetect -y 1
```

### References
- https://medium.com/@marcel.rickcen/setting-up-a-raspberry-pi-for-hardware-hacking-189c70107597

