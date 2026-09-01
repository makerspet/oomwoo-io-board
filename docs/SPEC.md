# OOMWOO I/O Board spec (work in progress)

See [vacuum BOM](https://github.com/makerspet/oomwoo/blob/main/BOM.md) for details.

## Motors

Most motors draw power directly from the 4S battery (not via a DC-DC converter). The battery is 14.4V nominal, 12V discharged and 16.8V fully charged.

| Type | Qty | Spec |
| --- | --- | --- |
| Drive wheel | 2 | DC 14.4V 19 Ohm, 3.5A stall (TODO check), H-bridge DRV8231, DRV8871 or similar |
| Suction fan | 1 | BLDC 14.4V 10A (TODO check) high-side load switch P-FET, PWM input to fan, FG feedback to STM32 |
| LiDAR | 1 | 5V 0.35A max, Mabuchi-style RF-500TB-14350 or similar, low-side load switch N-FET |
| Main brush | 1 | DC 14.4V 22A?? (TODO check) PRI-390SV-24100, JLS-395PH-2248A, RS-390WM-3107GCF or similar (bridge or FET TBD) |
| Side brush | 1 | DC 14.4V 1.3A stall (TODO check) RC500-KW/14440/DV, PR-500EV-14440 or similar (bridge or FET TBD) |
| Mop | 2 | GM-RS385Y-24065 or similar, DC 14.4V |
| Mop lift | 1 | Likely MG90S servo |
| Mop arm | 1 | Likely MG90S servo |
| Water pump | 1 | TBD |
| Side brush arm | 1 | Likely MG90S servo |

Motor pinouts

```
Roborock S5 Max wheel assembly - JST ZH 1.5mm male 7p (mates board f)
// Also see https://github.com/makerspet/oomwoo/tree/main/contributions/part-specs/Scowt
7 wheel-drop-switch on
6 wheel-drop-switch com
5 orange hall 5V VDD?
4 blue hall signal OUT?
3 brown hall GND?
2 MOT -?
1 MOT +?

BL24131607 suction fan DC 14.4V - JST PH2.0 female 5p (mates m-m fan-to-board cable)
1 ID
2 FG
3 SP
4 -
5 +

DC 14.4-15V 4S fans:
- 20N704R990F suction fan
- 20N704R990F suction fan
- MSD-D suction fan
- 20N709U020 suction fan
  JST PH2.0 female 4p (mates m-m fan-to-board cable)
  pin 4 VMOT
  pin 3 GND
  pin 2 PWM, low == off; drive at 5V?
  pin 1 TACH open collector

22N704V160 suction fan DC 14.4V - 5-pin 2mm pitch with latch female (PA?)

BL27302101 suction fan DC 14.4V - 6-pin 2mm pitch with latch female (PA?)

BL24131616 suction fan DC 14.4V - 5-pin 2mm pitch with latch female (PA?)

MSD-C-3 suction fan - 4-pin like PH, but looser vertically

MSD-G-V1 suction fan - LHE MX3.0 2x2 (4-pin) 3mm pitch with latch male (aka Molex Micro-Fit 3.0)
```

## Compute + Camera

- 2x 15-pin ArduCam-style connectors for OV5647
- TODO add USB to I/O board

Undecided TODO 
- maybe provision an M.2 slot, route a PICe lane, populate later - to experiment with NPU accelerator(s) like Hailo
- USB-C 3.0+, CM5 only - to experiment with accelerator(s) like Coral TPU
- Keep the compute socket able to take an integrated-NPU module too (Radxa CM5) or premium-upgradeable (CM5 + M.2 Hailo).
- Flag it to the PCB contractor as a design item: M.2 E-key (WiFi) + an M.2 M-key/PCIe (NPU or NVMe), PCIe lane routing, and the thermal path for a few-watt accelerator in a suction-cooled enclosure

## Charging

View [BRR-2P4S-5200FL battery datasheet](https://images.thdstatic.com/catalog/pdfImages/55/55d2f7f6-2ed9-44ed-ab4e-fb20d231c897.pdf) as a sample.

```
Battery BRR-2P4S-5200 14.4V nominal - 4-pin 3mm pitch with latch male LHE MX3.0 (C3001-H04), Molex Micro-Fit 3.0
[o66o] 4321 BAT+ 10.7K/NTC 0.62M/ID GND
```

### Robot

- the robot has 2 power inputs: USB-C and dock
  - robot receives 20-24V fixed DC from the dock
  - USB-C power use PD, request 20-24 V minimum (to step it down to 4S battery)
  - optional PPS
  - if a low-power USB-C 5V, 9V or 15V brick source is attached (no 20V/PPS), either charge slowly (optional) through a boost path or cleanly refuse and signal "insufficient charger" rather than misbehaving
- robot requires 65W minimum input (from the dock) with system power-path charger (a charger IC with a SYS rail)
  - support the vacuum charging and Raspberry Pi running simultaneously
  - assume Raspberry Pi is always on (to handle user access over Wi-Fi at any time)
  - Pi 5 worst case ~25 W (5 V/5 A) + housekeeping ≈ up to ~25–30 W
  - Healthy charge ~40 W (~0.5C into the 75 Wh pack)
  - ~65–70 W total
- cap charge at ~0.5C regardless of charging adapter power

### Dock

- the dock is powered from an external certified 24/25.2 V DC brick (~200–350 W)
  - use external brick for safety, the dock enclosure only ever sees 24 V DC
  - reuse a 25.2 V stick-vac motor for auto-empty, e.g. Dreame M10-E-4 25.2 V/310 W
- the dock has only 2 contacts: DOCK+ and GND
  - dock contacts feed a fixed 24V+ DC
  - the dock detects load/robot presence, energizes DOCK+ only when robot is detected (reliably, after a couple of seconds)
  - dock contacts are spring loaded, gold-coated pogo pins ≥4 A, placed rear-vertical, above water line
- ambient fan(s) for mop drying
  - no hot mop dry for now
- 2x water pumps: clean-feed + dirty-evacuate
  - diaphragm, 12–24 V
- dock PCB
  - ESP32 (WiFi + BLE + control)
  - Pump/fan drivers (brushed DC)
  - IR beacon LEDs + driver
  - robot/load presence-detect + charging contact energize FET
  - Level sensors (float/capacitive) clean-low, dirty-full
  - high-side FET for auto-empty blower
  - fuse, DC inlet, TVS
  - buck DC-DC 24V to 5V, 3.3V for ESP32, sensors

### Power path

Standard capability of power-path charger ICs - TI bq25 family and similar.

```
USB-C 20V ─► [PD sink] ─► [power-path charger] ─┬─► SYS rail ─► 14.4→5V buck ─► Pi (always-on)
                                                └─► charges 4S pack
Battery ────────────────────────────────────────┘ (supplements SYS if input insufficient)
```

- Docked: the Pi runs from the input via SYS; the battery charges from the surplus; once full, charge current → 0 and the Pi keeps running off input — no needless battery cycling while docked (also a longevity win).
- Undocked: SYS seamlessly falls back to the battery — the Pi never browns out during the handoff. This is exactly what makes "pause → return to charge → resume" and "app connects anytime" work cleanly.
- Input-limited: if only a weak brick is attached, the battery supplements SYS so the Pi stays up, and charge current backs off. Graceful degradation for free.

Details

1. 65 W = 20 V / 3.25 A → e-marked cable required. Above 3 A / 60 W, USB-C needs a 5 A e-marked cable. Unavoidable at 65 W (20 V is PD's max fixed voltage) — just document it. 65 W bricks ship with an e-marked cable anyway.
2. Dynamic power management (VIN/IIN-DPM): the charger must throttle charge current to keep total draw within the negotiated PD budget, prioritizing the Pi/SYS load. Standard on power-path chargers.
3. Cap charge current at ~0.5C (~2.6 A into the pack) for cell life, regardless of surplus — don't let a big brick fast-charge the cells.
4. Two DC inputs, one charger: the robot's own USB-C port and the dock contacts both present ~20 V DC → OR them into the charger's VBUS with priority/ideal-diode selection (both are the same voltage, so it's clean).
5. Dock side: dock has its own PD sink + 65 W brick, passes ~20 V to the contacts. At 20 V, 65 W ≈ 3.25 A over the contacts → size the pogo/spring contacts for ~4 A with margin.

Net spec

1. USB-C PD, 65 W minimum (20 V / 3.25 A), on both the robot port and the dock (each with its own PD sink); e-marked cable expected.
2. Power-path 4S charger with a SYS rail feeding the Pi's 5 V buck, so the Pi is always-on from input when docked and from battery when not, with seamless handoff and battery-supplement under load.
3. DPM + 0.5C charge-current cap; OR the two DC inputs into one VBUS; dock contacts rated ~4 A.

## LiDAR pinouts

```
X-WPFTB-V2.6.2 PCB marking - JST GH 1.25mm 4-pin female (needs m)

D-WPFTBCD-V1.0.1 PCB marking - JST GH 1.25mm 4-pin female (needs m)

LDROBOT LD14P lookalike - JST GH 1.25mm 4-pin female (needs m)

Mystery mini - JST GH 1.25mm 5-pin female (needs m)
```

## Front sensors module board

- 2x VL53L7CH (or VL53L7CX) 60° horizontal FoV each
  - each turned 30° left, right to cover 120° horizontal FoV
- 2x OV5647, 5M wide-angle for stereo depth + object recognition
  - off-the-shelf breakout boards for now
  - possibly $2 imaging ICs later
- NIR illumination LEDs with a projection pattern
- breaks into multiple PCBs using holes
  - central - 2x TSOP38238 (separated by a baffle) for dock homing
  - left - VL53L7CH pointed 30° left
  - right - VL53L7CH pointed 30° right
  - stereo depth camera 2x OV5647 with NIR illumination LEDs

## Side sensors module board

- TSOP38238 for dock detection
- consumer vacuums use analog Sharp short-range distance sensor
  - use VL6180V1NR/1 C2655167 $1.03 100pcs 18° FoV diagonal? Obsolete, similar to VL53L4CD
  - use VL53L0CX GY-530, TMF8806 or similar instead? Dust may be an issue.
  - VL53L0CXV0DH/1 C91199 $2.11 100pcs
  - VL53L4CDV0DH/1 C3178291 $2.19 100pcs

## Sensors

- VL53L7CH
  - Arduino library https://github.com/stm32duino/VL53L7CH
  - hookup schematic https://eu.mouser.com/en/new/stmicroelectronics/stm-vl53l7ch-tof-sensor
  - LPn pin sets I2C address

## Pump

- 6V DC motor, peristaltic; ~0.6A rated, 1A max
- make DC settable by replacing resistors

## GPIO

Please see the [PCB schematic](https://github.com/makerspet/oomwoo-io-board/tree/main/kicad/PDF) for up-to-date GPIO list.

TODO before layout/fabrication: confirm whether GPIO entries 36 and 46 are intentionally separate bumper inputs or a duplicate label.
