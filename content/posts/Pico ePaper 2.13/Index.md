+++
title = "Pi Pico 2W with Waveshare Pico ePaper 2.13"
date = 2026-04-19
draft = true
+++

I recently picked up a Pico-ePaper-2.13 from Waveshare. This is a great, small e-ink display with a fast refresh time designed for interfacing with the Pi Pico. I want to make a few things with this, but I'm going to start out making a clock/weather widget display for your desk. I'm using a Pi Pico 2W for the extra ooomph, altho a Pi Pico 1W would work for this application.

![Pi Pico 2W ePaper 2.13 clock widget](title.jpg)

Parts list:

[Pi Pico 2W](https://www.adafruit.com/product/6315)

[Waveshare Pico ePaper 2.13](https://www.waveshare.com/pico-epaper-2.13.htm)

I'm not going to include wiring diagrams or schematics on this specific project, this ePaper device is ready for the Pi Pico out of the box. As long as the Pi Pico has the headers soldered on, you can just fit the ePaper to the microcontroller and it's ready for code. Be sure to pay attention to the solder mask on the ePaper, it has a "USB" marking for orienting the Pi Pico correctly. I'm coding with MicroPython using the IDE Thonny.

![USB Alignment mark](pico-ePaper-USB.jpg)

To start the project, I'm going to need a free API that I can pull weather data from for my location. I've done some testing with this display, there are text fonts that exist, but they are all small. These fonts will not be readable from a distance, so I will need to make a custom font for the larger letters. I'm also gonna use a time API to check up on the accuracy of the Pi Pico every hour. The free API for the weather data is from Open Meteo.

First thing to do code-wise is to make the bitmaps for the large size letters/numbers. I formatted the numerals so that it is easy to see the bits that are mapped to the screen:

```Python
DIGIT_BITMAPS = {
    '0': [0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b00000,
          0b10001,
          0b10001,
          0b10001,
          0b11111],
    '1': [0b00100,
          0b01100,
          0b00100,
          0b00100,
          0b00000,
          0b00100,
          0b00100,
          0b00100,
          0b01110],
    '2': [0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111,
          0b10000,
          0b10000,
          0b10000,
          0b11111],
    '3': [0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111],
    '4': [0b10001,
          0b10001,
          0b10001,
          0b10001,
          0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b00001],
    '5': [0b11111,
          0b10000,
          0b10000,
          0b10000,
          0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111],
    '6': [0b11111,
          0b10000,
          0b10000,
          0b10000,
          0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b11111],
    '7': [0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b00000,
          0b00001,
          0b00001,
          0b00001,
          0b00001],
    '8': [0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b11111],
    '9': [0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111],
}
```

If you look at the ones in the field of zeroes, you can see the numerals drawn out. I also made bitmaps for the letters of the alphabet, but didn't do the fancy formatting. You can if you wanna tho.


```Python
LETTER_BITMAPS = {
    'A': [0b01110,0b10001,0b10001,0b10001,0b11111,0b10001,0b10001,0b10001,0b10001],
    'B': [0b11110,0b10001,0b10001,0b10001,0b11110,0b10001,0b10001,0b10001,0b11110],
    'C': [0b01111,0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b01111],
    'D': [0b11110,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b11110],
    'E': [0b11111,0b10000,0b10000,0b10000,0b11110,0b10000,0b10000,0b10000,0b11111],
    'F': [0b11111,0b10000,0b10000,0b10000,0b11110,0b10000,0b10000,0b10000,0b10000],
    'G': [0b01111,0b10000,0b10000,0b10000,0b10011,0b10001,0b10001,0b10001,0b01111],
    'H': [0b10001,0b10001,0b10001,0b10001,0b11111,0b10001,0b10001,0b10001,0b10001],
    'I': [0b01110,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100,0b01110],
    'J': [0b00111,0b00010,0b00010,0b00010,0b00010,0b00010,0b10010,0b10010,0b01100],
    'K': [0b10001,0b10010,0b10100,0b11000,0b10000,0b11000,0b10100,0b10010,0b10001],
    'L': [0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b11111],
    'M': [0b10001,0b11011,0b10101,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001],
    'N': [0b10001,0b11001,0b10101,0b10011,0b10001,0b10001,0b10001,0b10001,0b10001],
    'O': [0b01110,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b01110],
    'P': [0b11110,0b10001,0b10001,0b10001,0b11110,0b10000,0b10000,0b10000,0b10000],
    'Q': [0b01110,0b10001,0b10001,0b10001,0b10001,0b10101,0b10011,0b10001,0b01111],
    'R': [0b11110,0b10001,0b10001,0b10001,0b11110,0b11000,0b10100,0b10010,0b10001],
    'S': [0b01111,0b10000,0b10000,0b10000,0b01110,0b00001,0b00001,0b00001,0b11110],
    'T': [0b11111,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100],
    'U': [0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b01110],
    'V': [0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b01010,0b01010,0b00100],
    'W': [0b10001,0b10001,0b10001,0b10001,0b10001,0b10101,0b10101,0b11011,0b10001],
    'X': [0b10001,0b10001,0b01010,0b01010,0b00100,0b01010,0b01010,0b10001,0b10001],
    'Y': [0b10001,0b10001,0b01010,0b01010,0b00100,0b00100,0b00100,0b00100,0b00100],
    'Z': [0b11111,0b00001,0b00010,0b00100,0b01000,0b01000,0b10000,0b10000,0b11111],
}
```

![Getting Weather](fetch-weather.jpg)

The weather API from Open Meteo requires your lattitude and longitude, timezone, what specific info you want from them, and your preferred temperature format (C or F). It will return a "weather code", which can be translated into the current weather conditions. This is the translation layer for the weather API output:

```Python
WMO = {
    0:"Clear", 1:"Mostly Clear", 2:"Partly Cloudy", 3:"Overcast",
    45:"Foggy", 48:"Icy Fog",
    51:"Lt Drizzle", 53:"Drizzle", 55:"Hvy Drizzle",
    61:"Lt Rain", 63:"Rain", 65:"Hvy Rain",
    71:"Lt Snow", 73:"Snow", 75:"Hvy Snow", 77:"Sleet",
    80:"Showers", 81:"Showers", 82:"Hvy Showers",
    85:"Snow Showers", 86:"Hvy Snow Showers",
    95:"Thunderstorm", 96:"T-storm+Hail", 99:"Hvy T-storm",
}
```

![Clock Start](clock-widget-start.jpg)

This should take care of the majority of the code, there will be a little bit more to tie it all together. The clock display will update every minute. We're storing the time in the Pico, but checking every hour that we haven't drifted off too badly. We are going to hit the weather API every 15 minutes to update the latest temp and forecast. The Open Meteo API has a limit of 10,000 requests per day, however I want to limit my usage even further to be kind to this free API provider. The full code is as follows:

```Python
"""
Clock Widget - Waveshare 2.13" E-ink Display (Landscape Mode)
Hardware: Raspberry Pi Pico 2W
Canvas:   250x128px landscape

Layout:
  Left  (~148px): Large segmented clock  +  day/date below
  Right (~100px): Current weather summary

The clock uses a hand-drawn 5x9 pixel digit font scaled 3x
so each digit is 15x27px — readable at arm's length.
Colon separators are 5px wide. AM/PM sit beside the time.

APIs:
  - Time:    worldtimeapi.org  (synced on boot, re-synced hourly)
  - Weather: Open-Meteo       (free, no key)

Location: Nashville, TN — change LAT/LON to your coords.
"""

from machine import Pin, SPI, RTC
import framebuf
import utime
import network
import time

# ── Config ─────────────────────────────────────────────────
WIFI_SSID     = "Your SSID"
WIFI_PASSWORD = "Your Password"

LATITUDE   = 36.1628
LONGITUDE  = -85.5016
TEMP_UNIT  = "fahrenheit"
TEMP_SUFFIX = "F"

PARTIAL_INTERVAL = 60    # seconds — redraw every minute for the clock
DARK_MODE = True         # True = black background, white text

# ── E-Paper Constants ───────────────────────────────────────
EPD_WIDTH  = 250
EPD_HEIGHT = 128
RST_PIN  = 12
DC_PIN   = 8
CS_PIN   = 9
BUSY_PIN = 13
FULL_UPDATE = 0
PART_UPDATE = 1

lut_full_update = [
    0x80,0x60,0x40,0x00,0x00,0x00,0x00,
    0x10,0x60,0x20,0x00,0x00,0x00,0x00,
    0x80,0x60,0x40,0x00,0x00,0x00,0x00,
    0x10,0x60,0x20,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,0x00,0x00,
    0x03,0x03,0x00,0x00,0x02,
    0x09,0x09,0x00,0x00,0x02,
    0x03,0x03,0x00,0x00,0x02,
    0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,
    0x15,0x41,0xA8,0x32,0x30,0x0A,
]
lut_partial_update = [
    0x00,0x00,0x00,0x00,0x00,0x00,0x00,
    0x80,0x00,0x00,0x00,0x00,0x00,0x00,
    0x40,0x00,0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,0x00,0x00,
    0x0A,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,0x00,0x00,0x00,
    0x15,0x41,0xA8,0x32,0x30,0x0A,
]

# ── EPD Driver ──────────────────────────────────────────────
class EPD_2in13(framebuf.FrameBuffer):
    def __init__(self):
        self.reset_pin = Pin(RST_PIN, Pin.OUT)
        self.busy_pin  = Pin(BUSY_PIN, Pin.IN, Pin.PULL_UP)
        self.cs_pin    = Pin(CS_PIN, Pin.OUT)
        self.dc_pin    = Pin(DC_PIN, Pin.OUT)
        self.spi = SPI(1)
        self.spi.init(baudrate=4000_000)

        self.width    = EPD_WIDTH
        self.height   = EPD_HEIGHT
        self.fb_width = 256          # padded to multiple of 8
        self.buffer   = bytearray(self.fb_width * self.height // 8)
        super().__init__(self.buffer, self.fb_width, self.height, framebuf.MONO_HLSB)

        self.full_lut    = lut_full_update
        self.partial_lut = lut_partial_update
        self.full_update = FULL_UPDATE
        self.part_update = PART_UPDATE
        self.init(FULL_UPDATE)

    def _cmd(self, c):
        self.dc_pin.value(0); self.cs_pin.value(0)
        self.spi.write(bytearray([c])); self.cs_pin.value(1)

    def _dat(self, d):
        self.dc_pin.value(1); self.cs_pin.value(0)
        self.spi.write(bytearray([d]) if isinstance(d, int) else bytearray(d))
        self.cs_pin.value(1)

    def _wait(self):
        while self.busy_pin.value() == 1:
            utime.sleep_ms(10)

    def reset(self):
        self.reset_pin.value(1); utime.sleep_ms(50)
        self.reset_pin.value(0); utime.sleep_ms(2)
        self.reset_pin.value(1); utime.sleep_ms(50)

    def _on_full(self):
        self._cmd(0x22); self._dat(0xC7); self._cmd(0x20); self._wait()

    def _on_part(self):
        self._cmd(0x22); self._dat(0x0C); self._cmd(0x20); self._wait()

    def init(self, update):
        self.reset()
        if update == FULL_UPDATE:
            self._wait()
            self._cmd(0x12); self._wait()
            self._cmd(0x74); self._dat(0x54)
            self._cmd(0x7E); self._dat(0x3B)
            self._cmd(0x01)
            self._dat(0xF9); self._dat(0x00); self._dat(0x00)
            self._cmd(0x11); self._dat(0x03)
            self._cmd(0x44); self._dat(0x00); self._dat(0x0F)
            self._cmd(0x45)
            self._dat(0x00); self._dat(0x00); self._dat(0xF9); self._dat(0x00)
            self._cmd(0x3C); self._dat(0x03)
            self._cmd(0x2C); self._dat(0x55)
            self._cmd(0x03); self._dat(self.full_lut[70])
            self._cmd(0x04)
            self._dat(self.full_lut[71]); self._dat(self.full_lut[72]); self._dat(self.full_lut[73])
            self._cmd(0x3A); self._dat(self.full_lut[74])
            self._cmd(0x3B); self._dat(self.full_lut[75])
            self._cmd(0x32)
            for b in self.full_lut[:70]: self._dat(b)
            self._cmd(0x4E); self._dat(0x00)
            self._cmd(0x4F); self._dat(0x00); self._dat(0x00)
            self._wait()
        else:
            self._cmd(0x2C); self._dat(0x26); self._wait()
            self._cmd(0x32)
            for b in self.partial_lut[:70]: self._dat(b)
            self._cmd(0x37)
            for b in [0x00,0x00,0x00,0x00,0x40,0x00,0x00]: self._dat(b)
            self._cmd(0x22); self._dat(0xC0); self._cmd(0x20); self._wait()
            self._cmd(0x3C); self._dat(0x01)

    def _rotate(self):
        """Rotate 250x128 landscape framebuf 90° CW → 128x250 portrait panel buffer."""
        src_fw = self.fb_width   # 256 (padded row stride)
        src_h  = self.height     # 128
        src_w  = self.width      # 250 (real columns)
        bpr    = 16              # dest bytes per row (128px / 8)
        out    = bytearray(b'\xff' * (bpr * 250))
        for sy in range(src_h):
            for sx in range(src_w):
                idx = sy * src_fw + sx
                pixel = (self.buffer[idx >> 3] >> (7 - (idx & 7))) & 1
                dx = src_h - 1 - sy
                dy = sx
                di = dy * bpr + (dx >> 3)
                if pixel:
                    out[di] |=  (0x80 >> (dx & 7))
                else:
                    out[di] &= ~(0x80 >> (dx & 7))
        return out

    def display(self):
        buf = self._rotate()
        self._cmd(0x24); self._dat(buf); self._on_full()

    def display_base(self):
        buf = self._rotate()
        self._cmd(0x24); self._dat(buf)
        self._cmd(0x26); self._dat(buf)
        self._on_full()

    def display_partial(self):
        buf = self._rotate()
        self._cmd(0x24); self._dat(buf)
        self._cmd(0x26); self._dat(buf)
        self._on_part()

    def sleep(self):
        self._cmd(0x10); self._dat(0x03)
        utime.sleep_ms(2000)
        self.reset_pin.value(0)


# ── Large Digit Font ────────────────────────────────────────
# Each digit is a 5-wide × 9-tall bitmap (list of 9 rows, each row is
# a 5-bit integer, MSB = leftmost pixel).
# Scaled 3× → 15×27px per digit. Colon is 3×27.
#
# Segment map (viewed as a 5×9 grid):
#   Row 0:     top bar
#   Rows 1-3:  upper verticals
#   Row 4:     middle bar
#   Rows 5-7:  lower verticals
#   Row 8:     bottom bar
#
DIGIT_BITMAPS = {
    '0': [0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b00000,
          0b10001,
          0b10001,
          0b10001,
          0b11111],
    '1': [0b00100,
          0b01100,
          0b00100,
          0b00100,
          0b00000,
          0b00100,
          0b00100,
          0b00100,
          0b01110],
    '2': [0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111,
          0b10000,
          0b10000,
          0b10000,
          0b11111],
    '3': [0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111],
    '4': [0b10001,
          0b10001,
          0b10001,
          0b10001,
          0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b00001],
    '5': [0b11111,
          0b10000,
          0b10000,
          0b10000,
          0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111],
    '6': [0b11111,
          0b10000,
          0b10000,
          0b10000,
          0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b11111],
    '7': [0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b00000,
          0b00001,
          0b00001,
          0b00001,
          0b00001],
    '8': [0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b11111],
    '9': [0b11111,
          0b10001,
          0b10001,
          0b10001,
          0b11111,
          0b00001,
          0b00001,
          0b00001,
          0b11111],
}

DIGIT_W = 5   # bitmap columns
DIGIT_H = 9   # bitmap rows
SCALE   = 4   # pixel scale factor  (was 3 → now 4)
CELL_W  = DIGIT_W * SCALE   # 20px per digit
CELL_H  = DIGIT_H * SCALE   # 36px per digit

# ── A-Z bitmap font (5×9, same grid as digits) ──────────────
# Used for spelled-out day/month names at 2× scale (10×18px/char)
LETTER_BITMAPS = {
    'A': [0b01110,0b10001,0b10001,0b10001,0b11111,0b10001,0b10001,0b10001,0b10001],
    'B': [0b11110,0b10001,0b10001,0b10001,0b11110,0b10001,0b10001,0b10001,0b11110],
    'C': [0b01111,0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b01111],
    'D': [0b11110,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b11110],
    'E': [0b11111,0b10000,0b10000,0b10000,0b11110,0b10000,0b10000,0b10000,0b11111],
    'F': [0b11111,0b10000,0b10000,0b10000,0b11110,0b10000,0b10000,0b10000,0b10000],
    'G': [0b01111,0b10000,0b10000,0b10000,0b10011,0b10001,0b10001,0b10001,0b01111],
    'H': [0b10001,0b10001,0b10001,0b10001,0b11111,0b10001,0b10001,0b10001,0b10001],
    'I': [0b01110,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100,0b01110],
    'J': [0b00111,0b00010,0b00010,0b00010,0b00010,0b00010,0b10010,0b10010,0b01100],
    'K': [0b10001,0b10010,0b10100,0b11000,0b10000,0b11000,0b10100,0b10010,0b10001],
    'L': [0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b10000,0b11111],
    'M': [0b10001,0b11011,0b10101,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001],
    'N': [0b10001,0b11001,0b10101,0b10011,0b10001,0b10001,0b10001,0b10001,0b10001],
    'O': [0b01110,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b01110],
    'P': [0b11110,0b10001,0b10001,0b10001,0b11110,0b10000,0b10000,0b10000,0b10000],
    'Q': [0b01110,0b10001,0b10001,0b10001,0b10001,0b10101,0b10011,0b10001,0b01111],
    'R': [0b11110,0b10001,0b10001,0b10001,0b11110,0b11000,0b10100,0b10010,0b10001],
    'S': [0b01111,0b10000,0b10000,0b10000,0b01110,0b00001,0b00001,0b00001,0b11110],
    'T': [0b11111,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100,0b00100],
    'U': [0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b01110],
    'V': [0b10001,0b10001,0b10001,0b10001,0b10001,0b10001,0b01010,0b01010,0b00100],
    'W': [0b10001,0b10001,0b10001,0b10001,0b10001,0b10101,0b10101,0b11011,0b10001],
    'X': [0b10001,0b10001,0b01010,0b01010,0b00100,0b01010,0b01010,0b10001,0b10001],
    'Y': [0b10001,0b10001,0b01010,0b01010,0b00100,0b00100,0b00100,0b00100,0b00100],
    'Z': [0b11111,0b00001,0b00010,0b00100,0b01000,0b01000,0b10000,0b10000,0b11111],
}

def ordinal_suffix(n):
    """Return 'st', 'nd', 'rd', or 'th' for integer n."""
    if 11 <= (n % 100) <= 13:
        return 'th'
    return {1:'st', 2:'nd', 3:'rd'}.get(n % 10, 'th')

def draw_digit(epd, ch, x, y, color=0x00):
    """Draw a single large digit at (x, y) top-left."""
    bmap = DIGIT_BITMAPS.get(ch)
    if bmap is None:
        return
    for row in range(DIGIT_H):
        bits = bmap[row]
        for col in range(DIGIT_W):
            if bits & (0x10 >> col):
                for dy in range(SCALE):
                    for dx in range(SCALE):
                        epd.pixel(x + col*SCALE + dx, y + row*SCALE + dy, color)

def draw_char(epd, ch, x, y, scale=2, color=0x00):
    """
    Draw a single character (digit or letter) from the bitmap fonts
    at an arbitrary scale. scale=2 → 10×18px, scale=4 → 20×36px.
    Returns the width consumed (DIGIT_W * scale).
    """
    bmap = DIGIT_BITMAPS.get(ch) or LETTER_BITMAPS.get(ch.upper())
    if bmap is None:
        return DIGIT_W * scale   # skip unknown chars (treat as space)
    for row in range(DIGIT_H):
        bits = bmap[row]
        for col in range(DIGIT_W):
            if bits & (0x10 >> col):
                for dy in range(scale):
                    for dx in range(scale):
                        epd.pixel(x + col*scale + dx, y + row*scale + dy, color)
    return DIGIT_W * scale

def draw_string(epd, text, x, y, scale=2, gap=1, color=0x00):
    """
    Draw a string of letters/digits using bitmap font at given scale.
    Returns ending x position.
    """
    cx = x
    for ch in text:
        if ch == ' ':
            cx += (DIGIT_W * scale) // 2 + gap
        else:
            cx += draw_char(epd, ch, cx, y, scale, color) + gap
    return cx

def draw_colon(epd, x, y, color=0x00):
    """Draw a colon — two 3×3 dots, vertically centered in CELL_H."""
    dot_y1 = y + CELL_H // 3 - 1
    dot_y2 = y + 2 * CELL_H // 3 - 1
    for dy in range(4):
        for dx in range(4):
            epd.pixel(x + dx, dot_y1 + dy, color)
            epd.pixel(x + dx, dot_y2 + dy, color)

def draw_clock(epd, h, m, x, y, color=0x00):
    """Draw HH:MM in large digits starting at (x, y)."""
    time_str = f"{h:02d}{m:02d}"
    COLON_W = 10
    GAP     = 3

    cx = x
    for ch in time_str[:2]:
        draw_digit(epd, ch, cx, y, color=color)
        cx += CELL_W + GAP
    draw_colon(epd, cx, y, color=color)
    cx += COLON_W
    for ch in time_str[2:]:
        draw_digit(epd, ch, cx, y, color=color)
        cx += CELL_W + GAP
    return cx


# ── WiFi ────────────────────────────────────────────────────
def connect_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    if wlan.isconnected():
        return True
    print("Connecting WiFi...")
    wlan.connect(WIFI_SSID, WIFI_PASSWORD)
    for _ in range(20):
        if wlan.isconnected():
            print("WiFi:", wlan.ifconfig()[0])
            return True
        time.sleep(1)
    return False


# ── Time sync ───────────────────────────────────────────────
# Uses MicroPython's built-in ntptime (much more reliable than worldtimeapi).
# NTP always returns UTC, so we apply a manual offset.
UTC_OFFSET_HOURS = -6   # confirmed via NTP debug: UTC 21:35 = local 15:35

def sync_time():
    try:
        import ntptime
        ntptime.host = "pool.ntp.org"
        ntptime.settime()   # sets RTC to UTC
        utc = utime.localtime()
        print(f"NTP sync OK — UTC is {utc[3]:02d}:{utc[4]:02d}:{utc[5]:02d}")
        return True
    except Exception as e:
        print("Time sync error:", e)
        return False

def local_time():
    """Return localtime tuple adjusted for UTC_OFFSET_HOURS."""
    return utime.localtime(utime.time() + UTC_OFFSET_HOURS * 3600)


# ── Weather ─────────────────────────────────────────────────
WMO = {
    0:"Clear", 1:"Mostly Clear", 2:"Partly Cloudy", 3:"Overcast",
    45:"Foggy", 48:"Icy Fog",
    51:"Lt Drizzle", 53:"Drizzle", 55:"Hvy Drizzle",
    61:"Lt Rain", 63:"Rain", 65:"Hvy Rain",
    71:"Lt Snow", 73:"Snow", 75:"Hvy Snow", 77:"Sleet",
    80:"Showers", 81:"Showers", 82:"Hvy Showers",
    85:"Snow Showers", 86:"Hvy Snow Showers",
    95:"Thunderstorm", 96:"T-storm+Hail", 99:"Hvy T-storm",
}

def get_weather():
    try:
        import urequests
        url = (
            f"https://api.open-meteo.com/v1/forecast"
            f"?latitude={LATITUDE}&longitude={LONGITUDE}"
            f"&current=temperature_2m,apparent_temperature,weather_code,precipitation_probability"
            f"&temperature_unit={TEMP_UNIT}"
            f"&forecast_days=1"
            f"&timezone=America%2FChicago"
        )
        r = urequests.get(url, timeout=20)
        if r.status_code != 200:
            r.close(); return None
        d = r.json(); r.close()
        cur = d['current']
        return {
            'temp':       round(float(cur['temperature_2m'])),
            'feels_like': round(float(cur['apparent_temperature'])),
            'condition':  WMO.get(cur['weather_code'], "Unknown"),
            'precip':     cur.get('precipitation_probability', 0),
        }
    except Exception as e:
        print("Weather error:", e); return None


# ── Helpers ──────────────────────────────────────────────────
DAYS   = ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"]
MONTHS = ["January","February","March","April","May","June",
          "July","August","September","October","November","December"]

def bold(epd, text, x, y, color=None):
    c = FG if color is None else color
    epd.text(text, x,   y, c)
    epd.text(text, x+1, y, c)

def splash(epd, line1, line2=""):
    epd.fill(BG)
    epd.text(line1, 10, 55, FG)
    if line2: epd.text(line2, 10, 68, FG)
    epd.display()


# ── Screen layout ────────────────────────────────────────────
#
#  x=0                 x=152        x=250
#  ┌────────────────────┬────────────┐  y=0
#  │                    │ Nashville  │
#  │   10:42  •         │ 52*F       │
#  │                    │ Feels 48*F │
#  │   Thursday Feb 19  │ Pt Cloudy  │
#  │                    │ Precip 20% │
#  └────────────────────┴────────────┘  y=127
#
PANEL_X   = 152
DIVIDER_X = 151

# Derive foreground/background from DARK_MODE flag
BG = 0x00 if DARK_MODE else 0xFF   # fill color
FG = 0xFF if DARK_MODE else 0x00   # text/line color

def draw_screen(epd, weather):
    epd.fill(BG)

    t    = local_time()
    hour = t[3]
    mins = t[4]
    h12  = hour % 12 or 12
    ampm = "AM" if hour < 12 else "PM"

    # ── Large clock (4× scale, 20×36px digits) ──────────────
    clock_x = 4
    clock_y = 14
    draw_clock(epd, h12, mins, clock_x, clock_y, color=FG)

    # A or P — single letter at 2× scale, sits right of digit block
    ampm_letter = "A" if hour < 12 else "P"
    draw_char(epd, ampm_letter, clock_x + 108, clock_y + 10, scale=2, color=FG)

    # ── Day + Date (2× scale = 10×18px per char) ────────────
    day_name   = DAYS[t[6]]
    month_name = MONTHS[t[1] - 1]
    day_num    = t[2]
    suffix     = ordinal_suffix(day_num)

    # Pulled up slightly from the bottom edge
    date_y2 = 101
    date_y1 = 80

    draw_string(epd, day_name,  clock_x, date_y1, scale=2, gap=1, color=FG)
    end_x = draw_string(epd, month_name, clock_x, date_y2, scale=2, gap=1, color=FG)
    end_x += 4
    end_x = draw_string(epd, str(day_num), end_x, date_y2, scale=2, gap=1, color=FG)
    epd.text(suffix, end_x + 1, date_y2, FG)

    # ── Vertical divider ────────────────────────────────────
    epd.vline(DIVIDER_X, 0, 128, FG)

    # ── Weather panel ───────────────────────────────────────
    wx = PANEL_X + 3

    if weather:
        epd.text("Nashville", wx, 10, FG)
        epd.hline(DIVIDER_X, 20, 250 - DIVIDER_X, FG)

        # Temperature
        temp_str = str(weather['temp'])
        end = draw_string(epd, temp_str, wx, 26, scale=2, gap=1, color=FG)
        epd.text(f"*{TEMP_SUFFIX}", end + 1, 26, FG)

        # Feels like
        epd.text("Feels", wx, 50, FG)
        feel_str = str(weather['feels_like'])
        end = draw_string(epd, feel_str, wx + 44, 46, scale=2, gap=1, color=FG)
        epd.text(f"*{TEMP_SUFFIX}", end + 1, 46, FG)

        # Condition
        cond = weather['condition'][:11]
        epd.text(cond, wx, 70, FG)

        # Precip
        epd.text("Precip", wx, 90, FG)
        prec_str = str(weather['precip'])
        end = draw_string(epd, prec_str, wx + 50, 86, scale=2, gap=1, color=FG)
        epd.text("%", end + 1, 86, FG)

    else:
        epd.text("Weather", wx, 40, FG)
        epd.text("N/A", wx, 54, FG)


# ── Main ─────────────────────────────────────────────────────
def main():
    print("Clock Widget starting...")
    epd = EPD_2in13()

    splash(epd, "Clock Widget", "Starting...")
    time.sleep(1)

    splash(epd, "Connecting WiFi...")
    if not connect_wifi():
        splash(epd, "WiFi Failed!", "Check credentials")
        return

    splash(epd, "Syncing time...")
    if not sync_time():
        time.sleep(2)
        sync_time()

    splash(epd, "Fetching weather...")
    weather = get_weather()

    # First full render
    draw_screen(epd, weather)
    epd.display()
    print("Initial display done")

    update_count   = 0
    last_time_sync = time.time()
    last_weather   = time.time()

    while True:
        try:
            time.sleep(PARTIAL_INTERVAL)
            update_count += 1

            # Re-sync time every hour
            if time.time() - last_time_sync > 3600:
                sync_time()
                last_time_sync = time.time()

            # Re-fetch weather every 15 minutes
            if time.time() - last_weather > 900:
                w = get_weather()
                if w:
                    weather = w
                last_weather = time.time()

            draw_screen(epd, weather)
            epd.display()
            print(f"Update #{update_count} done")

        except KeyboardInterrupt:
            print("\nShutting down...")
            epd.init(FULL_UPDATE)
            epd.sleep()
            break
        except Exception as e:
            print(f"Loop error: {e}")
            time.sleep(30)


if __name__ == '__main__':
    main()
```


The only things needed to adapt this code to your usage is to add your SSID and password at the top of the file, your latitude and longitude, and your city name. Once these are in place, you should have a working desktop clock/weather widget device. This is a great first project to kick off the blog, and I will revisit it in the future to add features. The latest version of this code is also available on [GitHub](https://github.com/bitb4nger/pico-clock) as well. Check back in for more embedded dev projects, we're just getting started!!

![End](end.jpg)
