# Maker Calendar - Day 3

Day 3 of the Maker Calendar introduces push-buttons and using the Pico's GPIO pins as inputs by using five different pieces of demo code.  Roughly, these demonstrate:
* Using a single button to detect inputs, and outputting messages over serial when the button is pressed.
* Checking multiple buttons as inputs in the same program using a slightly clunky `if` approach
* A multi-step example which starts with checking multiple buttons using a slightly more elegant `if elif` approach, followed by using `if elif else` statements to detect if nothing is pressed, and finally compiling all of this into an example which illuminates LEDs depending on which button, or combination of buttons, is pressed.
* An example which introduces `or` statements to give the same output regardless of which button is pressed
* Finally, there is an example of using buttons to toggle outputs, rather than simply give outputs when the button is held down.

## Initial setup

First, you need to connect the buttons to the Pico H as shown in ThePiHut's diagram on the [*Maker Advent Calendar Day #3: Bashing Buttons!* page](https://thepihut.com/blogs/raspberry-pi-tutorials/maker-advent-calendar-day-3-bashing-buttons).
* The Red LED should be connected to physical pin 24/GPIO18, with a 330 ohm resistor to ground.
* The Yellow LED should be connected to physical pin 25/GPIO19, with a 330 ohm resistor to ground.
* The Green LED should be connected to physical pin 26/GPIO20, with a 330 ohm resistor to ground.

The buttons should be plugged into the mini-breadboard which came with this day's box.  Look carefully at the buttons, and make sure you get them plugged in with the correct orientation.

* One button should be plugged into physical pin 5/GPIO3
* One button should be plugged into physical pin 11/GPIO8
* One button should be plugged into physical pin 17/GPIO13

The other pin of each button should be connected to the Pico's 3.3V pin, which is physical pin 36.

## Activity 1: Button Test Program

This activity checks constantly to see if a button is pressed, and if it is, it prints a message on the serial bus to confirm this.  The Python code for this activity is quite straightforward.  It introduces setting a pin as an input, and setting the pull-down resistor on the pin. If you're not familiar with pull-downs, they're a resistor which connects the pin to ground.  The idea is that electronic wiring can, theoretically, act as an antenna, picking up radio signals and causing the voltage on the wire to fluctuation occasionally.  This can cause the reading on the pin to seem high even when the button isn't pressed.  The pull-down resistor to ground gives somewhere for a low amount of current/voltage to esacape, holding the pin low.  When the button is pressed most of the current can't escape to ground because the resistor restricts it, and so the pin goes high.

<details><summary>Python Code for the Button Test Program</summary>
  
```python
# Imports
from machine import Pin
import time

# Set up our button name and GPIO pin number
# Also set the pin as an input and use a pull down
button1 = Pin(13, Pin.IN, Pin.PULL_DOWN)

while True: # Loop forever

    time.sleep(0.2) # Short delay
    
    if button1.value() == 1: #If button 1 is pressed
        print("Button 1 pressed")
```
  
</details>

The C/++ code is a good bit more complicated than the Python, mostly because where Python can set a pin as an input and set the pull-down resistor in a single function call, C/++ does most of these thing in separate function calls.

```C
#include <stdio.h>
#include "pico/stdlib.h"

int main() {

    /* Initialise the serial/UART bus */
    stdio_init_all();
    
    /*set up the button pin as an input with a pull-down resistor*/
    const uint BUTTON_PIN = 13;
    gpio_init(BUTTON_PIN);
    gpio_set_dir(BUTTON_PIN, GPIO_IN);
    gpio_pull_down(BUTTON_PIN);

    while (true) {
        sleep_ms(200);
        if (gpio_get(BUTTON_PIN) == 1){
            printf("Button 1 is pressed!\n");
        }
    }
}
```

* `stdio_init_all()` sets up common IO interfaces, including the UART/serial bus.
* The next few lines set `BUTTON_PIN` to be 13, then initialises that GPIO pin for standard I/O functions, and then sets it as an input pin, and then activates the pull-down resistor on that pin.
* Finally, we get into a `while` loop which constantly checks to see if the button is pressed, and if so, it prints a message on the serial bus.

Upload the code, connect to the Pico using PuTTY (see [Day 2](https://github.com/UnfinishedStuff/C-Development-for-Raspberry-Pi-Pico/tree/main/Maker_Advent_Calendar/Day2) for instructions on doing this), and every time you push the button connected to GPIO13 you should see `Button 1 is pressed!` appear in the terminal.

## Activity 2: Multiple button inputs

Activity 1 only works with a single button.  Activity 2 does more or less the exact same thing, but uses multiple `if` statements to check if any of the three buttons are pressed.  This is not the most elegant way to do this, but it introduces `if` statements so that the more elegant approach can be taken later. 

<details><summary>Python code for Multiple Button Inputs</summary>
  
```python
# Imports
from machine import Pin
import time

# Set up our button names and GPIO pin numbers
# Also set pins as inputs and use pull downs
button1 = Pin(13, Pin.IN, Pin.PULL_DOWN)
button2 = Pin(8, Pin.IN, Pin.PULL_DOWN)
button3 = Pin(3, Pin.IN, Pin.PULL_DOWN)

while True: # Loop forever
    
    time.sleep(0.2) # Short Delay
        
    if button1.value() == 1: # If button 1 is pressed
        
        print("Button 1 pressed")
        
    if button2.value() == 1: # If button 2 is pressed
        
        print("Button 2 pressed")
        
    if button3.value() == 1: # If button 3 is pressed
        
        print("Button 3 pressed")
```
  
</details>

The C/++ equivalent code looks as follows:

```C
#include <stdio.h>
#include "pico/stdlib.h"

int main() {

    /* Initialise the serial/UART bus */
    stdio_init_all();
    
    /*set up the button pins */
    const uint BUTTON_ONE = 13;
    gpio_init(BUTTON_ONE);
    gpio_set_dir(BUTTON_ONE, GPIO_IN);
    gpio_pull_down(BUTTON_ONE);

    const uint BUTTON_TWO = 8;
    gpio_init(BUTTON_TWO);
    gpio_set_dir(BUTTON_TWO, GPIO_IN);
    gpio_pull_down(BUTTON_TWO);

    const uint BUTTON_THREE = 3;
    gpio_init(BUTTON_THREE);
    gpio_set_dir(BUTTON_THREE, GPIO_IN);
    gpio_pull_down(BUTTON_THREE);

    while (true) {
        sleep_ms(200);
        if (gpio_get(BUTTON_ONE) == 1){
            printf("Button one is pressed!\n");
        }
        if (gpio_get(BUTTON_TWO) == 1){
            printf("Button two is pressed!\n");
        }
        if (gpio_get(BUTTON_THREE) == 1){
            printf("Button three is pressed!\n");
        }
    }
}
```

Essentially, this does the exact same thing as Activity 1, but triplicates everything to set up and check the additional pins one after another.  This creates a situation where pressing any combination of buttons will lead to each of their messages being printed, in the order of `Button 1 > Button 2 > Button 3` as that's the order in which the code checks them.

## Activity 3: Multiple Button Inputs with elif and else

<details><summary> Python code for Multiple Button Inputs with elif and else</summary>

The first stage of Activity 3 does essentially the same as Activity 2, except it introduces the `else if` keyword.  Essentially, when the first clause in the `if` block is not true, the program will then check the next `else if` block to see if that is true.  If not, the program will continue on to subsequent `else if` blocks until one is true or all have been checked and found to be `false`.
  
```python
# Imports
from machine import Pin
import time

# Set up our button names and GPIO pin numbers
# Also set pins as inputs and use pull downs
button1 = Pin(13, Pin.IN, Pin.PULL_DOWN)
button2 = Pin(8, Pin.IN, Pin.PULL_DOWN)
button3 = Pin(3, Pin.IN, Pin.PULL_DOWN)

while True: # Loop forever
    
    time.sleep(0.2) # Short Delay
        
    if button1.value() == 1: # If button 1 is pressed
        
        print("Button 1 pressed")
        
    elif button2.value() == 1: # If button 2 is pressed
        
        print("Button 2 pressed")
        
    elif button3.value() == 1: # If button 3 is pressed
        
        print("Button 3 pressed")
```


</details>

The C/++ equivalent to the Python code is again much longer than this:

```C
#include <stdio.h>
#include "pico/stdlib.h"

int main() {
    
    /* Initialise the serial/UART bus */
    stdio_init_all();

    /*set up the button pins */
    const uint BUTTON_ONE = 13;
    gpio_init(BUTTON_ONE);
    gpio_set_dir(BUTTON_ONE, GPIO_IN);
    gpio_pull_down(BUTTON_ONE);

    const uint BUTTON_TWO = 13;
    gpio_init(BUTTON_TWO);
    gpio_set_dir(BUTTON_TWO, GPIO_IN);
    gpio_pull_down(BUTTON_TWO);

    const uint BUTTON_THREE = 13;
    gpio_init(BUTTON_THREE);
    gpio_set_dir(BUTTON_THREE, GPIO_IN);
    gpio_pull_down(BUTTON_THREE);

    while (true) {
        sleep_ms(200);
        if (gpio_get(BUTTON_ONE) == 1){
            printf("Button one is pressed!\n");
        }
        else if (gpio_get(BUTTON_TWO) == 1){
            printf("Button two is pressed!\n");
        }
        else if (gpio_get(BUTTON_THREE) == 1){
            printf("Button three is pressed!\n");
        }
    }
}
```

* Rather than use separate `if` statements to check each button individually, we're now using `if / else if` statements to do this.  One side effect here is that if you press multiple buttons at the same time, you will only get the message assigned to the first pressed button which is checked.  In comparison, Activity 2 with individual `if` statements would print a message for every button pressed, in the order in which the code checks the buttons.

Activity 3 then expands into adding an `else` clause telling the program what to do if none of the buttons were pressed.  The Python code for this really just needs a couple of extra lines:

<details><summary>Activity 3 with else statements</summary>
  
```python
# Imports
from machine import Pin
import time

# Set up our button names and GPIO pin numbers
# Also set pins as inputs and use pull downs
button1 = Pin(13, Pin.IN, Pin.PULL_DOWN)
button2 = Pin(8, Pin.IN, Pin.PULL_DOWN)

while True: # Loop forever
    
    time.sleep(0.2) # Short Delay
        
    if button1.value() == 1: # If button 1 is pressed
        
        print("Button 1 pressed")
        
    elif button2.value() == 1: # If button 2 is pressed
        
        print("Button 2 pressed")
        
    else: # If no buttons are being pressed
        
        print("No button presses")
  ```
  
  </details>
  
This Activity drops down to just two buttons to illustrate the point here.

```C
#include <stdio.h>
#include "pico/stdlib.h"

int main() {

    /* Initialise the serial/UART bus */
    stdio_init_all();

    /* Set up the button pins */
    const uint BUTTON_ONE = 13;
    gpio_init(BUTTON_ONE);
    gpio_set_dir(BUTTON_ONE, GPIO_IN);
    gpio_pull_down(BUTTON_ONE);

    const uint BUTTON_TWO = 8;
    gpio_init(BUTTON_TWO);
    gpio_set_dir(BUTTON_TWO, GPIO_IN);
    gpio_pull_down(BUTTON_TWO);


    while (true) {
        sleep_ms(200);

        if (gpio_get(BUTTON_ONE) == 1){
            printf("Button one is pressed!\n");
        }
        else if (gpio_get(BUTTON_TWO) == 1){
            printf("Button two is pressed!\n");
        }
        else{
            printf("No buttons are pressed.\n");
        }
    }
}
```

This code should continuously print "No button is pressed." to your PuTTY terminal unless you press a button.

Finally, Activity 3 branches out to integrating LEDs with the buttons.  If no buttons are pressed, the red LED is lit.  If button 1 is pressed then the amber LED is lit, while if both button 1 and button 2 are pressed then the green LED is lit.  To achieve this, it introduces the `&&` operator, which checks to see if two specific buttons are pressed at the same time.  This is often known as the `AND` operator, because it only outputs `true` if both input A *and* input B are `true`.

<details><summary>Python code for Activity 3 with AND clauses and LEDs</summary>
  
```python
# Imports
from machine import Pin
import time

# Set up our button names and GPIO pin numbers
# Also set pins as inputs and use pull downs
button1 = Pin(13, Pin.IN, Pin.PULL_DOWN)
button2 = Pin(8, Pin.IN, Pin.PULL_DOWN)
button3 = Pin(3, Pin.IN, Pin.PULL_DOWN)

# Set up our LED names and GPIO pin numbers
red = Pin(18, Pin.OUT)
amber = Pin(19, Pin.OUT)
green = Pin(20, Pin.OUT)

while True: # Loop forever
    
    time.sleep(0.2) # Short Delay
        
    if button1.value() == 1 and button2.value() == 1: # If button 1 and 2 are pressed at the same
        
        print("Buttons 1 and 2 pressed")
        green.value(1) # green LED on
        
    else if button1.value() == 1: # If button 1 is pressed
        
        print("Button 1 pressed")
        amber.value(1) # amber LED on
        
    else: # If no buttons are being pressed
        
        red.value(1) # red LED on
        amber.value(0) # amber LED off
        green.value(0) # green LED off
```
  
</details>

Now, if you look at the Python code from ThePiHut's guide, I'm pretty sure it's missing a few things.  If no buttons are pressed, the red LED is turned on and the green and amber LEDs are turned off.  However, if you then press button 1, the green LED is turned on, but the red LED isn't turned off.  In fact, there's no code here at all which turns the red LED off.  So, I've modified the code below to add code to the `if` and `else if` sections to turn off whichever LEDs aren't turned on, just to make sure that only one LED is on at a time.

```C
#include <stdio.h>
#include "pico/stdlib.h"

int main() {

    /* Initialise the serial/UART bus */
    stdio_init_all();

    /* Set up the button pins */
    const uint BUTTON_ONE = 13;
    gpio_init(BUTTON_ONE);
    gpio_set_dir(BUTTON_ONE, GPIO_IN);
    gpio_pull_down(BUTTON_ONE);

    const uint BUTTON_TWO = 8;
    gpio_init(BUTTON_TWO);
    gpio_set_dir(BUTTON_TWO, GPIO_IN);
    gpio_pull_down(BUTTON_TWO);

    const uint BUTTON_THREE = 3;
    gpio_init(BUTTON_THREE);
    gpio_set_dir(BUTTON_THREE, GPIO_IN);
    gpio_pull_down(BUTTON_THREE);

    /* Set up the LEDs */
    const uint LED_GREEN = 18;
    const uint LED_AMBER = 19;
    const uint LED_RED = 20;

    gpio_init(LED_GREEN);
    gpio_init(LED_AMBER);
    gpio_init(LED_RED);

    gpio_set_dir(LED_GREEN, GPIO_OUT);
    gpio_set_dir(LED_AMBER, GPIO_OUT);
    gpio_set_dir(LED_RED, GPIO_OUT);

    while (true) {
        sleep_ms(200);

        if (gpio_get(BUTTON_ONE)==1 && gpio_get(BUTTON_TWO)==1){
            printf("Buttons one and two are pressed!\n");
            gpio_put(LED_GREEN, 1);
            gpio_put(LED_RED, 0);
            gpio_put(LED_AMBER, 0);
        }
        else if (gpio_get(BUTTON_ONE)){
            printf("Button one is pressed!\n");
            gpio_put(LED_AMBER, 1);
            gpio_put(LED_GREEN, 0);
            gpio_put(LED_RED, 0);
        }
        else{
            printf("No button presses\n");
            gpio_put(LED_RED, 1);
            gpio_put(LED_AMBER, 0);
            gpio_put(LED_GREEN, 0);
        }
    }
}
```

## Activity 4: This button or that button?

The fourth activity introduces `or`-ing inputs together to trigger a set response if one *or* another button is pressed.  While Python uses the `OR` keyword for that, C/++ uses `||` for this instead.

<details><summary>Python code for This button or that button?</summary>
  
```python
# Imports
from machine import Pin
import time

# Set up our button names and GPIO pin numbers
# Also set pins as inputs and use pull downs
button1 = Pin(13, Pin.IN, Pin.PULL_DOWN)
button2 = Pin(8, Pin.IN, Pin.PULL_DOWN)

# Set up our LED names and GPIO pin numbers
green = Pin(20, Pin.OUT)

while True: # Loop forever
    
    time.sleep(0.2) # Short Delay
        
    if button1.value() == 1 or button2.value() == 1: # If button 1 OR button 2 is pressed
        
        print("Button 1 or 2 pressed")
        
        green.value(1) # green LED on
        time.sleep(2)
        green.value(0) # green LED off
```
  
</details>

This activity only uses two buttons and one LED.

```C
#include <stdio.h>
#include "pico/stdlib.h"

int main() {

    /* Initialise the serial/UART bus */
    stdio_init_all();

    /* Set up the button pins */
    const uint BUTTON_ONE = 13;
    gpio_init(BUTTON_ONE);
    gpio_set_dir(BUTTON_ONE, GPIO_IN);
    gpio_pull_down(BUTTON_ONE);

    const uint BUTTON_TWO = 8;
    gpio_init(BUTTON_TWO);
    gpio_set_dir(BUTTON_TWO, GPIO_IN);
    gpio_pull_down(BUTTON_TWO);

    /* Set up the LEDs */
    const uint LED_GREEN = 18;
    gpio_init(LED_GREEN);
    gpio_set_dir(LED_GREEN, GPIO_OUT);

    while (true) {
        sleep_ms(200);

        if ((gpio_get(BUTTON_ONE) == 1) || (gpio_get(BUTTON_TWO) == 1)){
            printf("Button one or two pressed!\n");
            gpio_put(LED_GREEN, 1);
            sleep_ms(2000);
            gpio_put(LED_GREEN, 0);
        }
    }
}
```

## Activity 5: Toggling with buttons

The final exercise is about detecting a button press and then flipping the state of an LED, turning it on or off respectively.  The LED will then hold that state until the button is pressed again.

<details><summary>Python code for Toggling with buttons</summary>
  
```python
# Imports
from machine import Pin
import time

# Set up our button names and GPIO pin numbers
# Also set pins as inputs and use pull downs
button1 = Pin(13, Pin.IN, Pin.PULL_DOWN)

# Set up our LED names and GPIO pin numbers
red = Pin(18, Pin.OUT)

while True: # Loop forever

    time.sleep(0.5) # Short delay
            
    if button1.value() == 1: #If button 1 is pressed
        
        print("Button 1 pressed")
        
        red.toggle() # Toggle Red LED on/off
```
  
</details>

This creates a slight problem for C/++ code: the Python example uses the `toggle()` function to set the LED state to whatever it currently isn't.  However, there doesn't seem to be an equivalent in the C/++ SDK.  However, what we can do is use the `gpio_get_out_level` function to see what state the LED is currently in, invert this using the NOT operator (`!`), and then set the LED to this value.  This will set the LED to whichever state it isn't currently in.

```C
#include <stdio.h>
#include "pico/stdlib.h"

int main() {

    /* Initialise the serial/UART bus */
    stdio_init_all();

    /* Set up the button pin */
    const uint BUTTON_ONE = 13;
    gpio_init(BUTTON_ONE);
    gpio_set_dir(BUTTON_ONE, GPIO_IN);
    gpio_pull_down(BUTTON_ONE);

    /* Set up the LED */
    const uint LED_RED = 18;
    gpio_init(LED_RED);
    gpio_set_dir(LED_RED, GPIO_OUT);

    while (true) {
        sleep_ms(500);

        if (gpio_get(BUTTON_ONE)==1){
            printf("Button one pressed!\n");
            gpio_put(LED_RED, !gpio_get_out_level(LED_RED));

        }
    }
}
```

And that's it for Day 3.  We've looked at how to set up a button as an input and read whether or not it is being pressed; we've looked at several approaches to detecting inputs from multiple buttons, as well as how to `AND` as well as `OR` them together to detect inputs from multiple buttons in relation to each other; and we've looked at how to toggle buttons.
