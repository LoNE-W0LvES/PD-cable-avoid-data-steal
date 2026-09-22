# PD-Cable: Anti-Juice-Jacking Smart Power Delivery Bridge

A hardware-level cybersecurity and power electronics concept designed to prevent **"Juice Jacking"** and data theft at public charging stations, while preserving high-speed **USB Power Delivery (USB-PD)** and fast-charging capabilities.

---

## 🛡️ Problem Statement: The Fast-Charging Dilemma

### What is Juice Jacking?
When mobile devices connect to untrusted public USB charging stations (at airports, cafes, hotels, or public transport), malicious charging kiosks or compromised ports can initiate unauthorized data transfers over USB data lines ($D+/D-$ and SuperSpeed pairs). This allows attackers to exfiltrate private photos, contacts, credentials, or install malicious payloads and spyware.

### The Limitation of Traditional "USB Data Blockers"
Standard commercial USB "condoms" or data-blocking adapters simply sever or disconnect the physical $D+$ and $D-$ lines. While this stops data theft, it introduces a major flaw:
1. **Destroys Fast-Charging:** Quick Charge, Samsung AFC, Apple 2.4A, and legacy protocols require handshakes on $D+/D-$. Without them, charging drops to standard USB baseline ($5\text{V} @ 0.5\text{A} = 2.5\text{W}$ or $5\text{V} @ 1.5\text{A}$ BC1.2), causing agonizingly slow charging.
2. **USB-PD CC Line Vulnerabilities:** While USB Power Delivery uses the Configuration Channel (CC) pin for negotiation, passive adapters cannot intelligently translate or sanitize protocols between untrusted chargers and smart devices without exposing lines or breaking contracts.

---

## 💡 The Solution: Active Dual-Controller Air-Gap Architecture

Instead of a passive pass-through cable, this project introduces an **active intermediary bridge** (or micro-powerbank / dongle) that acts as an **air-gap firewall for data**, while actively negotiating fast charging on both sides independently.

```
+-----------------------------------------------------------------------------------+
|                           ANTI-DATA-STEAL PD BRIDGE                               |
|                                                                                   |
|  [ Public Charger / Kiosk ]                                   [ Smartphone / PC ] |
|            |                                                           ^          |
|            v                                                           |          |
|   +-----------------+                                         +-----------------+ |
|   |  Port A (Sink)  |                                         | Port B (Source) | |
|   |  UFP Controller |                                         |  DFP Controller | |
|   | (e.g., CH224K / |                                         | (e.g., SW3516 / | |
|   |    IP2721)      |                                         |     IP6518)     | |
|   +--------+--------+                                         +--------+--------+ |
|            |                                                           ^          |
|     VBUS   |  (High Voltage DC: 9V / 12V / 15V / 20V)                  |  VBUS    |
|            +-------------------+---------------------------------------+          |
|                                |                                                  |
|                        +-------v-------+                                          |
|                        | Power Path &  |                                          |
|                        | DC-DC / Buffer|                                          |
|                        | (Optional Bat)|                                          |
|                        +---------------+                                          |
|                                                                                   |
|   DATA LINES (D+/D-):                                     DATA LINES (D+/D-):     |
|   Terminated Locally /                                    Handled Locally by      |
|   Isolated from Phone                                     Internal DFP IC         |
|                                                                                   |
|   ===================== NO DATA CONNECTION PASS-THROUGH ======================   |
+-----------------------------------------------------------------------------------+
```

---

## ⚙️ How It Works

### 1. Port A (Facing Public Charger - Sink / UFP)
- **Role:** Emulates a high-power sink device.
- **Controller:** Dedicated PD Sink IC (e.g., WCH CH224K, Ingenic IP2721, or STMicroelectronics STUSB4500).
- **Function:** Negotiates the highest available power profile from the public charging kiosk (e.g., 9V, 12V, 15V, or 20V at up to 3A–5A).
- **Security:**
  - $D+$ and $D-$ from the public charger are strictly terminated into the sink controller or left disconnected.
  - **No physical trace or digital pathway connects Port A's data lines to Port B.**
  - SuperSpeed data lines ($TX/RX$) are completely omitted from the PCB.

### 2. Isolation & Power Regulation Stage
- **Power Path Management:** VBUS power from Port A passes through an overvoltage protection (OVP) and overcurrent protection (OCP) circuit with an ideal diode (e.g., TI LM66200 or Maxim MAX40200) to prevent reverse current.
- **Optional Micro-Buffer / Battery (Powerbank Mode):** A small single-cell Li-ion buffer (e.g., 500mAh–2000mAh) can be integrated to smooth out unstable public power rails or provide battery top-offs.

### 3. Port B (Facing Mobile Device - Source / DFP)
- **Role:** Acts as an independent, certified power source.
- **Controller:** Dedicated PD Source / Buck-Boost Controller (e.g., Ismartware SW3516 / Injoinic IP6518 / TI TPS25750).
- **Function:** Advertises clean Power Data Objects (PDOs) to the connected smartphone, negotiating full-speed USB-PD 3.0, PPS, or QC fast-charging directly.
- **Security:**
  - The smartphone communicates exclusively with the internal onboard controller.
  - The phone never has physical or logical contact with the external public kiosk.

---

## 🔒 Security vs Performance Matrix

| Feature | Direct Public Cable | Standard Data Blocker | **This PD Bridge Concept** |
| :--- | :---: | :---: | :---: |
| **Juice-Jacking Protection** | ❌ None (High Risk) | ✅ 100% (Physical cut) | ✅ **100% (Air-Gapped Data)** |
| **USB Power Delivery (PD)** | ✅ Fast | ❌ Broken / Disabled | ✅ **Full Speed (Up to 65W–100W)** |
| **Quick Charge / AFC / FCP** | ✅ Fast | ❌ Broken / Disabled | ✅ **Full Speed Handshake** |
| **Malware / Payload Injection** | ❌ Vulnerable | ✅ Immune | ✅ **Immune** |
| **BadUSB / HID Spoofing** | ❌ Vulnerable | ✅ Immune | ✅ **Immune** |
| **Voltage Spike / Surge Isolation** | ❌ None | ❌ None | ✅ **Built-in OVP/OCP Clamping** |

---

## 🛠️ Proposed Hardware Implementation & IC Candidates

### Architecture A: Direct Dual-Controller Bridge (No Battery)
- **PD Sink (Port A):** **CH224K** / **IP2721** (Automatic PD/QC decoy negotiation to request 9V/12V/15V/20V).
- **DC-DC Step-Down / Source (Port B):** **SW3516** or **IP6518** (Multi-protocol fast-charging step-down power controller supporting PD3.0, PPS, QC4+, SCP, FCP, AFC).
- **Form Factor:** Compact inline dongle or molded cable head.

### Architecture B: Integrated Micro-Powerbank (With Battery Buffer)
- **Battery Management:** **IP5356** or **SW6208** (Bidirectional fast-charging powerbank SoC with integrated dual-port PD negotiation).
- **Battery:** Flat 3.7V Li-Po cell.
- **Advantage:** Completely decouples public electrical noise and provides temporary off-grid charging.

---

## 🗺️ Roadmap

- [x] Theoretical system design & isolation concept.
- [ ] Schematic drafting in EasyEDA / KiCad (CH224K sink + SW3516 source).
- [ ] 2-layer compact PCB layout with thermal relief for 30W–65W continuous dissipation.
- [ ] Prototype fabrication and testing with USB power analyzer and oscilloscope.
- [ ] 3D enclosure design for portable keychain / cable dongle.