# Audio reactive LED strip
This is a guide from electronics begineer trying to have some fun with LED strip and ESP8266 based microboard :) I will show you how to build your own audio reactive LED strip using the Wemos D1 Mini Pro microboard. I suppose you have some basic knowledge of microboards and other stuff, so I won't explain everything in detail (but of course you can ask me in issues). Some demonstrations (click for YouTube video):

[![Video #1](https://img.youtube.com/vi/WMm4V7VpMeQ/0.jpg)](https://www.youtube.com/watch?v=WMm4V7VpMeQ)
[![Video #2](https://img.youtube.com/vi/8EOnbsJb-b4/0.jpg)](https://www.youtube.com/watch?v=https://youtu.be/8EOnbsJb-b4)

## Hardware
I have bought most of the HW at AliExpress, so I give you also links. Tools are not listed.

- [Wemos D1 Mini Pro](https://www.aliexpress.com/store/1331105?spm=a2g0s.9042311.0.0.AFiCMF)
- [WS2813 LED strip](https://www.aliexpress.com/item/1m-4m-5m-WS2813-Dual-signal-wires-30-60-pixels-leds-m-Smart-led-pixel-strip/32699391341.html?spm=a2g0s.9042311.0.0.AFiCMF)
  - WS2813 is updated WS2812b ([difference](https://www.elecrow.com/blog/ws2813-vs-ws2812/)). Best advantage is that it has two data lines to provide backup in case some LED gets damaged -- then subsequent LEDs don't lose data signal. But of course you can also use WS2812b, which is also a little bit cheaper.
- [5V power supply for LED strip](https://www.aliexpress.com/item/5V-2A-3A-4A-5A-8A-10A-12A-20A-30A-40A-60A-Switch-LED-Power-Supply/32670505021.html?spm=a2g0s.9042311.0.0.AFiCMF)
  - I got the 30A one to be sure I can handle a long strip. Power consumption of these LED strips (60 LEDs/m) is ~18W/m, so ~0.3W/LED. Don't forget to buy [power cable](https://images.obi.cz/product/CZ/1500x1500/468962_1.jpg).
  - For shorter strips (i.e. < ~60 LEDs/m) you can use wall adapter. But definitely calculate expected power consumption in advance!
- [Breadboard, dupont cables](https://landzoelectronic.aliexpress.com/store/428351?spm=a2g0s.9042311.0.0.AFiCMF)
  - Honestly, quality of this breadboard is not very good (some holes have bad contact), so maybe look somewhere else.
- 1000μF 6.3V capacitor. I have bought [this one](https://www.gme.cz/ce-1000u-6-3vit-hit-exr-8x12-rm3-5-bulk).

## How to connect it
This is wiring diagram from https://learn.adafruit.com/adafruit-neopixel-uberguide/basic-connections (very good reading!). It's for WS2812b, but in case of WS2813 there is only one difference: there are two data lines, so there are two green wires leading from strip and connected to single pin on microboard.

![abc](https://cdn-learn.adafruit.com/assets/assets/000/030/892/original/leds_Wiring-Diagram.png?1456961114)

How it looks like in real:

![Overall](https://ctrlv.cz/shots/2017/11/26/WyfY.png)
![Power supply detail](https://ctrlv.cz/shots/2017/11/26/qfGA.png)
![LED strip detail](https://ctrlv.cz/shots/2017/11/26/ORLr.png)
![Microboard detail](https://ctrlv.cz/shots/2017/11/26/gSdd.png)

Wiring is simple:
- First connect power cable to power supply. **YOU ARE DOING THIS ON YOUR OWN RESPONSIBILITY. I am not responsible for damage to your health or property. If you aren't sure how to properly connect it, leave it to qualified electrician. Be sure your country is using same color schema for electric wires!** (This is from the Czech Republic.)
  - Brown cable goes to **L** (live).
  - Blue cable goes to **N** (neutral).
  - Green-yellow cable goes to [**Earth symbol**](http://www.clker.com/cliparts/4/4/d/4/12236156551925934261rsamurti_RSA_IEC_Ground_Symbol.svg.hi.png) (protective).
- Connect the LED strip to power supply using shorter wires from the side where arrows go to opposite side of LED strip (as in the first picture). **Wire colors and names on LED strip can be different manufacturer to manufacturer!**
  - GND wire (white in the picture) goes to -V on power supply.
  - +5V wire (red in the picture) goes to +V on power supply.
  - Connect capacitor between -V and +V on power supply. If you use electrolytic capacitor, then connect longer leg (+) to +V and shorter leg (-) to -V. Otherwise capacitor will explode! According to AdaFruit, "The capacitor buffers sudden changes in the current drawn by the strip.".
- Finally connect LED strip to microboard (as in the last image).
  - GND goes to GND pin.
  - Data lines both on WS2812b and WS2813 (on my LED strip called B0 and D0) go to D4 pin.
  
## How to make it reactive (software section)
We are using [Audio Reactive LED Strip](https://github.com/scottlawsonbc/audio-reactive-led-strip) library to create these beatiful LED strip audio reactions. You can read the instructions there to get more details, but if you follow my simplified instructions here, you will be probably also successful.

### Computer side
- Install [**conda**](https://conda.io/miniconda.html). Really, it's the easiest distribution system and it doesn't matter whether you are using Windows or Linux.
- Create virtual environment (venv) and install all the dependencies:
  - `$ conda create --name ledstrip python=3 numpy scipy pyqtgraph`
  - Switch to new venv: `$ source activate ledstrip` (`$ activate ledstrip` on Windows)
  - install the last dependency: `$ pip install pyaudio`
- Download Audio Reactive LED Strip repository, unzip it somewhere.
- Open `python/config.py` and set up the constants. `UDP_IP`, `UDP_PORT` and `N_PIXELS` must match the constants in `ws2812_controller.ino` (see below)!
- [Set up audio input](https://github.com/scottlawsonbc/audio-reactive-led-strip#audio-input). In Windows, the easiest is the stereo mix method.

### Microboard side
- Download Arduino IDE and follow [these instructions](https://github.com/esp8266/Arduino#installing-with-boards-manager) to prepare it for ESP8266 platform.
- Using the `Sketch -> Include Library -> Manage Libraries` in Arduino IDE, install these libraries: WebSockets (by Markus Sattles), NexPixelBus (by Makuna).
- Now there is a difference from original Audio Reactive LED Strip project. We are using NeoPixelBus library to control the LED strip. So in downloaded Audio Reactive LED Strip repository, replace `arduino/ws2812_controller/ws2812_controller.ino` with [this modified one](https://github.com/joeybab3/audio-reactive-led-strip/blob/master/arduino/ws2812_controller/ws2812_controller.ino).
- Open `ws2812_controller.ino` in Arduino IDE and set it up:
  - Set constants: `ssid`, `password` etc.
  - Change line `NeoPixelBus<NeoGrbFeature, Neo800KbpsMethod> ledstrip(NUM_LEDS, PixelPin);` (more [here](https://github.com/Makuna/NeoPixelBus/wiki/NeoPixelBus-object)):
    - If you are using WS2812b, change it to `NeoPixelBus<NeoGrbFeature, NeoEsp8266Uart800KbpsMethod> ledstrip(NUM_LEDS);`
    - If you are using WS2813, change it to `NeoPixelBus<NeoGrbFeature, NeoEsp8266UartWs2813Method> ledstrip(NUM_LEDS);`
  - Upload it to your Wemos.
  
### Run it
- `$ activate ledstrip`
- Go to `python` directory.
- Finally play some music and run `$ python visualization.py`. Enjoy the visualization! :)
