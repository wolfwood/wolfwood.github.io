---
layout: post
---
# {{ page.title }}

This post proposes a modification to the improved square keyboard matrix in order to support a low power wake-on-keypress mode, which has been a deficiency of this design until now.

## Matrix Backstory
![an illustration of a keyboard matrix, borrowed from dovenyi's KBD.news duplex matrix explainer](/images/2022-13-08/keyboard-matrix-6x2-col2row.png)
**an illustration of a keyboard matrix, borrowed from [dovenyi's KBD.news duplex matrix explainer](https://kbd.news/The-Japanese-duplex-matrix-1391.html)**

A keyboard matrix is a way of detecting key presses without wiring each switch to its own GPIO pin on the micrcocontroller. This is important because pins are a somewhat scarce resource, and when higher pin count versions of a given chip are available they are more expensive. The basic approach is to arrange switches in a grid with one side connected to a row wire and one to a column wire. One pin is assigned to each row and column wire. Either rows or columns are chosen as inputs and the other as outputs. One output pin is set high at a time and then the inputs are read to detect closed switches (keypresses). This is known as scanning the matrix. To properly detect multiple simultaneous keypresses, each switch must be wired in series with a diode so that current can't flow "backward" through switches assigned to different output pins than the currently active one, which could otherwise create ghost keypresses.

## The Matrix, Evolved
{% include image.html url="/images/2022-13-08/4-pin-improved-square-matrix.png"  description="an improved square matrix, from [KBD.news](https://kbd.news/Improved-square-matrix-1415.html)" height="500" align="right" %}

The [improved square (or round-robin) matrix](https://kbd.news/Improved-square-matrix-1415.html) is a method of sensing switch activations using the fewest number of GPIO pins possible (as far as we know). The key insight is that rather than statically assigning pins as inputs or outputs (and rows and columns), we can dynamically reconfigure them to serve both roles, with the caveat that a switch cannot have the same pin as both input and output. The extra connectedness, compared to a traditional matrix also means more opportunities for ghosting ([see the precursor write up on the round robin matrix for details](https://kbd.news/Square-or-round-robin-matrix-1400.html)). This can be addressed with larger voltage drops either by using diodes with higher values, or by doubling up on the 1N4148 diodes traditionally used. The improved version observes that rather than using 2 diodes per switch, we can 'factor out' one diode and instead use a one diode per GPIO pin, between input (or column, in these diagrams) segment and the output (or row) segment.

## Saving Power with Interrupt Mode
Pete Johanson raised the difficulty in supporting a low power keyboard mode at the time of the original KBD.news post about the round robin matrix. He has since expanded on it as part of his [excellent piece on designing keyboards for wireless connectivity and low power use](https://kbd.news/Designing-for-Wireless-1784.html).

Power use is a significant concern for battery-powered wireless keyboards, which aim to extend their bat;tery life as long as possible. Keyboards spend a significant amount of time idle and therefore signficant power can be saved by putting the CPU to sleep. However, there must be a way to detect when a key is pressed and wake a sleeping keyboard controller back up.

The approach used on a traditional matrix, which is illustrated in detail in Pete's post, is to enable interrupts on the input pins and to set _all_ output pins to high at once. This ensures that if any key is pressed, the CPU will be awoken. Note that because all the output pins are high, the interrupt will not be informative as to which key was pressed, so the keyboard must resume scanning the matrix as usual. Interrupts are not a substitute for scanning a matrix, just a means to resume it.

## Interrupt Mode for the Improved Square Matrix
With the improved square matrix, its easy to see how to listen for keypresses: we must treat all pins as inputs, pulling them low and enabling interrupts. However, this leaves us unsure about the other part: how do we connect the other side of the switches to a detectable voltage?

![the left side of the schematic shows my proposed addition to the square matrix circuit](/images/2022-13-08/4-pin-improved-square-matrix-low-power.png){:  height="500"}
**the left side of the schematic shows my proposed addition to the square matrix circuit**

The improved matrix makes this possible by separating the output segments (rows in this illustration) from the input segments (columns) by a diode, with the GPIO pins on the input side. We can add a single additional pin to supply a voltage to all the output segments, using diodes to prevent them from from becoming connected during normal matrix scanning.

With the additional pin pulled low, scanning can be done as normal. When it is time for the CPU to sleep, we pull the new pin high and configure the other pins as low inputs with interrupts enabled. Any keypress will wake the CPU, as with the traditional matrix low power mode.

We also don't need to worry that the new diodes will drop the voltage below the voltage threshold, as they subsitute for the diodes the improved matrix introduced to combat ghosting. If the matrix registers keypresses correctly, then the interrupt mode will as well.

I've written this post to get feedback on any potential deficiencies with the interrupt mode proposal, and to share this idea to help us save both pins and power. I hope we will all treat power-savings as a goal, even on our wired keyboards.

#### Coda: A Personal Plea for More Pins
I hope that microcontroller board designers will not use the efficiencies granted by the improved square matrix as justification to produce boards with even fewer pins broken out. Being able to direct wire switches for handwired split keyboards has made experimentation significantly easier, but folks keep sticking new MCUs into tiny footprints. I understand the desire to retain compatibility, but I hope people take inspiration from the Proton-C and break out addition GPIOs on a snap-off extension (sadly, the Proton-C still only exposes 23 of 37 GPIOs). Matrix tricks are cool, but not as cool as never needing to use one :)
