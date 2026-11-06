# noble-radio

The seam between the Link Layer of the noble Bluetooth Low Energy stack and
a chip.  The package holds five traits, `Radio`, `Timebase`, `Cipher`, `Ecdh`
and `CsRadio`, and the types they name: the advertising `Pdu` with its codec,
the PHY constants and the on-air time functions, the session key `EncKey`, the
`RadioProfile` the Controller derives its timing budgets from, and the plan
and result records of Channel Sounding.  It holds
no register and no chip.  It depends on nothing, and it has no state and no
effects of its own: a trait takes the effect row of the backend that
implements it.

| Module | Holds |
|---|---|
| `ll_radio` | `Timebase` (the clock), `Radio` (the transceiver), the constants of LE 1M, `RadioProfile` and the budgets derived from it |
| `ll_cipher` | `Cipher` (AES-CCM and one AES-128 block) and `EncKey` |
| `ll_ecdh` | `Ecdh` (the P-256 engine) and the Debug-mode private key of the Security Manager |
| `ll_cs_radio` | `CsRadio` (a radio that measures, for Channel Sounding; every method has a body that answers none) and the records of its capabilities, timing, calibration, the head and the steps of a plan, and the head and the steps of a raw result |
| `ll_pdu` | `Pdu`, a fixed buffer for one PDU, and the builders and readers of the seven legacy advertising PDU types |
| `ll_phy` | the PHY numbers, masks and options, the time a packet takes on air, and the payload a time allows |

A Controller is generic over the traits, so it costs the effects of the
backend that implements them and no call goes through a table.  The
functions of `ll_pdu` and `ll_phy` are plain arithmetic over bytes and
integers.

```novo
use ll_pdu
use ll_phy

// An ADV_IND with the six-octet address C0..C5 is 8 octets with its header; on LE 1M it takes 128 microseconds.
fn adv_ind_air_time() -> Int
    var p = ll_pdu.pdu_new()
    var a: Vec[u8; 6] = Vec.new()
    for i in 0..6
        let _x = a.push((0xC0 + i) as u8)
    ll_pdu.pdu_begin(var p, ll_pdu.ADV_IND, 1, 0)
    ll_pdu.pdu_put_addr(var p, var a)
    ll_pdu.pdu_finish(var p)
    ll_phy.air_us(ll_phy.PHY_1M, false, p.n)
```

The Controller (`noble-controller`) depends on this package, and a vendor
board package implements the traits; the Nordic nRF54 package does.
`docs/radio.md` describes what each trait asks of a chip.

## Install, build and test

```sh
novo pkg add noble-radio
```

adds the package to the `[dependencies]` table of the consumer's `novo.toml`.
In a checkout of the package:

```sh
novo build
novo test
```

`novo build` type-checks and effect-checks every module under `src/`; the
package is a library and writes no binary.  `novo test` runs the suites
under `tests/`, one for each module (`tests/ll_pdu_tests.nv`, for example
`novo test tests/ll_pdu_tests.nv`).  The suites hold a small `Radio` and
`Timebase`, `Cipher` and `Ecdh` of their own, which also show how a trait is
implemented.  The tests are not part of the published package.
