These instructions are out of date as of July 2019 version of Sming. Minor amendments need to be made to comply with [building.md](https://github.com/SmingHub/Sming/blob/develop/Sming/building.md)


## Prerequisites:
_(You might already have it)_

##### Xcode command line tools
```shell
xcode-select --install
```

##### Homebrew
```shell
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Install build toolchain
```shell 
brew install binutils coreutils automake wget gawk libtool gettext gperf grep
```

## Install GNU sed

Install gnu-sed and make sure that it is the default `sed` command line application. 

### Option 1: using $PATH

Install gnu-sed
```
brew install gnu-sed
```

Add `<brew install root>/opt/gnu-sed/libexec/gnubin` to your `$PATH` to make gnu-sed the default `sed` command. Include this in your `~/.profile` or equivalent to set the value permanently. 

```
export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"
```

### Option 2: using --with-default-names
_(Note: this doesn't work with Homebrew 2.1.11, `--with-default-names` isn't available.)_

For the installation use the commands below:
```shell
brew install gnu-sed --with-default-names
```

If you have already installed `gnu-sed` but it is not the default one then make sure to uninstall it and install it again with the correct options. This can be done using the commands below:

```shell
brew uninstall gnu-sed
brew install gnu-sed --with-default-names
```

## Install prebuilt Open ESP SDK
```
cd ~/
curl -L -O  https://dl.bintray.com/sminghub/toolchain/esp-open-sdk-2.0-macosx.tgz
sudo mkdir -p /opt/
sudo tar -zxf esp-open-sdk-2.0-macosx.tgz -C /opt/
sudo chmod -R 775 /opt/esp-open-sdk
```
You can also build it yourself [with Homebrew](https://github.com/pfalcon/esp-open-sdk#macos) or [with MacPorts](http://www.esp8266.com/wiki/doku.php?id=setup-osx-compiler-esp8266).

## Get And Build Sming Core:
```shell
cd <your-favourite-development-folder>/

git clone https://github.com/raksagro/Sming.git
# Warning: Do NOT use the --recursive option for the command above. 
#          Our build mechanism will take care to get the third-party sources and patch them, if needed.

cd Sming/Sming

# You will get a copy of our `develop` branch which intended for developers 
# and it is the one where all new cool (unstable) features are landing. 

# HERE is where you will go to Raks branch
git checkout masterRaks

# And finally set the SMING_HOME environment variable to point to <your-favourite-development-folder>/Sming/Sming

export SMING_HOME=`pwd`

make
```

## Configure Environment Variables
Open with a text editor the `.profile` file in your home directory. And add at the end the following lines:

```shell
export ESP_HOME=/opt/esp-open-sdk
export SMING_HOME=<your-favourite-development-folder>/Sming/Sming
```

Make sure to replace `<your-favourite-development-folder>` in the command above with the actual directory on your local disk.

## Python/Serial Problems?
In case an error involving serial or python happens, make sure you have python installed e the pyserial module

Installing python:
```
brew install python
```
Installing pyserial:
```
pip install pyserial==3.4
```

### (Optional step) 
_(used by Make and Eclipse - make sure to quit Eclipse first)_

If you installed Eclipse manually, substitute `/opt/homebrew-cask/Caskroom/eclipse-cpp/4.5.1/Eclipse.app` to `/Applications/Eclipse/eclipse.app`
```
sudo /usr/libexec/PlistBuddy -c "Add :LSEnvironment:ESP_HOME string '/opt/esp-open-sdk'" /opt/homebrew-cask/Caskroom/eclipse-cpp/4.5.1/Eclipse.app/Contents/Info.plist
sudo /usr/libexec/PlistBuddy -c "Add :LSEnvironment:SMING_HOME string '/opt/sming/Sming'" /opt/homebrew-cask/Caskroom/eclipse-cpp/4.5.1/Eclipse.app/Contents/Info.plist
sudo /System/Library/Frameworks/CoreServices.framework/Frameworks/LaunchServices.framework/Support/lsregister -v -f /opt/homebrew-cask/Caskroom/eclipse-cpp/4.5.1/Eclipse.app
```

## Using Eclipse (Oxygen) on Mac
If you are using Eclipse to build the samples you need to make sure the path is set correctly for the make process.

First make sure the Makefile_macos.mk references the correct serial port for your ESP8266. This file is found in SMING_HOME and the entry for Default COM Port should show the device name your ESP8266 gets when it is plugged in.

Next Ensure that you can build the sample Basic_Blink from a terminal window by changing to the sample folder and executing 'make'.
If this works without errors then type 'echo $PATH' and copy the resulting path to the clipboard.

Now fire up Eclipse and go to

_Eclipse==>Preferences==>C/C++==>Build==>Environment_

and add a new variable PATH. Paste in the path saved from the terminal session above.
You can also add SMING_HOME and ESP_HOME variables here the same way as you set in the export commands above which will then be set for all the projects.

The standard make files use miniterm.py to provide a serial terminal for debugging the ESP8266. Miniterm does not work inside Eclipse so you should open Makefile_macos.mk in the SMING_HOME folder and comment out the line

`KILL_TERM    ?= pkill -9 -f "$(COM_PORT) $(COM_SPEED_SERIAL)" || exit 0`

replacing it with 

`KILL_TERM ?=`

Next comment out the line

`TERMINAL     ?= python -m serial.tools.miniterm $(COM_PORT) $(COM_SPEED_SERIAL) $(COM_OPTS)`

and replace it with

`TERMINAL ?=`

This will prevent Eclipse from trying to launch miniterm and throwing an error about Inappropriate ioctl for device.

You can use the built in terminal in Eclipse Oxygen by adding it using

_Window==>Show View==>Terminal_

then setting terminal type to Serial and setting the port to the port the ESP8266 is connected to. Remember to disconnect before tying to re-flash the device though.

## Compile Sming Examples
You can find all examples coming with Sming under the `samples` folder. 

```shell
cd $SMING_HOME/../samples/
```

If you want to test some of the examples the following sequence of commands can help you:
```
cd $SMING_HOME/../samples/
cd Basic_Blink
make
# The command below will upload the sample code to your ESP8266 device
make flash  