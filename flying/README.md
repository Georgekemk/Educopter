# 2. Assembling the EDUCOPTER Quadcopter

## Installing the Holybro Power Distribution Board

Place the **Holybro PM07 power distribution board** inside the drone frame base plate.

Plug the four ESC power leads into the ports of the Holybro board.

Ensure:

- polarity is correct
- connections are secure
- cables are not under tension
- space is left above the board for mounting the EDUCOPTER controller

Insert image here

---

## Mounting the EDUCOPTER Flight Controller

Fix the EDUCOPTER board to the top of the drone frame using a **3D printed mounting structure** designed for your frame geometry.

An example mounting structure is shown below.

Insert image here

Ensure:

- the board is rigidly mounted
- the IMU is firmly secured
- there is no wobble in the controller mounting
- the Raspberry Pi USB-C port remains accessible

Loose IMU mounting will cause incorrect attitude estimation during flight.

---

## Connecting the GPS and RC Receiver

Connect the GPS module and RC receiver to the EDUCOPTER board.

Connections:

- GPS → GPS UART port
- SBUS receiver → RC input UART port

Ensure:

- correct voltage rails are used
- wiring polarity is correct
- the SBUS inversion circuit is operating correctly if used

Insert image here

---

## Connecting ESC Signal Outputs to the PCA9685

Attach the ESC signal wires to the signal and ground pins of the PCA9685.

ESC numbering follows the PCA9685 channel order from left to right:

- ESC 1 → Channel 0
- ESC 2 → Channel 1
- ESC 3 → Channel 2
- ESC 4 → Channel 3

The **QUAD-X motor configuration must be followed**.

Insert diagram here

Incorrect ordering will prevent stable flight.

---

## Mounting the Motors

Attach the motors to the drone frame arms using the supplied nuts and bolts.

Once mounted, connect each motor to its ESC.

Do not attach propellers yet.

Motor rotation direction will be verified later during calibration.

---

## Installing the Battery and Power Connections

Attach the battery to the underside of the drone frame.

Connect power as follows:

Battery  
→ power distribution splitter  
→ Holybro PM07  
→ 5V 3A regulator  
→ Raspberry Pi USB-C power input

Ensure:

- the Raspberry Pi receives a stable regulated 5V supply
- polarity is correct before connecting the battery
- the Raspberry Pi is not powered directly from ESC rails

---

## Performing ArduPilot Calibration

Once assembly is complete, calibration can begin using Mission Planner.

Complete the following calibrations:

- accelerometer calibration
- compass calibration
- RC calibration

ESC calibration depends on ESC type. The standard ArduPilot ESC calibration process works for most ESCs but should be checked against manufacturer documentation before use.

---

## Testing RC Outputs Before Flight

Ensure propellers are **not attached** before testing.

Arm the throttle using either:

the MAVProxy command

arm throttle

or by holding the RC throttle hard left for several seconds.

Test outputs by adjusting:

- throttle
- yaw
- pitch
- roll

Observe PWM outputs from the PCA9685 channels.

---

## Verifying Motor Rotation Direction

Use the Mission Planner motor test function to confirm correct motor rotation.

Expected QUAD-X rotation directions:

- front right → clockwise
- front left → counter-clockwise
- rear left → clockwise
- rear right → counter-clockwise

If rotation is incorrect, swap any two ESC-motor wires.

---

## First Flight

Once calibration and testing are complete, attach the propellers.

Move to an open outdoor space before attempting flight.

Before take-off confirm:

- battery secure
- GPS lock achieved
- transmitter connected
- failsafe configured
- correct flight mode selected

Set the initial flight mode to:

STABILIZE

Ensure compliance with local drone regulations before flying.

Happy flying!
