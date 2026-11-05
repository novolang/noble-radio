# What a chip implements

`noble-radio` states what the noble Link Layer asks of a chip.  A vendor
package implements the traits for its registers; the Link Layer, written
over the traits, then runs on that chip unchanged.  This page lists what
each trait asks, the effect a backend declares, and how to check a backend
before it meets a radio.

## The four traits

| Trait | The Link Layer asks it for |
|---|---|
| `Timebase` | the time in microseconds on 64 bits, a free-running 32-bit counter for short spans and its rate, the core's cycle counter, and a wait until a time |
| `Radio` | the access address, CRC seed and PHY; the channel, power, whitening and ramp-up; five packet buffers; what the radio does by itself at the end of a packet; starting the transmitter or receiver; the events of a packet and the waits for them; the marks that turn an access address into a time |
| `Cipher` | AES-CCM sealing and opening of a data PDU in place, one AES-128 block, and a count of the operations the engine failed to finish |
| `Ecdh` | the P-256 public key of a private key, the DHKey of a private key and a peer's public key, and a count of the failures |

A chip that has no AES or P-256 engine does not implement `Cipher` or `Ecdh`
in registers: `noble-controller` holds a software `Cipher` (`SoftCipher`),
and `orbit/ble` holds a software `Ecdh` over `crypto-ble-ecc`.  A `Radio` and
a `Timebase` are always the chip's.

## The effect row

A trait takes an effect parameter, `Radio[e]`, and a backend names the
effects its methods perform.  A backend over registers writes
`impl Radio[hw] for MyAir`; a backend that also writes state it owns adds
`mutate`.  The Link Layer is generic over the backend (`R: Radio[e] +
Timebase[t]`), so a program built on a register backend carries `[hw]` and a
program built on a scripted backend carries no effect.  This is why the
traits are in a `core` package and a chip implementation is not.

## What a Radio promises

- **The configuration comes back.**  `config(aa, crc_init, maxlen)` points
  the radio at an access address and a CRC seed, and brings the PHY back to
  LE 1M, the ramp-up back to the one the interframe space is qualified for,
  whitening back on and the power back to 0 dBm.  `phy`, `ramp`, `whitening`
  and `power` change one setting from the next start on.
- **Leads are the radio's.**  `tx_lead_us` and `rx_lead_us` are the time
  from a start to the first bit on air and to a receiver that listens; the
  program starts the radio that long before the time it wants.  They follow
  the PHY and the ramp-up.
- **The turn-around is the radio's.**  `arm(next, rssi)` makes the radio
  rest, turn to the transmitter or turn to the receiver T_IFS (150
  microseconds) after the end of a packet, with no software in the path.
  `disarm` and `halt` take the arming away.
- **Events are cleared by name.**  `clear_all`, `clear_address`,
  `clear_packet` and `clear_done` forget the events of a packet; the waits
  `wait_address`, `wait_phyend` and `wait_rx` answer `false` or `RX_TIMEOUT`
  when the event does not come within a bound.  Every spin on a flag is
  bounded in the backend.
- **Buffers are numbered.**  `BUF_TX`, `BUF_RX`, `BUF_ANS`, `BUF_CUR` and
  `BUF_NXT` name five buffers of 260 octets; `put`, `read`, `read_pdu`,
  `octet` and `set_octet` move octets between a `Pdu` and a buffer.
- **A radio may refuse a PHY.**  `supports_phy` answers `false` for a PHY the
  chip cannot send and receive; the Controller then refuses it everywhere it
  is named.

`ramp_us`, `has_fast_ramp` and `octet_work_ns` are what the Controller reads
once, through `profile_of`, to derive its budgets (the lead before a
booking, the guard before a higher one, the spacing of the subevents of a
stream).  A port reports the figures it measures.

## What a Timebase promises

`now_us` never goes back.  A backend that extends a narrower counter needs
a call at least as often as the counter wraps.  `stamp` ticks `stamps_per_us`
times a microsecond and wraps at 32 bits, so a span is
`(stamp() - t0) & 0xFFFFFFFF`.  `wait_until(t)` returns once `now_us()` has
reached `t`, or after a bound that a stopped clock cannot hold.

## What a Cipher and an Ecdh promise

`seal` encrypts the payload in place and appends the four octets of the MIC,
so the length octet grows by 4; `open` checks the MIC, decrypts in place and
answers `false` when it does not match.  An empty PDU passes through both
unchanged; a payload shorter than a MIC does not open.  `dir` carries the
directionBit in bit 0 and says in bits 1 and 2 whether the PDU is a
Connected or a Broadcast Isochronous PDU, whose header octets mask different
bits.  An engine that fails to finish leaves an empty PDU on `seal`, so that
nothing clear is sent, and counts the failure in `faults`.

`Ecdh.public_key` answers `false` for a private key that is zero or not
below the order of the curve, so that the caller draws another;
`Ecdh.dhkey` answers `false` for a peer key that is no point of the curve.
Keys cross the interface in the order HCI carries them: X then Y, 32 octets
each, least significant octet first.  `debug_private_key` is the private key
of the Security Manager's Debug mode in that order.

## Checking a backend

The suites under `tests/` show the shape.  `tests/ll_radio_tests.nv` holds a
radio and a clock with no registers (`FakeAir`) and a generic function over
`R: Radio[e] + Timebase[t]` that runs an advertising event; a chip package
can run the same function against its own backend on a bench, and
`tests/ll_cipher_tests.nv` and `tests/ll_ecdh_tests.nv` do the same for a
cipher and a curve.  The Controller package holds a scripted air that runs
its whole radio program on a PC.

## The PDU and PHY types

`Pdu` is a fixed buffer of 260 octets with the number written: the radio's
S0 field is octet 0 and its LENGTH field is octet 1, so a `Pdu` is the bytes
the radio sends or fills, header first.  The functions of `ll_pdu` build and
read the seven legacy advertising PDU types; every function takes a `Pdu` as
a `var` parameter so that the call passes its address, and none allocates.
`ll_phy` numbers the PHYs as HCI does (`PHY_1M`, `PHY_2M`, `PHY_CODED`),
keeps their masks and options, and gives the time a packet takes on air
(`air_us`) and the payload a time allows (`fit_octets`).  Nothing there
knows a chip.
