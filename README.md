# noble-radio

The seam between the noble Link Layer (`orbit/ble`, `src/controller/`) and a
chip.  The package holds traits and the types they name, and no register.

| Module | Holds |
|---|---|
| `ll_radio` | `Timebase`, `Radio`, `RadioProfile` and the constants of LE 1M |
| `ll_cipher` | `Cipher` and `EncKey` |
| `ll_ecdh` | `Ecdh` |
| `ll_pdu` | `Pdu` and the legacy PDU codec |
| `ll_phy` | the PHY constants and the on-air time functions |

The Controller depends on this package.  A vendor board package implements the
traits: the nRF54 implementations are in `orbit/bsp/nordic/nrf54/blocks/`.  The
design is in `planning/BLE-RADIO-TO-BSP-MOVE.md`.

`novo test --isolate src/ll_pdu.nv` in this directory runs the
tests of a module.
