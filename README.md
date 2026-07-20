# UPDI Serial Auto Programmer

A hardware tool that automatically switches between standard UART serial communication and UPDI programming modes using USB RTS control, featuring a CH340E bridge and integrated RS2228 analog multiplexer.

![Rendering](https://github.com/Grunwalskii/UPDI-Serial-Auto-Programmer/blob/main/Images/3DRendering.png)

## 🔌 Project Overview

This repository contains the design files for the **UPDI Serial Auto Programmer**, a highly integrated development tool designed for modern AVR microcontrollers (such as ATtiny 0/1/2-series, ATmega 0-series, and AVR Dx/Ex families). 

Unlike standard UPDI programmers that require physical jumpers or manual swapping between programming and serial monitoring modes, this board features an **automatic multiplexing circuit**. By utilizing the USB UART's **RTS (Request to Send)** line, the programmer dynamically switches between standard full-duplex UART serial communication and single-wire UPDI programming.

---

## 🛠️ Hardware Architecture & Features

### 1. USB & UART Core
* **USB Interface:** Standard **USB Type-C** (16-pin) connector configured with proper $5.1	{ k}\Omega$ pull-down resistors (`USB_R1`, `USB_R2`) on the CC lines for wide compatibility with host devices.
* **USB-to-UART Bridge:** Powered by the compact **CH340E** IC, ensuring excellent cross-platform driver compatibility and reliable high-baud-rate communication.

### 2. Auto-Switching Circuitry
* **High-Speed Multiplexer:** An **RS2228XUTQK10** dual SPDT analog switch (`U7`) acts as the core multiplexing engine.
* **RTS Control:** The `RTS` signal from the CH340E manages the multiplexer's control pin (`OE#`/`S`). 
  * **High / Default (Serial Monitor Mode):** Routes standard full-duplex `TXD` and `RXD` to the target header.
  * **Low (UPDI Programming Mode):** Couples the lines through a protective diode and resistor network (`1N5819WS` and $470\,\Omega$ / $1	ext{ k}\Omega$ resistors) to interface with the target's single-wire UPDI line automatically.
* **Timing Network:** An RC filter configuration ($120	ext{ k}\Omega$, $5.1	ext{ k}\Omega$, and $100	ext{ nF}$) conditions the `RTS` logic line transition profile (Rise time to $1.6	ext{ V}  pprox 4.51	ext{ ms}$; Fall time to $0.5	ext{ V}  pprox 25	ext{ ms}$).

### 3. Power Management
* **Dual Voltage Support:** Features a **AMS1117-3.3** Low Dropout (LDO) linear regulator to derive a stable 3.3V rail from the 5V USB bus line, supported by $10\,\mu	ext{F}$ filtering capacitors.
* **Rail Selection:** An `MS-22D28-G020` DPDT slide switch allows seamless manual switching between 5V and 3.3V power routing to the target.
* **Status LEDs:**
  * **Red LED:** Indicates the system is actively operating on the 5V rail.
  * **Green LED:** Indicates the system is operating on the 3.3V rail.

### 4. Target Interface Connectors
The output interface exposes a 7-pin header (`X6511WR-07H`) breaking out all necessary signals for the target board:
1. `GND`
2. `VCC` (Selected via slide switch)
3. `3.3V` (Direct output)
4. `5V` (Direct output)
5. `RXD`
6. `TXD`
7. `UPDI`

---

## 🚀 How It Works

1. **Standard Operation:** During normal terminal operations or debugging, `RTS` remains high. The analog switch links the target directly to the CH340E's hardware TX/RX lines for bidirectional serial printing.
2. **Programming Trigger:** When your programming tool chain (e.g., `pymcuprog` or `avrdude`) initiates a connection, it drops the `RTS` line low. 
3. **Automated Handoff:** The analog switch instantly redirects the TX/RX paths into the shared, bi-directional single-wire UPDI configuration required by the modern AVR architecture. 
4. **Post-Flash Restoration:** As soon as programming concludes and the software releases the port, `RTS` toggles back, re-enabling the transparent serial terminal mode instantly without resetting your hardware layout manually.
   
⚠️ **Hardware Dependency:** The automatic switching relies on the USB-UART chip's **RTS pin** toggling states; the programming IDE must support switching this line. The **Arduino IDE (via megaTinyCore)** handles this natively, whereas tools like PlatformIO or raw avrdude command lines may require custom upload flags to toggle the RTS line properly.

## Support

If you enjoy this project and want to support its development, you can buy me a coffee:

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/W1L623I6WG)
