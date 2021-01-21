# BlinkyBlocks
Easily deploy Blinky Blocks projects.

This repo gather both CLI and GUI interfaces, and their dependencies.

`git clone --recurse-submodules <thisRepo>`
This will automatically initialize and update each submodule in the repository

If you already cloned the project and forgot to update submodule
`git submodule update --init --recursive`

___

# Installation and Programming Blinky Blocks V2
In order to contribute you need to install qtcreator open-source IDE.

## Ubuntu installation of the Qt Creator IDE
`sudo apt-get install qt5-default`

`sudo apt-get install libqt5serialport5-dev`

`sudo apt-get install qtcreator`

Now two projects have to be compiled, first the libraries that contains the code, and then the CLI.

### Generate library libblinkyApploader.so
Switch to *layer2_base* branch and download [apploader2.zip](https://disc.univ-fcomte.fr/gitlab/flassabe/apploader2/commits/layer2_base)
Unzip in your local workspace directory

`cd BBV2/apploader2.git`

`qtcreator blinkyApploader.pro`

Select _Configure Project_
Then execute _Clean_, _qmake_, _Build all_

The file `/build-blinkyApploader-Desktop-Debug/libblinkyApploader.so.1.0.0` (> 1MB) should have been created.
___
Download [apploader-cli.zip](https://disc.univ-fcomte.fr/gitlab/flassabe/apploader-cli/)
Unzip in your local workspace directory

`cd ../apploader-cli.git`

`qtcreator apploader-cli.pro`

Select _Configure Project_
In the .pro check these lines

`INCLUDEPATH += $$PWD/../apploader2.git`

`DEPENDPATH += $$PWD/../apploader2.git`

Then execute _Clean_, _Build all_

The file `/build-blinkyApploaderCLI-Desktop-Debug/blinkyApploaderCLI` should have been created.
___
Install library in the default system path for libraries

`sudo cp ../build-blinkyApploader-Desktop-Debug/libblinkyApploader.so.1.0.0 /usr/lib/`

Link the library to the different existing revisions

`sudo ln -s /usr/lib/libblinkyApploader.so.1.0.0 /usr/lib/libblinkyApploader.so.1.0`

`sudo ln -s /usr/lib/libblinkyApploader.so.1.0.0 /usr/lib/libblinkyApploader.so.1`

`sudo ln -s /usr/lib/libblinkyApploader.so.1.0.0 /usr/lib/libblinkyApploader.so`
___
Now everything should be ready to launch. Try to display the help with :

`cd build-blinkyApploaderCLI-Desktop-Debug/`

`./blinkyApploaderCLI -h`
You must get:

```javascript
Usage: ./blinkyApploaderCLI
    -h display this message
    -s <SERIAL> sets serial interface name
    -c <R,G,B> sets connected blinky to color R,G,B
    -t applies command to blocks ensemble (builds a spanning tree)
    -e erase configuration
    -p <HEX file> starts sending HEX file to connected blinky ensemble
    -j 0xAAAAAAAA jumps to application address AAAAAAAA
    -k <ID min> sets blinky blocks software ID, starting with <ID min>
    -g requests blinky block to send its configuration
    -w <VAR ID,VAR VALUE> sets blinky block variable <VAR ID> to value <VAR VALUE>
                          	Application autostart -> 1 (jump auto after bootloading)
                          	Application autostart delay -> 2
                          	ID sur le spanning tree (doublons) -> 3
                          	Application address -> 4 (default is 0x8010000, param must be decimals)
                          	Commit configuration -> 128 (copy of previous -w modifications from RAM to Flash, with param 0)
```

### QTCreator Troubleshoot
Sometimes you can encounter errors while building Apploader2 for the first time. It is mostly related to gcc version and here are the steps to update it to version 9 and tell your system to use this version:

`sudo add-apt-repository ppa:ubuntu-toolchain-r/test`

`sudo apt update`

`sudo apt install gcc-9 g++-9`

`sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 60 --slave /usr/bin/g++ g++ /usr/bin/g++-9`

## Quitting the blinkyApploaderCLI program
You will see that, aside from running with the -h option, the application remains active even when network communications between the computer and the blinky blocks ended. Therefore, in the following instructions, when you read "Quitting"  in the output, it means that the program has been closed by the Ctrl-C signal. Indeed, for each step, you must send this signal to your application before being able to send the next command.

## Set ids to an ensemble of BB
First create a spanning tree to propagate the IDs

`sudo ./blinkyApploaderCLI -t`

```javascript
Opening serial port on /dev/ttyUSB0
Quitting
Closing port
```
Check the spanning tree in changing colors, all the BBs must change their color (here to black). If only the block directly connected to serial changes color, then the spanning tree is not available.

`sudo ./blinkyApploaderCLI -c 0`

```javascript
Opening serial port on /dev/ttyUSB0
Quitting
Closing port
```
Change the ids, the connected BB (8) receives 1000, the next one 1001, and so on 3F0 is equal to 1000 + 8  the number of the last updated id.

`sudo ./blinkyApploaderCLI -k 1000`

```javascript
Opening serial port on /dev/ttyUSB0
8A EA 00 00 A0
03 00 00 03 F0
Quitting
Closing port
```
Set the color to red

`sudo ./blinkyApploaderCLI -c 100,0,0`

```javascript
Opening serial port on /dev/ttyUSB0
Quitting
Closing port
```
## Programming a set of Blocks

### Install the development environment
We now know how to send basic commands to the BB and we will learn how to send a whole application/program through the CLI/IHM.
Applications are located at [/gitlab/flassabe/bbv2-application](https://disc.univ-fcomte.fr/gitlab/flassabe/bbv2-application)
Let’s take `distance_gradient` as an example. First, you need to download *OpenSTM32CubeIDE* and extract `STM32IDE-1.0.1.tar.gz` to your favorite folder.
To launch the IDE run `/stm32cubeide_x.x.x/stm32cubeide`
Then import our example as follows:
_File_ > _Open Projects from File System…_ > _Import Source_ > _Archive_ > *_appli.tar.gz_* > _Finish_

### Compilation
Now the STM32 on our hardware accepts *.hex* compiled files. By default if you build the code you will obtain a .elf file. We need to tell our IDE to post-build convert it to .hex as follows:
_Project_ > _Properties_ > _C/C++ Build_ > _Settings_ > _Tab : Build Steps_ > _Post-build steps_ > _Commands_
And write: `arm-none-eabi-objcopy -O ihex "${BuildArtifactFileBaseName}.elf" "${BuildArtifactFileBaseName}.hex"`
_Apply & Close_

You can now build all and search inside /Debug or /Release folder of your project, an Appli-ng.hex should have appeared aside the Appli-ng.elf. For flash size requirements, and since the BB has no debugging interface, you shall build the release version of the application.

### Upload
To upload and deploy the application on the BB go back to the CLI and enter:

`sudo ./blinkyApploaderCLI -p ~/yourpathtoproject/Appli-ng/Release/Appli-ng.hex`

A series of Request for chunk XXX should then appear.
If nothing reflects on the BB disconnect/reconnect them and run the spanning tree command beforehand:

`sudo ./blinkyApploaderCLI -t`

You shall see each BB with increasing green light intensity one after the other as a progress bar materializing the deployment of the bootloader and the application over each BB.

### Run
When every BB is green and ready to run your example application, you need to tell them to jump to the address of the application within the flash memory with:

`sudo ./blinkyApploaderCLI -j 0x8010000`

___

# Upload And Run A New Application

## Resetting Configuration
Reset to default configuration parameters from previous changes with the following:

`./blinkyApploaderCLI -t -c 0`

`./blinkyApploaderCLI -e`

Then cut the power and wait for all of them to turn off (including the white LED indicator from power supply blocks) and switch power back on.

## Set BB IDs
`./blinkyApploaderCLI -t -c 0`

`./blinkyApploaderCLI -k ID_START` (ID_START is an *INT* describing the first ID to be used from which to increment the others)

You shall see to output lines, only the second one is of interest where the two last bytes hexadecimal values are the 16 bits of the last allocated ID (check if it equals to ID_START + QTY_of_BB - 1)
Cut the power and wait for all of them to switch off (including the white LED indicator from power supply blocks) and switch power back on.

## Upload & Run

`./blinkyApploaderCLI -t -c 0`

Enable advanced programmation, all BB should turn magenta when ready:

`./blinkyApploaderCLI -a 1`

Upload you application hexadecimal file you downloaded or converted from .elf to .hex:

`./blinkyApploaderCLI -p APPLICATION_HEX_FILE`

Now jump to the start address of the application in Flash memory to run it with 1 sec delay:

`./blinkyApploaderCLI -j 0x8010000`

___

# Application Auto Start At Boot

Instead of the usual uploading of an application on the BB for it to replicate, then jump to start address in Flash memory, auto-start mode from booting is available. You can do so enabling the mode in configuration whether manually with the hexadecimal address or using a previously uploaded application start address.

Hard reboot the BBv2
Spread the spanning tree:

`./blinkyApploaderCLI -t -c 0`

Reset current configuration to default:

`./blinkyApploaderCLI -e`

Hard reboot the BBv2 (power off, then on), then rebuild the spanning tree

`./blinkyApploaderCLI -t -c 0`

Set IDs:

`./blinkyApploaderCLI -k 1 (1 is for the sake of the example)`

Hard reboot the BBv2 (power off, then on), then rebuild the spanning tree

`./blinkyApploaderCLI -t -c 0`

Switch to advanced configuration mode BBPP:

`./blinkyApploaderCLI -a 1`

Upload and run the new application the manual way for validation:

`sudo ./blinkyApploaderCLI -p ~/Downloads/Appli-ng.hex`
`sudo ./blinkyApploaderCLI -j 0x8010000`

Hard reboot the BBv2

`./blinkyApploaderCLI -t -c 0`

Check the current configuration

`./blinkyApploaderCLI -g`

```
00000001 00000000 00000001 00000000 08008000 FFFFFFFF
```
Only 5 x 32 bits words out of 16 are currently reserved variables located in the bootloader configuration section, but it is definitely possible to develop an app with up to 11 variables (the remaining ones, not used by the bootloader):
1. Configuration section @ (for internal wear leveling use)
2. Auto start flag
3. Delay (sec)
4. ID
5. Application start @
6. First unused variable
Configure auto start and execution delay:

`sudo ./blinkyApploaderCLI -w 1,1 -w 2,2`

Until now all configuration update are stored in RAM in order to reduce useless Flash writings. Therefore, we shall write these changes in the Flash once and for all when we are ready, and check the new configuration:

`sudo ./blinkyApploaderCLI -w 128,0` (0 parameter not used)

`./blinkyApploaderCLI -g`

```
00000004 00000001 00000002 00000001 08010000 FFFFFFFF
```
Hard reboot the BBv2
After 2 seconds as previously configured, the application shall start.

___

# Ugrade Bootloader With Bootloader Application

## Install the custom application dedicated to flash the bootloader
Currently the upgrade is performed sequentially, when the process is reliable enough we will devise to switch to parallel programming from the bootloader to the application. The following tutorial specifies a serie of commands where in-between the group of blinky blocks **must not** be restarted, unless it is explicitly written.

When we want to send control commands from the CLI towards the BB we first proceed with the creation of the spanning tree, and resetting colors to rgb(0,0,0) as follows:

`./blinkyApploaderCLI -t -c 0`

Now we want to deploy an application whose only purpose is to upgrade the bootloader

`./blinkyApploaderCLI -p 2020-07-08-Bootloader-as-application-aligned.hex` [download file]()

All the BB should gradually turn to green (as if it was a progress bar)
When all of them are a 100% green, jump to the start address of the application in Flash memory with:

`./blinkyApploaderCLI -j 0x8010000`

All the BB should turn magenta when they are ready

_For a reminder of available commands please refer to [help](https://disc.univ-fcomte.fr/gitlab/flassabe/apploader-cli/wikis/Installation-and-Programming-of-Blinky-Blocks-V2#generate-library-libblinkyapploader-so)._

## Upgrade the bootloader from the application you uploaded
Now the BB are ready to receive the new bootloader, we upload the file

`./blinkyApploaderCLI -t -c 0`
`./blinkyApploaderCLI -p 2020-09-21-Bootloader.hex` [download file]()

## Rebooting the bootloader
When all the BB are a 100% green, cut the power and wait for all of them to switch off (including the white LED indicator from power supply blocks)
Switch power back on, all the BB should light up in yellow meaning they are started in the new bootloader.
