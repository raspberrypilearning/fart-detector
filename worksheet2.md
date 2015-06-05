# Fart Detector - Part 2

Connect a sensor to your Raspberry Pi to warn you when there are noxious gases about!

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

## Monitoring for farts and raising the alarm

Now that we have successfully calibrated the sensor in normal air, we can add some code to wait for the trigger pin to go from LOW to HIGH, then we can play the alarm sound. Let's continue editing our program.

```bash
nano farts.py
```

We should add code directly after the line `print "Calibrated to", fresh_air` so that it will only run if we have a successful calibration. Look at the code below and modify yours to match it.

```python
fresh_air = calibrate(trace = True, sleep_time = 0.5)

if fresh_air != -1:
    print "Calibrated to", fresh_air

    print "Waiting for fart..."

    while not GPIO.input(TRIGGER): #wait as long as trigger is LOW
        time.sleep(.1)

    fart = calibrate(sleep_time = 0.1) #quickly recalibrate to get the fart level

    if fart > fresh_air or fart == -1:
        print "Fart level", fart, "detected!"

        mixer.music.play(-1) # -1 to loop the sound
        time.sleep(10) #let it play for 10 seconds
        mixer.music.stop()
else:
     print "Could not calibrate"
```

So first we use a `while` loop with the syntax `while not GPIO.input(TRIGGER)`, with a sleep inside the loop. This will hold up the code from progressing onto the lines below while the trigger pin reads `0` LOW. Cue a fart and the output voltage of the sensor should increase enough for the trigger pin to go back into HIGH, which will cause the loop to exit. We can then reuse the `calibrate` function as a way to measure the fart potency!

To do this we call the `calibrate` function again but we pass in a 0.1 second `sleep_time` parameter, because we want to do this quickly in order to sound the alarm. We store the result of this in a variable called `fart` so that we can compare it to `fresh_air`. We should only sound the alarm if `fart` is greater (worse air quality) than `fresh_air`, or if `fart` was a failed calibration meaning the air quality can't get any worse. So we use the `if fart > fresh_air or fart == -1` syntax to do this; inside the `if` statement we can print out the level of the fart, and put the three lines of code to play the alarm sound for ten seconds.

Let's run the code. Press `Ctrl - O` then `Enter` to save, followed by `Ctrl - X` to quit.
Remember to use the `sudo` command when you run the code.

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
Waiting for fart...
```

I would suggest using a deodorant can to test that the air quality sensor is working. Most deodorants use a gas called isobutane: the sensor is very sensitive to isobutane, so this gives us a good way to simulate farts on demand.

You only need a very small squirt to set it off, so spray some in the general direction of the sensor and wait. The message `Fart level x detected!` should appear and the *evacuate* alarm should go off. If you get a `-1` then you probably sprayed too much; you may need to wait a bit to be able to successfully calibrate the next time you run the code.

Make sure you don't spray the deodorant directly onto the sensor: if too much isobutane gets inside it, it may well not calibrate again for several hours.

## Continuous monitoring and recalibration

You'll notice that currently the program will only wait for one fart, sound the alarm and then exit. It may be that you want to set this up in a semi-permanent way to provide an early warning system in the home. It would be better in this case if the program could sound the alarm for a while, and then recalibrate to wait for another fart.

The sensor also has a dependency on temperature and humidity, which will manifest as the output voltage of the air quality sensor shifting under normal air conditions. A potential failure mode could see our ladder calibration level becoming wrong as conditions change throughout the day, causing the alarm to go off on its own without a fart. To mitigate this, we can put a timeout in our code to force a recalibration to run every two minutes.

Let's continue editing our program.

```bash
nano farts.py
```

The first problem is easy to solve. All we have to do is enclose our current code in a `while` loop and perhaps add a five-second `sleep` where we were unable to calibrate, to wait for the air to clear or the sensor heater to warm up.

Then we need to limit how long we wait while the trigger pin is LOW. To measure time in code you have to record the time now, wait for something to happen, and then subtract the time you recorded from the current time. Take a look at the code below and modify yours to match it:

```python
while True:
    fresh_air = calibrate(trace = True, sleep_time = 0.5)

    if fresh_air != -1:
        print "Calibrated to", fresh_air
        start_time = time.time()

        print "Waiting for fart..."

        while not GPIO.input(TRIGGER) and time.time() - start_time < 120: #wait as long as trigger is LOW or < 2 min
            time.sleep(.1)

        if time.time() - start_time < 120: #make sure this is not just a timeout
            fart = calibrate(sleep_time = .1) #quickly recalibrate to get the fart level

            if fart > fresh_air or fart == -1:
                print "Fart level", fart, "detected!"
                mixer.music.play(-1) # -1 to loop the sound
                time.sleep(10) #let it play for 10 seconds
                mixer.music.stop()
        else:
            print "Time out, recalibrating..."
    else:
        print "Could not calibrate"
        time.sleep(5)
```

Let's go through this. Firstly, we have added the `while True` syntax just above the fresh air calibration. All of the existing code has been indented by four spaces to make it belong to this `while` loop. We then put a `time.sleep(5)` under the `else` clause where the calibration was unsuccessful.

When you wrap an existing block of code in a `while` loop or `if` statement, you must make sure everything is correctly indented. [Indentation](http://en.wikipedia.org/wiki/Python_syntax_and_semantics#Indentation) in Python is very important.

You'll notice that we have added the line `start_time = time.time()` just after a calibration has been successful. This is to record the time of the calibration so we can measure how much time has elapsed since then. Next, there is a change to the `while` loop where we monitor the trigger pin. We've added the syntax `and time.time() - start_time < 120`, for the condition when the trigger pin is LOW *and* less than 120 seconds have elapsed since the start. So after 120 seconds, the loop will exit.

We now need to be careful, as we will arrive at this point every time the 120-second timeout occurs. So we should only test for farts and sound the alarm if the 120 seconds has not fully elapsed. This would imply that the trigger pin must have gone HIGH to cause the `while` loop to exit. To do this we can just measure the elapsed time again, using an `if` statement with the syntax `if time.time() - start_time < 120`. We can also add an `else` clause to explicitly indicate that a timeout has happened.

Let's run the code. Press `Ctrl - O` then `Enter` to save, followed by `Ctrl - X` to quit.
Remember to use the `sudo` command when you run the code.

```
sudo ./farts.py
```

Wait for two minutes and the output should look something like this:

```
0 0
1 1
2 10
3 11
4 100
Calibrated to 4
Waiting for fart...
Time out, recalibrating...
0 0
1 1
2 10
3 11
4 100
Calibrated to 4
Waiting for fart...
```

Get the deodorant can out again, spray some at the sensor and ensure that the alarm is still working. After the alarm has gone off, you may find that you will see several `Could not calibrate` messages before you get a successful calibration. Just allow some time for the air to return to normal, and perhaps open a window, then you should eventually get another `Waiting for fart...` message. Thus we now have a continuous fart monitoring solution that will recalibrate to natural changes in temperature and humidity during the day.

You can press `Ctrl - C` to abort the program.

## Customise the output

Here are some tricks you can use in your code to make it more funny. The first is to show the *fart detected* message in colour. There are a number of [ANSI escape codes](http://en.wikipedia.org/wiki/ANSI_escape_code) that we can use in conjunction with the `print` command to change the background and foreground colour of the terminal.

For example:

```python
print "\033[0;32mGREEN TEXT\033[0m"
```

This may look complicated, but the format can be broken down like this:

`START_SEQ``x;y;zm``DISPLAY_TEXT``END_SEQ`

- Start sequence: `\033[`
- x; y; z m: `ANSI colour codes delimited by a semicolon` followed by `m`
- Display text: `The text you want to show`
- End sequence: `\033[0m`

Here is a table of the codes you might want to use:

Text color | Code | Text style | Code | Background color | Code
--- | --- | --- | --- | --- | ---
Black | 30 | No effect | 0 | Black | 40
Red | 31 | Bold | 1 | Red | 41
Green | 32 | Underline | 4| Green | 42
Yellow | 33 | Blink | 5 | Yellow | 43
Blue | 34 | Inverse | 7 | Blue | 44
Purple| 35 | Hidden | 8 | Purple | 45
Cyan | 36 | | | Cyan | 46
White | 37 | | | White | 47

A good way to simplify this is to use Python [string formatting](https://docs.python.org/2/library/string.html#string-formatting). This is quite a sophisticated way to manipulate strings, and has a number of advantages over using the older *%s* method. For example:

```python
alarm_template = "\033[5;43;31m{0} {1} {2}\033[0m"

print alarm_template.format("Fart level", fart, "detected!")
```

We create a string variable, which contains the ANSI escape codes that we want to use as a template. Within the template there are some numbered markers `{0} {1} {2}`, which specify the parts of the string that should be replaced when you use the `format` command.

Secondly, you could use a `if...elif...elif` statement to show different messages depending on the fart potency. So essentially, if `fart` is greater than one number and less than another number, then show one message; else show another message. It would look something like this:

```python
if fart >= 0 and fart < 5:
    print alarm_template.format("Huh only level", fart, "detected, call that a fart?")
elif fart >= 5 and fart < 10:
    print alarm_template.format("Fart level", fart, "detected!")
elif fart >= 10 and fart < 15:
    print alarm_template.format("DANGER! Fart level", fart, "detected!")
elif fart == -1:
    print alarm_template.format("GAS GAS GAS! FART LEVEL", "EXTREME", "DETECTED! EVACUATE EVACUATE EVACUATE!")
```

So, given these changes, the final code will be this:

```python
#!/usr/bin/python
import time, RPi.GPIO as GPIO

from pygame import mixer
mixer.init()
mixer.music.load("evacuate.mp3")

alarm_template = "\033[5;43;31m{0} {1} {2}\033[0m"

TRIGGER = 4
GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM) #use BCM pin layout
GPIO.setup(TRIGGER, GPIO.IN)

def set_pin(pin, ison):
    if ison:
        GPIO.setup(pin, GPIO.OUT)
        GPIO.output(pin, GPIO.LOW)
    else:
        GPIO.setup(pin, GPIO.IN)

def set_dac(bitwise):
    set_pin(17, bitwise & 1 == 1)
    set_pin(18, bitwise & 2 == 2)
    set_pin(22, bitwise & 4 == 4)
    set_pin(23, bitwise & 8 == 8)

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

while True:
    fresh_air = calibrate(trace = True, sleep_time = 0.5)

    if fresh_air != -1:
        print "Calibrated to", fresh_air
        start_time = time.time()

        print "Waiting for fart..."

        while not GPIO.input(TRIGGER) and time.time() - start_time < 120: #wait as long as trigger is LOW or < 2 min
            time.sleep(.1)

        if time.time() - start_time < 120: #make sure this is not just a timeout
            fart = calibrate(sleep_time = .1) #quickly recalibrate to get the fart level

            if fart > fresh_air or fart == -1:
                if fart >= 0 and fart < 5:
                    print alarm_template.format("Huh only level", fart, "detected, call that a fart?")
                elif fart >= 5 and fart < 10:
                    print alarm_template.format("Fart level", fart, "detected!")
                elif fart >= 10 and fart < 15:
                    print alarm_template.format("DANGER! Fart level", fart, "detected!")
                elif fart == -1:
                    print alarm_template.format("GAS GAS GAS! FART LEVEL", "EXTREME", "DETECTED! EVACUATE EVACUATE EVACUATE!")

                mixer.music.play(-1) # -1 to loop the sound
                time.sleep(10) #let it play for 10 seconds
                mixer.music.stop()
        else:
            print "Time out, recalibrating..."
    else:
        print "Could not calibrate"
        time.sleep(5)
```

There is one other trick you could do, which is to make the alarm continue until the fart has dissipated and a successful calibration has been made. All you need to do for this is to move the `mixer.music.stop()` line from its current location to just after you show the `Waiting for fart...` message. This means the alarm will sound for a minimum of 10 seconds and will continue for however many calibration attempts are required, which could be several minutes.

## Troubleshooting

We've tried to anticipate the problems you may encounter and have listed the most common ones below.

### Will not calibrate

The heater in the air quality sensor requires some time to warm up from cold before it will work, so please wait a while and try again. In some cases it can take up to 20 minutes before you'll get a successful calibration. It depends largely on the background air quality, temperature, and humidity.

If you are still unable to get a successful calibration then it's a good idea to do the *Optional activity using a multimeter* described above (refer to the end of step 5). This will allow you to observe the voltage level that is reaching the trigger pin as the calibration code runs. You can then confirm that the voltage level never gets down to the 1.1 to 1.4 region required to make the trigger pin go LOW.

If that *is* the case I recommend replacing only the 47kΩ `R0` resistor with a lesser 10kΩ resistor (refer to the diagram at the end of step 3). This will siphon off a greater amount of voltage by default and should allow you to get a successful calibration.

### Alarm goes off in normal air

If the alarm goes off in normal air without any farts or deodorant, one of two things is probably happening:

1. There is some unseen or unknown air contaminant which is setting it off. Did someone drop an SBD?
1. The background temperature and/or humidity has changed, causing the calibration to become wrong.

Don't be too surprised if this happens. It happened to us when we were doing the testing for this resource quite a few times. There are a couple of things you can do to mitigate this though. First and easiest is to reduce the calibration timeout; this is currently set to 120 seconds. You could try and reduce this to 60 seconds in your code and see if that helps.

Find the following line and change the `120` to `60` in your code.

```python
while not GPIO.input(TRIGGER) and time.time() - start_time < 120:
```

You'll also need to change the `120` on the following line below the `while` statement. Consider using a variable for this:

```python
if time.time() - start_time < 120:
```

If that doesn't help there is one other option. This is to force the ladder DAC into a lower resistance or higher binary number configuration than was returned by the `calibrate` function in your code. This will make the fart detector slightly less sensitive, meaning it should no longer give false alarms; it will, consequently, need a stronger fart to set it off.

We will have to respect the upper limit of 15 which is `1111` in binary. So firstly we can use an `if` statement to check the calibration level. If it is less than 15 then we can add 1 to `fresh_air` and call the `set_dac` function again.

```python
if fresh_air < 15:
    fresh_air += 1
    set_dac(fresh_air)
```

We then need to put this `if` statement into the right place in our code. It should go just before the `print "Calibrated to", fresh_air` line like so:

```python
while True:
    fresh_air = calibrate(trace = True, sleep_time = 0.5)

    if fresh_air != -1:
        if fresh_air < 15:
            fresh_air += 1
            set_dac(fresh_air)

        print "Calibrated to", fresh_air
```

The consequence of this change is that you are making the fart detector less sensitive, but only by one position on its scale of 0 to 15, so it should still function as expected.

### My farts don't set it off

If you farted and the alarm didn't go off then it is likely that the fart didn't smell, which happens more often than you might think. To ensure that your farts can set the detector off, you need to eat something that will ferment in the gut and produce hydrogen and methane, which the sensor can easily detect. Some carbohydrates cannot be digested and absorbed by the intestines, and so they pass down into your colon where they ferment and produce these gases.

Foods that contain a high amount of unabsorbable carbohydrates include beans, broccoli, cabbage, cauliflower, artichokes, raisins, pulses, lentils, onions, prunes, apples and brussels sprouts. Need I say more?
