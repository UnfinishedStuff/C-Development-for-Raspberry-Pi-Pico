# Maker Calendar - Day 2

Day 2 of the Maker Calendar sees us doing more with LEDs, this time using breadboarded components instead of the on-board LED, and using three different colours of LED instead of just the green on-board LED.  The first thing you want to do is wire everything up according to the [*Maker Advent Calendar Day #2: Let’s Get Blinky!* guide page](https://thepihut.com/blogs/raspberry-pi-tutorials/maker-advent-calendar-day-2-let-s-get-blinky).  In short, using the breadboard from Day 1 or a separate additional breadboard, each LED should be connected to GPIO 18, 19, and 20, and then should connect to ground via the 330 Ohm resistors.  There's no reason you *have* to use those three pins, but as they're used in the official guide I'll use them here as well.

## Day2_triple_blink

The first program today is very straightforward: we make the three LEDs turn on for five seconds, and then turn off for five seconds.  The MicroPython code is quite straightforward:

<details> <summary> Day 2 MicroPython for basic LED blinking </summary>
    
```python
from machine import Pin
import time

red = Pin(18, Pin.OUT)
amber = Pin(19, Pin.OUT)
green = Pin(20, Pin.OUT)

red.value(1)
amber.value(1)
green.value(1)

time.sleep(5)

red.value(0)
amber.value(0)
green.value(0)
```
</details>


The C/++ equivalent code takes a few more lines, as each pin needs to be set up initialised separately and then their direction set.  Create a custom program as described [in the Day 1 guide](https://github.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/blob/main/Maker_Advent_Calendar/Day1/README.md), and use the following code:

```C
#include "pico/stdlib.h"

int main() {

    const uint LED_GREEN = 18;
    const uint LED_AMBER = 19;
    const uint LED_RED = 20;

    gpio_init(LED_GREEN);
    gpio_init(LED_AMBER);
    gpio_init(LED_RED);

    gpio_set_dir(LED_GREEN, GPIO_OUT);
    gpio_set_dir(LED_AMBER, GPIO_OUT);
    gpio_set_dir(LED_RED, GPIO_OUT);


    gpio_put(LED_GREEN, 1);
    gpio_put(LED_AMBER, 1);
    gpio_put(LED_RED, 1);
        
    sleep_ms(5000);

    gpio_put(LED_GREEN, 0);
    gpio_put(LED_AMBER, 0);
    gpio_put(LED_RED, 0);
    
}
```

Basically everything here is covered in the Day 1 guide, so we'll move on to the next program.

## Day2_counter_blink

The next MicroPython script on Day 2 introduces `while()` loops.  <details> <summary>Day 2 MicroPython for blinking *n* times</summary>
    
```python
# Imports
from machine import Pin
import time

#Set up our LED names and GPIO pin numbers
red = Pin(18, Pin.OUT)
amber = Pin(19, Pin.OUT)
green = Pin(20, Pin.OUT)

counter = 1 # Set the counter to start at 1

while counter < 11: # While count is less than 11...
    
    print(counter) # Print the current counter
    
    # LEDs all on
    red.value(1)
    amber.value(1)
    green.value(1)
    
    time.sleep(0.5) # Wait half a second
    
    # LEDs all off
    red.value(0)
    amber.value(0)
    green.value(0)
    
    time.sleep(0.5) # Wait half a second
    
    counter += 1 # Add 1 to our counter
 ```
</details> 
    
This script simply blinks the three LEDs on and off while the `counter` variable is less than 11 (i.e. 10 times).  The C/++ isn't so different to what we saw earlier.
    
```C
#include "pico/stdlib.h"

int main() {

    const uint LED_GREEN = 18;
    const uint LED_AMBER = 19;
    const uint LED_RED = 20;

    gpio_init(LED_GREEN);
    gpio_init(LED_AMBER);
    gpio_init(LED_RED);

    gpio_set_dir(LED_GREEN, GPIO_OUT);
    gpio_set_dir(LED_AMBER, GPIO_OUT);
    gpio_set_dir(LED_RED, GPIO_OUT);

    int counter = 1;

    while (counter < 11) {
        gpio_put(LED_GREEN, 1);
        gpio_put(LED_AMBER, 1);
        gpio_put(LED_RED, 1);
        sleep_ms(500);

        gpio_put(LED_GREEN, 0);
        gpio_put(LED_AMBER, 0);
        gpio_put(LED_RED, 0);
        sleep_ms(500);

        counter += 1;
    }
}
```
                        
The main new concepts here are:
* Declaring `int counter = 1;`, a variable of type integer which is initialised to 1.  This stores the current number of times the `while` loop has run, and stops blinking the LEDs when it reaches the 11th time.
* Wrapping the LED blinking in a `while()` loop so that it automatically repeats the instructions in that code while `counter` is less than 11.
* Incrementing the `counter` variable by 1 each time we reach the end of the `while()` block.  You can alternatively use `counter = counter + 1`, but `counter += 1` does essentially the same thing for less typing.
                        
## Day2_sequence_blink
                        
The final program blinks each of the LEDs in sequence 10 times.  ThePiHut clearly say that their MicroPython code in't the most efficient or sensible way to do this, but it's a relatively simple way to do it early on.
                        
<details><summary> Day2 MicroPython for blinking the LEDs in sequence </summary>
    
```python
# Imports
from machine import Pin
import time

#Set up our LED names and GPIO pin numbers
red = Pin(18, Pin.OUT)
amber = Pin(19, Pin.OUT)
green = Pin(20, Pin.OUT)

counter = 1 # Set the counter to 1

while counter < 11: # While count is less than 11
    
    print(counter) # Print the current counter
    
    # Red ON
    red.value(1) # ON
    amber.value(0) # OFF
    green.value(0) # OFF
    
    time.sleep(0.5) # Wait half a second
    
    # Amber ON
    red.value(0) # OFF
    amber.value(1) # ON
    green.value(0) # OFF
    
    time.sleep(0.5) # Wait half a second
    
    # Green ON
    red.value(0) # OFF
    amber.value(0) # OFF
    green.value(1) # ON
    
    time.sleep(0.5) # Wait half a second
    
    counter += 1 # Add 1 to our counter               
```
 </details>
    
The C/++ code is very similar to the previous exercise, except now we only turn on one LED at a time, wait half a second, switch to the next LED, and then do the same for the final LED.  There aren't really any new concepts here.
    
```C
#include "pico/stdlib.h"

int main() {

    const uint LED_GREEN = 18;
    const uint LED_AMBER = 19;
    const uint LED_RED = 20;

    gpio_init(LED_GREEN);
    gpio_init(LED_AMBER);
    gpio_init(LED_RED);

    gpio_set_dir(LED_GREEN, GPIO_OUT);
    gpio_set_dir(LED_AMBER, GPIO_OUT);
    gpio_set_dir(LED_RED, GPIO_OUT);

    int counter = 1;

    while (counter < 11) {
        gpio_put(LED_GREEN, 1);
        gpio_put(LED_AMBER, 0);
        gpio_put(LED_RED, 0);
        sleep_ms(500);

        gpio_put(LED_GREEN, 0);
        gpio_put(LED_AMBER, 1);
        gpio_put(LED_RED, 0);
        sleep_ms(500);

        gpio_put(LED_GREEN, 0);
        gpio_put(LED_AMBER, 0);
        gpio_put(LED_RED, 1);
        sleep_ms(500);

        counter += 1;
    }
}
```
                        
That's the end of Day 2.  We've blinked three LEDs on and then off, then we've blinked all of the LEDs on and off ten times, and then we've blinked each LED on and off in sequence 10 times.  It's quite straightforward, but it's also a good demonstration of different methods of LED control, and also of using through-hole LEDs with a breadboard.
