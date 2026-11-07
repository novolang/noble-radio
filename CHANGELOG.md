# Changelog

Every published version, newest first. This file is on the publish
allow-list, so it travels with the package: it is the only thing a
consumer deciding whether to upgrade can read.

## Unreleased

`CsPlan` has two fields more: `ant_sync` (the antenna element CS_SYNC uses) and `fae_peer` (the peer's table of 72 frequency actuation errors in 1/32 ppm, empty when the Controller has none).  `cs_fae_index` gives a channel's place in the table and `cs_peer_fae` reads the value for a channel from a plan.  A radio that takes the peer's table into its mode 0 measurement reads it from the plan.

## 0.1.0

First release.  The traits between the noble Link Layer and a chip: `Radio`
and `Timebase` (the transceiver and the clock), `Cipher` (AES-CCM and the
AES-128 block) and `Ecdh` (the P-256 engine), with the types they name: the
legacy advertising `Pdu` and its codec, the PHY constants and the on-air
time functions, `EncKey`, and `RadioProfile`, which holds the ramp-up times
the Controller derives its budgets from.  The package holds no register and
no chip; a vendor package implements the traits.  It has no
dependencies.
