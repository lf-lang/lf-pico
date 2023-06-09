# Chapter 1
## Tools and Environments
This chapter will overview the different tool required for software development on embedded
systems. Microcontrollers often have limited resources and interaction methods due to being headless devices. The tasks will introduce a common development workflows for these
platforms.

This section will explore the RP2040 microcontroller, a low cost dual-core arm based chip with various useful and unique hardware peripherals. The chip is embedded in the popular Raspberry Pi Pico and is tailored for education.

## Prelab
### Resources
Here are some documents from which this chapter was derived from that detail the various components of the setup.

Build System
- Bash Scripting
- Nix
- Command Line

RP2040
- Datasheet
- Getting Started Pico
- Pico SDK

Lingua Franca
- Introduction
- 

Textbook
- Memory Architecture
- Embedded Processors

### Exercises
1. TODO
2. TODO
3. TODO

## Repo
This book uses the [lf-pico](https://github.com/lf-lang/lf-pico) repository as a framework that contains setup scripts and examples for the applications described. Make sure the copy of lf-pico is up to date with the current documentation. Clone or navigate to the repository, pull the latest and compare the commit sha.
```
git clone https://github.com/lf-lang/lf-pico.git
git pull
git log -1
```
latest commit sha: 9fc422f05d934fdc45b7ebace22cfdaa12b3c3f9

## Setup
One challenge of working with embedded systems is installing tooling. To create a reproducible unix shell environment that installs all required dependency applications, we use the nix package manager. Install [nix](https://nixos.org/download.html) first for your preferred platform. There is support for windows (wsl), macos and linux. 

After installation, run the following in the shell to enable the experimental nix flakes feature.
``` bash
mkdir -p ~/.config/nix
echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf
```

To launch the lf-pico shell environment, run the following in the root of the lf-pico repository. 
```bash
nix develop
```

This should automatically download and install specific revisions of the gcc-arm toolchain, openocd and picotool. These packages will be required compiling, flashing and debugging C code for the RP2040.

## Blink
A common "Hello, World" application for embedded platforms is the blink application which periodically flashes an LED connected on a GPIO pin of the controller. Many of the low level instructions required to interact with the GPIO peripheral are abstracted using the [pico-sdk](https://github.com/raspberrypi/pico-sdk/tree/master) C/C++ library. Using the gcc-arm toolchain, the library provides cmake support for generating binaries for the rp2040.  More details [here](./Pico-SDK-Primer).

We will also introduce the Lingua Franca (LF) coordination framework as a high level scheduling framework. The framework emphasizes modeling computation as a network of concurrent reactor interactions. Reactors are similar to [actors](https://en.wikipedia.org/wiki/Actor_model) but also order simultaneous input reactions and timestamp messages. This allows for highly reproducible and testable code. More details [here](./Lingua-Franca-Primer)

To start, build the `src/Blink.lf` example using the following command. This will invoke the LF compiler which generates C sources from the LF file and builds elf, uf2 and hex binaries for the rp2040.
``` shell
lfc src/Blink.lf
```

## Flashing
Before flashing the binary to your rp2040 based board, the board must be placed into ``BOOTSEL`` mode. 
- On a regular Raspberry Pi Pico this can be entered by holding the ``RESET`` button while connecting the board to the host device. 
- The 3PI+ robot can enter ``BOOTSEL`` mode by holding the ``B`` button and pressing the ``RESET`` button while connected through usb.
The picotool application installed in the nix shell is an easy way to interact with boards.
With the application you can check what programs are currently flashed and can directly load program binaries.
Run the following to load the Blink application on to your board.

``` shell
picotool load bin/Blink.elf
```

TODO : Explanation bubble on elf files
TODO : Bootloaders
TODO : Flash 


## Debugging
Debugging is done using a secondary RPI pico board flashed with the picoprobe firmware.
This [thesis](https://openocd.org/files/thesis.pdf) dedicated to the development of the open-ocd software contains a good overview of the problems with debugging embedded platforms.
For this reason, openocd runs on the host device and uses different hardware debug protocols to interface with gdb, the gnu debugger. The debugging frontend can be changed,
and for use with vscode, a simple run configuration must be provided.

Flash the picoprobe firmware from [here](https://github.com/raspberrypi/picoprobe/releases/download/picoprobe-cmsis-v1.02/picoprobe.uf2) using picotool on the RPI Pico device that will be used
as the debugger. The wiring diagram here shows what pins to connect in order to debug the 3pi robot.
TODO: diagram

Run openocd to start the debugging server and load a specific elf file. This will create a gbd server that can be connected to. 

