## Testing the board for I2C and SPI detection, and UART verification ##

When porting a new board to ArduPilot, many changes are required in the Linux_HAL source code. As a result, making a mistake is easy and can be the source of much frustration. This section aims to tackle the de-bugging systematically to rule-out hardware defects or configuration errors.

---

##  Hardware Verification ##

The first problem solving step is to test the hardware functionality and wiring. This can be done by verifying whether the Raspberry Pi detects the Baro, IMU, PCA9685, RC receiver and GPS.

## I2C device tests ##

The two devices connected via I2C are the PCA9685 and the BMP280 barometer. They can be detected on the I2C bus by downloading the 'i2cdetect' package like so:

sudo apt update
sudo apt install i2c-tools
