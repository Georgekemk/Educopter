# Hardware Setup – EDUCOPTER

This document explains how to construct and assemble the **EDUCOPTER flight controller hardware**. It covers sensor connections, PCB manufacturing, component soldering, wiring of external peripherals, and power supply requirements.

The EDUCOPTER board is intentionally designed to be **simple to manufacture and assemble**, using a **single-sided PCB** and only the sensors required for ArduPilot operation.

---

# 1. Sensor and Peripheral Connections

The EDUCOPTER board uses standard interfaces supported by **ArduPilot on Linux-based flight controllers**. Wherever possible, commonly used pin assignments have been used so that ArduPilot can detect and initialise sensors without extensive configuration.

## Sensor Interface Overview

| Device | Interface | Purpose |
|------|------|------|
| IMU (MPU9250) | SPI | Attitude estimation (gyro + accelerometer) |
| Barometer | I2C | Altitude estimation |
| PCA9685 PWM driver | I2C | Motor PWM outputs |
| GPS module | UART | Position and velocity |
| RC Receiver (SBUS) | UART | Remote control input |

### SPI – IMU

The **IMU** is connected using the SPI interface. SPI provides fast and reliable communication required for inertial measurements.

The IMU pins include:

- MOSI  
- MISO  
- SCLK  
- CS (chip select)

These are connected to the Raspberry Pi SPI bus using standard Linux ArduPilot mappings.

---

### I2C – Barometer and PCA9685

Both the **barometer** and the **PCA9685 PWM driver** are connected using the **I2C bus**.

Advantages of using I2C:

- Multiple devices can share the same bus  
- Only two signal wires are required  
- Widely supported by ArduPilot drivers  

Devices connected on I2C:
