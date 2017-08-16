## Play a test alarm sound

Plug your Raspberry Pi back in, boot up, and log in as usual. If you are using headphones or a speaker on the Raspberry Pi, you will need to run the following terminal command to redirect sound to the headphone socket:

```bash
sudo amixer cset numid=3 1
```

First, we need to download a sound file that will be the alarm: we've chosen a robotic voice saying "evacuate". Enter the following terminal command to download it:

```bash
wget https://raw.githubusercontent.com/raspberrypilearning/fart-detector/master/data/evacuate.mp3 --no-check-certificate
```

Now let's do some programming. Enter the following command to start editing a blank file:

```bash
nano farts.py
```

Now either copy and paste or enter the following code by hand:

```python
#!/usr/bin/python
import time

from pygame import mixer
mixer.init()
mixer.music.load("evacuate.mp3")

mixer.music.play(-1) # -1 to loop the sound
time.sleep(10) #let it play for 10 seconds
mixer.music.stop()
```

This code uses the pygame mixer to load a sound file and play it in a loop for 10 seconds.
Press `Ctrl - O` then `Enter` to save, followed by `Ctrl - X` to quit.

Next, mark the file as executable with the following command:

```bash
chmod +x farts.py
```

Now we can run the code. When you do, the alarm should play for 10 seconds and then stop.

```bash
./farts.py
```

