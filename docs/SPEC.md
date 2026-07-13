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
