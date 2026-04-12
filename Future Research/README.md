# Research Presentation and ideas for future designs

This project was developed not only as a working ArduPilot controller for Planes, Copters, Rover and Subs, but as a **research platform demonstrating how ArduPilot can be ported to a minimal Linux-based hardware architecture**. A key objective of EDUCOPTER is to make the internal structure of the ArduPilot Linux Hardware Abstraction Layer (HAL) easier to understand and extend for educational use.

Two architecture diagrams are included within this repository to support this goal.

## Linux porting software architecture

<img src="images/HAL_software_architecture.png" width="800">

This diagram presents the **Linux porting software architecture**, showing how the vehicle firmware interacts with the Linux HAL and how SPI, I2C and UART sensor drivers are integrated into the ArduPilot build system using `waf`.

The purpose of this diagram is to help future developers understand:

- how ArduPilot interfaces with Linux hardware drivers
- how custom Linux board subtypes are introduced
- how sensor drivers connect to the vehicle firmware
- how additional peripherals can be supported within the existing framework


## EDUCOPTER communication architecture

<p align="center">
  <img src="hardware/images/EDUCOPTER_Communication_Architecture.png" width="600"/>
</p>

This diagram presents the **minimum working hardware configuration required to achieve stable quadcopter flight using ArduPilot on Linux**.

It highlights the communication links between:

- the Raspberry Pi flight computer
- the IMU (SPI)
- the barometer (I2C)
- the PWM driver (I2C)
- the GPS module (UART)
- the RC receiver (UART)

Together, these diagrams demonstrate both the **software integration workflow** and the **hardware communication structure** required to reproduce the EDUCOPTER platform.


## EDUCOPTER as a research platform

Unlike commercial flight controllers, EDUCOPTER is intentionally designed as a **foundation for experimentation rather than a finished compact autopilot product**.

The simplified architecture allows future users to:

- increase hardware compactness through PCB redesign
- integrate additional sensing capabilities
- add payload support such as cameras or telemetry radios
- modify or extend the Linux HAL board definition
- migrate the architecture to alternative embedded processors

Because of this, EDUCOPTER can act as a starting point for students and researchers interested in exploring **custom ArduPilot flight controller development**.


# Future development opportunities

Several extensions could be explored to further develop the EDUCOPTER platform.

## LEDs and buzzer integration

One immediate improvement would be the integration of **status LEDs and a buzzer**, similar to those implemented in the OBAL board.

These components provide useful feedback during:

- boot status
- GPS lock acquisition
- calibration procedures
- arming conditions
- failsafe events

Adding these indicators would improve usability during setup and flight testing.


## Integrated power distribution

Another possible improvement would be integrating a **power distribution circuit directly into the EDUCOPTER PCB** in order to produce a more compact flight controller.

This approach was considered during development but was intentionally avoided in order to:

- reduce board complexity
- simplify manufacturing
- lower overall project cost
- avoid routing high-current battery power on a single-sided PCB

Routing higher voltages across small PCB tracks introduces additional safety constraints, particularly when operating from a 4S LiPo supply. For this reason, integrated power distribution would likely be more appropriate for **smaller drone platforms operating at lower voltages**.

Future multi-layer PCB versions of EDUCOPTER could explore this direction further.


## Additional payload integration

The communication architecture used in this project allows straightforward integration of additional peripherals such as:

- cameras
- telemetry radios
- optical flow sensors
- rangefinders
- companion processing modules

For example, integrating a camera would allow:

- live video transmission
- visual navigation experiments
- onboard autonomy research

Because ArduPilot already supports MAVLink-based payload interaction, these additions can be implemented without significant architectural changes.


## Real-time ESP32-based autopilot variants

A further research direction would be implementing a similar architecture using a **real-time microcontroller platform such as the ESP32 running a real-time operating system instead of Linux**.

Linux provides flexibility and accessibility for development, but it does not provide deterministic timing behaviour for control loops. Microcontroller-based systems running RTOS environments can achieve higher loop execution stability and are commonly used in dedicated flight controllers.

A comparable architectural structure to the Linux porting workflow shown in this repository can be found within:

