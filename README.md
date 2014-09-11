# Fart Detector

Bust people for farting with your Raspberry Pi

![](images/cover.png)

All kids understand that farts are funny. As we grow older, some of us lose our sense of humour in this area. You'll be glad to hear we believe that building a real working fart detector has both educational and comedic value. If you have a relative or dog who frequently likes to stink up the family home, you'll be able to provide the advanced warning necessary to evacuate the room.

## Lesson objectives

- Understand the difference between analogue and digital
- Understand binary counting
- Understand voltage, current and resistance
- Understand how an air quality sensor works
- Understand fart potency

## Lesson outcomes

- To have built and programmed a working fart detector
- To have made a rudimentary DAC (digital to analogue converter) 
- Gained experience in Python programming
- Gained experience using the Raspberry Pi GPIO pins

## Requirements

### Hardware

- Raspberry Pi (any model)
- Micro USB power adaptor
- An SD Card with Raspbian already set up through NOOBS ([setup guide](http://www.raspberrypi.org/help/noobs-setup/))
- USB keyboard
- USB mouse
- HDMI cable
- Ethernet cable
- LAN with Internet connection
- A monitor or TV
- Breadboard (try [Pimoroni](http://shop.pimoroni.com/products/solderless-breadboard-400-point) or [Maplin](http://www.maplin.co.uk/p/ad-102-breadboard-ag10l))
- **Male** to **Female** jumper wires, at least 8 (try [Pimoroni](http://shop.pimoroni.com/products/jumper-jerky))
- **Male** to **Male** jumper wires, at least 6 (try [Pimoroni](http://shop.pimoroni.com/products/jumper-jerky))
-	2 x 47k ohm resistor, through hole (try [Pimoroni](http://shop.pimoroni.com/products/resistor-grab-bag) or [Maplin](http://www.maplin.co.uk/c/components/resistors))
-	1 x 22k ohm resistor, through hole (try [Pimoroni](http://shop.pimoroni.com/products/resistor-grab-bag) or [Maplin](http://www.maplin.co.uk/c/components/resistors))
- 1 x 10k ohm resistor, through hole (try [Pimoroni](http://shop.pimoroni.com/products/resistor-grab-bag) or [Maplin](http://www.maplin.co.uk/c/components/resistors))
- 1 x 4.7k ohm resistor, through hole (try [Pimoroni](http://shop.pimoroni.com/products/resistor-grab-bag) or [Maplin](http://www.maplin.co.uk/c/components/resistors))
-	Air quality sensor (try [RS Components](http://uk.rs-online.com/web/p/gas-detection/5389960))

### Software

- python-pygame

### Extras

- A digital multimeter (try [Pimoroni](http://shop.pimoroni.com/products/digital-multimeter) or [Maplin](http://www.maplin.co.uk/p/uni-trend-ut30b-digital-compact-multimeter-n15by))
- A can of deodorant or perfume (to simulate farts)

### Time required

- 2 to 3 hours

## Steps

1. Setting up your Pi
1. Wire up the Air Quality sensor
1. Wire up the trigger pin
1. Build a resistor ladder DAC
1. Play a test alarm sound
1. Write code to calibrate the ladder
1. Monitoring for farts and raising the alarm
1. Continuous monitoring and recalibration
1. Make the output even more amusing

## Worksheet & included files

- [The worksheet](WORKSHEET.md)
- (Optional) Final version of Python code [farts-final.py](farts-final.py)
    - Download to your Pi with the following commands:

    ```bash
    wget https://raw.githubusercontent.com/raspberrypilearning/fart-detector/master/sounds/evacuate.mp3 --no-check-certificate
    wget https://raw.githubusercontent.com/raspberrypilearning/fart-detector/master/farts-final.py --no-check-certificate
    chmod +x farts-final.py
    sudo ./farts-final.py
    ```
## Licence

Unless otherwise specified, everything in this repository is covered by the following licence:

![Creative Commons License](http://i.creativecommons.org/l/by-sa/4.0/88x31.png)

***Fart Detector*** by the [Raspberry Pi Foundation](http://raspberrypi.org) is licenced under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

Based on a work at https://github.com/raspberrypilearning/fart-detector
