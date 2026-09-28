# What this actually is

Precise hardware identity, established by direct inspection (revision docs
+ `i2cdetect` + `dmesg` + live device tree), not assumed from the general
"PinePhone" spec sheet — several things about this specific unit differ
from what you'd expect from generic PinePhone docs. Verified 2026-09-28.

## Chassis

**PinePhone Beta Edition.** Confirmed against Pine64's own hardware
revision notes: the Beta Edition uses motherboard revision **1.2b**, and
its hardware (aside from the sensor substitution below) is identical to
the other community-edition production runs — "Beta" was a software-only
distinction at the time these units shipped, not a rougher board.

Revision 1.2b matters specifically because it's the **newest, most-fixed**
hardware revision (has the 1.2a USB-C CC-pin fix and the 1.2b VBUS/
brightness fix), but also because it's one of the less-travelled variants
in most distros' device trees — see
[postmarketos-1.2b-fixes.md](postmarketos-1.2b-fixes.md) for what that
actually costs you.

## Bootloader

**Tow-Boot**, already installed on the eMMC (not stock U-Boot). If you
ever need to check whether a PinePhone has it: power on and watch for the
LED — solid **blue** during a Volume-Up hold means you're in Tow-Boot's
USB mass-storage mode; a red-then-yellow sequence with its own boot
screen means you're looking at the Tow-Boot *installer* (different
thing, only relevant if Tow-Boot isn't on the device yet at all).

## Sensors — the important substitution

| Sensor | Chip | Notes |
|---|---|---|
| Accelerometer + gyroscope | InvenSense **MPU-6050** | Standard, well-supported everywhere |
| Ambient light + proximity | Sensortek **STK3310** (family: STK3311/STK3335) | Standard, well-supported everywhere |
| **Magnetometer/compass** | Voltafield **AF8133L / AF8133J** | **Not** the LIS3MDL that most PinePhone docs assume |
| Cellular + GNSS | Quectel **EG25-G** | USB-attached; GNSS (GPS/GLONASS/BeiDou/Galileo/QZSS + AGPS) shares the same chip |
| PMIC | X-Powers **AXP803** | Also exposes board temp + rail voltages/currents as a bonus IIO device |

**The magnetometer substitution is the one thing that'll bite you if you
don't know about it.** Most PinePhone units shipped with an ST LIS3MDL
compass. When LIS3MDL became hard to source, Pine64 substituted the
Voltafield AF8133L/AF8133J on Beta Edition units specifically — same
board position, same I2C address (`0x1c` on `i2c1`), same reset pin (PB1),
same power rail (`reg_dldo1`), different chip. Confirmed present on this
exact unit via `i2cdetect -y 1` — a device also answers at `0x1c` on
`i2cdetect -y 5`, which we haven't chased down (could be the same
controller exposed under two bus numbers, could be something else
entirely at a coincidentally-matching address; hasn't caused a problem
either way, but don't take it as confirmed).

Both chips' device-tree nodes already exist upstream, sharing the same
electrical description, gated by `status = "okay"`/`"disabled"` — see the
fixes doc for what it actually takes to get the right one active.

## Camera

Four `/dev/video*` nodes exist, but only **one** (`sun6i-csi-capture`) is
real capture. The others are `cedrus` (H.264 decode VPU, unrelated to
capture), `sun8i-rotate`, and `sun8i-di` (deinterlacer). Only one physical
camera is usable right now — mainline dual-camera support on the original
PinePhone is still incomplete.

## Confirming your own unit's revision

Don't assume from the model name alone. Options, cheapest first:
- Once booted: `cat /proc/device-tree/model` — reports the *currently
  loaded* device tree's claimed revision, which may not match reality if
  the wrong one is loaded (this happened to us — see the fixes doc).
- `i2cdetect -y <bus>` across `/dev/i2c-*` for address `0x1c`: responds
  regardless of which chip is actually populated, but combined with
  `lsmod`/driver probe success tells you which one really answers.
- Physical: the board revision is silkscreened on the PCB, if you're
  willing to open the case.
