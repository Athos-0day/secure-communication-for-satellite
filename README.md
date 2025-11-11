# 🛰️ STM32H7 Secure Communication PoC

> Proof-of-Concept demonstrating AES-GCM encryption/decryption on an **STM32H7 (ART-Pi / DFR0942)**,  
> using the **hardware cryptographic accelerator (CRYP)** and **RNG peripheral**.  
>  
> Two PoCs are defined:  
> 1. **PoC1 – CPU-driven AES-GCM** (implemented and tested)  
> 2. **PoC2 – DMA-assisted AES-GCM** (to be extended)

---

## Overview

This repository contains a minimal, documented demonstration of **AES-GCM (AES-256)** on the STM32H7 platform using the **CRYP hardware peripheral**.

The PoC simulates **secure communication between a ground station and a satellite**:
- Telecommands (**TC**) are received as encrypted packets and **decrypted** on-board.
- Telemetry (**TM**) is produced by the satellite, **encrypted**, and transmitted securely.

The objective is to validate:
- the STM32H7’s hardware crypto workflow,  
- and the correct enforcement of **confidentiality**, **integrity**, and **authenticity**.

---

## Features

- AES-GCM encryption/decryption via hardware CRYP peripheral  
- RNG-based IV generation  
- Example framing: `[Header | IV | Ciphertext | AuthTag]`  
- UART loopback simulation for ground↔satellite flow  
- Verification of `AuthTag` to ensure message integrity  
- PoC2 (future): DMA integration for throughput improvement  

---

## Hardware & Tools

| Component | Description |
|------------|-------------|
| **Board** | ART-Pi STM32H7 (DFR0942) |
| **MCU** | STM32H750 / STM32H743 / STM32H753 compatible |
| **IDE** | STM32CubeIDE v1.12.1+ |
| **Toolchain** | arm-none-eabi (bundled with CubeIDE) |
| **Programmer** | ST-LINK (on-board debugger) |
| **Console** | 115200 8N1 via UART3 (`PC10/PC11`) |

---

## Repository layout

```
STM32H7_SecureComms_PoC/
│
├─ PoC1_AESGCM_CPU.zip        # Exported STM32CubeIDE project (ready to import)
├─ PoC2_AESGCM_DMA.zip
│
├─ report.pdf
│
└─ README.md
```

---

## 🚀 Build & Run

### 🧩 Using STM32CubeIDE (recommended)

1. Open **STM32CubeIDE**  
2. Go to **File → Import → Existing Projects into Workspace**
3. Select and **import `PoC1_AESGCM_CPU.zip`**
4. Connect the **ART-Pi via USB-DBG**
5. Build the project → `Project → Build Project`
6. Flash/debug → `Run → Debug` or `Run → Run`
7. Open serial terminal (115200 8N1) to monitor output

You should observe the encryption/decryption test output on UART —  
with plaintext, ciphertext, and decrypted buffers matching (confirming tag verification).

---

### 🧑‍💻 CLI / Headless build (optional)

If you prefer to build from terminal:

```bash
# Flash with STM32CubeProgrammer CLI
STM32_Programmer_CLI -c port=SWD -w PoC1_AESGCM_CPU/Debug/PoC1_AESGCM_CPU.bin 0x08000000
```

On macOS/Linux, ensure `STM32_Programmer_CLI` is installed (via ST’s package).

---

## 🔐 How the PoC1 works

### Data flow (simplified)
```
Ground Station (UART TX) 
   ↓
  [Ciphertext + Header + Tag]
   ↓
MCU (UART RX → CRYP Peripheral)
   ↓
AES-GCM Decryption → Tag Verification
   ↓
Plaintext in RAM → Execution
```

And for telemetry:
```
MCU (RAM Payload)
   ↓
AES-GCM Encryption → Generate Tag
   ↓
UART TX → Ground Station
```

---

### Frame format

```
[ Header (AAD) | IV (12 bytes) | Ciphertext (N bytes) | AuthTag (16 bytes) ]
```

| Field | Description |
|-------|--------------|
| **Header (AAD)** | Authenticated but not encrypted (e.g., source, destination) |
| **IV (Nonce)** | 96-bit unique value per encryption |
| **Ciphertext** | Encrypted message data |
| **AuthTag** | 128-bit tag proving integrity & authenticity |

---

### Security properties verified

- **Confidentiality:** AES-256 encryption ensures message secrecy.  
- **Integrity:** `AuthTag` detects any modification of header/ciphertext.  
- **Authenticity:** Valid tag confirms correct source and key usage.  

---

## Notes & Best Practices

- **IV Uniqueness:** Always generate a new IV for each encryption (use RNG or counters).  
- **Key Protection:** Never hardcode production keys. Use secure provisioning and storage.  
- **Header Usage:** Include metadata like message type and counters to prevent replay.  
- **Tag Check:** Always verify before accepting or processing decrypted data.  
- **Word Alignment:** CRYP HAL expects 32-bit words (pad incomplete blocks with zeros).  

---

## 🧩 Next Step — PoC2 (DMA Mode)

The second PoC extends this concept by enabling **DMA transfers** between RAM and CRYP,  
reducing CPU load and improving throughput — ideal for high-frequency telemetry frames.

---

## 📜 License

This project is released under the **MIT License**.  
See the `LICENSE` file for details.

---

## 👤 Author

**Arthur Morain**  
*OBC Developer – TOLOSAT*  
