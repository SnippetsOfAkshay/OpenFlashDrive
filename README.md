# 💾 OpenFlashDrive
A from-scratch USB-C mass-storage device built around the Prolific PL2732 USB 3.0-to-eMMC controller, with two 64 GB eMMC chips presented to the host as a single 128 GB volume.
![Flash Drive render](Render/Flash%20Drive.png)
Status: schematic and PCB design study · EDA: KiCad 10.0.5 · Link speed: USB 3.0 SuperSpeed (5 Gbps), USB 2.0 fallback

---

## 📋 Overview

A complete schematic capture of a USB 3.0 flash drive, built to learn how storage, high-speed USB, USB-C orientation handling, power distribution, and ESD protection come together on one board.

The design is a hierarchical, multi-sheet KiCad schematic. It is the design only: nothing here has been fabricated or brought up on hardware yet.

What it does:
- Two 64 GB eMMC devices combined into one ~128 GB USB storage volume (JBOD)
- Full USB-C support: reversible orientation, SuperSpeed and USB 2.0, CC detection
- Dedicated ESD and VBUS protection on every externally exposed line
- A regulated power tree feeding the controller and memory from a single 5 V input

---

## 💸 Why this exists

A finished 128 GB USB-C drive sells for about $27. The parts to build one as an individual, ordering a single unit, come to roughly $110-120. The two most important chips, the controller and the memory, aren't stocked at the usual Western catalog distributors at all, and most of that cost is the memory.

This exists to show what goes into a device people treat as disposable, and what low-volume hardware sourcing looks like.

---

## 🏗️ System architecture

Signal path, following the data from the plug inward:

![Architecture](Architecture.svg)

Key points the schematic makes explicit:
- ESD protection and VBUS protection run on separate paths. TVS arrays clamp the data lines; a separate device limits current and guards the VBUS rail. Signal and power are protected independently.
- One chip, the HD3SS3220, handles both orientation detection and SuperSpeed muxing. It reads CC1/CC2 to find the plug orientation, then switches the SuperSpeed TX/RX pairs so the controller always sees them correctly, whichever way the cable went in.
- USB 2.0 skips the mux. Only the SuperSpeed pairs need orientation switching, so the D+/D- lines route straight to the controller.
- The controller generates most of its own rails and takes a 1.2 V core voltage from the external buck off the protected VBUS.

---

## 🧩 Key components

| Ref | Part | Function |
|-----|------|----------|
| U1 | Prolific **PL2732** | USB 3.0-to-eMMC storage controller (the bridge) |
| U2, U3 | FORESEE **FEMDNN064G-C9A61** | 64 GB eMMC devices (×2, combined to 128 GB) |
| U4 | TI **HD3SS3220** | USB-C orientation detection + SuperSpeed mux |
| U8 | TI **TPD1S514** | VBUS protection and current limiting |
| U5, U6, U7, U9 | TI **TPD4EUSB30** | ESD / TVS arrays on high-speed data lines |
| U10 | TI **TPS62807** | Step-down buck for the 1.2 V core rail |
| Y1 | Abracon **ABM8W-30.000MHZ** | 30 MHz reference crystal |
| J1 | **USB4155-03-C** | USB Type-C receptacle |
| U11 | **AT25XE512C** | Optional SPI flash for firmware/config (DNP, write-disabled) |

---

## ⚡ Power architecture

A single 5 V USB VBUS input is protected, then regulated into the rails the board needs:

| Rail | Source | Used by |
|------|--------|---------|
| VBUS 5 V (protected) | TPD1S514 from USB VBUS | Buck input, controller |
| 1.2 V core | TPS62807 buck (1 µH, ~0.2 A) | PL2732 core |
| 3.3 V | Controller-generated | eMMC core, SPI flash |
| 1.8 V | Controller-generated | eMMC I/O |

The 1.2 V core rail was validated in TI WebBench before the layout was committed, holding roughly 1.27-1.34 V across the load range in simulation. Local high-frequency decoupling sits at the controller, mux, memory, and regulator.

---

## 📑 Schematic sheets

Hierarchical design, one concern per sheet:

| # | Sheet | Contents |
|---|-------|----------|
| 1 | Root | Top-level block interconnect |
| 2 | USB_CONNECTOR | USB-C receptacle, TVS arrays, TPD1S514, HD3SS3220 mux |
| 3 | USB_FLASH_CONTROLLER | PL2732, crystal, reset, decoupling |
| 4 | eMMC_BANK | Two 64 GB eMMC devices, separate clock/command per device |
| 5 | POWER | TPS62807 buck + WebBench simulation reference |
| 6 | SPI_FLASH | Optional AT25XE512C (DNP) |

The full schematic is in [`USB-TO-MEM.pdf`](USB-TO-MEM.pdf), with source files under the KiCad project.

---

## 📶 Signal integrity & layout notes

- Controlled-impedance routing for the high-speed USB pairs, targeting ~90 Ω differential
- Each eMMC device has its own dedicated command and clock channels and an 8-bit data path
- Separate power and ground strategy for the high-speed digital interfaces
- Connector shield tied to system ground

---

## 📁 Repository structure

```
OpenFlashDrive/
├── README.md
├── USB-TO-MEM.pdf              # Full schematic (all sheets)
├── CAD Files/
│   ├── USB-TO-MEM.kicad_pro    # KiCad project
│   ├── USB-TO-MEM.kicad_sch    # Root schematic
│   ├── *.kicad_sch             # Hierarchical sheets
└──  docs/
    └── board-render.png        # 3D render

```

*(Adjust to match your actual folders as you add files.)*

---

## 🛠️ Opening the project

Built in KiCad 10.0.5. Clone the repo and open `hardware/USB-TO-MEM.kicad_pro` in KiCad 10 or later.

```bash
git clone https://github.com/SnippetsOfAkshay/OpenFlashDrive.git
```

---

## 🚧 Project status

This is a design study. The schematic is complete; the board hasn't been fabricated, assembled, or tested on hardware. Expect some assumptions (impedance targets, power behavior, footprint choices) to need revision on the first spin.

---

## ⚠️ Disclaimer

A personal reference design, shared for learning. It hasn't been validated on physical hardware. If you reuse any of it, check every rail, footprint, and impedance target against the current datasheets before fabrication.

---

## 📜 License

Pick a license before publishing. For a design meant to be learned from and reused, [MIT](https://choosealicense.com/licenses/mit/) (permissive) or [CERN-OHL-S](https://choosealicense.com/licenses/cern-ohl-s-2.0/) (open hardware, share-alike) both work.

---

## 👤 Author

Akshay · [add your LinkedIn URL]

Spot a mistake or have a better way to do something? Open an issue. Corrections welcome.
