## Write code to calibrate the ladder DAC

We need to calibrate the ladder to bring the output voltage of the air quality sensor down to just below the threshold of the trigger pin, so that it reads LOW. That way, any increase in output voltage caused by a fart will tip the trigger from LOW into HIGH, which we can easily detect in code.

Enter the following command:

```bash
nano farts.py
```

Remove the three lines below. These are no longer needed, but will be used again later.

```python
mixer.music.play(-1) # -1 to loop the sound
time.sleep(10) #let it play for 10 seconds
mixer.music.stop()
```

Next, we need to import the GPIO library. This allows us to configure the GPIO pins as desired.
Add `RPi.GPIO as GPIO` at the top of the file as shown below:

```python
#!/usr/bin/python
import time, RPi.GPIO as GPIO

from pygame import mixer
mixer.init()
mixer.music.load("evacuate.mp3")
```

We now need to set up the trigger pin (GPIO 4) as an input. We can create a variable called `TRIGGER` which will hold the number 4; we can then refer to this variable everywhere else in the code when we access the trigger pin.

```python
TRIGGER = 4

GPIO.setmode(GPIO.BCM) #use BCM pin layout
GPIO.setup(TRIGGER, GPIO.IN)
```

The `GPIO.setmode` line above configures the GPIO library to use the Broadcom pin layout, which matches the graphic in the breadboard diagrams above. Then the `GPIO.setup` line actually configures the trigger pin to be an `INPUT`; this allows us to read the state of it to test if it is HIGH or LOW. The next task is for us to loop through all the possible on/off combinations for the ladder, 0 to 15 in binary, and locate the threshold of this trigger pin.

So the algorithm will be something like:

- Loop for each number between 0 to 15
    - Configure the laddder to the binary form of the number
        - if GPIO 4 is LOW
            - Exit loop
- Wait for GPIO 4 to go HIGH
    - Sound fart alarm

If we break this task down there are three things we need to do:

- Turn resistor pins on and off.
- Set all resistor pins in the ladder to represent a binary value.
- Loop between 0 and 15 to calibrate the ladder DAC.

### Turn resistor pins on and off

In order to switch a resistor on or off we just use the `GPIO.setup` command with different parameters. If the resistor/pin is *on* we configure the pin to use `OUTPUT` mode and drive it LOW. This will connect the pin to ground and some voltage will then flow from the sensor output through to ground. If the sensor is *off* we configure the pin to use `INPUT` mode, which means the pin is not connected to anything and nothing will flow through it.

We can define a function called `set_pin` as follows to do this. Manually enter or copy and paste this into your code:

```python
def set_pin(pin, ison):
    if ison:
        GPIO.setup(pin, GPIO.OUT)
        GPIO.output(pin, GPIO.LOW)
    else:
        GPIO.setup(pin, GPIO.IN)
```

The function takes two parameters: `pin` and `ison`. The `pin` parameter will be the GPIO pin number, and `ison` will be a boolean (True/False) variable to say whether the resistor/pin is in an on or off state. We then just use an `if` statement and call the appropriate GPIO commands passing in `pin`. When we call the function we can write `set_pin(18, True)`, for example.

### Set all resistor pins in the ladder to represent a binary value

Next, we need a function to call `set_pin` multiple times for each of the ladder GPIO pins (17, 18, 22 and 23).
Since we're setting the entire ladder DAC we can define a function called `set_dac` to do this; enter or copy and paste this into your code.

```python
def set_dac(bitwise):
    set_pin(17, bitwise & 1 == 1)
    set_pin(18, bitwise & 2 == 2)
    set_pin(22, bitwise & 4 == 4)
    set_pin(23, bitwise & 8 == 8)
```

The function takes one parameter called `bitwise`. Because each resistor/pin represents a binary bit position, we now need to call the `set_pin` function accordingly, based on whether or not the corresponding binary bit is set to `1` in `bitwise`.

8's MSB | 4's | 2's | 1's LSB
--- | --- | --- | ---
GPIO 23 | GPIO 22 | GPIO 18 | GPIO 17
4.7k | 10k | 22k | 47k

For example, if `bitwise` was 9, this would be `1001` in binary. So, working from LSB to MSB (right to left):

- `1` = `GPIO 17` ON
- `0` = `GPIO 18` OFF
- `0` = `GPIO 22` OFF
- `1` = `GPIO 23` ON

If we do a logical [and operation](http://en.wikipedia.org/wiki/Bitwise_operation#AND) between `bitwise` and the value of the bit position, we can test to see if a bit is set to `1` or not. It works by comparing the bit positions of both numbers: if they are `1` in both then the answer will also be `1`, otherwise it's `0`.

For example, testing the number 5 for the first bit (LSB):

```
    0101 (decimal 5)
AND 0001 (decimal 1)
  = 0001 (decimal 1)
```

Another example, testing for the rightmost bit (MSB):

```
    1101 (decimal 13)
AND 1000 (decimal 8)
  = 1000 (decimal 8)
```

We can use [bitwise operators](https://wiki.python.org/moin/BitwiseOperators#The_Operators:) to do this in our code. So if we use `bitwise & x == x` (bitwise and x is equal to x) this will give a boolean (True/False) result, depending on whether the bit `x` is set or not. Take another look at the `set_dac` function now and you'll notice that we use this trick to call the `set_pin` function multiple times for each bit value/position. This ensures that the `ison` parameter inside the `set_pin` function will always be True or False.

When we call the `set_dac` function we can write `set_dac(x)`, where x is a number between 0 and 15.

### Loop between 0 and 15 to calibrate the ladder DAC

Now that we have the ability to configure the DAC, we need some code that will loop from 0 to 15 calling `set_dac` and testing the input trigger pin to find the HIGH to LOW threshold.

Let's call this function `calibrate`: enter or copy and paste this into your code.

```python
def calibrate(trace = False, sleep_time = 0):
    result = -1
    for i in range(0, 16):
        set_dac(i)
        if trace:
            print i, "{0:b}".format(i) #binary format
        time.sleep(sleep_time)
        if not GPIO.input(TRIGGER):
            result = i
            break

    return result
```

The function takes two parameters: `trace` and `sleep_time`. The `trace` parameter is a boolean (True/False) value to say whether we want to trace/print the ladder steps we're working through. The `sleep_time` parameter specifies how long the code should pause for between configuring the DAC and taking the measurement from the trigger pin. You'll notice that they are both *optional* parameters since we provide a default value for them.

Inside the function we define a variable called `result`. This will be returned at the end of the function, so that the main program can know which step on the ladder (between 0 and 15) the threshold was found. Next is a `for` loop for the range 0 to 16. Why 16? In Python you specify the number to start at and the number to stop at, so stopping at 16 means 15 will be the last time around the loop. The syntax `for i in range` means that the variable `i` changes each time around the loop.

Inside the loop we pass `i` into `set_dac`; we then print out what ladder step we're on (in both decimal and binary), sleep as necessary and then test the value of the trigger pin using the `GPIO.input` command. This command will return `1` for HIGH or `0` for LOW. We can then use the `if not` syntax to test whether `0` was returned. If we get `0` then we've found the HIGH to LOW threshold step, so we set `result = i` and `break` out of the loop.

It is possible that the air quality could be so bad that the threshold is never found: in this case, `result` will still be `-1` and will end up being the return value of the function. This allows the main code to know if the calibration was successful or not. When the calibration is unsuccessful you can only wait for the air to clear and try again.

Now let's program the main code that will use the `calibrate` function!

Enter or copy and paste this code at the very bottom of your file:

```python
fresh_air = calibrate(trace = True, sleep_time = 0.5)

if fresh_air != -1:
    print "Calibrated to", fresh_air
else:
    print "Could not calibrate"
```

This calls the `calibrate` function and stores the result in a variable called `fresh_air`; under normal conditions this should be somewhere below 6 or 7. We can then use an `if` statement to test if `fresh_air` is equal to `-1` or not. So if not equal `!=` to `-1` then we have a successful calibration, otherwise it failed.

Let's run the code. Press `Ctrl - O` then `Enter` to save, followed by `Ctrl - X` to quit.

GPIO functions require root access on your Pi, so from now on you must use the `sudo` command to run your code. If you don't use `sudo` you'll see the following error: `No access to dev/mem. Try running as root!`

```bash
sudo ./farts.py
```

The output should look something like this:

```
0 0
1 1
2 10
3 11
4 100
Calibrated to 4
```

It's important to remember that the heater in the air quality sensor needs to have warmed up before this will work. So if you turn off your Pi now and come back to this later, you may need to wait a few minutes for the heater to warm up before you can get a successful calibration.

If it goes all the way up to 15 and you see the `Could not calibrate` message, then just wait a minute or two and run the code again. Depending on the background air quality, temperature and humidity it can take anywhere up to 20 minutes to calibrate. If you experience problems in this area see the Troubleshooting section at the end.

### Optional activity using a multimeter

If you connect a multimeter to the breadboard as shown, and configure it to display voltage, you will be able to observe the voltage level changing in real time as the calibration code runs. You may need to use some male-to-male jumper wires, and a friend to help hold the multimeter terminals onto the bare ends of the jumper wires.

![](images/fzz_multimeter.png)

If you want the calibration to run more slowly to give you more time to see the values on the multimeter, then you can just increase the `sleep_time` parameter on the call to the `calibrate` function on the line below:

```python
fresh_air = calibrate(trace = True, sleep_time = 0.5)
```

Try `1.5` instead of `0.5` and see how that looks. Run the code again with `sudo ./farts.py` when you're ready.

When you're doing this, bear in mind that the HIGH vs LOW threshold for a GPIO pin is around 1.1 to 1.4 volts. The expected result is for the voltage to start around 2 to 3 volts, then drop in stages until it reaches the threshold. When the GPIO pin goes LOW you should see the `Calibrated to x` message on the screen.

