---
title: DRAFT Implementation Guidance
subtitle: OSDP Transparent Mode
---

\newpage
\tableofcontents
\newpage {}

# 1. Concept of Operations #

The document describes the expected usage scenario, enumerates some
sample credential formats expected to be used, and suggests a recommended
message flow, using current-generation OSDP messages.

# 1.1 Assumptions #

- the PD is assumed to be a contactless reader with no keypad.
- exactly what kind of credential the PD is prepared to interact with is out of
scope, as long as _some_ credential can be used to exercise the command/response sequences.
- SPE is not addressed.
- the "Card Info Report" is not used.
- use of "downstream readers" to implement contact/contactless readers is not addressed.
- extended APDU mode and resultant fragmentation is not addressed.
- This is not a "change the spec" exercise, this is intended to use the
existing spec as corrected.  Therefore proposed enhancements while
relevant to the specification process are not discussed here.

# 1.2 Example credentials #

- initialized but unpopulated cards only capable of returning an ISO 7816-4  UID 1,2 or 3 value
- multi-credential cards containing more than one application.
- single-applet PKOC v1/v2 cards
- PK-PACS X.509 cards
- PIV cards
- TWIC cards
- NXP AN-14223 
- NXP AN-10957
- LEAF Verified
- Aliro
- cards using proprietary formats based on 7816-4 (DESFire, Mifare Classic, SEOS, DUOX, NEON, etc.)

\newpage {}

# 1.3. Expected Scenario #

This is the scenario testing is meant to represent:

- ACU and PD initialize communications and the PD is configured in transparent
mode
- an arbitrary ISO 14443 credential is presented
- the PD provides an "object in field" indication to the ACU
- the ACU performs certain operations using ISO 7816-4 commands to interact
with the credential.  The osdp_XWR command is used to send explicit
7816-4 commands to the credential.  The osdp_XRD response is used
to send credential responses back to the ACU.

\newpage {}

# 2. Implementation Recommendations #

# 2.1 ACU Implementation #

- the ACU ___SHOULD___ implement osdp_ACURXSIZE.
- the ACU MUST be prepared to accept an OSDP response that contains a whole 7816-4 APDU 
(see appendix B for calculation.)
- the ACU SHOULD send osdp_KEEPACTIVE after each osdp_XRD response that contains an APDU.
- the ACU MUST implement:
    - set transparent mode
    - exit transparent mode>
    - osdp_XWR w/APDU
    - osdp_XRD w/APDU
    - osdp_XRD for get status.
- the ACU ___MAY___ refuse to use transparent mode with a PD that does not set the smartcard bit
and/or does not indicate a sufficient receive buffer size.

# 2.2 PD Implementation #

- the PD ___SHOULD___ implement a minimum receive message size sufficient
to accept an osdp_XRW command that contains a whole 7816-4 APDU (see Appendix B for calculation.)
- the PD ___SHOULD___ send the smartcard capability with the blah bit set and bits x to y set to zero
- the PD ___SHOULD___ implement osdp_ACURXSIZE
- the PD ___MAY___ NAK transparent mode commands if the ACU does not assert it can handle full size APDU's
since the default OSPD message size is 128 bytes.
- the PD ___MUST___ implement:
    - osdp_XWR and osdp_XRD response with mode status
    - osdp_XRD response with card present
    - osdp_XWR with APDU
    - osdp_XRD response with APDU

\newpage {}

# 2.3 Suggested Message Dialog #

Given the current state of the standard, including
commands and responses that post-date the inclusion of
transparent mode, this is the suggested sequence of operations:

1. ACU sends osdp_CAP to PD
2. PD sends osdp_PDCAP capabilities report.  Must include smartcard capability 0x01.
Must include max message receive size of at least (258+an OSDP secure channel 1/2 wrapper.)
ELSE ACU must not use transparent mode with this PD.
3. ACU sends osdp_ACURXSIZE of at least (258+an OSDP secure channel 1/2 wrapper.)
ELSE PD must nak all osdp_XWR commands (reason being bad parameter.)
4. ACU sends get-mode command, confirms reader is in mode 0.
This is done to double-check the reader supports transparent mode, since
technically PDCAP responses are optional.
5. ACU sends set-mode to mode 1, do another get-mode to make sure it sticks.
It is expected this test sequence will be used with new implementations
and so confirming the set-mode command worked is thought to be conservatively
appropriate.
6. ACU sends osdp_KEEPACTIVE so the PD keeps the coil energized.  Note this
must be done after every osdp_XWR command.
7. credential is placed in field
8. PD sends card-present osdp_XRD in response to the next poll.  It contains a 
status value of 0x00 under normal conditions.
9. the ACU may optionally send a card scan command.  The PD is expected to
return an osdp_XRD with a card information report.
10. the ACU sends a 7816 command (i.e. a select) using osdp_XWR.
11. he PD will respond, possible on a later poll cycle,
with an osdp_XRD containing an 7816 response.  The response may well
be an error response.  The ACU is responsible for determining the
card format and so it may send more than one commmand to initiate
access to the card.
12. assuming the credential responded positively the ACU sends
subsequent 7816 commands via osdp_XWR, the PD responds with subsequent osdp_XRD
responses, and a message exchange happens.

\newpage {}

# 3. Conformance Requirements #

# 3.1 ACU Conformance #

- must implement mode switch
- must implement apdu send
- must implement accept card present
- must implement accept apdu response
- should implement acurxsize
- should request and act on values returned in PDCAP

# 3.2 PD Conformance #

- (optional) implement ACURXSIZE
- (optional) PD capabilities reports sufficient max rx size
- (optional) implement smartcard capability in capabilities report
- (must) report SIA 2.2.2 or later
- (must) implement osdp_XWR command processing sufficient for test cases
- (must) implement osdp_XRD response processing sufficient for test cases
- (must) be capable of accepting a whole sized 7816-4 APDU in a command
- (must) be capable of transmitting a whole sized 7816-4 APDU in a response

# 4. Testing Suggestions #

(see basecamp response post)

\newpage {}

# Appendix #

# A.1 Colophon #

Source in markdown in github smithee-solutions/transparent26.

# A.2 References #

1. SIA OSDP Application Specific Messages, July 2015.
2. SIA 2.2.2
3. ISO 7816-4-2020
4. OSDP Verified Test Cases github Security-Industry-Association/osdp-conformance.
5. PSIA PKOC NFC spec
7. NXP AN-14223
8. NXP AN-10957
9. NIST SP800-73-5
10. TWIC
11. PK-PACS spec

\newpage {}

# B. Packet Size Calculation #

Assuming Secure Channel (2.2.2 style) a whole OSDP message used for 7816-4 APDU's (not in 
extended APDU mode) this is the packet size calculation.

| Length | Field          |
| ------ | -------------- |
|        |                |
| 1      | SOM            |
|        |                |
| 1 | Address |
|        |                |
| 2 | Length |
|        |                |
| 1 | Control |
|        |                |
| 2 | SC1 SCS Header |
|        |                |
| 1      | Command/Response |
|        |                |
| 3      | XWR/XRD header |
|        |                |
| 258    | Max size 7816 APDU |
|        |                |
| 4      | SC1 MAC        |
|        |                |
| 2      | CRC               |
|        |                |
| 274    | TOTAL=258+16 |

\newpage {}

# C. Command/Response Messages #

# C.1. Command Overview #

These are the named commands.  They represent various but not all command parameter combinations.

- osdp_PR00REQ
- osdp_PR00SET
- osdp_PR01XMIT
- osdp_PR01SPE
- osdp_PR01SCSCAN

or as raw hex, including the OSDP Command code:

```
A1 00 01
A1 00 01 00 00
A1 00 01 00 01
A1 00 01 01 00 <APDU>
A1 01 02 00
A1 01 03 00 <SPE>
A1 01 04 00
```

Each command contains a _mode_ and a _pcmnd_ field and an optiona _pdata_ field.  Some commands include
a (vestigial) _reader number_ values as the first octet of the pdata field.

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

# C.2. Transparent Mode Commands #

# C.2.1 osdp_PR00REQ #

| Offset | Field | Value | Contents |
| -- | ------ | ------ | ------ |
| |         |          |    |
| - | osdp_XWR | 0xA1 | OSDP command |
| |         |          |    |
| 0 | XRW_MODE | 0x00  | Mode 0 command  |
| |         |          |    |
| 1 | XWR_PCMND | 0x01 | Read Mode Setting |

# C.2.2 osdp_PR00SET (Set Mode 0) #

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

\newpage {}

# C.2.3 osdp_PR00SET (Set Mode 1) #

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

# C.2.4 osdp_PR01XMIT (Send APDU) #

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

# C.2.5 osdp_PR01SCDONE (Terminate connection) #

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

# C.2.6 osdp_PR01SPE (Secure PIN Entry) #

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

# C.2.7 osdp_PR01SCSCAN (Smart Card scan) #

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

\newpage {}

# C. Example Traces #

\newpage {}

# Punchlist #

- example traces
- incorporate basecamp testing response post

