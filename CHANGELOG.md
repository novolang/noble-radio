# Changelog

Every published version, newest first. This file is on the publish
allow-list, so it travels with the package: it is the only thing a
consumer deciding whether to upgrade can read.

## 0.1.2 — 2026-11-25

`ll_radio.work_us(p, base_us)` gives the time the program's own work takes on the core the radio reports: `base_us` when the core takes no more than `WORK_NS_REF` (3,000, the nRF54LM20's figure) for an octet (`Radio.octet_work_ns`), and in proportion to the time it takes when it takes more.  `lead_us` and `stream_guard_us` use it for `LEAD_WORK_US` and `STREAM_WORK_US`: a radio whose core is slower than the nRF54LM20's at 128 MHz is given a longer lead before a booking.  A radio that was not asked, one that reports 3,000 or less, and every other budget are unchanged, and no trait changed.

2026-11-23: a second backend implements `Radio` and `Timebase`, the nRF52840's (`orbit/bsp/nordic/nrf52/blocks/radio_nrf52.nv`); the traits did not change.  `docs/radio.md` says which chips implement them.

`ll_radio.tifs_register_us(base_us, phy, s8, rx_s2, s2_late_us, s2_after_s2_us)` gives the value of the interframe register of a radio that times the turn-around itself, for a transmitter on `phy` (`s8` says LE Coded sends S=8) answering a packet that is S=2 when `rx_s2` is true: `base_us` on every PHY and coding, `base_us - s2_late_us` on LE Coded with S=2 after an S=8 packet, and `base_us - s2_late_us + s2_after_s2_us` on LE Coded with S=2 after an S=2 packet, for a radio that starts the transmitter of S=2 `s2_late_us` microseconds later than that of S=8 and answers an S=2 packet `s2_after_s2_us` earlier than an S=8 packet.  A radio with no such differences passes 0 and 0.  Nothing existing changes, and the function allocates nothing.

`ll_radio.tifs_register_us` adds `s2_after_s2_us` for a transmitter of either coding on LE Coded when the packet answered is S=2 (it was added for a transmitter of S=2 only).  `Radio.tx_address_us(phy)` gives the microseconds from the first bit of a packet the radio sends to its ADDRESS event; its body answers `ll_phy.aa_end_us(phy)`, so a radio that does not override it is unchanged.

## 0.1.1 — 2026-11-10

`ll_cs_radio`: the seam between the Channel Sounding procedures of the Link Layer and a radio that measures.  The trait `CsRadio` (`supports_cs`, `cs_caps`, `cs_timing`, `cs_cal`, `cs_fae`, `cs_arm`, `cs_load`, `cs_run`, `cs_step_raw` and `cs_abort`) has a body for every method that answers that the radio has none, so a chip without Channel Sounding writes none of them.  The records it names are `CsCaps`, `CsTiming`, `CsCal`, `CsStep`, `CsPlan`, `CsTone`, `CsStepRaw` and `CsRaw`, with the constants `CS_OK` to `CS_ABORTED`, the roles, the modes and the limits on steps and subevents.  Nothing existing changes, and no function allocates.

`cs_sync_preamble_bits`, `cs_sync_bits`, `cs_sync_bit_ns`, `cs_sync_preamble` and `cs_sync_trailer` describe a CS_SYNC packet on the three CS_SYNC PHYs: the preamble (8 bits on LE 1M, 16 on LE 2M and LE 2M 2BT, the first of them the least significant bit of the access address), the access address, the trailer (4 bits that continue the alternation of the last address bit) and the sequence, with the time a bit takes.  A backend uses them to program its packet format on LE 2M.  No behaviour changes.

`CsPlan` carries two fields the Controller fills from the procedure: `ant_sync` (the antenna element CS_SYNC uses) and `fae_peer` (the peer's table of 72 frequency actuation errors in 1/32 ppm, empty when the Controller has none).  `cs_fae_index` gives a channel's place in the table and `cs_peer_fae` reads the value for a channel from a plan.  A radio that takes the peer's table into its mode 0 measurement reads it from the plan.

## 0.1.0

First release.  The traits between the noble Link Layer and a chip: `Radio`
and `Timebase` (the transceiver and the clock), `Cipher` (AES-CCM and the
AES-128 block) and `Ecdh` (the P-256 engine), with the types they name: the
legacy advertising `Pdu` and its codec, the PHY constants and the on-air
time functions, `EncKey`, and `RadioProfile`, which holds the ramp-up times
the Controller derives its budgets from.  The package holds no register and
no chip; a vendor package implements the traits.  It has no
dependencies.
