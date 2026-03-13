# EDUCOPTER – Running Educopter binary on a Raspberry Pi

This repository documents the software setup required to run **ArduPilot Educopter on a Raspberry Pi** using the custom **EDUCOPTER hardware configuration**.

It includes:

- Raspberry Pi OS installation
- ArduPilot source modifications for the EDUCOPTER board
- Building ArduPilot on a Linux VM
- Copying the build to the Raspberry Pi
- Downloading MAVProxy to the Raspberry Pi
- MAVLink communication with MAVProxy and Mission Planner
- System service configuration

The aim of this readme is to:
1. Allow the reader to directly replictae this project
2. Detail the steps that would enable the creation of a new Linux board

---

# 1. Preparing the Raspberry Pi

## Install Raspberry Pi OS

Download **Raspberry Pi Imager** from the Raspberry Pi website onto your laptop.

Install the following image:

**Raspberry Pi OS Lite (32-bit)**

The Lite version is recommended because:

- it is lightweight
- it reduces CPU usage
- it avoids unnecessary desktop processes that can slow the ArduPilot main loop

Insert an SD card into a reader and flash the image using the imager.

---

## Naming the Pi

For simplicity and to avoid directory errors in the Pi's terminal, set the following details for the Raspberry Pi

- **Name** 'pi'
- **Password** 'password'

## Enable WiFi using the laptop hotspot

Ensuring the Pi connects to your laptop hotspot makes ssh to Pi possible with the laptop connected to any network

To allow the Pi to connect to the laptop hotspot during first boot:

1. In Raspberry Pi Imager open **Advanced Options**
2. Enable **WiFi configuration**
3. Enter the hotspot credentials

Example:

- **SSID Name:** `LaptopSSID_name`
- **Password:** `**********`
- **Country:** `GB`

---

# 2. Preparing the Build Environment (Linux VM)

ArduPilot is built on a **Linux virtual machine** rather than directly on the Raspberry Pi. This allows faster compilation and avoids installing unnecessary development tools on the Pi, whilst enabling an easy ssh connection. Alternatively ssh can be performed using VSCode.

I used Oracle VirtualBox box to run a version of Ubuntu but you can use any VM that you prefer. The guide for to install Ubuntu on Oracle VirtualBox can be found here:
https://ubuntu.com/tutorials/how-to-run-ubuntu-desktop-on-a-virtual-machine-using-virtualbox

---

## 3. First boot

Insert the SD card into the Raspberry Pi and power it on using the 5V usbc port.

Connect via SSH from your Linux virtual machine's terminal. Find the Pi's IP address by looking in your hotspot's connected devices and enter the following:

`ssh pi@<pi_ip_address>`

e.g.

`ssh pi@192.168.137.119`

Update packages:

`sudo apt update`

`sudo apt upgrade`

---

# 4. Clone the ArduPilot source code

**IMPORTANT** Clone the ardupilot repository on your Linux virtual machine. When you eventually build the board subtype using waf, this will save time and also keeps the workflow organised. Of course you can clone it straight to the Pi, but building the binary will take approx 45mins to an hour each time.

In a terminal on the VM, clone the repository:

`git clone https://github.com/ArduPilot/ardupilot.git`

Enter the directory:

`cd ~/ardupilot`

Initialise the submodules:

`git submodule update --init --recursive`

---

# 5. Adding the EDUCOPTER Hardware Definition

Create the directory:

`ardupilot/libraries/AP_HAL_Linux/hwdef/EDUCOPTER`

You can do this using the command:

`sudo nano ardupilot/libraries/AP_HAL_Linux/hwdef/EDUCOPTER`

Copy the `hwdef.dat` file from this repository into that directory.

This file defines:

- Sensors
- Communication methods (I2C, SPI)
- GPIO mappings
- Board subtype
- Log file directories

used by the EDUCOPTER board.

---

# 6. Required Source Code Modifications

A small number of changes must be made to ArduPilot's Linux HAL so the EDUCOPTER board is recognised correctly.

These modifications allow the system to:

- recognise the new board subtype
- enable Raspberry Pi specific utilities
- configure flight sensors
- configure RC input handling (SBUS)

Only the relevant lines should be changed so that the setup remains easier to maintain when ArduPilot updates.

---

## Modify `AP_HAL_Boards.h`

File:

`libraries/AP_HAL/AP_HAL_Boards.h`

Add a new board subtype definition for EDUCOPTER.

### Why this is needed

This allows ArduPilot to distinguish the board from other Linux targets such as Navio or BeagleBone.

<img src="images/HAL_Boards.h.png" width="400">


---

## Modify `HAL_Linux_Class.cpp`

File:

`libraries/AP_HAL_Linux/HAL_Linux_Class.cpp`

This file:
- Defines board subtype
-  Maps RC Input Protocol
-  Maps RC Output (signal to the ESCs)

The modification steps are as follows:

1. Add EDUCOPTER board subtype definition
   
<img src="images/HAL_Linux_Class.cpp board_definition.png" width="400">

2. Add EDUCOPTER RCInputProtocols definition

<img src="images/HAL_Linux_Class.cpp RCInput.png" width="400">

But what exactly does the line do?

static RCInput_RCProtocol rcinDriver{"/dev/ttyAMA3", nullptr};

This line instantiates the RCInput_RCProtocol driver, which is responsible for reading RC receiver data from a Linux serial device and passing it to ArduPilot’s AP_RCProtocol decoder. This decoder can interpret multiple RC protocols such as iBUS, SBUS and CRSF.

For the EDUCOPTER board, the receiver is connected to the Raspberry Pi UART exposed as /dev/ttyAMA3. The second constructor argument is set to nullptr, indicating that no secondary 115200-baud RC input device is used.

On the Raspberry Pi, /dev/ttyAMA3 corresponds to the UART mapped to GPIO4 and GPIO5, which is used to receive the serial RC data from the receiver. The configuration of this UART is discussed in greater detail in Step X of this README.

3. Add EDUCOPTER RCOutput definition

<img src="images/HAL_Linux_Class.cpp RCOutput.png" width="400">

static RCOutput_PCA9685 rcoutDriver(i2c_mgr_instance.get_device_ptr(1, PCA9685_PRIMARY_ADDRESS), 0, 0, RPI_GPIO_<17>());

This line was kept from OBAL. EDUCOPTER uses the PCA9685 to generate PWM signals to the ESCs, in turn driving the motors. 'RPI_GPIO_<17>()' corresponds to the pin that connects the PCA enable switch. EDUCOPTER has this line grounded to reduce complexity, but the line has been left in for educational purposes

---

## Modify `Util_RPI.cpp`

File:

`libraries/AP_HAL_Linux/Util_RPI.cpp`

Add the EDUCOPTER subtype to the Raspberry Pi utility functions.

### Why this is needed

This allows the board to use Raspberry Pi specific functionality such as:

- GPIO handling
- CPU detection
- system utilities

### Screenshot placeholder
![Util_RPI Change](screenshots/util-rpi.png)

---

## Modify `RCInput_RCProtocol.cpp`

File:

`libraries/AP_HAL_Linux/RCInput_RCProtocol.cpp`

Add support for the RC input configuration used by the EDUCOPTER board.

### Why this is needed

The board uses a serial RC protocol for receiver input, so the Linux HAL must initialise the correct RC input driver.

### Screenshot placeholder
![RCInput_RCProtocol CPP Change](screenshots/rcinput-cpp.png)

---

## Modify `RCInput_RCProtocol.h`

File:

`libraries/AP_HAL_Linux/RCInput_RCProtocol.h`

Add the necessary declarations for the EDUCOPTER board’s RC input handling.

### Why this is needed

This ensures the RC input class supports the required board-specific configuration.

### Screenshot placeholder
![RCInput_RCProtocol H Change](screenshots/rcinput-h.png)

---

# 7. Creating the Python Build Environment

ArduPilot uses **Python tools during compilation**, so the build should be done inside a virtual environment.

Create a virtual environment named `ardupilot-venv`:

`python3 -m venv ardupilot-venv`

Activate it:

`source ardupilot-venv/bin/activate`

Install the required Python packages:

`pip install future`

`pip install empy`

`pip install pyserial`

`pip install pyyaml`

`pip install numpy`

If additional ArduPilot Python dependencies are required on your machine, install them into the same environment.

### Screenshot placeholder
![Python Venv](screenshots/ardupilot-venv.png)

---

# 8. Building ArduPilot

From inside the ArduPilot directory, with the virtual environment activated, configure the build for the EDUCOPTER board:

`./waf configure --board=EDUCOPTER`

Then build ArduCopter:

`./waf copter`

If successful, the binary will appear at:

`build/EDUCOPTER/bin/arducopter`

### Why a virtual environment is used

Using a dedicated Python environment avoids conflicts with system Python packages and keeps the build reproducible.

### Screenshot placeholder
![Waf Build](screenshots/waf-build.png)

---

# 9. Copying the Binary to the Raspberry Pi

Use `scp` to transfer the binary to the Raspberry Pi.

Example:

`scp build/EDUCOPTER/bin/arducopter pi@raspberrypi:/home/pi/`

Rename it if desired:

`mv arducopter /home/pi/arducopter`

### Screenshot placeholder
![SCP Transfer](screenshots/scp-transfer.png)

---

# 10. Installing MAVProxy on the Raspberry Pi

MAVProxy is used for MAVLink communication and debugging.

Create a Python virtual environment on the Pi named `mavproxy-venv`:

`python3 -m venv mavproxy-venv`

Activate it:

`source mavproxy-venv/bin/activate`

Install MAVProxy:

`pip install MAVProxy`

If required, install any supporting Python packages into the same environment.

### Screenshot placeholder
![MAVProxy Venv](screenshots/mavproxy-venv.png)

---

# 11. MAVLink Communication

ArduPilot sends MAVLink telemetry via UDP.

Example configuration in `ardupilot.service`:

`--serial1 udp:0.0.0.0:14550`

This allows MAVProxy or Mission Planner to connect.

Example MAVProxy command:

`mavproxy.py --master=udp:127.0.0.1:14550`

### Screenshot placeholder
![MAVProxy Connection](screenshots/mavproxy-connection.png)

---

# 11. Systemd Service

The repository includes an example `ardupilot.service` file.

Copy it to:

`/etc/systemd/system/`

Enable the service:

`sudo systemctl enable ardupilot`

Start it:

`sudo systemctl start ardupilot`

Check the status:

`sudo systemctl status ardupilot`

### Screenshot placeholder
![Systemd Service](screenshots/systemd-service.png)

---

# 12. Parameter File

The repository also includes an `ardupilot.parm` file.

Copy it to the location expected by the binary, for example:

`/home/pi/ardupilot.parm`

This file contains the startup parameters required for the EDUCOPTER configuration, including telemetry and GPS serial settings.

### Screenshot placeholder
![Parameter File](screenshots/parm-file.png)

---

# Repository Contents

`software/`
- `README.md`
- `ardupilot.service`
- `ardupilot.parm`

`hwdef/`
- `README.md`
- `EDUCOPTER/hwdef.dat`

---

# Future Work

Future improvements may include:

- automated patch scripts for ArduPilot source modifications
- wiring diagrams
- hardware schematics
- troubleshooting notes
- flight testing results

---

# Notes

This repository documents the **software setup only**. Hardware wiring, PCB layout, power distribution, and sensor integration should be documented separately in the hardware section of the project.

Where ArduPilot source files must be edited, only the relevant changes should be documented rather than copying entire upstream files. This makes the instructions more robust to future ArduPilot updates.
