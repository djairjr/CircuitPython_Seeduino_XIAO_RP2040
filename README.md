I decided to write this tutorial because the manufacturer's website contains some errors regarding the use of this board with Circuitpython. Although the site contains the information that the firmware for the board is the same as that of the Raspberry Pi Pico, I noticed that some features were not present. Especially when we use the expansion board. The examples on the Seeed Wiki are mostly suitable for use with the Arduino IDE. But the examples for Circuitpython are not well documented.

**Supplies**
1. Seeeduino XIAO Rp2040
2. Seeeduino XIAO Expansion Board
3. Circuitpython Source


**Step 1: Unboxing... I2C Not Working?**

As soon as the card arrived, I installed the firmware version for Raspberry Pi Pico, according to the manufacturer's guidelines. As I had already acquired the expansion card before, I started my tests trying to display text on the Oled display (I wrote another tutorial about it). When I made the call to the I2C bus, I received an error message informing me of the absence of Pullups on the SDA and SCL lines. I tested several resistor values, going from 4K7 to 47k and none of them worked. It was then that I visited the Seeed support forum and found a user commenting on the same issues with the I2C bus, but this time using the Arduino IDE. The user's message informed them that they had changed the native pinouts for I2C and this had allowed them to use the bus. The same user commented in another post about the board-specific Circuitpython firmware.

I ended up putting the two ideas together and did a search on Circuitpython's Github to check for a Port for the board. For some reason, the Port for Xiao RP2040 is ready in the source code, but has not yet been made available for download on the site. So to get the firmware for the card, you'll need to compile from scratch (if you don't want to venture into this step, I made the firmware compiled on my Github available).

**Step 2: Compiling the Firmware**

The Adafruit team has already made an excellent tutorial on this step.

https://learn.adafruit.com/building-circuitpython/build-circuitpython

You'll just need to adapt a few things:

In the Adafruit tutorial, in the Build Circuitpython step, you'll need to go to the correct port folder.

In our case, instead of

```
cd ports/atmel-samd 
make BOARD=circuitplayground_express 
```

you must type:

```
cd ports/raspberrypi
make BOARD=seeeduino_xiao_rp2040
```
The later steps are basically the same. The compiled firmware will stay in the folder ports/raspberrypi/build-seeeduino_xiao_rp2040/firmware.uf2

Simply reset XIAO RP2040 with the BOOT button pressed and then upload the firmware.

Now is the time to run some tests on the board. First the OLED display.

**Step 3: Oled Display**

The OLED display is accessed via the I2C bus, at the 0x3C, just like in my other tutorial.

First, download the circuitpython-specific libraries in https://circuitpython.org/libraries

You will need to copy some libraries (adafruit_displayio_ssd1306, adafruit_bus_device, adafruit_bitmap_font, adafruit_display_text, adafruit_display_notification, adafruit_display_shapes, adafruit_displayio_layouts) to your card's lib folder.

```
import board
import busio
import displayio
import terminalio
import adafruit_displayio_ssd1306
from adafruit_display_text import label

displayio.release_displays()
i2c = busio.I2C (scl=board.SCL, sda=board.SDA)

display_bus = displayio.I2CDisplay (i2c, device_address = 0x3C) # The address of my Board

display = adafruit_displayio_ssd1306.SSD1306(display_bus, width=128, height=64)
splash = displayio.Group() # no max_size needed
display.show(splash)

color_bitmap = displayio.Bitmap(128, 64, 1) # Full screen white
color_palette = displayio.Palette(1)
color_palette[0] = 0xFFFFFF  # White
 
bg_sprite = displayio.TileGrid(color_bitmap, pixel_shader=color_palette, x=0, y=0)
splash.append(bg_sprite)
 
# Draw a smaller inner rectangle
inner_bitmap = displayio.Bitmap(118, 54, 1)
inner_palette = displayio.Palette(1)
inner_palette[0] = 0x000000  # Black
inner_sprite = displayio.TileGrid(inner_bitmap, pixel_shader=inner_palette, x=5, y=4)
splash.append(inner_sprite)
 
# Draw a label
text = "Nicolau dos"
text_area = label.Label(terminalio.FONT, text=text, color=0xFFFF00, x=28, y=15)
splash.append(text_area)

text = "Brinquedos"
text_area = label.Label(terminalio.FONT, text=text, color=0xFFFF00, x=32, y=25)
splash.append(text_area)
 
while True:
    pass

```

**Step 4: NEOPIXEL Led**

It's time to turn on the lights! First let's use the Adafruit example, with the changes to work with the board.

Adapted from https://learn.adafruit.com/circuitpython-essentials

```
import time
import board
from rainbowio import colorwheel
import neopixel

pixel_pin = board.NEOPIXEL #The neopixel can be accessed in this way
num_pixels = 1 #only one pixel

pixels = neopixel.NeoPixel(pixel_pin, num_pixels, brightness=0.3, auto_write=False)


def color_chase(color, wait):
    for i in range(num_pixels):
        pixels[i] = color
        time.sleep(wait)
        pixels.show()
    time.sleep(0.5)


def rainbow_cycle(wait):
    for j in range(255):
        for i in range(num_pixels):
            rc_index = (i * 256 // num_pixels) + j
            pixels[i] = colorwheel(rc_index & 255)
        pixels.show()
        time.sleep(wait)


RED = (255, 0, 0)
YELLOW = (255, 150, 0)
GREEN = (0, 255, 0)
CYAN = (0, 255, 255)
BLUE = (0, 0, 255)
PURPLE = (180, 0, 255)

while True:
    pixels.fill(RED)
    pixels.show()
    # Increase or decrease to change the speed of the solid color change.
    time.sleep(1)
    pixels.fill(GREEN)
    pixels.show()
    time.sleep(1)
    pixels.fill(BLUE)
    pixels.show()
    time.sleep(1)

    color_chase(RED, 0.1)  # Increase the number to slow down the color chase
    color_chase(YELLOW, 0.1)
    color_chase(GREEN, 0.1)
    color_chase(CYAN, 0.1)
    color_chase(BLUE, 0.1)
    color_chase(PURPLE, 0.1)

    rainbow_cycle(0)  # Increase the number to slow down the rainbow
```

**Step 5: Red Green Blue Leds**

The board has three colored LEDs, which can be accessed in this way:

```
import time
import board
from digitalio import DigitalInOut, Direction, Pull

red = DigitalInOut(board.LED_RED)
green = DigitalInOut(board.LED_GREEN)
blue = DigitalInOut(board.LED_BLUE)

red.direction = Direction.OUTPUT
green.direction = Direction.OUTPUT
blue.direction = Direction.OUTPUT

while True:
    blue.value = not blue.value
    time.sleep (0.2)
    red.value = not red.value
    time.sleep (0.2)
    green.value = not green.value
    time.sleep (0.2)
```

We could test the User button too... (adapted from https://learn.adafruit.com/circuitpython-essentials)

```
import time
import board
from digitalio import DigitalInOut, Direction, Pull
red = DigitalInOut(board.LED_RED)

red.direction = Direction.OUTPUT

switch = DigitalInOut(board.D1) #Expansion Board User Button
switch.direction = Direction.INPUT
switch.pull = Pull.UP

while True:
    red.value = not switch.value
    
```

**Step 6: Buzzer**

Here a scale of notes using the Buzzer (pin A3).

```
import time
import board
import pwmio

piezo = pwmio.PWMOut(board.A3, duty_cycle=0, frequency=440, variable_frequency=True)

while True:
    for f in (262, 294, 330, 349, 392, 440, 494, 523):
        piezo.frequency = f
        piezo.duty_cycle = 65535 // 2  # On 50%
        time.sleep(0.25)  # On for 1/4 second
        piezo.duty_cycle = 0  # Off
        time.sleep(0.05)  # Pause between notes
    time.sleep(0.5)
```

This also plays an mp3 file.

The buzzer does not play the mp3 properly, but the code works. https://learn.adafruit.com/circuitpython-essentials/circuitpython-mp3-audio

```
import board
import digitalio

from audiomp3 import MP3Decoder

try:
    from audioio import AudioOut
except ImportError:
    try:
        from audiopwmio import PWMAudioOut as AudioOut
    except ImportError:
        pass  # not always supported by every board!

button = digitalio.DigitalInOut(board.D1)
button.switch_to_input(pull=digitalio.Pull.UP)

# The listed mp3files will be played in order
mp3files = ["begins.mp3", "xfiles.mp3"]

# You have to specify some mp3 file when creating the decoder
mp3 = open(mp3files[0], "rb")
decoder = MP3Decoder(mp3)
audio = AudioOut(board.A3)

while True:
    for filename in mp3files:
        # Updating the .file property of the existing decoder
        # helps avoid running out of memory (MemoryError exception)
        decoder.file = open(filename, "rb")
        audio.play(decoder)
        print("playing", filename)

        # This allows you to do other things while the audio plays!
        while audio.playing:
            pass

        print("Waiting for button press to continue!")
        while button.value:
            pass
```
**Step 7: SD Card Reader**

The expansion board also has an SD Card interface, using SPI. 
In the XIAO RP2040 diagram, the CS pin of the SPI interface is pin D7. 
But in the Expansion Board, the indicated pin is D2 (which is the correct one).

Don't forget to copy the adafruit_sdcard library to your /lib folder.

```
import os

import adafruit_sdcard
import board
import busio
import digitalio
import storage

# In XIAO EXPANSION BOARD, SPI CS is D2 Pin
SD_CS = board.D2

# Connect to the card and mount the filesystem.

spi = busio.SPI(board.SCK, board.MOSI, board.MISO)
cs = digitalio.DigitalInOut(SD_CS)
sdcard = adafruit_sdcard.SDCard(spi, cs)
vfs = storage.VfsFat(sdcard)
storage.mount(vfs, "/sd")

def print_directory(path, tabs=0):
    for file in os.listdir(path):
        stats = os.stat(path + "/" + file)
        filesize = stats[6]
        isdir = stats[0] & 0x4000

        if filesize < 1000:
            sizestr = str(filesize) + " by"
        elif filesize < 1000000:
            sizestr = "%0.1f KB" % (filesize / 1000)
        else:
            sizestr = "%0.1f MB" % (filesize / 1000000)

        prettyprintname = ""
        for _ in range(tabs):
            prettyprintname += "   "
        prettyprintname += file
        if isdir:
            prettyprintname += "/"
        print('{0:<40} Size: {1:>10}'.format(prettyprintname, sizestr))

        # recursively print directory contents
        if isdir:
            print_directory(path + "/" + file, tabs + 1)

print("Files on filesystem:")
print("====================")
print_directory("/sd")

```

This simple script just show whatever is your SDCard in REPL.

**Step 8: Real Time Clock**

And here we can see the RTC working.

Copy the adafruit_pcf8563 and adafruit_register libraries to your lib folder.

Terminal test:

```
import time
import board
import busio
import adafruit_pcf8563

i2c_bus = busio.I2C(board.SCL, board.SDA)

rtc = adafruit_pcf8563.PCF8563(i2c_bus)

days = ("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday")

# pylint: disable-msg=using-constant-test
if True:  # change to False if you don't want to set the time!

    #                     year, mon, date, hour, min, sec, wday, yday, isdst

    t = time.struct_time((2021, 12, 26, 11, 22, 0, 0, -1, -1))

    # you must set year, mon, date, hour, min, sec and weekday

    # yearday is not supported, isdst can be set but we don't do anything with it at this time

    print("Setting time to:", t)  # uncomment for debugging

    rtc.datetime = t

    print()

# pylint: enable-msg=using-constant-test



# Main loop:

while True:

    if rtc.datetime_compromised:

        print("RTC unset")

    else:

        print("RTC reports time is valid")

    t = rtc.datetime

    # print(t)     # uncomment for debugging

    print(

        "{} {}/{}/{}".format(

            days[int(t.tm_wday)], t.tm_mday, t.tm_mon, t.tm_year

        )

    )

    print("{}:{:02}:{:02}".format(t.tm_hour, t.tm_min, t.tm_sec))

    time.sleep(1)  # wait a second
```

Then on the OLED display.

```
import time
import displayio
import terminalio
import busio
import board
import adafruit_displayio_ssd1306
import adafruit_pcf8563

from adafruit_display_text import label

displayio.release_displays()

i2c = busio.I2C(scl=board.SCL, sda=board.SDA)

font = terminalio.FONT

display_bus = displayio.I2CDisplay(i2c, device_address=0x3C)  # The address of my Board
rtc = adafruit_pcf8563.PCF8563(i2c)

oled = adafruit_displayio_ssd1306.SSD1306(display_bus, width=128, height=64)

while True:
    current = rtc.datetime

    hour = current.tm_hour % 12
    if hour == 0:
        hour = 12

    am_pm = "AM"
    if current.tm_hour / 12 >= 1:
        am_pm = "PM"

    time_display = "{:d}:{:02d}:{:02d} {}".format(hour, current.tm_min, current.tm_sec, am_pm)
    date_display = "{:d}/{:d}/{:d}".format(current.tm_mon, current.tm_mday, current.tm_year)
    text_display = "CircuitPython Time"

    clock = label.Label(font, text=time_display)
    date = label.Label(font, text=date_display)
    text = label.Label(font, text=text_display)

    (_, _, width, _) = clock.bounding_box
    clock.x = oled.width // 2 - width // 2
    clock.y = 5

    (_, _, width, _) = date.bounding_box
    date.x = oled.width // 2 - width // 2
    date.y = 15

    (_, _, width, _) = text.bounding_box
    text.x = oled.width // 2 - width // 2
    text.y = 25

    watch_group = displayio.Group()
    watch_group.append(clock)
    watch_group.append(date)
    watch_group.append(text)

    oled.show(watch_group)
```

**Step 9: Weather Station**
Finally, I included a Humidity and Temperature Sensor, DHT11 and with some changes in the previous code, I managed to set up a small meteorological station.

Plug your DHT11 sensor into the Groove connector on the expansion board, at pin D0.

Just save the file to your CIRCUITPY drive with the name code.py. So it will run every time the system starts up.
I started from the example indicated in the link:

https://learn.adafruit.com/dht/dht-circuitpython-code

```
import time
import displayio
import terminalio
import busio
import board
import adafruit_displayio_ssd1306
import adafruit_pcf8563
import adafruit_dht

from adafruit_display_text import label

displayio.release_displays()

i2c = busio.I2C(scl=board.SCL, sda=board.SDA)

font = terminalio.FONT

display_bus = displayio.I2CDisplay(i2c, device_address=0x3C)  # The address of my Board
rtc = adafruit_pcf8563.PCF8563(i2c)
dht = adafruit_dht.DHT11(board.D0)

oled = adafruit_displayio_ssd1306.SSD1306(display_bus, width=128, height=64)

while True:
    current = rtc.datetime

    try:
        temp_data = dht.temperature
        hum_data = dht.humidity
        old_temp = temp_data
        old_hum = hum_data
    except RuntimeError as e:
        temp_data = old_temp
        hum_data = old_hum

    hour = current.tm_hour % 12
    if hour == 0:
        hour = 12

    am_pm = "AM"
    if current.tm_hour / 12 >= 1:
        am_pm = "PM"

    time_display = "{:d}:{:02d}:{:02d} {}".format(hour, current.tm_min, current.tm_sec, am_pm)
    date_display = "{:d}/{:d}/{:d}".format(current.tm_mon, current.tm_mday, current.tm_year)
    temp_display = "Temp: {:.1f} *C".format (temp_data)
    hum_display =  "Umid: {}%".format (hum_data)

    clock = label.Label(font, text=time_display)
    date = label.Label(font, text=date_display)
    temp = label.Label(font, text=temp_display)
    hum = label.Label(font, text=hum_display)

    (_, _, width, _) = date.bounding_box
    date.x = oled.width // 2 - width // 2
    date.y = 9

    (_, _, width, _) = clock.bounding_box
    clock.x = oled.width // 2 - width // 2
    clock.y = 19

    (_, _, width, _) = temp.bounding_box
    temp.x = oled.width // 2 - width // 2
    temp.y = 29

    (_, _, width, _) = hum.bounding_box
    hum.x = oled.width // 2 - width // 2
    hum.y = 39

    watch_group = displayio.Group()
    watch_group.append(clock)
    watch_group.append(date)
    watch_group.append(temp)
    watch_group.append(hum)

    oled.show(watch_group)
```
