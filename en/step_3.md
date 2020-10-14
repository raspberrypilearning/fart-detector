## Giving your Pi a 'nose': how it works

Humans can detect farts easily, as we are sensitive to the volatile sulphuric compounds which make up 1% of flatulence (i.e. the compounds which make farts smell).
To detect a fart with the Raspberry Pi, however, is a slightly different proposition: we need to give the Raspberry Pi a 'nose', by connecting it to a sensor which is responsive to one of the gasses present in flatulence. The sensor recommended for this project is the Figaro TGS2600, which is sensitive to the hydrogen, another gas contained in flatulence. When air enters the sensor, it is energised by a small heater which allows its electrical resistance to be measured. This is done by passing a low level of electricity across a small gap of energised air. The more contaminated the air is, the less resistance it has and the better it will conduct electricity (like a variable resistor). The output of the sensor is therefore an analogue voltage that goes up and down according to how contaminated the air is. The more contaminants, the higher the voltage output.

### Analogue vs Digital

We also need to understand that the air quality sensor gives us an analogue signal, and the difference between an analogue signal and a digital one. Digital signals are essentially binary: 1 or 0; on or off. Analogue signals, on the other hand, have the full range between on or off. Think of a car steering wheel: the wheel is analogue because there is a full range of steering available to the driver. You can steer very gently around a long sweeping corner, you can turn the wheel to full lock, or anywhere in between. If you wanted to steer a car digitally you would basically have full lock left and full lock right only.

### Reading an analogue signal with a digital device

The challenge we face is being able to read an analogue signal on a digital computer. The Raspberry Pi GPIO pins can be used as inputs or outputs. Output mode is for when you want to supply voltage to something like an LED or a buzzer. If we use input mode, a GPIO pin has a value that we can read in our code. If the pin has voltage going into it, the reading will be `1` (*HIGH*); if the pin was connected directly to ground (no voltage), the reading would be `0` (*LOW*). So the pins are digital, allowing only `1` or `0`.

How can we solve this? One way would be to use an ADC chip ([Analogue to Digital Converter](http://en.wikipedia.org/wiki/Analog-to-digital_converter)), which would convert the analogue voltage from the sensor to a digital number in our code.However you would only need to use an ADC if a really accurate reading from the sensor was required. In practice, we just want to make an alarm go off when a fart has been detected, so everyone can run! So if you think about it, this *is* a digital detection. There *is* a fart or there *is no* fart: on or off, binary 1 or 0. We don't have to worry about the analogue fidelity coming from the air quality sensor.

We already know that the sensor is like a variable resistor: the worse the air quality, the lower the resistance and the more voltage is let through. Logically, when the sensor comes into contact with a fart, the output voltage should spike. Therefore, we just need to detect these voltage spikes and that *can* be done digitally. We can make it so that when a spike occurs a GPIO pin goes from LOW to HIGH; we can then detect this change in our code and play an alarm sound file!

### The high and low threshold

How does the Raspberry Pi know if a GPIO pin is HIGH or LOW?

The answer to this question is actually part of our solution. The GPIO pins work at 3.3 volts. So if you set a pin to be HIGH in output mode, that pin will give/supply 3.3 volts. If you set it to output LOW, though, it will be connected to ground but could form the return path for completing a circuit.

In input mode things work slightly differently. You might assume that the reading of the pin would be HIGH if it was connected to 3.3 volts and LOW if connected to ground. There is actually a voltage *threshold* that lies somewhere around 1.1 to 1.4 volts. Below the threshold is LOW and above it is HIGH; so for example 1.0 volt would read LOW, despite there actually being some voltage there, whereas 1.6 volts would read HIGH, despite this being a lot less than 3.3.

If we use some resistors to bring the output voltage of the air quality sensor down to *just below* this threshold, then the spike caused by a fart will tip it over from LOW to HIGH, and we have our digital fart detection.

