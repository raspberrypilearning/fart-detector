# Fart Detector

Bust people for farting with your Raspberry Pi

![](cover.png)

All kids understand that farts are funny. As we grow older, some of us lose our sense of humour in this area. You'll be glad to hear that we believe that building a real working fart detector has both educational and comedic value. If you have a relative or dog who frequently likes to stink up the family home, you'll be able to provide the advance warning necessary to evacuate the room.

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

- Breadboard (try [Pimoroni](http://shop.pimoroni.com/products/solderless-breadboard-400-point) or [Maplin](http://www.maplin.co.uk/p/ad-102-breadboard-ag10l))
- **Male** to **Female** jumper wires, at least 8 (try [Pimoroni](http://shop.pimoroni.com/products/jumper-jerky))
- **Male** to **Male** jumper wires, at least 6 (try [Pimoroni](http://shop.pimoroni.com/products/jumper-jerky))
-	2 x 47k ohm resistor, through hole (try [Pimoroni](http://shop.pimoroni.com/products/resistor-grab-bag) or [Maplin](http://www.maplin.co.uk/c/components/resistors))
-	1 x 22k ohm resistor, through hole (try [Pimoroni](http://shop.pimoroni.com/products/resistor-grab-bag) or [Maplin](http://www.maplin.co.uk/c/components/resistors))
- 1 x 10k ohm resistor, through hole (try [Pimoroni](http://shop.pimoroni.com/products/resistor-grab-bag) or [Maplin](http://www.maplin.co.uk/c/components/resistors))
- 1 x 4.7k ohm resistor, through hole (try [Pimoroni](http://shop.pimoroni.com/products/resistor-grab-bag) or [Maplin](http://www.maplin.co.uk/c/components/resistors))
-	Air quality sensor (try [RS Components](http://uk.rs-online.com/web/p/gas-detection/5389960))

### Extras

- A digital multimeter (try [Pimoroni](http://shop.pimoroni.com/products/digital-multimeter) or [Maplin](http://www.maplin.co.uk/p/uni-trend-ut30b-digital-compact-multimeter-n15by))
- A can of deodorant (to simulate farts)

## Worksheet & included files

- [The worksheet](worksheet.md)
- (Optional) Final version of Python code [farts-final.py](farts-final.py)
    - Download to your Pi with the following commands:

        ```bash
        wget http://goo.gl/DrvYfl -O evacuate.mp3 --no-check-certificate
        wget http://goo.gl/OJBFEf -O farts-final.py --no-check-certificate
        chmod +x farts-final.py
        sudo ./farts-final.py
        ```

## Licence

Unless otherwise specified, everything in this repository is covered by the following licence:

[![Creative Commons License](http://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

***Fart Detector*** by the [Raspberry Pi Foundation](http://raspberrypi.org) is licenced under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

Based on a work at https://github.com/raspberrypilearning/fart-detector
