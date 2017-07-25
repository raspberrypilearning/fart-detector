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

