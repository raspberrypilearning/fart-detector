## Introduction

![](images/dude.png)

### How can we detect a fart?

Flatulence or *farts* are essentially gasses that are produced in the stomach and bowels by bacterial fermentation during the process of digestion. The scientific study of farts and fart gases is known as *flatology*, a future career for some of you perhaps? It is perfectly normal for human beings to pass wind every day although the amount varies greatly between individuals, it can range from a tiny amount up to and in excess of two litres.

You'll be surprised to learn that 99% of fart gasses do not smell at all. These include oxygen, nitrogen, carbon dioxide, hydrogen and methane. The remaining 1% is what gives farts their smell and these are mostly volatile sulphuric compounds. The same stuff that makes rotten eggs smell.

To detect a fart with the Raspberry Pi we need to use a sensor that is responsive to one or more of these gasses. Essentially we need to give the Raspberry Pi a *nose*. The sensor recommended for this project is the Figaro TGS2600 and costs around £10.

Here is a close-up of it:

![](images/figaro.png)

Firstly it is important for us to understand how this sensor works. The sensor is designed to measure air quality or rather how *contaminated* the air is. The datasheet for those of you who want it can be found [here](http://www.figarosensor.com/products/2600pdf.pdf).

To summarise it has six holes to allow air to go inside. The air is then energised by a small heater which allows its electrical resistance to be measured. This is done by passing a low level of electricity across a small gap of energised air. Generally speaking the more contaminated the air is the less resistance it has and the better it will conduct electricity (like a variable resistor). The output of the sensor is therefore an analogue voltage that goes up and down according to how contaminated the air is. The more contaminants the higher the voltage output.

### Analogue vs Digital

![](images/analogue_digital.png)

We also need to understand that the air quality sensor gives us an *analogue* signal. So let's look at what analogue means as opposed to digital as a concept. Digital signals are essentially binary 1 or 0, on or off. Analogue signals have the full range *between* on or off. Think of a car steering wheel. The wheel is analogue because you have a full range of steering available to the driver. You can steer very gently around a long sweeping corner, you can turn the wheel to full lock or anywhere in between. If you wanted to steer a car digitally you would basically have full lock left and full lock right only, like steering using the indicator stick.

Those of you who play computer games may have experienced this before. Look at your control pad and consider the differences between the use of the analogue thumb joystick and the digital D-pad in the games that you play. Analogue and digital both have their place and often one works better for a particular task than the other. For a game like a flight simulator you would want analogue control to aim the plane, whereas for something simple like a jump, run and shoot platform game digital control is better.

I expect you can think of further examples of analogue and digital beyond just these.

### But the Raspberry Pi is digital?

![](images/gpio_b_plus.png)

So the challenge we face is being able to read an *analogue* signal on a *digital* computer. The Raspberry Pi GPIO pins can be used as inputs or outputs. Output mode is for when you want to supply voltage to something like an LED or a buzzer. If we use input mode, a GPIO pin has a value that we can read in our code. If the pin has voltage going into it, the reading will be `1` (*HIGH*); if the pin was connected directly to ground (no voltage), the reading will be `0` (*LOW*). So they are digital allowing only `1` or `0`.

How can we solve this? One way would be to use an ADC chip ([Analogue to Digital Converter](http://en.wikipedia.org/wiki/Analog-to-digital_converter)) or something like an [Arduino](http://arduino.cc/en/Main/Products). By connecting the output of the air quality sensor to the input of an ADC we can convert the analogue voltage from the sensor to a digital number in our code.

However this does complicate matters slightly. You would only need to use an ADC if a really accurate reading from the sensor was needed, for example if you wanted to know how many [parts per million](http://en.wikipedia.org/wiki/Parts-per_notation) of methane was present. In practise we just want to make an alarm go off when a fart has been detected so everyone can run! So if you think about it... this is a digital detection. There *is* a fart. There *is no* fart. On or off, binary 1 or 0. We can get away without having to worry about the analogue fidelity coming from the air quality sensor.

We already know that the sensor is like a variable resistor, the worse the air quality the lower the resistance and the more voltage is let through. So logically when the sensor comes into contact with a fart the output voltage should spike. Therefore we just need to detect these voltage spikes and that *can* be done digitally. We can make it so that when a spike occurs a GPIO pin goes from `LOW` to `HIGH`, we can then detect this change in our code and play an alarm sound file!

### The high and low threshold

You now might be wondering how the Raspberry Pi knows if a GPIO pin is `HIGH` or `LOW`?

The answer to this question is actually part of our solution. You may already know that the GPIO pins work at 3.3 volts. So if you set a pin to be `HIGH` in output mode that pin will give/supply 3.3 volts. If you set it to output `LOW` though it will be connected to ground but could form the return path for completing a circuit.

In input mode things work slightly differently. Naturally you would assume that the reading of the pin would be `HIGH` if it was connected to 3.3 volts and `LOW` if connected to ground. There is actually a voltage *threshold* that lies somewhere around 1.1 to 1.4 volts. The actual threshold varies slightly between different hardware revisions of the Raspberry Pi (but we can cope with this). Below the threshold is `LOW` and above it is `HIGH`. So for example 1.0 volt would read `LOW`, despite there actually being some voltage there where as 1.6 volts would read `HIGH` despite this being a lot less than 3.3.

This is quite a hacky way to do it but if we use some resistors to bring the output voltage of the air quality sensor down to *just below* this threshold then the spike caused by a fart will tip it over from `LOW` to `HIGH` and we have our digital fart detection.

## Step 0: Setting up your Pi

First check that you have all the parts you need to get your Raspberry Pi set up and working.

- Raspberry Pi
- Micro USB power adaptor
- An SD card with Raspbian already set up through NOOBS
- USB keyboard
- USB mouse
- HDMI cable
- A monitor or TV

### Activity checklist:

1. Place the SD card into the slot of your Raspberry Pi.
1. Next connect the HDMI cable from the monitor or TV.
1. Plug in the USB keyboard and mouse.
1. Plug in the micro USB power supply.
1. When prompted to login type:

    ```bash
    Login: pi
    Password: raspberry
    ```

## Step 1: Wire up the Air Quality sensor

![](images/pinout.png)

This is the **bottom** view... the pin numbers have the following functions:

1. Heater (-)
1. Sensor electrode (-)
1. Sensor electrode (+)
1. Heater (+)

So there are two distinct circuits that we need to accommodate. First is the *heater* (pins 1 and 4) which is used to energise the air and the other is the *sensor* itself (pins 2 and 3). The output (-) side of the sensor is where we'll be doing our hacky trick with a GPIO pin threshold. Take the breadboard and push the four pins of the sensor into it so that it straddles the central gap as shown below. You may need to bend the pins a little, this will not harm the sensor. Ensure the little tab is in the same orientation as shown.

*Note:* On a breadboard like this the holes on the outside are connected *vertically* and on the inside they're connected *horizontally*. Look at the green highlighting in the diagram.

![](images/fzz_a.png)

The sensor can run on 5 volts but we're going to run it on 3.3 volts here since this is safer for use with a GPIO input. Use the jumper wires to make the orange connections shown above, this will supply 3.3 volts to pins 3 and 4 of the sensor (both positive electrodes). The colour of the wire you use doesn't matter. Next connect the negative (-) terminal of the heater directly to ground as shown above by the black wires.

We still need to do something with the negative side of the sensor, row 1 in the top right corner of the breadboard. See below.

## Step 2: Wire up the trigger pin

Next let's connect the output of the sensor to one of the GPIO pins, this will be the *trigger* pin which we will monitor in our code to see if a fart has occurred. Use GPIO 4 for this (or pin number 7 if you're counting horizontally from the top). Take a jumper wire and make the white connection shown below.

![](images/fzz_b.png)

Next take a 47k ohm resistor (resistors are [colour coded](http://en.wikipedia.org/wiki/Electronic_color_code#Resistor_color-coding) to help you identify them) and connect it between the sensor output and ground as shown above. This will essentially siphon off a portion of the voltage coming from the sensor output to help bring it down to the (1.1 to 1.4 volt) region of the GPIO threshold for our trigger pin. This single resistor is not going to be enough to get the job done though, read on.

## Step 3: Build a resistor ladder DAC

The problem we now have is that despite the addition of the 47k resistor the air quality sensor has quite a large output voltage range. 0 volts would be in a vacuum (no air, e.g. space) and the maximum 3.3 would be in a terrible, eye watering, silent but deadly fart. Depending on the background quality of the air the output voltage of the sensor can be anywhere within that range. So we need a reliable way to always bring that voltage down to just below the GPIO threshold under different air quality conditions.

To do this we need *another* variable resistor, so that we can vary the amount of voltage that we siphon off to ground. We could use a [potentiometer](http://en.wikipedia.org/wiki/Potentiometer) for this but you would always need to manually tune it to the background air before it could be used. This is not ideal if you want to set the trap and wait for an unsuspecting victim. The background air quality can change naturally in the mean time and thus the alarm might go off without a fart. Awkward.

It would be a lot better to have control of this from within our code. Then we can program it to keep adjusting to the background air quality and the trap would not need manual intervention if the air quality changed.

A clever trick we can use here is the [resistor ladder](http://en.wikipedia.org/wiki/Resistor_ladder). This is where we have a set of repeating resistors that we can independently turn on and off in our code. If each resistor has a different value in ohms we can use different combinations of them to give us something which approximates the behaviour of a variable resistor / potentiometer.

Look at the diagram below. This *schematically* shows how a resistor ladder would be connected to the TGS2600 air quality sensor. So essentially the output voltage of the sensor is coming out of pin number `2` and this is connected to GPIO 4. However in between that we have several places where we can siphon off voltage to bring the voltage down to the GPIO pin threshold as required.

### Some theory

![](images/ladder_schematic.png)

So far only the 47k ohm `R0` is present on your breadboard which is hard wired directly to ground. The other resistors (R1 to R4) are connected in parallel to a different GPIO pin. This gives us digital control over whether each resistor is on or off. If we configure the GPIO pin in our code to be in `INPUT` mode this switches the resistor off because the GPIO pin not internally connected to anything. However if we set it to use `OUTPUT` mode and then drive the pin `LOW` this will connect the resistor to ground and thus some voltage will be siphoned off through it.

Since the ladder is controlled digitally by turning resistors on and off with the output being a variable level of resistance (that effects an analogue voltage) the circuit can be called a *digital to analogue converter* or DAC for short. This is the opposite of an ADC mentioned earlier.

Ideally we need to vary the resistance in a linear way and have a good number of possible on/off combinations that will accommodate the range of the air quality sensor output voltage. Consider what would happen if all the resistors had the same value in ohms, how many possible combinations of resistance values could there be?

The answer is only 5. Look at the table below:

R1 | R2 | R3 | R4 
--- | --- | --- | ---
x | x | x | x
ON | x | x | x
ON | ON | x | x
ON | ON | ON | x
ON | ON | ON | ON

This is problematic since it doesn't give us much of a range to choose from, there are only 5 possible steps. It might work but it would be quite hard to bring the voltage down to the GPIO threshold correctly every time. However if we used *different* resistance values for R1 to R4 then we could combine them many more ways affording more combinations. We could borrow from the [binary counting system](http://en.wikipedia.org/wiki/Binary_number#Counting_in_binary) here? So that each resistor represents a binary digit with an associated magnitude.

8's | 4's | 2's | 1's
--- | --- | --- | ---
`0` | `0` | `0` | `0`

In binary each digit position has twice the value of the position to the right. So the right most column is 1's, the next colum is 2's, then 4's and so on. Look at the table above. So to represent the number *3* you need one lot of 2 and one lot of 1, so the decimal number 3 in binary is `0011`. The decimal number *9* is one lot of 8 and one lot of 1 giving `1001`.

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
10 | `1010`
11 | `1011`
12 | `1100`
13 | `1101`
14 | `1110`
15 | `1111`

In a perfect world the resistance values we use should mirror the binary digit position values. For example:

R0 | R1 | R2 | R3 | R4
--- | --- | --- | --- | ---
Rx8 | Rx8 | Rx4 | Rx2 | R 

or

R0 | R1 | R2 | R3 | R4
--- | --- | --- | --- | ---
R | R | R/2 | R/4 | R/8 

