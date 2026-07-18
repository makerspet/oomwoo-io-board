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
Roborock S5 Max wheel assembly - JST ZH 1.5mm male 7p (needs f)
['''''''] wheel-drop-switch on, wheel-drop-switch com, orange hall TBD, blue hall TBD, brown hall TBD, MOT, MOT

BL24131607 suction fan DC 14.4V - JST PH2.0 female 5p (needs m)
['''''] ID FG SP - +

20N704R990F suction fan -  JST PH2.0 female 4p (needs m)
[''''] Pinout TBD

20N704R990F suction fan DC 15V - JST PH2.0 female 4p (needs m)
[''''] Pinout TBD

MSD-D suction fan - JST PH2.0 female 4p (needs m)
[''''] Pinout TBD

20N709U020 suction fan - JST PH2.0 female 4p (needs m)
[''''] Pinout TBD

22N704V160 suction fan DC 14.4V - 5-pin 2mm pitch with latch female (not PH)

BL27302101 suction fan DC 14.4V - 6-pin 2mm pitch with latch female (not PH)

BL24131616 suction fan DC 14.4V - 5-pin 2mm pitch with latch female (not PH)

MSD-C-3 suction fan - 4-pin like PH, but looser vertically

MSD-G-V1 suction fan - LHE MX3.0 2x2 (4-pin) 3mm pitch with latch male (aka Molex Micro-Fit 3.0)
```

## Compute + Camera

- 2x 15-pin ArduCam-style connectors for OV5647
- TODO add USB to I/O board

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

## GPIO

1. Power source current sense (analog in)
2. VBat sense (analog in)
3. Main fan sense (analog in)
4. anti-fall left up sensor (analog in because IR sensors are analog)
5. anti-fall left down sensor (analog in)
6. anti-fall right up sensor (analog in)
7. anti-fall right down sensor (analog in)
8. wheel motor left driver in1 (digital output)
9. wheel motor left driver in2 (digital output)
10. wheel motor left driver encoder (digital input)
11. wheel motor right driver encoder (digital input)
12. Power button (digital input)
13. CPU (e.g. Raspberry Pi) power on/off (digital output)
14. STM32 SWDIO
15. STM32 SWCLK
16. Vacuum power on/off (digital output)
17. Wheel motor right current sense (analog in)
18. Wheel motor left current sense (analog in)
19. Main brush motor current sense (analog in)
20. IMU SPI SCLK (digital out)
21. IMU SPI MISO
22. IMU SPI MOSI
23. IMU SPI CS
24. Wheel motor right driver in1 (digital out)
25. Motors power enable (digital out)
26. Wheel motor right driver in2 (digital out)
27. Water pump sense (analog in)
28. Side brush left front motor sense (analog in)
29. Side brush right front motor sense (analog in)
30. CPU reset (e.g. Raspberry Pi)
31. Dock IR sensor 1 (analog in)
32. Dock IR sensor 2 (analog in)
33. Water pump motor PWM (digital out)
34. Main brush motor PWM (digital out)
35. Lidar motor PWM (digital out)
36. Bumper switch 1 (digital in)
37. UART1 TX
38. UART RX
39. Side brush motor right PWM (digital out)
40. Side brush motor left PWM (digital out)
41. Power LED on/off (digital out)
42. Home LED on/off (digital out)
43. Home button (digital in)
44. Battery charge sense (digital in)
45. Charge status (digital out)
46. Bumper switch 1 (digital in)
47. Bumper switch 2 (digital in)
48. Test/program
49. Test/program
50. Main fan motor PWM (digital out)
51. Main fan motor current sense (analog in)
52. IMU interrupt 2 (digital in)
53. IMU interrupt 1 (digital in)
54. IMU FSYNC (digital in)
55. Side proximity IR sensor left (analog in)
56. Side proximity IR sensor right (analog in)
57. Side proximity IR LED left PWM (digital out)
58. Side proximity IR LED right PWM (digital out)
59. Wheel drop sensor left (digital in)
60. Wheel drop sensor right (digital in)

TODO before layout/fabrication: confirm whether GPIO entries 36 and 46 are intentionally separate bumper inputs or a duplicate label.
