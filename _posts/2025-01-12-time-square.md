I had three LCD screens (16x2), one 3.5" RPI display, and two Raspberry Pis lying around and gathering dust. It had been three years since I had worked on an Arduino/Raspberry Pi project. It was time to see how much I still remembered about making circuits work.

As usual, I decided to challenge myself. I put forth two challenges,

1. Use Art and Craft materials to make something different
2. Use Vim to write/edit the code

I was able to use some of my creative juice to beautify my usual digital clock, but I gave up on my second challenge. 

A few years back, I read a tweet, 

> I couldn't exit Vim, so I learned it.

Well, I did learn to split the screen vertically and horizontally, copy from one file to another, and use commands such as `p, P, d, dd, set nu or set compatible`, etc., but it was high time I learned to exit Vim and speed up the development process.

The idea was to develop a digital clock to show the date, time, and temperature.

But it has to be different. I decided to align the LCD screen vertically to show the dates.

Check the image below to get the idea.

![TIME SQUARE](/what-i-learnt/assets/time_square.jpg)

The concepts I learned were - 

1. Not all 16x2 LCD screens are the same
2. Creating custom characters
3. `kivy` for desktop app creation
4. Connecting the RPI screen not using the GPIO provided at the back but using a minimum number of jumper wires
5. Connecting multiple LCDs to Raspberry Pi
6. and finally, a few things about Raspberry Pi

Let's take one at a time.

### RASPBERRY PI 

`RTFM` has been the mantra for becoming a good programmer, but in the days of 30-second YouTube shorts, it has been reduced to `TL; DR.` However, sometimes, it is good to refer to documentation.

For example, I used to Google for the `pin` header diagram, but Raspberry Pi has an out-of-the-box command. 

```
pinout

J8:
 3V3  (1) (2)  5V    
 GPIO2  (3) (4)  5V    
 GPIO3  (5) (6)  GND   
 GPIO4  (7) (8)  GPIO14
 GND  (9) (10) GPIO15
GPIO17 (11) (12) GPIO18
GPIO27 (13) (14) GND   
GPIO22 (15) (16) GPIO23
 3V3 (17) (18) GPIO24
GPIO10 (19) (20) GND   
 GPIO9 (21) (22) GPIO25
GPIO11 (23) (24) GPIO8 
 GND (25) (26) GPIO7 
 GPIO0 (27) (28) GPIO1 
 GPIO5 (29) (30) GND   
 GPIO6 (31) (32) GPIO12
GPIO13 (33) (34) GND   
GPIO19 (35) (36) GPIO16
GPIO26 (37) (38) GPIO20
 GND (39) (40) GPIO21

J2:
GLOBAL ENABLE (1)
 GND (2)
 RUN (3)

J14:
TR01 TAP (1) (2) TR00 TAP
TR03 TAP (3) (4) TR02 TAP

```

Secondly, the RPI is loaded with three LCDs and one RPI screen. The board would become hot, and to avoid overheating, I had to put in heat sinks and a fan (5v). This meant I needed a way to check the current voltage, amperage, and temperature. 

Here comes `vcgencmd`.

```
tushar@raspberrypi:~ $ vcgencmd measure_temp
temp=50.6'C
tushar@raspberrypi:~ $ vcgencmd measure_volts core
volt=0.8563V
tushar@raspberrypi:~ $ vcgencmd get_throttled
throttled=0x0
```

There was no way to check the exact amperage without buying additional hardware, so I had to chuck it. I could have used a multimeter, but then I just didn't see RPI complaining about overheating, so I chucked it.

### LCD Screens

#### Not all LCD screens are the same

Initially, I was using Adafruit's [CircuitPython_CharLCD](https://docs.circuitpython.org/projects/charlcd/en/latest/). This works if we connect the LCD's individual pin to RPI, but it fails when I use an LCD with an `i2c` interface. I was on the verge of giving up when I came across [RPLCD](https://rplcd.readthedocs.io/en/stable/). The document mentioned that -

```
Supported I²C Port Expanders

PCF8574 (used by a lot of I²C LCD adapters on Ali Express)
MCP23008 (used in Adafruit I²C LCD backpack)
MCP23017
```

That's it. My LCDs were PCF8574. I found this by using a magnifying glass to check the `i2c` adapter.

Check the image below.

![TIME SQUARE](/what-i-learnt/assets/i2c.jpeg)

#### Custom Character

The LCD has 16 columns and 2 rows. The library has a wrapper to write each character(predefined and baked into the library) in one of the cells. I used custom characters for all 16 columns and 2 rows to display one character. For that, I had to know that each cell has 5 tiny lights in a row and 8 such rows. Each row could be on or off using `0` or `1`. Using `b00111` will light up the three 
lights on the right, and two will be turned off. There is an excellent website to generate such binary codes for 1 cell.

[LCD Custom Code Generator](https://maxpromer.github.io/LCD-Character-Creator/)

#### Connecting Multiple LCD

Since the LCDs use `i2c`, it should be enabled (by default, it is disabled on Raspberry Pi) by navigating to `raspi-config` and then to `display` and turning on the `i2c` option.

By default, the LCD port was `x27`; one can check by using the command - 

```
tushar@raspberrypi:~ $ i2cdetect -y 1
 0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- 23 -- -- 26 27 -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --                         
tushar@raspberrypi:~ $ 
```

> I have 3 LCDs connected, it shows ports `x23,26 & 27`.

`i2c` uses SDA and SCL to transmit data, and there is only 1 such pin in the Raspberry Pi (GPIO 2 & GPIO 3). To individually manage each LCD, it should listen to three different ports. This can be done by soldering `A0, A1 & A2` pins. Check out the image.

[LCD PORT MANIPULATION](/what-i-learnt/assets/i2c_port.jpeg)

This refers to a `3-bit` offset; if two boxes in column `A2` are soldered, it represents binary `100` (the index number represents the position; the two in A2 mean the second position from the right, starting with zero). Since `100` is `4` in decimal, the new port is `27-4=23.`

### 3.5" RPI Screen

The RPI screen is a plug-and-play screen; the display will latch comfortably onto the RPI GPIO pins. Since I had other wires from LCDs, I had to use jumper wires to connect them. Now I can take 26 wires and connect each pin, but then what's the fun in that. Refer to the excellent wiki -

[LCD Wiki](http://www.lcdwiki.com/3.5inch_RPi_Display)

The section on the interface refers to the mapping and function of each pin. I used only one power connection, one ground, and multiple pins except those categorized as `NC` or `Not Connected.` I didn't need the touch ability, so I removed the touch-related pins, but the display didn't work, so I added them back.

### `kivy`

I find `tkinter` not so fancy. ``qt` is fancy but too complicated. Here comes `kivy`!! I am surprised that I didn't know about kivy, an excellent library for creating desktop apps.

[Kivy Documentation](https://kivy.org/doc/stable/)

The project has been a roller coaster ride for almost a month, but it felt worth it when I completed it today.

Now, I'm moving on to another project: converting my LED strips to a grow light specification. Until then, have fun!!