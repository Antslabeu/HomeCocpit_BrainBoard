# BrainBoard – STM32G0 CAN-FD Base Board

BrainBoard is a small base board built around an STM32G0B1KET6 MCU and a CAN-FD transceiver.
It is intended as a “brain” for various cockpit / IO modules, providing 3V3 power, CAN-FD
communication and a common expansion interface.

Repo: https://github.com/Antslabeu/HomeCocpit_BrainBoard  

> **Status:** prototype / WIP – use at your own risk.

---

## Main Features

- MCU: **STM32G0B1KET6**
  - CAN-FD, SPI, I²C, UART, timers/PWM, ADC, GPIO exposed on board connectors
- CAN-FD transceiver: **MCP2542FDxMF**
- Power architecture:
  - Input: **12 V**
  - On-board DC/DC step-down (2× **TPS62170DSG**):
    - **3.3 V / 1 A** rail for MCU and logic
    - **5 V** rail available on board
- PCB:
  - **4-layer** board, 1.6 mm
  - Stackup: **JLC04161H-7628** (JLCPCB)

---

## Programming Connector – `STM_PROG` (J1)

6-pin programming/debug connector for SWD + UART:

| Pin | Signal | Net    | Description                      |
|-----|--------|--------|----------------------------------|
| 1   | VCC    | 3.3V   | Target supply (3.3 V)           |
| 2   | DIO    | SW_DIO | SWD data I/O                    |
| 3   | RST    | SW_RST | MCU reset (NRST)                |
| 4   | CLK    | SW_CLK | SWD clock                       |
| 5   | TX     | SW_TX  | MCU UART TX (debug / bootloader)|
| 6   | GND    | GND    | Ground                          |

Compatible with ST-Link, J-Link and other SWD programmers.

---

## IO Connectors (Module Interface)

The board exposes module connectors that carry:

- **SPI bus** (shared)
- **4× chip select (CS) lines**
- **UART**
- **2× PWM outputs**
- **4× ADC inputs**
- **I²C bus** with **4.7 kΩ pull-ups** to 3.3 V
- **4× IO Board ID lines** – used to identify the type of module plugged into the connector  
  *(this Board ID mechanism is WIP and may change)*

Exact pinout is defined in the KiCad schematics (`connectors.kicad_sch`).

---

## CAN-FD Bus

- Transceiver: **MCP2542FDxMF** (CAN 2.0 / CAN FD up to 8 Mbit/s).
- CANH / CANL (and GND / power, if needed) are routed to dedicated connectors.
- Standard **120 Ω termination** is intended **only at the ends of the main bus**
  (check schematic for jumpers / solder bridges).

---

## Power

- **Input:** nominal **12 V**
- **Converters:** 2× **TPS62170DSG** buck regulators  
  (3 V–17 V in, up to 0.5 A each)  [Texas Instruments](https://www.ti.com/lit/ds/symlink/tps62170.pdf?utm_source=chatgpt.com)
- **Outputs:**
  - 12 V → **3.3 V / 1 A**
  - 12 V → **5 V** (aux rail)

### Typical consumption (board only, no external loads – rough estimate)

- STM32G0B1 in Run mode @ 64 MHz: typ. ~8–9 mA @ 3.3 V  [STMicroelectronics](https://www.st.com/resource/en/datasheet/stm32g0b1cc.pdf?utm_source=chatgpt.com)
- MCP2542FD supply current:
  - recessive: typ. ~2.5–5 mA @ 5 V
  - dominant: typ. ~55–70 mA @ 5 V  [DigiKey](https://www.digikey.kr/htmldatasheets/production/1897584/0/0/1/mcp2542-44fd-wfd.html?utm_source=chatgpt.com)
- TPS62170 quiescent current: ~17 µA from VIN each  [Texas Instruments](https://www.ti.com/lit/ds/symlink/tps62170.pdf?utm_source=chatgpt.com)

So, approximately:

- **Idle bus (recessive):**  
  - P_out ≈ 30 mW (MCU) + 25 mW (CAN) ≈ 55 mW  
  - With ~85% efficiency, input power ≈ 65 mW → **I_in ≈ 5–6 mA @ 12 V**
- **Worst-case dominant transmit:**  
  - P_out ≈ 30 mW (MCU) + 350 mW (CAN) ≈ 380 mW  
  - Input ≈ 450 mW → **I_in ≈ 35–40 mA @ 12 V**

Real current will be higher once external modules load 3.3 V rail.

Always respect absolute maximum ratings from datasheets.

---

## Files and Tools

All hardware files are in the repo root:

- `BrainBoard.kicad_pro` – main KiCad 8 project
- `BrainBoard.kicad_sch` + hierarchical sheets:
  - `power.kicad_sch`
  - `mcu.kicad_sch`
  - `can.kicad_sch`
  - `connectors.kicad_sch`
- `BrainBoard.kicad_pcb` – PCB layout (4-layer, JLC04161H-7628)
- `BrainBoard.pdf` – schematic PDF
- `LICENSE` – MIT

**CAD tool:** KiCad **8.x** (designed with 8.0.4).

---

## Getting Started (Hardware)

1. **Open the project**  
   Clone the repo and open `BrainBoard.kicad_pro` in KiCad 8.x.

2. **Review schematics and layout**  
   Check power, CAN and connector pinouts before manufacturing.

3. **Manufacturing**  
   - Generate Gerbers from KiCad.
   - Use the **JLC04161H-7628** 4-layer stackup if ordering from JLCPCB.

4. **First power-up**  
   - Apply **12 V** to the main power input.  
   - Verify 3.3 V and 5 V rails on test points.  
   - Connect a SWD programmer to **STM_PROG** and flash a simple test firmware
     (blink + CAN init).

5. **Integrate into CAN-FD network**  
   - Connect CANH/CANL to your main CAN-FD backbone.  
   - Enable 120 Ω termination only on physical bus ends.

---

## Firmware / Software

Firmware, examples and Board ID handling are **WIP** and not yet part of this repository.

Planned:

- Reference firmware for:
  - CAN-FD communication
  - SPI-based module control
  - Board ID-based module detection

---

## License

This hardware design is released under the **MIT License**  
(see the `LICENSE` file in the repository).

---

## Disclaimer

This is a prototype board.  
No guarantees are given – **use at your own risk**, especially in any safety-critical context.
