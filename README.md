# BitstreamEvolution
Code for the BitstreamEvolution Open Source Toolkit

## Table of Contents
- [BitstreamEvolution](#bitstreamevolution)
  - [Table of Contents](#table-of-contents)
  - [Setup](#setup)
    - [Requirements](#requirements)
    - [Installing the dependencies](#installing-the-dependencies)
    - [Configuring the BistreamEvolution core](#configuring-the-bitstreamevolution-core)
      - [Primary Targets](#primary-targets)
      - [Targets for building Project Icestorm tools](#targets-for-building-project-icestorm-tools)
	  - [Clean targets](#clean-targets)
    - [Configuring the Arduino components](#configuring-the-arduino-components)
      - [Obtaining the arduino-cli tool](#obtaining-the-arduino-cli-tool)
    - [Determining the correct device files](#determining-the-correct-device-files)
      - [Finding device files with udevadm](#finding-device-files-with-udevadm)
      - [Finding device files with dmesg](#finding-device-files-with-dmesg)
    - [Setting up permissions](#setting-up-permissions)
    - [Issues with setup](#issues-with-setup)
      - [USB permission denied](#usb-permission-denied)
  - [Usage](#usage)
    - [Configuration](#configuration)
      - [GA parameters](#ga-parameters)
        - [Selection methods](#selection-methods)
        - [Initialization modes](#initialization-modes)
      - [Logging parameters](#logging-parameters)
      - [System parameters](#system-parameters)
      - [Hardware parameters](#hardware-parameters)
    - [Running](#running)
    - [Troubleshooting](#troubleshooting)
      - [Program hangs during FPGA programming](#program-hangs-during-fpga-programming)
  - [Contributing](#contributing)
  - [License](#license)

## Setup
Setting up BitstreamEvolution involves configuring the
BitstreamEvolution core, the [Project Icestorm](http://www.clifford.at/icestorm/)
tools, and the Arduino components. While BitstreamEvolution should run
on any Linux distribution and likely other Unix-based systems, the
instructions assume you are running a Debian-based distrubtion such as
Debian or Ubuntu.

### Requirements
BistreamEvolution requires the following libraries and packages:
  * build-essential
  * clang
  * bison
  * flex
  * libreadline-dev
  * gawk
  * tcl-dev
  * libffi-dev
  * libftdi-dev
  * git
  * mercurial
  * graphviz
  * xdot
  * pkg-config
  * python3
  * python3-pip
  * libboost-all-dev
  * cmake
  * make

BitstreamEvolution alse requires the following Python libraries:
  * pyserial
  * matplotlib
  * numpy
  * sortedcontainers

### Installing the dependencies
Each of the dependencies above has a corresponding `apt` package of the
same name. For other package managers the package name may be different.
For Debian-based distributions and other distributions that use `apt`,
the packages can all be installed at once with the following commands:

```bash
sudo apt update && sudo apt upgrade  # Optional, but recommended 
sudo apt install build-essential clang bison flex libreadline-dev gawk tcl-dev libffi-dev libftdi-dev mercurial graphviz xdot \ 
pkg-config python3 python3-pip libboost-all-dev cmake make
```
The Python libraries can be installed in one command in any Linux
distribution as follows:

```bash
python3 -m pip install pyserial numpy matplotlib sortedcontainers
```
### Configuring the BitstreamEvolution core
Although BitstreamEvolution doesn't require any building or
compilation (other than the Project Icestorm tools), it utilizes make
targets to simplify configuration. The simplest and recommended way to
configure the BitstreamEvolution core is to run `sudo make` in the root
directory of BitstreamEvolution, which will default to running the
target `all`. To allow users to optimize their configuration and save
space (BitstreamEvolution with all its tools is around 2.2 GB), other
targets are exposed. The description of each target is given in the
tables below.

#### Primary targets
These targets are responsible for the setup and configuration of
BitstreamEvolution and some of its dependencies. Note that the `all`
target performs the actions of all other targets (except for the clean
targets). One reason to utilize one of the targets other than `all` is
if the user wants to reconfigure something or if they would like to
manually install or have already installed the Project Icestorm
tools.

|Target|Actions|
|------|-------|
|`all`|Creates the directories for logging data, initializes the default configuration settings installs all the Project Icestorm tools, and creates and writes the udev rules for the Lattice ICE40 USB serial programmer|
|`init`|Creates the directories for logging and initializes the default configuration settings|
|`udev-rules`|Creates and writes the udev rules for the Lattice ICE40 USB serial programmer|

#### Targets for building Project Icestorm tools
These targets are used to individually build the tools
BitstreamEvolution depends on. In general, they are only useful if some
of the Project Icestorm tools have already been installed and the user
does not want to overwrite the previous installs or wants to save on
disk space.

|Target|Actions|
|------|-------|
|`icestorm-tools`|Builds and installs all the Project Icestorm tools|
|`icestorm`|Builds and install just the icestorm tools (e.g. icepack, iceprog)|
|`arachne-pnr`|Builds and installs just the arachne-pnr tool|
|`yosys`|Builds and installs just the yosys tools|

#### Clean targets
*In general, use of these targets is not recommended*. Since
BitstreamEvolution does not build any intermediate targets, clean
targets are used to get rid of other automatically generated files such
as the workspace defaults and the tools. In particular, cleaning the
latter is not recommended as this makes it more difficult to uninstall
the project Icestorm tools. So far, the primary use of these targets has
been for testing and maintainence of the project.

|Target|Actions|
|------|-------|
|`clean`|Removes the default permanent data logging directories and their contents as well as all the build directories for the Project Icestorm tools (*but it does not uninstall them*)|
|`clean-workspace`|Removes the default permanent data logging directories and their contents|
|`clean-tools`|Removes all the build directories for the Project Icestorm tools (*but it does not uninstall them*)|

<!-- Arduino-CLI Instructions -->
### Configuring the Arduino components
The Arduino components are used by BitstreamEvolution to control and
communicate with the microcontroller which is responsible for measuring
the signals from the FPGA. The Arduino component can be configured two
different ways. The first way is to utilize the Arduino GUI to compile
and upload the microncontroller code. The second and recommended way is
to use the official Arduino-cli tools. This section describes the latter
method.

Ideally, in the future, installation and configuration of the
arduino-cli tool could be done automatically with a make target.

#### Obtaining the arduino-cli tool
Download and extract the pre-built binary from the [Arduino webpage](https://arduino.github.io/arduino-cli/latest/installation/#latest-packages)

For an x86_64 machine running Linux, this can be done with the following
commands:
```bash
wget https://downloads.arduino.cc/arduino-cli/arduino-cli_latest_Linux_64bit.tar.gz
tar -xf arduino-cli_latest_Linux_64bit.tar.gz -z
```

In the same directory where you downloaded and extracted arduino-cli,
run the following commands to configure it:

```bash
./arduino-cli update
./arduino-cli upgrade
./arduino-cli core download arduino:avr
./arduino-cli core install arduino:avr
./arduino-cli compile -b arduino:avr:nano [PATH TO PROJECT i.e. ~/BitstreamEvolution/data/ReadSignal/ReadSignal.ino]
```

If you are using an Arduino microcontroller other than a nano, replace
"nano" in the last command with the name of the Arduino microcontroller
type you are using (e.g. "mega", "uno", "duo", et cetera). Note that
while the program is likely to work for any Arduino microcontroller, it
has only been thoroughly tested with a 5V nano.

After arduino-cli has been configured, you will need to upload the
sketch to the microcontroller with the arduino-cli tool. This requires
accessing the correct device file and having the proper priviledges for
it. The device file should look something like `/dev/ttyUSB#` where
`#` is a number. To determine which device file is the correct one and
how to enure you have proper access see the
[following section](#determining-the-correct-device-files).

Once you have found the correct file and ensured you can access it, run
the following command to upload the sketch, making sure to replace the
`#` with the appropriate number (or the entire filename if your system
uses a different type of device file):

```bash
./arduino-cli upload -b arduino:avr:nano -p /dev/ttyUSB# PATH/TO/SKETCH
```
Where `PATH/TO/SKETCH/` may look something like: 
`~/BitstreamEvolution/data/ReadSignal/ReadSignal.ino`

### Determining the correct device files
Correctly configuring BitstreamEvolution requires determining the device
files for the Arduino microcontroller and the Lattice ICE40 FPGA. The
device file for each will likely have the form `\dev\ttyUSB#` where `#`
is a number There are two ways to do determine the device file
associated with a particular device: one uses:
[`udevadm`](#finding-device-files-with-udevadm) and the other uses
[`dmesg`](#finding-device-files-with-dmesg).

#### Finding device files with udevadm
Information about a particular device file can be found with the
command `udevadm info <filename>`, where `<filename>` is the name of the
file you want to examine. From the output of this command, you can
discern whether a particular device file corresponds to the device you
are looking for.

Here is an example of using `udevadm` to examine the device file of the
Lattice ICE40 FPGA:
```bash
$ udevadm info /dev/ttyUSB0
P: /devices/pci0000:00/0000:00:15.0/0000:03:00.0/usb3/3-2/3-2:1.0/ttyUSB0/tty/ttyUSB0
N: ttyUSB0
L: 0
S: serial/by-id/usb-Lattice_Lattice_FTUSB_Interface_Cable-if00-port0
S: serial/by-path/pci-0000:03:00.0-usb-0:2:1.0-port0
E: DEVPATH=/devices/pci0000:00/0000:00:15.0/0000:03:00.0/usb3/3-2/3-2:1.0/ttyUSB0/tty/ttyUSB0
E: DEVNAME=/dev/ttyUSB0
E: MAJOR=188
E: MINOR=0
E: SUBSYSTEM=tty
E: USEC_INITIALIZED=23901723886
E: ID_BUS=usb
E: ID_VENDOR_ID=0403
E: ID_MODEL_ID=6010
E: ID_PCI_CLASS_FROM_DATABASE=Serial bus controller
E: ID_PCI_SUBCLASS_FROM_DATABASE=USB controller
E: ID_PCI_INTERFACE_FROM_DATABASE=XHCI
E: ID_VENDOR_FROM_DATABASE=Future Technology Devices International, Ltd
E: ID_MODEL_FROM_DATABASE=FT2232C/D/H Dual UART/FIFO IC
E: ID_VENDOR=Lattice
E: ID_VENDOR_ENC=Lattice
E: ID_MODEL=Lattice_FTUSB_Interface_Cable
E: ID_MODEL_ENC=Lattice\x20FTUSB\x20Interface\x20Cable
E: ID_REVISION=0700
E: ID_SERIAL=Lattice_Lattice_FTUSB_Interface_Cable
E: ID_TYPE=generic
E: ID_USB_INTERFACES=:ffffff:
E: ID_USB_INTERFACE_NUM=00
E: ID_USB_DRIVER=ftdi_sio
E: ID_PATH=pci-0000:03:00.0-usb-0:2:1.0
E: ID_PATH_TAG=pci-0000_03_00_0-usb-0_2_1_0
E: ID_MM_CANDIDATE=1
E: DEVLINKS=/dev/serial/by-id/usb-Lattice_Lattice_FTUSB_Interface_Cable-if00-port0 /dev/serial/by-path/pci-0000:03:00.0-usb-0:2:1.0-port0
E: TAGS=:systemd:
E: CURRENT_TAGS=:systemd:
```
References to Lattice in the output confirm that `/dev/ttyUSB0` is the
device file (in this example) for the Lattice ICE40 FPGA.

This is an example of using `udevadm` to examine the device file for the
Arduino nano microcontroller. Note that your output may differ depending
on the manufacturer of your microcontroller:
```bash
$ udevadm info /dev/ttyUSB0
P: /devices/pci0000:00/0000:00:15.0/0000:03:00.0/usb3/3-2/3-2:1.0/ttyUSB0/tty/ttyUSB0
N: ttyUSB0
L: 0
S: serial/by-path/pci-0000:03:00.0-usb-0:2:1.0-port0
S: serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
E: DEVPATH=/devices/pci0000:00/0000:00:15.0/0000:03:00.0/usb3/3-2/3-2:1.0/ttyUSB0/tty/ttyUSB0
E: DEVNAME=/dev/ttyUSB0
E: MAJOR=188
E: MINOR=0
E: SUBSYSTEM=tty
E: USEC_INITIALIZED=25223386657
E: ID_BUS=usb
E: ID_VENDOR_ID=1a86
E: ID_MODEL_ID=7523
E: ID_PCI_CLASS_FROM_DATABASE=Serial bus controller
E: ID_PCI_SUBCLASS_FROM_DATABASE=USB controller
E: ID_PCI_INTERFACE_FROM_DATABASE=XHCI
E: ID_VENDOR_FROM_DATABASE=QinHeng Electronics
E: ID_MODEL_FROM_DATABASE=CH340 serial converter
E: ID_VENDOR=1a86
E: ID_VENDOR_ENC=1a86
E: ID_MODEL=USB2.0-Serial
E: ID_MODEL_ENC=USB2.0-Serial
E: ID_REVISION=0263
E: ID_SERIAL=1a86_USB2.0-Serial
E: ID_TYPE=generic
E: ID_USB_INTERFACES=:ff0102:
E: ID_USB_INTERFACE_NUM=00
E: ID_USB_DRIVER=ch341
E: ID_USB_CLASS_FROM_DATABASE=Vendor Specific Class
E: ID_PATH=pci-0000:03:00.0-usb-0:2:1.0
E: ID_PATH_TAG=pci-0000_03_00_0-usb-0_2_1_0
E: ID_MM_CANDIDATE=1
E: DEVLINKS=/dev/serial/by-path/pci-0000:03:00.0-usb-0:2:1.0-port0 /dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
E: TAGS=:systemd:
E: CURRENT_TAGS=:systemd:
```
References to the manufacturer of the board (QinHeng Electronics) and to
its function as a "serial convertor" indicate that (in this example) the
device file `/dev/ttyUSB0` is associate with the Arduino nano
microcontroller.

#### Finding device files with dmesg
The device file for a particular device can also be found using the
demsg command as follows.

```bash
sudo dmesg | tail
```

If no messages referring to a device or device file appear, you may
want to look further back. You can run a command of the form:

```bash
sudo dmesg | tail -n N
```
Where `N` is the number of lines you want to look back.

<!--TODO include examples of running dmesg-->

### Setting up permissions
Accessing certain device files often requires special permissions. These
permission are often given by adding a user to a specified group. On
Debian-based Linux distributions, the group is often `plugdev` or
sometimes `dialout`.  Below is an example from a machine running Ubuntu:

```bash
$ ls -la /dev/ | grep ttyUSB
crw-rw---- 1 root dialout 188,  0 May 4 17:20 ttyUSB0
crw-rw----+ 1 root plugdev 188, 1 May 4 17:20 ttyUSB1
crw-rw----+ 1 root plugdev 188, 2 May 4 17:20 ttyUSB2
```

Long listing the specified device files shows that read/write permission
is restricted to members of the groups `dialout` for `ttyUSB0` and
`plugdev` for `ttyUSB1` and `ttyUSB2`.

BitstreamEvolution requires read/write permission for the device files
for the Lattice ICE40 FPGA and the Arduino microcontroller. To add
yourself as a member to appropriate group, you use the command:

```bash
sudo usermod -a -G GROUPNAME USERNAME
```

Where `GROUPNAME` is the name of the group that has permissions for the
appropriate device file and `USERNAME` is your username. For the devices
in the example listed above we would run the following the add the user
`USERNAME` to the appropriate groups:

```bash
sudo usermod -a -G dialout USERNAME
sudo usermod -a -G plugdev USERNAME
```

### Issues with setup:
This section describes some issues that can be encountered during
installation and how to address them. If the following steps do not
address your issue or you encounter other problems, please file an issue
[here](https://github.com/evolvablehardware/BitstreamEvolution/issues) so that
we can look into it.

#### USB Permission denied
If you get permission denied related to a USB and you have updated
the rules, try:
  * log out of your current session or reboot as the added rules may not have taken effect
  * run `ls -l /dev` and look at `ttyUSB0` and `ttyUSB1`. Ensure that
    the user you are running the program with is a member of the group
    that can access these devices (do not run the program as root!)

## Usage
This section describes how to run the configure and run
BitstreamEvolution. For the most part, BitstreamEvolution can be run
as-is without configuration; the only part of the configuration file
that needs to be modified is the
[Arduino device file path](#system-parameters).

<!--
  TODO Configuration options that need to be changed should be
  better highlighted and emphasized. Ideally, the program would ensure
  that the user has set these before running (and they would be unset by
  default
-->
### Configuration
The project has various configuration options that can be specified in
`data/config.ini`. The file`data/default_config.ini` contains the
default options for the configuration and should not be modified. Below
is a list of the options, their description, and their possible values:

<!-- NOTE Right now this only lists the most important options-->

#### GA parameters
| Parameter | Description | Possible Values | Recommended Values |
|-----------|-------------|-----------------|--------------------|
| Population size | The number of circuits to evolve | 2 - 1000+ | 10 - 50 |
| Generations | The maximum number of generations to iterate through | 2 - 1000+ | 50 - 500 |
| Mutation probability | The probability to flip a bit of the bitstream during mutation | 0.0 - 1.0 | (1 / genotypic length) = 0.0021 |
| Crossover probability | The probability of replacing a bit in one bitstream from a bit from another during crossover | 0.0 - 1.0 | 0.1 - 0.5 |
| Elitism fraction | The percentage of most fit circuits to protect from modification in a given generation | 0.0 - 1.0 | 0.1 |
| Desired frequency | The target frequency of the evolved oscillator | (In Hertz) 1 - 1000000 | 1000 |
| Selection | The type of selection to perform | SINGLE_ELITE, FRAC_ELITE, CLASSIC_TOURN, FIT_PROP_SEL| CLASSIC_TOURN |
| Variance threshold | The target signal variance from initial random search | 3-8 | 4 |
| Seed Mode | The method to generate the initial random circuits | *TODO Add Seed Modes to CircuitPopulation.Py and List them here* | RAND_FROM_SEED |

##### Selection methods
<!--TODO ALIFE2021 Describe the various selection methods-->
*TODO Describe the various selection methods*

##### Initialization modes
<!--TODO ALIFE2021 Describe the various initialization modes-->
*TODO Describe the various initialization modes*

#### Logging parameters
<!--TODO ALIFE2021 Describe these -->
*TODO Describe these*

#### System parameters
| Parameter | Description | Possible Values |
|-----------|-------------|-----------------|
| USB Path | The path to the USB device file | Any device file path (e.g. `/dev/ttyUSB0`) |

#### Hardware parameters
<!--TODO Describe these -->
*TODO Describe these*

### Running
From the root directory of BitstreamEvolution run:

```bash
python3 src/evolve.py
```

BitstreamEvolution will begin to run and display information in separate windows
that will appear (unless these have been disabled in the configuration).

BitstreamEvolution will continue to run until one of the following happens:
  * It has run through the specified number of generations
  * It has met the specified conditions
  * It is terminated in some other form (e.g. ctrl-c, shutdown, etc.)

### Troubleshooting
#### Program hangs during FPGA programming
BitstreamEvolution may hang indefinitely while attempting to program the
FPGA with `iceprog`. This often happens if the circuit upload process was
quit or interrupted for any reason.

A simple fix for this is to disconnect and reconnect the FPGA.

## Contributing
<!--TODO ALIFE2021 define the desired approach -->
*Don't know what Derek wants to do regarding contributing*.

## License
This project is licensed under the GNU GPL v3 or later. For more
information see [LICENSE](LICENSE).
