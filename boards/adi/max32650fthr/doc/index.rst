.. zephyr:board:: max32650fthr

Overview
********
The MAX32650FTHR evaluation kit provides a platform for evaluating the capabilities
of the MAX32650 ultra-low-power memory-scalable microcontroller designed specifically
for high-performance, battery-powered applications.

The Zephyr port is running on the MAX32650 MCU.

Hardware
********

- MAX32650 MCU:

  - Ultra Efficient Microcontroller for Battery-Powered Applications

    - 120MHz Arm Cortex-M4 with FPU
    - SmartDMA Provides Background Memory Transfers with Programmable Data Processing
    - 120MHz High-Speed and 50MHz Low-Power Oscillators
    - 7.3728MHz Low Power Oscillators
    - 32.768kHz and RTC Clock (Requires External Crystal)
    - 8kHz, Always-on, Ultra-Low-Power Oscillator
    - 3MB Internal Flash, 1MB Internal SRAM
    - 104µW/MHz Executing from Cache at 1.1V
    - Five Low-Power Modes: Active, Sleep, Background, Deep-Sleep, and Backup
    - 1.8V and 3.3V I/O with No Level Translators
    - Programming and Debugging

  - Scalable Cached External Memory Interfaces

    - 120MB/s HyperBus/Xccela DDR Interface
    - SPIXF/SPIXR for External Flash/RAM Expansion
    - 240Mbps SDHC/eMMC/SDIO/microSD Interface

  - Optimal Peripheral Mix Provides Platform Scalability

    - 16-Channel DMA
    - Three SPI Master (60MHz)/Slave (48MHz)
    - One QuadSPI Master (60MHz)/Slave (48MHz)
    - Up to Three 4Mbaud UARTs with Flow Control
    - Two 1MHz I2C Master/Slave
    - I2S Slave
    - Four-Channel, 7.8ksps, 10-bit Delta-Sigma ADC
    - USB 2.0 Hi-Speed Device Interface with PHY
    - 16 Pulse Train Generators
    - Six 32-bit Timers with 8mA Hi-Drive
    - 1-Wire® Master

  - Trust Protection Unit (TPU) for IP/Data and Security

    - Modular Arithmetic Accelerator (MAA), True Random Number Generator (TRNG)
    - Secure Nonvolatile Key Storage, SHA-256, AES-128/192/256
    - Memory Decryption Integrity Unit, Secure Boot ROM

- External devices connected to the MAX32650FTHR:

   - Battery Connector and Charging Circuit
   - Micro-SD Card Interface
   - USB 2.0 Full-Speed Device Interface
   - MAX11261 6-Channel, 24-Bit, 16ksps, ADC
   - Adafruit® Feather Board Compatible

Supported Features
==================

.. zephyr:board-supported-hw::

Programming and Debugging
*************************

.. zephyr:board-supported-runners::

Flashing
========
The MAX32650 MCU can be flashed by connecting an external debug probe to the
SWD port. SWD debug can be accessed through the Cortex 10-pin connector, J5.
Logic levels are fixed to VDDIO (1.8V).

Once the debug probe is connected to your host computer, then you can simply run the
``west flash`` command to write a firmware image into flash.

.. note::

   This board uses OpenOCD as the default debug interface. You can also use
   a Segger J-Link with Segger's native tooling by overriding the runner,
   appending ``--runner jlink`` to your ``west`` command(s). The J-Link should
   be connected to the standard 2*5 pin debug connector (J5) using an
   appropriate adapter board and cable

Debugging
=========
Please refer to the `Flashing`_ section and run the ``west debug`` command
instead of ``west flash``.

References
**********

- `MAX32650FTHR web page`_

.. _MAX32650FTHR web page:
   https://www.analog.com/en/resources/evaluation-hardware-and-software/evaluation-boards-kits/max32650fthr.html
