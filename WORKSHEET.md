## Introduction: how can we detect a fart?

![](./images/dude.png)

Flatulence or *farts* are essentially gasses that are produced in the stomach and bowels by bacterial fermentation during the process of digestion. The scientific study of farts and fart gases is known as *flatology*, a future career for some of you perhaps? It is perfectly normal for human beings to pass wind every day although the amount varies greatly between individuals, it can range from a tiny amount up to and in excess of two litres.

You'll be surprised to learn that 99% of fart gasses do not smell at all. These include oxygen, nitrogen, carbon dioxide, hydrogen and methane. The remaining 1% is what gives farts their smell and these are mostly volatile sulphuric compounds. The same stuff that makes rotten eggs smell.

To detect a fart with the Raspberry Pi we need to use a sensor that is responsive to one or more of these gasses. Essentially we need to give the Raspberry Pi a *nose*. The sensor recommended for this project is the Figaro TGS2600 and costs around £10.

Here is a close-up of it:

![](./images/figaro.png)

Firstly it is important for us to understand how this sensor works. The sensor is designed to measure air quality or rather how *contaminated* the air is. The datasheet for those of you who want it can be found [here](http://www.figarosensor.com/products/2600pdf.pdf).

To summarise it has six holes to allow air to go inside. The air is then energised by a small heater which allows its electrical resistance to be measured. This is done by passing a low level of electricity across a small gap of energised air. Generally speaking the more contaminated the air is the less resistance it has and the better it will conduct. The output of the sensor is therefore an analogue voltage that goes up and down according to how contaminated the air is.
