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

