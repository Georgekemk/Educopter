## Testing the board for I2C and SPI detection, and UART verification ##

When porting a new board to ArduPilot, several changes are required in the Linux HAL source code. As a result, configuration mistakes are common and can be difficult to diagnose. This section provides a systematic hardware verification procedure to rule out wiring faults or configuration errors before debugging the software.

---

##  Hardware Verification ##

The first step in troubleshooting is to verify the hardware functionality and wiring. This can be done by checking whether the Raspberry Pi detects the barometer, IMU, PCA9685 PWM driver, RC receiver and GPS module.

## I2C device tests ##

The two devices connected via I2C are the PCA9685 and the BMP280 barometer. They can be detected on the I2C bus using the i2c-tools package:

`sudo apt update`

`sudo apt install i2c-tools`

Then check that i2c exists by running the following command:

`ls /dev/i2c*`

This should return /dev/i2c-1. If the terminal replies with nothing, this is most likely due to I2C not being enabled. Follow step X in the software README file to enable I2C.

The I2C devices can then be detected using:

`sudo i2cdetect -y 1`

This should show two devices on the I2C bus as in the image below. The BMP280 should be on the 0x76 address and the PCA9685 on the 0x40.
