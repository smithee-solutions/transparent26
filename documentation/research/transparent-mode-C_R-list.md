---
title: OSDP Transparent Mode Command/Response Operations
---

\newpage {}

# 1. Command Overview #

- osdp_PR00REQ
- osdp_PR00SET
- osdp_PR01XMIT
- osdp_PR01SPE
- osdp_PR01SCSCAN

```
A1 00 01
A1 00 01 00 00
A1 00 01 00 01
A1 00 01 01 00 <APDU>
A1 01 02 00
A1 01 03 00 <SPE>
A1 01 04 00
```

| Offset | Field | Value | Contents |
| -- | ------ | ------ | ------ |
| |         |          |    |
| - | osdp_XWR | 0xA1 | OSDP command |
| |         |          |    |
| 0 | XRW_MODE | t.b.d.  |   |
| |         |          |    |
| 1 | XWR_PCMND | t.b.d. |  |
| |         |          |    |
| 2 | XWR_PDATA | t.b.d. | Optional |

\newpage {}

# 2. Transparent Mode Commands #

# 2.1 osdp_PR00REQ #

| Offset | Field | Value | Contents |
| -- | ------ | ------ | ------ |
| |         |          |    |
| - | osdp_XWR | 0xA1 | OSDP command |
| |         |          |    |
| 0 | XRW_MODE | 0x00  | Mode 0 command  |
| |         |          |    |
| 1 | XWR_PCMND | 0x01 | Read Mode Setting |

\newpage {}

# 2.2 osdp_PR00SET (Set Mode 0) #

This command switches the PD to transparent mode.

| Offset | Field | Value | Contents |
| -- | ------ | ------ | ------ |
| |         |          |    |
| - | osdp_XWR | 0xA1 | OSDP command |
| |         |          |    |
| 0 | XRW_MODE | 0x00  | Mode 0 command  |
| |         |          |    |
| 1 | XWR_PCMND | 0x01 | Set Mode Setting  |
| |         |          |    |
| 2 | Mode code | 0x00 | Mode to enable  |
| |         |          |    |
| 3 | Mode config | 0x00 | Disable card info |
|   |             | 0x01 | Enable card info |

# 2.3 osdp_PR00SET (Set Mode 1) #

| Offset | Field | Value | Contents |
| -- | ------ | ------ | ------ |
| |         |          |    |
| - | osdp_XWR | 0xA1 | OSDP command |
| |         |          |    |
| 0 | XRW_MODE | 0x00  | Mode 0 command  |
| |         |          |    |
| 1 | XWR_PCMND | 0x01 | Set Mode Setting  |
| |         |          |    |
| 2 | Mode code | 0x01 | Mode to enable  |
| |         |          |    |
| 3 | Mode config | 0x00 | Optional, default is 0x00 |

\newpage {}

# 2.4 osdp_PR01XMIT (Send APDU) #

| Offset | Field | Value | Contents |
| -- | ------ | ------ | ------ |
| |         |          |    |
| - | osdp_XWR | 0xA1 | OSDP command |
|   |          |       | |
| 0 | XRW_MODE | 0x01  | Mode 1 command  |
|   |          |       | |
| 1 | XWR_PCMND | 0x01  | Send APDU  |
|   |          |       | |
| 2 | Reader Number | 0x00  | Reader number (use 0x00 always)  |
|   |          |       | |
| 3-n | APDU | ...  | 7816 APDU  |

# 2.5 osdp_PR01SCDONE (Terminate connection) #

| Offset | Field | Value | Contents |
| -- | ------ | ------ | ------ |
| |         |          |    |
| - | osdp_XWR | 0xA1 | OSDP command |
|   |          |       | |
| 0 | XRW_MODE | 0x01  | Mode 1 command  |
|   |          |       | |
| 1 | XWR_PCMND | 0x02  | Terminate connection  |
|   |          |       | |
| 2 | Reader Number | 0x00  | Reader number (use 0x00 always)  |

\newpage {}

# 2.6 osdp_PR01SPE (Secure PIN Entry) #

| Offset | Field | Value | Contents |
| -- | ------ | ------ | ------ |
|   |          |       | |
| - | osdp_XWR | 0xA1 | OSDP command |
|   |          |       | |
| 0 | XRW_MODE | 0x01  | Mode 1 command  |
|   |          |       | |
| 1 | XWR_PCMND | 0x03  | Perform Secure PIN Entry  |
|   |          |       | |
| 2 | Reader Number | 0x00  | Reader number (use 0x00 always)  |
|   |          |       | |
| 3-22 | SPE command structure | ...  | See [1] for structure description.  |

# 2.7 osdp_PR01SCSCAN (Smart Card scan) #

| Offset | Field | Value | Contents |
| -- | ------ | ------ | ------ |
|   |          |       | |
| - | osdp_XWR | 0xA1 | OSDP command |
|   |          |       | |
| 0 | XRW_MODE | 0x01  | Mode 1 command  |
|   |          |       | |
| 1 | XWR_PCMND | 0x04 | Perform a Smart Card Scan  |
|   |          |       | |
| 2 | Reader Number | 0x00  | Reader number (use 0x00 always)  |

# 3. References #

[1] SIA OSDP Application Specific Messages, July 2015.
