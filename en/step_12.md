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

