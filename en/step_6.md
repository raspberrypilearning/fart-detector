## Build a resistor ladder DAC

The problem we now have is that despite the addition of the 47kΩ resistor, the air quality sensor has quite a large output voltage range. 0 volts would be what we'd find in a vacuum, whereas the maximum 3.3 would be what we'd see from a terrible, eye-watering, silent-but-deadly fart. Depending on the background quality of the air, the output voltage of the sensor can be anywhere within that range. So we need a reliable way to always bring that voltage down to just below the GPIO threshold, under different air quality conditions.

To do this we need *another* variable resistor, so that we can vary the amount of voltage that we siphon off to ground. We could use a [potentiometer](http://en.wikipedia.org/wiki/Potentiometer) for this, but you would always need to manually tune it to the background air before it could be used. This is not ideal if you want to set the trap and wait for an unsuspecting victim. The background air quality can change naturally in the meantime, and make the alarm go off without a fart. Awkward.

It would be a lot better to have control of this from within our code. Then we can program it to keep adjusting to the background air quality, and the trap will not need manual intervention if the air quality changes.

A clever trick we can use here is the [resistor ladder](http://en.wikipedia.org/wiki/Resistor_ladder). This is where we have a set of repeating resistors that we can independently turn on and off in our code. If each resistor has a different value in ohms, we can use different combinations of them to give us something approximating the behaviour of a variable resistor/potentiometer.

### The theory

This next section might seem a bit boring, but the topics covered will tremendously help your understanding of the project, so I advise you not to skip it!

The diagram below *schematically* shows how a resistor ladder would be connected to the TGS2600 air quality sensor. The output voltage of the sensor is coming out of pin number `2`, and this is connected to GPIO 4. However, in between that we have several places where we can siphon off voltage to bring the voltage down to the GPIO pin threshold as required.

![](images/ladder_schematic.png)

So far only the 47kΩ `R0` is present on your breadboard, which is hard-wired directly to ground. The other resistors (`R1` to `R4`) are each connected *in parallel* to a different GPIO pin. This gives us digital control over whether each resistor is on or off. If we configure the GPIO pin to use `INPUT` mode this switches the resistor off, because the GPIO pin is not internally connected to anything. However, if we set it to use `OUTPUT` mode and then drive the pin LOW, this will connect the resistor to ground and some voltage will be siphoned off through it.

A note about [parallel resistors](http://en.wikipedia.org/wiki/Series_and_parallel_circuits#Resistors_2). The total resistance of the ladder is *not* the sum of all the resistors that are turned on. It would be if you wired the resistors in series, though; that's because the voltage would need to flow through each resistor in turn. In parallel, the flow of voltage will divide equally among each resistor and the effect is that the total resistance *is less*. So the more resistors we turn on, the lower the total resistance will be, and the more voltage gets siphoned off to ground.

Since the ladder is controlled digitally by turning resistors on and off, but affects an analogue voltage from the sensor, the circuit can be called a [digital to analogue converter](http://en.wikipedia.org/wiki/Digital-to-analog_converter) or DAC for short. This is the opposite of the ADC mentioned earlier.

Ideally, we need to vary the resistance in a linear way and have a good number of possible on/off combinations that will accommodate the range of the air quality sensor output voltage. Consider what would happen if all the resistors had the same value in ohms; how many possible *unique* combinations of resistance values could there be?

The answer is only five. Look at the table below:

R1 | R2 | R3 | R4
--- | --- | --- | ---
x | x | x | x
ON | x | x | x
ON | ON | x | x
ON | ON | ON | x
ON | ON | ON | ON

This is problematic, since it doesn't give us much of a range to choose from: there are only five possible steps. It might work, but it would be quite hard to bring the voltage down to the GPIO threshold correctly every time. However, if we used *different* resistance values for `R1` to `R4` then we could combine them in many more ways, affording more combinations. We could borrow from the [binary counting system](http://en.wikipedia.org/wiki/Binary_number#Counting_in_binary) here, so that each resistor represents a binary digit with an associated bit significance/magnitude in ohms.

8s | 4s | 2s | 1s
--- | --- | --- | ---
`0` | `0` | `0` | `0`

In binary, each digit position has twice the value of the position to the right. So the rightmost column is 1s, the next column is 2s, then 4s, 8s and so on. Look at the table above. To represent the number *3* you need one lot of 2 and one lot of 1, so the decimal number 3 in binary is `0011`. The decimal number *9* is one lot of 8 and one lot of 1, giving `1001`.

This would then give us 16 on/off combinations (if we include zero). Look at the table below:

Decimal | Binary
--- | ---
0 | `0000`
1 | `0001`
2 | `0010`
3 | `0011`
4 | `0100`
5 | `0101`
6 | `0110`
7 | `0111`
8 | `1000`
9 | `1001`
- | `1010`
- | `1011`
- | `1100`
- | `1101`
- | `1110`
- | `1111`

In a perfect world, the resistance values for `R1` to `R4` should mirror binary bit significance. The term *bit significance* refers to the value or magnitude that each bit position has. For example, in a four-bit number the rightmost bit has a value of only one, and is called the [least significant bit](http://en.wikipedia.org/wiki/Least_significant_bit) or **LSB** for short. The leftmost bit has a value of eight and is the [most significant bit](http://en.wikipedia.org/wiki/Most_significant_bit) or **MSB** for short.

We need to think carefully now. Consider the amount of voltage that each resistor lets through. The higher the resistance in ohms, the *less* voltage is let through; conversely, the lower the resistance value, the *more* voltage is let through. Remember that a normal wire has almost no resistance and lets *all* voltage though. This means we ought to assign the least significant bit to have the *highest* resistance, since this lets through the least voltage; and the *lowest* resistance to the most significant bit, since this lets through the most voltage. For example:

8s MSB | 4s | 2s | 1s LSB
--- | --- | --- | ---
R/8 | R/4 | R/2 | R

The actual values we're going to use are below. These have been chosen for their universal quality and to make it easier for you to buy/obtain the physical resistors. You'll notice that they do not *perfectly* mirror binary bit significance, but they will be good enough for this project.

8s MSB | 4s | 2s | 1s LSB
--- | --- | --- | ---
4.7k | 10k | 22k | 47k

Take another look at the schematic diagram above. You'll see that there is a row of 1s to represent the four-bit binary number that will be the on/off state of the ladder; it shows `1111`, which is 15. So in our code we'll start the ladder at `0000`. With all the resistors turned off the output voltage will be much higher than the GPIO threshold, and so the trigger pin (GPIO 4) will read HIGH. We then incrementally work our way up to 15 or `1111`. On each step we decrease the resistance (or increase the amount of voltage siphoned off to ground), and check to see if GPIO 4 has gone from HIGH to LOW. Once we have found the HIGH/LOW threshold, the air quality sensor is then calibrated to normal air, and any increase in output voltage (caused by a fart) should be enough to tip the trigger pin back from LOW into HIGH. We then just need to wait for this to happen in our code and then sound the alarm!

### The practice

Let's go ahead and wire up the resistor ladder on our breadboard. Here is a quick reference table for the resistor values that you need to use, and the pins they should be connected to:

R0 | R1 | R2 | R3 | R4
--- | --- | --- | --- | ---
47k | 47k | 22k | 10k | 4.7k
GND | GPIO 17 | GPIO 18 | GPIO 22 | GPIO 23

The resistor [colour bands](http://en.wikipedia.org/wiki/Electronic_color_code#Resistor_color-coding) will match those on the diagram below. Use the jumper cables to make the connections shown; remember that the colour of the wire does not matter. You'll notice a jumper needs to go between H1 and F8 on the breadboard. This is just to expand the number of holes that are connected to pin 2 of the air quality sensor, allowing us to set off each of the resistors that make up the ladder.

![](images/fzz_c.png)

When you're done you should have something like this. You'll notice that for some of the connections we have just used bare wire (instead of a jumper), pushed flat to the breadboard. This can be a nice way of keeping things tidy, but either way will work fine.

![](images/breadboard_done.jpg)

We have now completed the hardware side of the project; we just need to bring it to life with some programming!

