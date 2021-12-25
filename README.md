# CircuitPython_Seeduino_XIAO_RP2040
The firmware version of CircuitPython for the Raspberry Pi Pico does not work properly for the XIAO RP2040.
There are some pinout settings that are different.

On https://github.com/adafruit/circuitpython, you can see that the card is already inserted into the ports, but for some reason, we didn’t find the firmware already compiled to download.

I managed to compile the firmware properly by following the steps described in Build CircuitPython | Building CircuitPython | Adafruit Learning System

In the “BUILD CIRCUIT PYTHON” step, you should go to the ports/raspberrypi directory ports/raspberrypi

cd ports/raspberrypi 
make BOARD=seeeduino_xiao_rp2040

The compiled firmware will be in the build-seeeduino_xiao_rp2040 folder.
