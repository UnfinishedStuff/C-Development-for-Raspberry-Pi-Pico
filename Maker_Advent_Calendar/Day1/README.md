### Maker Calendar – Day 1

Most of [the Official Day 1 Advent Calendar guide](https://thepihut.com/blogs/raspberry-pi-tutorials/maker-advent-calendar-day-1-getting-started) is about setting up Thonny for using MicroPython with the included Pico H dev board.  The C/++ equivalent is quite an involved process, and it's also something which isn’t exclusive to the calendar, so [I’ve detailed setting up the C/++ toolchain on Windows here instead](https://github.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/tree/main/Setting-up-the-Windows-toolchain).  This means that today’s project in C is more simple: we're going to create a custom program in VSCode, run a basic LED blink program, and then run the classic `Hello World!` example over serial.


## Creating a new program
This is something we’ll need to do for every new project.  It’s not as straightforward as just creating a new file, as the compilation process for C/++ involves a few other tools which need to know where your new program is.  In particular, we need a `CMakeLists.txt` file to tell the compiler how we want different things to be linked together.
* Start by opening VSCode (by typing `code` into the Developer Command Prompt for VS2022).  Open the explorer by using the icon in the top left, and open the `pico-examples` directory.
*	In the explorer pane which opens, create a new directory.  I called mine `day1_blink`.
*	Open one of the other programs (e.g., `blink` which came with the `pico-examples` files) and right-click the `CMakeLists.txt` file there, and click Copy
*	Right-click on `day1_blink`, and click Paste.  This should paste a copy of `CMakeLists.txt` into your new project directory.
*	If you now click on that `CMakeLists.txt` file to edit it, you’ll see that there are five or so points where it mentions the original `blink` program.  Change all of those to the name for your project (`day1_blink` in this case).  Save the file.
*	Next, navigate to the `CMakeLists.txt` file which is in the `pico-examples` directory and click it (this one should have a good bit more in it than the file in your project, and should start with `cmake_minimum_required(VERSION X.XX)`).  Add a new line to the bottom with the text `add_subdirectory(day1_blink)`.  This tells the compiler to include your new project in the list of things to build.
*	Go back to your project, right-click on the project title, and select `New File…`.  Call the file `day1_blink.c`.  This will contain the source code for your project.  Right, now we can actually start programming.

## Day1_blink
First, we're going to start by toggling the LED.  The micropython for day 1 is very simple:

```python
from machine import Pin
onboardLED = Pin(25, Pin.OUT)
onboardLED.value(1)
```
The first line imports the specific code for handling GPIO pins.  Line 2 sets the on-board LED pin (pin 25) as an output pin, and the third line turns the LED on.  Which is a bit boring really, so we'll actually make the LED blink.  The C/++ code is a bit more complex.

```C
#include "pico/stdlib.h"

int main() {
    /* Set up the LED pin */
    const uint LED_PIN = PICO_DEFAULT_LED_PIN;
    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
    
    /* Blink the LED */
    while (true) {
        gpio_put(LED_PIN, 1);
        sleep_ms(100);
        gpio_put(LED_PIN, 0);
        sleep_ms(1000);
    }
}
```
This repo is not meant to be an introduction to the C/++ language.  However, just for my own reference, I've made some quick notes on what this code does:
* The first line imports `stdlib.h`, which contains core functions and shortcuts for a lot of common features on the Pico’s RP2040 chip, including pin control.  This makes it far quicker to get hardware functions working, compared to manually setting things up at the lowest level possible.
* All programs in C/++ must have a function called `main()`, and the program normally starts here first.  While these programs can also have other functions, we’re only going to use `main` here.
*	The next line (`const uint LED_PIN = PICO_DEFAULT_LED_PIN;`) defines a constant unsigned integer called `LED_PIN` and sets it to `PICO_DEFAULT_LED_PIN`, one of the shortcuts which points to the GPIO pin for the on-board LED.  You can also manually set this to the pin number for the on-board LED, but this is different between the Pico (pin 25) and Pico W (actually one of the pins on the WiFi chip, and so it requires a module for interacting with the WiFi chip to blink the LED).
*	Next you need to initialise the pin, which is what `gpio_init(LED_PIN)` does.  This sets the pin to be active as a generic IO pin.  The line after this sets the pin as an output pin.
*	Finally, we come into a `while` loop.  This puts “1” onto the LED pin, turning it on.  The chip then sleeps for 1000 milliseconds (1 second), and then we put “0” on the LED pin (turning it off).  We then wait for another 1,000 milliseconds, and repeat constantly.
*	Once you’ve got the code ready, there are two things you can do.  First, you can try to only compile the program without uploading it to the Pico, useful if you want to see if the code throws any errors without uploading it.  You can do this using the “Compile Active File” button in the top right ![Compile Active File Icon](https://raw.githubusercontent.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/main/Maker_Advent_Calendar/Day1/CompileActiveFile.jpg).  By default this doesn’t seem to have a keyboard shortcut, but you can add one by going to `File` > `Preferences` > `Keyboard Shortcuts`.  Alternatively, if you're using a second Pico board with the Picoprobe firmware to flash your dev board, on the left, click the Debug icon (the triangle with the little bug), and then the Start Debugging (green Triangle) at the top of the debug pane.  This should compile and then upload the file to your Pico, and you will see the LED blinking.  Either way, make sure the correct program is selected in the toolbar at the bottom of VSCode (the last text in the bar is the "Select Target to Build" selector).

## Day1_hello_world
Next, we'll send the message "Hello world!" across the serial bus.  To receive this, you'll need the full Picoprobe wiring shown in the [Setting Up The Windows Toolchain](https://github.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/tree/main/Setting-up-the-Windows-toolchain) guide, and you'll need to have [PuTTY](https://putty.org/) installed as well in order to receive the data.
* Create a new program as before.  I called this one `day1_hello_world`.
* In `day1_hello_world.c`, add the following code:

```C
#include <stdio.h>
#include "pico/stdlib.h"

int main() {
    stdio_init_all();

    while (true){
        printf("Hello world!\n");
        sleep_ms(1000);
    }
    return 0;
}
```

* The first two lines import modules which are needed for general functions, as well as serial comms.  
* The `main()` function starts by initialising all IO pins, which is a shortcut to preparing the serial pins for data transfer.  
* Finally, there's a `while()` loop which prints "Hello world!" on the serial bus, and then repeats this every 1 second.
* Now, before we can build this we need to make a few changes to the `CMakeLists.txt` file for this program (*not* the general `CMakeLists.txt` file in the `pico-examples` directory).  Open that from the file explorer and add these to the bottom:

```
pico_enable_stdio_usb(day1_hello_world 0)
pico_enable_stdio_uart(day1_hello_world 1)
```

* The Pico actually has two potential serial output modes.  The first is via USB CDC, i.e. sending serial data directly over the USB connection.  However, if you've wired the whole thing up with a second Pico board programmed with Picoprobe, then you want to use the second option, which is to output data over the UART pins.  These are the orange and yellow wires in [the Picoprobe wiring diagram](https://raw.githubusercontent.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/main/Setting-up-the-Windows-toolchain/Picoprobe_wiring.jpg?token=GHSAT0AAAAAAB4LJFJD6JNAZNSVUUBV2IMEY4Y573A), and so the serial data is sent over the Rx/Tx wires to your Picoprobe, which then sends the data to your PC over USB.
* Compile and upload the program to your Pico as before.
* Now, to receive this data we need to open and configure PuTTY.  Open the Start Menu and open the Device Manager (you'll probably have to type the program name to search for it, it isn't normally in the Start Menu).  In the program which opens, click the arrow beside the `Ports (COM&LPT)` option.  In the drop-down list which appears, there should be an entry called `Picoprobe (Interface 0) (COMXX)`, e.g. `COM11`, where `XX` is a number.  Make a note of that number.
* Open PuTTY.  Change the Connection Type to `Serial`, change the Speed to `115200`, and change the Serial Line to the `COM` port you found in the Device Mnager.  Then click the Open button at the bottom.
* This should open a black serial terminal which will receive serial mesages from your Pico: you should see `Hello world!` being printed on the screen over and over again.

That's it, we've looked at how to create a custom program, how to blink the on-board LED, and how to receive serial data.  This is a little more than the official Day 1 guide does, but it sets things up nicely for Day 2.
