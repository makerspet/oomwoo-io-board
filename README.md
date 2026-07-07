# OOMWOO I/O Board (STM32G070 based)


**Current plan - the I/O board will (likely) accept CM4/CM5 Raspberry Pi compute module (and other compatible 3rd party compute modules).**


Schematic (PDF) for the [OOMWOO](https://github.com/makerspet/oomwoo) open-source robot
vacuum's controller + I/O board

Status: early reference schematic. NOT validated. Do NOT fabricate from it as-is.

Part of the [oomwoo](https://github.com/makerspet/oomwoo) project — an open-source, ROS 2,
3D-printable robot vacuum you build yourself.

## Architecture notes

Consumer vacuums typically split compute into CPU and MCU. The MCU ([real examples](https://robotinfo.dev/)) controls
- all motors - driving wheels, suction fan, side brushes, mop motors, mop lift servos, pump and so on
- all sensors except below - IR cliff, IR side proximity, IR docking, wheel drop, bumper (IR or micro-switches), motor encoders, ultrasonic carpet sensor
- all push buttons
- 2D LiDAR motor RPM
- power - battery charging, motors power on/of
- all safety sensors - overcurrent for all motors (cheap vacuums ignore overcurrent sensing)

I/O board also provides audio encoders/decoders, amp, speaker (connector), mics (if any).

CPU receives
- 2D LiDAR data over serial port
- IMU (sometimes IMU gets forwarded via MCU) over SPI/I2C
- cameras (2x MIPI for visual obstacle avoidance)

MCU (and CPU)) are usually always active. This allows the user to start cleaning at any time (especially via app remotely). Exceptions:
- no power (battery dead and no dock power)
- user explicitly shut down the vacuum by long-pressing the power button

MCU overarching constrait is to ensure robot safety. This is necesary to pass CE mark tests. Here is how MCU ensures safety:
- MCU firmware uses FreeRTOS or similar, static memory allocation, watchdog, guaranteed reaction time
- MCU acts as watchdog for CPU, resets CPU if CPU becomes unresponsive
- MCU connects to CPU over serial using a custom serial protocol ([concrete example](https://github.com/codetiger/VacuumRobot)). No micro-ROS to reduce dependencies, prevent bugs from updated 3rd party libraries slipping into safety-critical MCU firmware.
- MCU shuts down some (or all) motors when MCU detects overcurrent (brush stuck)
- MCU shuts down all motors (including power to all motors) when CPU becomes unresponsive (to prevent CPU sending garbage commands to MCU)
- MCU monitors battery charging, shuts it down when needed (battery too hot)
- MCU stops driving wheel motors when bumper sensors activate
- MCU tests motors, sensors
- MCU does not depend on CPU
- MCU toggles watchdog GPIO; otherwise an external-to-MCU circuit shuts down motor power

Consumer vacuums place CPU and MCU on the same PCB board. Since OOMWOO has to be hackable, it seems best to separate I/O from the compute. Therefore, this I/O board needs to have
- MCU and programming header
  - likely STM32G070RBT6 because it has 56 GPIOs, 16 ADC channels, costs $1 at JLCPCB, has LQFP package for easy PCBA - a rare combination
  - see tentative [GPIO budget](https://github.com/makerspet/oomwoo-io-board/blob/main/docs/GPIO.md)
- all vacuum motor drivers (with connectors), driven by the MCU
  - cliff, side proximity, docking, bumper, carpet, water tank full/empty, dust bin present, motor overcurrent, motor encoders
- MCU connected to all sensors (except those handled by CPU like LiDAR serial, MIPI cameras I/O, IMU); sensor connectors
- control circuit for 2D LiDAR's motor (controlled by MCU)
- battery charging, power management (motors power on/off, charging on/off, power shut down), power for CPU compute module (see below)
- push buttons (home, power, etc.) and UI LEDs
- dock connector (dock power, sense dock present)
- USB power for charging; USB for serial shell, etc.
- audio encoder/decoder, audio amp, mics (if any)
- SD card for hacking, cheap large storage
- connect IMU FSYNC to MCU to synchronize MCU timestamps with IMU data streaming to CPU

Consumer vacuums usually don't have a CPU fan/heatsink because the vacuum's suction fan can (indirectly) cool the CPU/MCU PCB. Therefore, no CPU fan on this I/O board.

As far as compute - currently it seems best to use a Raspberry Pi CM4/CM5 compute module. Why?
- CM4/CM5 modules cost a bit less than a full Raspberry Pi 4/5
- CN4/CM5 have lower profile/height vs a full Raspberry Pi 4/5, important to keep vacuum cleaner slim
- there are plenty of 3rd party compute modules pin-compatible with CM4/CM5, making the compute swappable, hackable
  - 3rd party compute modules with NPUs are especially interesting because we need NPU for real-time camera-based obstacle detection

Compute module headers should expose
- 2x MIPI camera I/O
- 1x serial for CPU-MCU comms
- 1x serial for CPU-LiDAR comms
- 1x SPI (or I2C) for CPU-IMU comms
- audio (digital?) I/O
- power from I/O board to CM
- a few TBD GPIOs
  - MCU GPIO to reset CPU

This board should be designed using an open-source PCB tool (KiCad seems like a good choice).

## License

[Apache License 2.0](LICENSE).
