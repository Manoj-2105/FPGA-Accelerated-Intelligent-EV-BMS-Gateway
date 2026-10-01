# ⚡ PolarFire SoC Battery Management System (BMS)

> A real-time, FPGA-accelerated Battery Management System for a 3S Li-ion pack, built on the **Microchip PolarFire SoC Icicle Kit** (RISC-V + FPGA fabric).

![System Architecture](docs/architecture.png)

---

## 📌 Overview

Battery packs fail silently: an over-voltage cell, a hot spot, or an unbalanced pack can degrade capacity or become a safety hazard. This project monitors a **3-cell lithium-ion pack** in real time, detects faults in hardware, estimates battery state, and exposes everything through a live web dashboard and an optional CAN bus link to a vehicle.

The key idea is a **hybrid architecture** on a single chip:

- **FPGA fabric** handles the time-critical work: sensor interfacing, filtering, cell balancing logic, and fault detection, all in hardware with deterministic latency.
- **RISC-V subsystem (Linux)** handles the application layer: SOC/SOH estimation, system management, communication and the dashboard.

---

## ✨ Features

- 🔋 **Per-cell voltage monitoring** for a 3S Li-ion pack (BQ76920)
- 🛡️ **Protection flags**: over-voltage (OV), under-voltage (UV), over-current (OCP), over-temperature (OT)
- 🌡️ **Temperature monitoring** via NTC thermistor
- ⚡ **Pack current & voltage sensing** (INA219 + shunt resistor)
- ⚖️ **Cell balancing logic** implemented in FPGA fabric
- 📊 **SOC calculation, SOH estimation & battery state analysis** on the RISC-V core
- 🌐 **Real-time web dashboard** over Ethernet / Wi-Fi (HTTP)
- 🚗 **Optional CAN bus (ISO 11898)** interface for vehicle communication
- 🔌 **Single-input power tree**: 12 V / 5 V in → 5 V → 3.3 V

---

## 🏗️ System Architecture

The system is split into five blocks (see the diagram above):

### 1. Battery Pack
A 3S Li-ion pack (3 × 3.7 V cells in series). Cell taps (VC0–VC2) go to the monitor IC, and a shunt resistor in the low-side path provides current sensing.

### 2. Battery Monitoring & Sensing

| Component | Role | Interface |
|---|---|---|
| **BQ76920** | Battery monitor IC: cell voltage measurement, OV/UV/OCP protection, temperature monitoring, status & fault flags | I²C (SCL, SDA) + `ALERT` interrupt |
| **INA219** | Pack current (charge/discharge) and pack voltage sensor | I²C (SCL, SDA) |
| **NTC thermistor** | Pack temperature, fed to the BQ76920 on `TS1` | Analog |
| **Shunt resistor** | Current sense element | Analog |

### 3. PolarFire SoC Icicle Kit

**FPGA Fabric (Hardware Accelerators)**
- Sensor Data Interface
- Signal Processing & Filtering
- Cell Balancing Logic (algorithm)
- Fault Detection (OV / UV / OCP / OT)

**RISC-V Subsystem (Application Layer)**
- SOC Calculation
- SOH Estimation
- Battery State Analysis
- System Management
- Communication (USB / Ethernet / CAN)
- Dashboard Interface (Embedded Linux)

The FPGA fabric and RISC-V subsystem communicate over an **AXI interconnect**, with a **System Controller & Data Management** block coordinating data between them.

### 4. User Interface
A web dashboard served from the board shows, in real time: **State of Charge, pack voltage, per-cell voltages (C1–C3), current, and temperature**.

### 5. Vehicle Communication (Optional)
A CAN transceiver (e.g., **SN65HVD230**) connects to the SoC and exposes `CANH` / `CANL` per ISO 11898, so BMS data can be shared with a vehicle ECU.

### ⚙️ Power Supply
`12 V / 5 V input → DC-DC converter (5 V) → LDO regulator (3.3 V)`
The 5 V rail powers the sensing hardware; the 3.3 V rails power the SoC kit and peripherals.

---

## 🔄 Data Flow

1. BQ76920 measures cell voltages and temperature; INA219 measures pack current and voltage.
2. Readings arrive in the FPGA fabric over **I²C**; the BQ76920 `ALERT` line raises an interrupt on faults.
3. The fabric filters the data, runs fault detection and cell balancing logic.
4. Processed data crosses the **AXI interconnect** to the RISC-V cores.
5. The application layer computes **SOC / SOH** and analyzes battery state.
6. Results are served to the **web dashboard** and, optionally, broadcast on **CAN**.

---

## 🧰 Hardware Requirements

- Microchip **PolarFire SoC Icicle Kit**
- **BQ76920** battery monitor IC (evaluation board or custom breakout)
- **INA219** current/voltage sensor module
- 10 kΩ **NTC thermistor**
- **Shunt resistor** for current sensing
- 3S Li-ion battery pack (3 × 3.7 V cells)
- *(Optional)* **SN65HVD230** CAN transceiver
- DC-DC converter (5 V) and LDO regulator (3.3 V)
- 12 V / 5 V power source

## 💻 Software & Tools

- Microchip **Libero SoC** (FPGA design & synthesis)
- **SoftConsole** / RISC-V toolchain
- Embedded **Linux** on the PolarFire SoC
- Web dashboard (HTTP served from the board)

---

## 📁 Repository Structure

> Update this to match your actual layout.

```
.
├── docs/
│   └── architecture.png       # System block diagram
├── fpga/                      # Libero project, HDL for accelerators
│   ├── sensor_interface/
│   ├── signal_processing/
│   ├── cell_balancing/
│   └── fault_detection/
├── software/                  # RISC-V application layer
│   ├── soc_estimation/
│   ├── soh_estimation/
│   └── dashboard/
├── hardware/                  # Schematics, wiring, BOM
└── README.md
```

---

## 🚀 Getting Started

### 1. Clone
```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

### 2. Wire the hardware
- Connect cell taps to **VC0, VC1, VC2** on the BQ76920 and the NTC to **TS1**.
- Place the shunt resistor in the pack's low-side current path.
- Connect BQ76920 and INA219 to the Icicle Kit's I²C pins, and `ALERT` to a fabric GPIO.
- Power the system via the 5 V / 3.3 V rails.

### 3. Build & program the FPGA
Open the project in **Libero SoC**, generate the bitstream, and program the Icicle Kit.

### 4. Build & run the software
```bash
# Fill in your build/run steps here
```

### 5. Open the dashboard
Connect the board over Ethernet / Wi-Fi and browse to `http://<board-ip>:<port>`.

---

## 📊 Dashboard

Live view of:

- State of Charge (%)
- Pack voltage (V)
- Cell voltages C1, C2, C3 (V)
- Pack current (A)
- Temperature (°C)

*(Add a real screenshot here: `docs/dashboard.png`)*

---

## 🧪 Current Status

> Edit this honestly, since judges appreciate clarity on what works.

- [ ] Cell voltage & temperature readout (BQ76920)
- [ ] Current / pack voltage readout (INA219)
- [ ] FPGA fault detection
- [ ] Cell balancing
- [ ] SOC estimation
- [ ] SOH estimation
- [ ] Web dashboard
- [ ] CAN integration

---

## 🔭 Future Work

- Extend to higher-cell-count packs (4S and above)
- Improved SOC/SOH models (e.g., Kalman filter or ML-based estimation)
- Data logging and cloud telemetry
- Full CAN protocol integration with vehicle ECUs
- Custom PCB replacing the evaluation setup

---

## 👥 Team

| Name | Role |
|---|---|
| _Your Name_ | _Role_ |
| _Teammate_ | _Role_ |

**Hackathon:** _Event name_ · **Year:** 2026

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for details.

---

## 🙏 Acknowledgements

- Microchip Technology for the PolarFire SoC Icicle Kit
- Texas Instruments for the BQ76920 and INA219 documentation and reference designs
