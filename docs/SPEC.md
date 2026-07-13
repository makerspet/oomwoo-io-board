# OOMWOO I/O Board spec (work in progress)

See [vacuum BOM](https://github.com/makerspet/oomwoo/blob/main/BOM.md) for details.

## Motors

Most motors draw power directly from the 4S battery (not via a DC-DC converter). The battery is 14.4V nominal, 12V discharged and 16.8V fully charged.

| Type | Qty | Spec |
| --- | --- | --- |
| Drive wheel | 2 | DC 14.4V 3.5A stall (TODO check), H-bridge DRV8231, DRV8871 or similar |
| Suction fan | 1 | BLDC 14.4V 10A (TODO check) high-side load switch P-FET, PWM input to fan, FG feedback to STM32 |
| LiDAR | 1 | 5V 0.35A max, Mabuchi-style RF-500TB-14350 or similar, low-side load switch N-FET |
| Main brush | 1 | DC 14.4V 22A?? (TODO check) PRI-390SV-24100, JLS-395PH-2248A, RS-390WM-3107GCF or similar (bridge or FET TBD) |
| Side brush | 2 | DC 14.4V 1.3A stall (TODO check) RC500-KW/14440/DV, PR-500EV-14440 or similar (bridge or FET TBD) |
| Mop | 2 | TBD |
| Mop lift | 2 | TBD |
| Water pump | 1 | TBD |
| Side brush arm | 2 | TBD |

## Charging

- USB-C PD, request 20 V minimum (to step it down to 4S battery)
- optional PPS
- 65W minimum with system power-path charger (a charger IC with a SYS rail)
  - support the vacuum charing and Raspberry Pi running simultaneously
  - assume Raspberry Pi is always on (to handle user access over Wi-Fi at any time)
  - Pi 5 worst case ~25 W (5 V/5 A) + housekeeping ≈ up to ~25–30 W
  - Healthy charge ~40 W (~0.5C into the 75 Wh pack)
  - ~65–70 W total
- cap charge at ~0.5C regardless of charging adapter power
- the two dock contacts are the
- if only a 5V, 9V or 15V source is attached (no 20V/PPS), either charge slowly (optional) through a boost path or cleanly refuse and signal "insufficient charger" rather than misbehaving
- assume dock contacts feeding a fixed 20V+ DC; the dock will likely have its own PD sink, converting the dock's brick to 20V DC fixed.

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
