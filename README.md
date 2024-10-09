# home-assistant-lowglow

Read your electricity meter in a battery friendly way by means of the pulse LED on your meter. Uses the ESP32 deep sleep capabilities to extend battery life. Inspired by Home Assistant Glow (<https://glow-energy.io/>)

## Motivation, the Journey and Beyond

My Electricity and Gas (not so) smart meters are outside the house and a long way from any power sources, how difficult could it be to run a glow setup from a battery! I soon discovered that a USB battery pack and the std glow config would only get me a few days of operation and that was assuming I could keep the dammed battery pack alive. So liking a challenge the Low(Power)Glow project was born. If I knew then that it would take me 6 months plus to get to this stage I may have thought twice!

In order to achieve this i've re-written the glow logic to use the ESP32 deep sleep function and control the battery pack using USB C along with a small LIPO battery keeping the ESP powered while charging the battery pack. My current setup gives me 20-25 days of use per charge from a 10600 mAh battery with an accuracy +0 -> -0.01kWh/day (based on a 1Wh pulse), obviously this duration depends on your usage but I have an EV and electric water heating so this is a good indication. I will probably buy a bigger battery pack as I want to get to over a month between charges.

I would like to point out that this is my first ESP HOME project so any suggestions as to improving the structure of the config and to reduce the power consumption would be greatly appreciated. The same goes for the hardware setup.

## Hardware

Please note - This is ESP32 specific

As per Glow with the following additions/variations:-

- Power efficient ESP (My choice FireBeetle ESP32 IOT Microcontroller (V3.0) uses just 10μA in deep-sleep mode)
- LIPO battery (i've used a 2000mAh)
- USB Battery Pack with USB C port
- USB C breakout board (remove resistors)
- Some resistors and a 2N7000 MOSFET (or two)
- A toggle switch (Deep Sleep Enable)
- A push button (Cold Start)
- USB cables

Sketch of hardware setup..
![USBC Voltage Control Circuit](https://github.com/user-attachments/assets/b6c43688-630c-4370-b26b-9aa6d84245e0)

Notes-

To be compliant with USB C standard you probably need separate MOSFET pull down on both CC1 and CC2.

The build....
![lowglow_v1](https://github.com/user-attachments/assets/90b7a078-8c47-48ba-8021-2f3b2c0df609)

## ESP Home Config

Highlights:-

In deep sleep mode following a pulse from the Photodiode and incrementing the counter the ESP will enter deep sleep until the next pulse or after a time delay which is adjusted to occur just before the next Hour transition when HA will be updated via the API.

Wake-up can take quite a while (250ms+), so there is a possibility of pulses being dropped during periods of high consumption. I have therefore made an attempt to calculate the duration between pulses and if below a certain limit keep the ESP awake until the frequency of pulses drops to an acceptable level.

If the LIPO battery voltage drops below 3.4V the USB battery is energized by pulling the CC pin to ground through a 4.7k resistor which simulated a cable insertion. The ESP is kept awake during this period.

When the LIPO battery voltage is above 4.1V the CC pin is left to float which simulates a cable removal and deep sleep is enabled again.
