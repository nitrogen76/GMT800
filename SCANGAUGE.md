## Current Commanded Gear:

| Field | Contents |
|-------|----------|
| TXD   | 6C10F122199A01 |
| RXF   | 04620519869A   |
| RXD   | 3008           |
| MTH   | 000100010000   |

## Transmission Fluid Temp:

| Field | Contents |
|-------|----------|
| TXD   | 6C10F122194001 |
| RXF   | 046205190640   |
| RXD   | 3008           |
| MTH   | 00090005FFD8   |

_temp in freedom units_

## Oil Life:

| Field | Contents |
|-------|----------|
| TXD   | 6C10F122119F01 |
| RXF   | 04628511069F   |
| RXD   | 3008           |
| MTH   | 00C800330000   |

_percent_

## Engine Torque

| Field | Contents |
|-------|----------|
| TXD   | 6C10F12219DE01 |
| RXF   | 0462851906DE   |
| RXD   | 3010           |
| MTH   | 000A00020000   |

_ft-lbs_


## What does all this mean??

# OBD2 Custom PID Fields (TXD, RXF, RXD, MTH)

When defining custom PIDs (such as for ScanGauge or OBDLink), the fields `TXD`, `RXF`, `RXD`, and `MTH` describe how to request and interpret data from a vehicle ECU.

---

## 🧠 Overview

Each custom PID consists of:

| Field | Purpose |
|------|--------|
| TXD  | The message sent to the ECU |
| RXF  | Filter to select the correct response |
| RXD  | Location of the data in the response |
| MTH  | Math used to convert raw data into a usable value |

---

## 📤 TXD — Transmit Data

This is the **raw request sent to the ECU**.

### Example

TXD: 6C10F122199A01


### Breakdown (typical GM J1850 VPW)
| Bytes | Meaning |
|------|--------|
| 6C   | Priority / header |
| 10F1 | Target + source address |
| 22   | Mode (0x22 = Read Data by Identifier) |
| 199A | PID (Parameter ID) |
| 01   | Subfunction / extra byte |

👉 TXD defines *what data you are asking for*.

---

## 📥 RXF — Receive Filter

This defines which responses should be accepted.

### Example
RXF: 04620519869A


### Purpose
- Matches response mode (e.g. `62` = reply to `22`)
- Matches PID echoed in response
- Filters out unrelated traffic on the bus

👉 RXF acts like a packet filter:
> Only accept responses that match this pattern.

---

## 📦 RXD — Receive Data Definition

This tells the scanner **where the actual data is located** in the response.

### Example
RXD: 3008

### Format
SSLL


| Part | Meaning |
|------|--------|
| SS   | Start position (offset) |
| LL   | Length in bits |

👉 RXD defines:
- where to start reading
- how many bits to extract

---

## 🧮 MTH — Math Conversion

This converts the raw extracted value into a human-readable number.

### Example
MTH: 000100010000


| Variable | Meaning |
|----------|--------|
| X        | Raw value from RXD |
| A        | Multiplier |
| B        | Offset |
| C        | Divisor |

👉 Used for scaling, unit conversion, and offsets.

---

## 🔧 Example End-to-End

### Request
TXD: 6C10F122199A01
### ECU Response (example)
6C F1 10 62 19 9A 03

### Interpretation
| Step | Description |
|------|------------|
| RXF  | Confirms this is the correct response (`62 19 9A`) |
| RXD  | Extracts `03` from the response |
| MTH  | Converts raw value to final output |

👉 Final result: `3` (e.g., current gear)

---

## ⚠️ Notes (GM / J1850 VPW)

- Mode `22` is manufacturer-specific (not standard OBD-II)
- Responses typically use mode `62`
- Data may be:
  - multi-byte
  - signed
  - scaled in non-obvious ways

👉 If values seem incorrect:
- Check byte alignment (RXD)
- Verify scaling (MTH)

---

## 🧩 Summary

| Field | Role |
|------|------|
| TXD  | Ask for data |
| RXF  | Identify correct response |
| RXD  | Extract raw value |
| MTH  | Convert to usable value |

---

## 💡 Tip

When reverse engineering:
- Capture request/response pairs
- Identify repeated patterns in responses
- Adjust RXD and MTH until values match expected real-world behavior

