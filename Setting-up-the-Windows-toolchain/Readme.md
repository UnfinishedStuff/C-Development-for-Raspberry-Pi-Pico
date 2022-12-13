# Setting up the Windows development toolchain for the Raspberry Pi Pico and C/++

Honestly, setting up the Windows toolchain is fraught with difficulties, and using a Raspberry Pi as the development environment is just easier.  However, there's something nice about being able to do development and compilation in VSCode on Windows if you use that OS routinely, and it's even possible to set up one-click build-and-upload using the Picoprobe firmware provided by the Pi Foundation.  I've documented the process I've used to set this up below, but be warned: you may experience errors with things which shouldn't error...

## Information Sources

Most of this information was drawn from the [***Getting started with Raspberry Pi Pico** C/C++ development with Raspberry Pi Pico and other RP2040-based microcontroller boards* PDF which you can find here](https://www.raspberrypi.com/documentation/microcontrollers/raspberry-pi-pico.html), particularly Section 9.2.  That doc is much more concise than this guide, so if you're an experienced dev it might be faster to work from that.  However, if you plan on using Picoprobe to debug your device, I'd point out that it's a faff and I wrote this guide to summarise workarounds for problems I experienced.

I've also used some information from [Shaun Hymel's *Raspberry Pi Pico and RP2040 - C/C++ Part 2: Debugging with VS Code* guide, written for Digikey, which you can find here](https://www.digikey.bg/en/maker/projects/raspberry-pi-pico-and-rp2040-cc-part-2-debugging-with-vs-code/470abc7efb07432b82c95f6f67f184c0).

I've probably also picked up a few things from miscellaneous places on the internet.  Sorry, but while debugging this process I went through a lot of troubleshooting steps on a lot of web pages, and can't remember most of them now!

## Installing the tools

### ARM development tools 

The ARM GNU toolchain is required to compile code to run on the ARM Cortex M0+ cores of the Pi pico.
* Download the ARM GNU Toolchain from here: https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads
  * Download the file ending with “-arm-none-eabi.exe)”, e.g. `arm-gnu-toolchain-12.2.MPACBTI-Bet1-mingw-w64-i686-arm-none-eabi.exe`
 * When installing, tick the box to register the path to the ARM compiler as an environment variable in the Windows shell, and then click Finish.  This means programs can call the compiler by using the program name without needing to know the full path to the program on the filesystem.

### Installing CMake

CMake is a set of utilities which help manage the compilation process.  It avoids the need for a very large command to be manually typed when running the compilation process.

* Download the Windows x64 installer from here: https://cmake.org/download/
  * E.g. `cmake-3.25.1-windows-x86_64.msi`
* In the screen before you hit install, tick “Add CMake to the system path for all users”.  Again, this will allow CMake to be run without needing the full path to the program to be used

### Installing the Build Tools for Visual Studio 2022

A few more tools are required for building the programs (e.g. nmake), and these come from a kit called Build Tools for Visual Studio 2022.
* Download the tools from here: https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022
* Run the installer.  When the options appear, in the Workloads tab, you only need to install the “Desktop development with C++” tools.
* In the Individual Components tab, make sure the Windows 10 SDK is checked (the most recent version)
*	 Note:  When I attempted this I got errors later on when trying to compile the example code claiming my machine couldn't find `nmake`.  It turned out that, for some reason, these Build Tools hadn't installed properly.  If you run this step, go to the Start Menu, find the Visual Studio 2022 folder and there are only two programs in it, try this step again.  There should be ~7 entries.

### Installing Python

Apparently, one of the build tools needs Python.  I've no ideas which.

* Download the Windows 64-bit installer for Python from here:	https://www.python.org/downloads/windows/
 * *Getting started with Raspberry Pi Pico* explicitly says that Python 3.10 should be installed, but I've used 3.11.1 without issues.
 * In the installer check the box labelled “Add python.exe to PATH” (again, this means you don't need the full file path to run `python`)

### Installing Git

If you're using this guide then you probably already know what Git is, but if not: it allows you to manage different versions of software as it is developed,  and can integrate with online repositories such as this really obscure one called `GitHub.com`.  There are a few settings which need ticked while installing here:
* Download the installer from https://git-scm.com/download/win
  * E.g. `64-bit Git for Windows Setup`
*	In the "Select Components" window, don’t change anything
*	In the “Choosing the default editor used by Git” window, change the default editor to e.g. Notepad or Notepad++.  
*	In the “Configuring the line ending conversations” window, select “checkout as-is, commit as-is”
*	In the “Configuring the terminal emulator to use with Git Bash” window, select “Use Windows’ default console window”
*	In the “Configuring experimental options” window, tick “Enable experimental support for pseudo consoles”


## Getting the SDK and example code

The Pi Foundation have produced a Source Developer's Kit (SDK) which contains helpful shortcuts and configuration files, which save an enormous amount of work compared to doing everything from first-principles.  There is also a comprehensive folder of examples.

* Open an instance of the Windows Command Prompt (open the Start Menu, type `cmd` and the Command Prompt should appear in the seach list).
*	Navigate to where you want to store everything:
  * If you're unfamiliar with the Windows Command Prompt, the `cd` command is for "Change Directory" (i.e. take you to the place you want to install everything, e.g. `Cd C:\Users\UnfinishedStuff\Desktop`
  * Make a directory caled `pico`: `Mkdir pico`
  * To list the directories in your current location, use the `dir` command.  You should now see the `pico` directory.
  * Enter the `pico` directory using `cd pico`
* Download the pico-sdk into this directory using git: `git clone https://github.com/raspberrypi/pico-sdk.git --branch master`
* Enter the `pico-sdk` directory: `cd pico-sdk`
* Update the submodules of the SDK.  Not everything here was written by the Pi Foundation: TinyUSB is used as well for USB processes on the Pico, and the following command will ensure this is up to date: `git submodule update --init`
* Go back up to the `pico` directory: `cd ..`
* Download the directory of example code:	`git clone https://github.com/raspberrypi/pico-examples.git --branch master`
 * At this point, if you run the `dir` command you should see two directories (`pico-examples` and `pico-sdk`), as well as `.` and `..`


## Doing the initial build of the example code

The example code is provided as raw source code.  In order to use any of them straight away you'll need to build them into files which can be uploaded to the Pico.

* In the start menu, go to Windows > Visual Studio 2022 and run `Developer Command Prompt for VS 2022`
* Type	`setx PICO_SDK_PATH "C:\Users\UnfinishedStuff\Desktop\pico\pico-sdk"`.  You should change the file path to the location of `pico-sdk` on your system.  This adds the location to the PATH on Windows, essentially allowing other programs to find parts of the SDK more easily.
* Close the prompt, and re-open it
* Navigate to the `pico-examples` directory.
* Make a directory called "build": `mkdir build`
* Enter `build`: `cd build`
* Next we run CMake.  This will help to prepare the files for compilation: `cmake -G "NMake Makefiles" ..`
 * If you get an error about not finding `nmake`, make sure the Build Tools for Visual Studio 2022 installed the C++ dev tools properly
* Finally, build the code: `nmake`

## VSCode

This is, strictly speaking, all you need to do to set up the build-chain.  However, rather than work with basic tools it's possible to use an IDE with all the creature comforts.  The Pi Foundation seem to recommend VSCode, so let's use that.

* Download and install VS Code from this website: https://code.visualstudio.com/
* Open a Developer Console Command Prompt for VS2022 again (NOTE: I had to run this next step as an admin, otherwise I got an access denied error)
  * In the Start Menu, go to Visual Studio 2022 > Developer Command Prompt for VS2022 
* In that prompt, type “code”.  This will load VSCode up.
* Click on the Extensions button to the left-hand side ![Extensions icon](https://raw.githubusercontent.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/main/Setting-up-the-Windows-toolchain/Extensions.jpg?token=GHSAT0AAAAAAB4LJFJDOPP5D67ONJSTB7XQY4Y5V6Q), and search for and install `CMake Tools`
* Click on the cogwheel in the bottom left ![Settings Button](https://raw.githubusercontent.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/main/Setting-up-the-Windows-toolchain/Settings.jpg?token=GHSAT0AAAAAAB4LJFJDWDA63GGVCFLVFJPSY4Y5V7Q) and select Settings.  In the window which opens select Extensions and then CMake Tools.  Scroll down to the section titled `Cmake: Configure environment`.  Click the blue `Add New` button, and in the Item box type `PICO_SDK_PATH`, in Value type `..\..\pico-sdk`
* Continue scrolling down to the section called `Cmake:Generator`, and type `NMake Makefiles` into the box.  Close the settings window.
* Go to File > Open Folder, find and select the `pico-examples` folder, and click “Open Folder”.  If needed, click “Yes, I trust the authors” on the security popup.
*	A little message will pop up in the bottom-right of VSCode asking if you’d like to configure the project.  Click Yes, and in the drop-down list which appears at the top, select `GCC [version no.] arm-none-eabi`
*	When the project finishes configuring (see the progress bar in the bottom-right), go to the bottom of VSCode and click on “Build” (white text on a light blue background, it has a little cogwheel beside it).  This will build all of the examples, and will probably take some time.

That's it!  So, to write and build programs:
* In the File Explorer of VSCode, navigate to PICO-EXAMPLES > blink > blink.c.  Change the sleep_ms() times to whatever you like. At the bottom of VSCode, click “all” beside Build, search for `blink`, and select `blink EXECUTABLE`.  Then click Build
* Once finished, in Windows Explorer, navigate to `pico-examples/build/blink`.
* Copy the `blink.UF2` file, this is the compiled program.  Hold down the Boot button of your pico, plug it in, and copy the UF2 file over to the removable drive which should have appeared.  Ta-da!  It should now be running the program.


## Using Picoprobe with VSCode

Manually copying the UF2 files over to the Pico, after having put it into bootloader mode, isn't the most convenient thing, especialy given that the official Pico boards do not come with reset buttons.  It's possible to bypass all of this and create a single-click build-and-upload workflow using Picoprobe.

This part, more than any, seems to be fragile and has a tendency to throw errors.  I've replicated this process twice now, so hopefully I've ironed out most of the kinks and/or found shortcuts for the parts most prone to breaking.

* First, you'll need a second Pico or Pico-like board.  You can use another Pico for this, but other RP2040 boards work tooL I've successfully done this using a [Pimoroni Tiny2040](https://shop.pimoroni.com/products/tiny-2040?variant=39560012300371).
* You'll need to flash it with the Picoprobe firmware.  [You can find it at this page in the "Debugging using another Raspberry Pi Pico" section](https://www.raspberrypi.com/documentation/microcontrollers/raspberry-pi-pico.html#debugging-using-another-raspberry-pi-pico).
* You'll then need to wire your Picoprobe board to the board you want to run code on as per Appendix A in the *Getting Started with Raspberry Pi Pico* document.

<p align="center">
<img src="https://raw.githubusercontent.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/main/Setting-up-the-Windows-toolchain/Picoprobe_wiring.jpg?token=GHSAT0AAAAAAB4LJFJCBK4MQFBNUQG5DYYQY4Y5V7A" width="600" height="379">
</p>
 
* Next, we need a program called OpenOCD.  The *Getting started with Raspberry Pi Pico* doc explains how to compile this.  However, it's a bit fiddly, seems to go wrong quite often, and isn't actually necessary: Shaun Hymel wrote a very nice [Picoprobe with VSCode guide here](https://www.digikey.bg/en/maker/projects/raspberry-pi-pico-and-rp2040-cc-part-2-debugging-with-vs-code/470abc7efb07432b82c95f6f67f184c0), and as part of that he has a pre-compiled version of OpenOCD with some other files which are required.  Scroll down to the *Build OpenOCD* section just below the wiring diagram, look for the Windows guide, and download the executable and .dll files Shaun has available there.  Unzipping these should create two folders, one called `tcl` and once called `src`.  Copy these to a folder called `openocd` in the `pico` directory, alongside `pico-examples` and `pico-sdk`.  
* You then need to add `openOCD` to your `path`.
  * In the Start Menu, type `env`.  An option called `Edit environment variables for your account` should appear.  Click that.
  * In the Environment Variables window which pops up, in the upper section, double-click on the variable called `Path`
  * A new window titled "Edit environmental variable" should appear.  Click New, and then type the path to your `openocd\src` directory.  Hit OK to close the dialogue boxes.
* Open VSCode (remember that you need to go via the Developer Command Prompt).
* Open the Extensions menu, search for and install the `Cortex-Debug` package.
* Pick a program using the Explorer button (I've used `blink`).
* Press `Control-Shift-D` on your keyboard to open up the debug pane.
* Next, we need to sort-of follow section 7.3. *Debugging a Project* in the *Getting Started* guide:
  * In the `pico-examples` directory, make a directory called `.vscode`
  * In that directory, make a file called `launch.json`.  The *Getting Started* PDF has an example of what should be in here in Section 7.3, but I've made a few changes to get it working.
    * [Get the launch.json template from here](https://github.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/blob/main/Setting-up-the-Windows-toolchain/launch.json)
    * `gdbPath` was changed from `gdb-multiarch` to `arm-none-eabi-gdb`
    * In the `configFiles` section, `"interface/raspberrypi-swd.cfg"` was changed to `"interface/picoprobe.cfg"` as we're using a Picoprobe for this, not a Raspberry Pi single board computer.
    * `runToMain` was changed from `true` to `false`.  This seemed to cause the board to stop with a break point at `main()`, rather than run straight into running the program after it was uploaded to the Pico.
    * The `"postRestartCommands"` section was also removed for the same reason.  The comments in the original `launch.json` file says that this section should *prevent* the program stopping at `main()`, but that's not what I've observed it doing.
    * `"searchDir": ["C:/Users/UnfinishedStuff/Desktop/pico/openocd/tcl"]` was added.  You should point this to the `tcl` folder you downloaded and unzipped from Shaun Hymel's tutorial earlier.
  * In the `.vscode` directory, make a file called `settings.json`.  Again, *Getting Started* has an example and this time I've made no changes.  
    * [Get the settings.json template from here](https://github.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/blob/main/Setting-up-the-Windows-toolchain/settings.json)

* Finally, you *may* need to set the USB driver for the Picoprobe device.  The *Getting Started* guide says that this was required in older versions of Picoprobe but isn't needed anymore.  However, when I did this in December 2022 I still needed to set the USB driver.
  * Download Zadig from here: https://zadig.akeo.ie/
  * Run the program.  If you can't see a device called `Picoprobe (Interface 2)` in the drop-down menu, click Options > List All Devices.  Click on `Picoprobe (Interface 2)`.  The driver on the left of the green arrow is what is currently installed, the right shows what Zadig will install.  Change the right-hand box to `WinUSB [version number]` and click Reinstall Driver.  It may take some time to update.

That's all (!).  Now, to compile and upload a program:
* Open VSCode (remember you need to go via the Developer Console Command Prompt for VS2022)
* In the Explorer pane on the left, navigate to the `blink` director, and click on `blink.c`
* Change the program as desired
* Press `Control-Shift-D` to open the debug panel, and click the green triangle at the top.
* You should see the program being compiled in the Output at the bottom, and then VSCode should shift to the Debug Console where you will see OpenOCD using your Picoprobe-flashed Pico (or alternative) board uploading the program to your development Pico board.
* As an afterthought, OK, it's not *really* one-click compile-and-upload.  When you do this, the program will start running on the Pico, and in VSCode a little toolbar will appear with several symbols.  You'll need to press the red square to stop the Pico running the code before uploading the next version, so it really takes two clicks.  But hey, you don't have to hold down the `boot` button while resetting a board which doesn't come with a reset button and then drag-and-drop the UF2 file over.

# Summary

Setting up the toolchain for Pico development, including one-click compiling-and-uploading, is a bit of a journey.  However, with that out of the way, you can now totally use all that spare time you've got left to learn how to program the Pico!
