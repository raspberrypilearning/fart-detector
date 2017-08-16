## Troubleshooting

We've tried to anticipate the problems you may encounter and have listed the most common ones below.

### Will not calibrate

The heater in the air quality sensor requires some time to warm up from cold before it will work, so please wait a while and try again. In some cases it can take up to 20 minutes before you'll get a successful calibration. It depends largely on the background air quality, temperature, and humidity.

If you are still unable to get a successful calibration then it's a good idea to do the *Optional activity using a multimeter* described above (refer to the end of step 5). This will allow you to observe the voltage level that is reaching the trigger pin as the calibration code runs. You can then confirm that the voltage level never gets down to the 1.1 to 1.4 region required to make the trigger pin go LOW.

If that *is* the case I recommend replacing only the 47kΩ `R0` resistor with a lesser 10kΩ resistor (refer to the diagram at the end of step 3). This will siphon off a greater amount of voltage by default and should allow you to get a successful calibration.

### Alarm goes off in normal air

If the alarm goes off in normal air without any farts or deodorant, one of two things is probably happening:

- There is some unseen or unknown air contaminant which is setting it off. Did someone drop an SBD?
- The background temperature and/or humidity has changed, causing the calibration to become wrong.

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

