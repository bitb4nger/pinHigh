+++
title = "Pi Pico 2W with Waveshare Pico ePaper 2.13"
date = 2026-04-19
draft = true
+++

I recently picked up a Pico-ePaper-2.13 from Waveshare. This is a great, small e-ink display with a fast refresh time designed for interfacing with the Pi Pico. I want to make a few things with this, but I'm going to start out making a clock/weather widget display for your desk. I'm using a Pi Pico 2W for the extra umph, altho a Pi Pico 1W would work for this application.

Parts list:

Pi Pico 2W                          https://www.adafruit.com/product/6315
Waveshare Pico ePaper 2.13          https://www.waveshare.com/pico-epaper-2.13.htm

I'm not going to include wiring diagrams or schematics on this specific project, this ePaper device is ready for the Pi Pico out of the box. As long as the Pi Pico has the headers soldered on, you can just fit the ePaper to the microcontroller and it's ready for code. Be sure to pay attention to the solder mask on the ePaper, it has a "USB" marking for orienting the Pi Pico correctly. I'm coding with MicroPython using the program/IDE Thonny.
