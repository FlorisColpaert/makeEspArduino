# makeEspArduino
A makefile for ESP8266/Arduino projects.

The main intent for this project is to provide a minimalistic yet powerful and easy configurable
makefile for projects using the ESP8266/Arduino framework available at: https://github.com/esp8266/Arduino

Using make instead of the Arduino IDE makes it easier to do more production oriented builds of software projects.

This makefile basically gives you a command line tool for easy building and loading of the ESP8266/Arduino examples as
well as your own projects.

The makefile can use the ESP8266/Arduino environment either from the installation within the Arduino IDE or in a separate
git clone of the environment. The latter can be useful in project where you want stringent control of the environment
version e.g. by using it as a git submodule.

You basically just have to specify your main sketch file and the libraries it uses. The libraries can be from arbitrary
directories without any required specific hierarchy or any of the other restrictions which normally apply to builds made from within the
Arduino IDE. The makefile will find all involved header and source files automatically.

The makefile will also automatically produce header and c files which contain information about the time when the build (link)
was performed. This file also includes the git descriptions (tag) of the used version of the ESP8266/Arduino environment and the project source (when applicable).
This can be used by the project source files to provide stringent version information from within the software.

Rules for building the firmware as well as upload it to the ESP8266 are provided.

It is also possible to let the makefile generate and upload a complete SPIFFS file system based on an arbitrary directory of files.

The intension is to use the makefile as is. Possible specific configuration is done via via makefile variables supplied on the command line
or in separate companion makefiles.

The makefile can be used on Linux, Mac OS and Microsoft Windows (Cygwin).

The actual build commands (compile, link etc.) are extracted from the Arduino description files (platform.txt etc.).

#### Installing

First make sure that you have the environment installed as described at:  https://github.com/esp8266/Arduino <br>
If you don't want to use the environment installed in the Arduino IDE, then you can to clone it into a separate
directory instead, see below.

Then start cloning the makeEspArduino repository.

    cd ~
    git clone https://github.com/plerup/makeEspArduino.git

After this you can test it. Attach your ESP8266 and execute the following commands:

    cd makeEspArduino
    make -f makeEspArduino.mk upload

After this you will have the example "HelloServer" in your ESP. This is the default demo example which the makefile chooses
if no sketch has been specified.

If you want to use a clone of the environment instead then do something like this:

    cd ~
    git clone https://github.com/esp8266/Arduino.git esp8266
    cd esp8266

Determine which version you want to use. [See releases.](https://github.com/esp8266/Arduino/releases) Example:

    git checkout tags/2.3.0

Then install the required environment tools by issuing the following commands:

    cd tools
    python get.py


To test this installation you have to specify the location of the environment when running make

    cd ~/makeEspArduino
    make -f makeEspArduino.mk upload ESP_ROOT=~/esp8266

#### Building projects

You can now use the makefile to build your own sketches or any of the examples in the ESP8266/Arduino environment. The makefile will automatically search
for a sketch in the current directory and build it if found. It is also possible to specify the location of the sketch on the command line.
You may want to specify an alias first to minimize typing.

    alias espmake="make -f ~/makeEspArduino/makeEspArduino.mk"
    # Or when using a clone
    alias espmake="make -f ~/makeEspArduino/makeEspArduino.mk ESP_ROOT=~/esp8266"

##### Some examples

In current directory:

    cd ~/.arduino15/packages/esp8266/hardware/esp8266/2.3.0/libraries/Ticker/examples/TickerBasic
    espmake

Explicit naming of a default directory:

    espmake -C ~/.arduino15/packages/esp8266/hardware/esp8266/2.3.0/libraries/Ticker/examples/TickerBasic

Explicit naming of the sketch:

    espmake SKETCH=~/.arduino15/packages/esp8266/hardware/esp8266/2.3.0/libraries/Ticker/examples/TickerBasic/TickerBasic.ino
    # Or like this
    espmake SKETCH="\$(ESP_ROOT)/libraries/Ticker/examples/TickerBasic/TickerBasic.ino"

#### Advanced usage

The makefile has several variables which control the build. There are different ways to change the defaults of these variables.

The simplest and most direct way to do this is by specifying the variables and their values on the command line.

The more permanent way is to create a special makefile with the appropriate values for the variables and then include this in the build. This can be achieved
either by including makeEspArduino.mk in this file or the other way around by letting makeEspArduino.mk include it. The advantage with the latter method is that
the makefile doesn't need to know the location of makeEspArduino.mk, more about this in the examples below.

The two main variables in the makefile are named SKETCH and LIBS.

**SKETCH** is the path to the main source file. As stated above, if this is missing then makeEspArduino will try to locate it in the current directory. If no file is found
the demo sketch will be used.

**LIBS** is a variable which should contain a list of explicit source files and/or directories with multiple source files, which are to be compiled and used as libraries
in the build. Please note that there is no restrictions regarding location and naming of these files as in the Arduino IDE build system.
If this variable is not defined makeEspArduino tries to locate all required libraries by parsing the include statements in the sketch source file. Only
libraries in the ESP8266/Arduino library structure are detected though so if you have your own libraries you have to explicitly list them in this variable.
All source files located in the same directory as the sketch will also be included automatically.

There are some other important variables which corresponds to the settings which you normally do in the "Tools" menu in the Arduino IDE.

| Variable  | Function |
| :------------- | :------------- |
| **BOARD**  | The type of ESP8266 board you are using.<br>Default: generic.  |
| **FLASH_DEF**  | Flash memory configuration.<br>Default: 4M3M e.g. 4 MB flash with 3 MB SPIFFS.  |
| **UPLOAD_SPEED**  | Upload serial baudrate.<br>Default is taken from boards.txt.  |
| **UPLOAD_PORT**  | Upload serial port.<br>Default: /dev/ttyUSB0.  |

There are some other variables as well, check the first section in the makefile for more information about this.

##### Including the makefile

The easiest way to control the makefile is by defining the desired values of the control variables in your own makefile and then include makeEspArduino.mk. Example:


    # My makefile
    SKETCH = $(ESP_ROOT)/libraries/Ticker/examples/TickerBasic/TickerBasic.ino

    UPLOAD_PORT = /dev/ttyUSB1
    BOARD = esp210

    include $(HOME)/makeEspArduino/makeEspArduino.mk

Another possibility is to do this the other way around, i.e. let makeEspArduino include your makefile instead. This can be achieved by naming
your makefile "config.mk". makeEspArduino will always check for a file with this name in the current directory or in the same directory as the sketch.
If you want to use another name for your makefile you can specify this via the variable PROJ_CONF on the command line. Example of such a makefile:

    # config.mk
    THIS_DIR := $(realpath $(dir $(realpath $(lastword $(MAKEFILE_LIST)))))
    ROOT := $(THIS_DIR)/..
    LIBS = $(ESP_LIBS)/SPI \
      $(ESP_LIBS)/Wire \
      $(ESP_LIBS)/ESP8266WiFi \
      $(ROOT)/libraries \
      $(ROOT)/ext_lib

    UPLOAD_SPEED = 115200

It is of course also always possible to control the variable values in the makefile by defining them as environment variables in the shell. Exmaple:

    export UPLOAD_PORT=/dev/ttyUSB2

#### Building a file system

There are also rules in the makefile which can be used for building and uploading a complete SPIFFS file system to the ESP. This is basically the same functionality
as the one available in the Arduino IDE, https://github.com/esp8266/Arduino/blob/master/doc/filesystem.md#uploading-files-to-file-system

The file system content is made up of everything within a directory specified via the variable **FS_DIR**. By default this variable is set to a subdirectory named
**data** in the sketch directory.

Use the rule **upload_fs** to generate a file system image and upload it to the ESP.

All the settings for the file system are taken from the selected board's configuration.


#### Additional control options

There are some additional variables worth mentioning which control more advanced build options.

| Variable  | Function |
| :------------- | :------------- |
| **VERBOSE**  | By default the build process runs in silent mode, i.e. the commands are not echoed.<br>Set this variable to 1 in order to change this.  |
| **SINGLE_THREAD**  | The build is by default using multiple threads for parallel operations.<br>This variable can be set to 1 in order to force single threaded builds.  |
| **USE_PREBUILD**  | Later versions of ESP8266/Arduino has pre-build operations which creates a header file for git information. As makeESPArduino already has this function this is not enabled by default. Mainly because it will trigger unnecessary builds.<br>If you do want this function here as well set this variable to 1.  |
