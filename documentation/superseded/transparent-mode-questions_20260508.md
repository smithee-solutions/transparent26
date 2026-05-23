---
title: Transparent Mode in OSDP
subtitle: Call for feedback
author: Rodney Thayer rodney@smithee.solutions
date: 20260508
---


\newpage {}
# 1. Introduction #

This document discusses the draft proposal to describe
the use of transparent mode as
identified for evaluation in the OSDP Verified program.

The document describes the expected usage scenario, enumerates some
sample credential formats expected to be used, and suggests a recommended
message flow, using current-generation OSDP messages.  A list of questions 
is then presented.

This is not a "change the spec" exercise, this is intended to use the
existing spec as corrected.  Therefore proposed enhancements while
relevant to the specification process are not discussed here.

# 2. Expected Scenario #

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

# 3. Suggested OSDP 2.2.2 Example Message Flow #

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

# 4. Example credentials #

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

# 5. Questions #

(See also discussion thread on Basecamp.)

1. is "smartcard/1/0" the expected capability setting for transparent mode?
2. is it mandatory that a transparent-mode reader wake up in mode 0?
3. is it correct to assume there is currently an implied ACURXSIZE sufficient for
a whole standard APDU?
4. is it mandatory to do a card scan?
5. do we need to use osdp_ACURXSIZE?
6. do we need to use osdp_KEEPACTIVE?
7. are the message sizes for osdp_XWRD and osdp_XRD tracking the max message sizes in the
ACU and PD?
8. what are the expected NAK values for osdp_XWR?  (A NAK/Message Check has been
seen.)
9. What is the expected behavior if a card in extended apdu mode attempts to return
more data than the established maximum sized OSDP packet?

\newpage {}

# References #

[1] ISO 7816-4-2013

[2] PSIA PKOC NFC spec

[3] NXP AN-14223

[4] NXP AN-10957

[5] NIST SP800-73-5

[6] TWIC

[7] PK-PACS spec

[8] SIA OSDP 2.2.2

