# OpenSolder
Fully open source JBC T245-compatible soldering station and tool holder  
![](/mechanical/images/front.png)

## Introduction
When my cheap T12 clone soldering station went up in smoke, I started looking into either buying a JBC CDB, or building a proper soldering station. The internet is already crowded with DIY JBC compatible stations, but most are poorly documented, or does not have the quality-of-life features that the original have.

The [unisolder project](https://github.com/sparkybg/UniSolder-5.2) is very impressive, but way to complex and "universal" for my needs. [Marco Reps](https://youtu.be/GYIiOkr6x9o) built a cool C470 station that i liked the simplicity of, and this project is highly inspired by that.

## Goals:
- _T12 clone_ form factor that does not take up valuable bench space
- JBC T245 tip compatibility, with similar performance to the original station
- QoL features that the CDB station have, like auto standby, tip remover, holder, tip cleaner etc.
- Simple design using cheap off-the-shelf parts
- A compact handpiece stand with these features:
	- Detection when the handpiece is in the tool holder (auto standby)
	- Detection when you're about to swap tips (cut power)
	- Integrated tip remover, tip storage, tip cleaner and a soldering wire spool holder
	- Simple to build

## Design Specifications
- 24V toroidal transformer
- Zero Crossing AC switching
- OLED display and rotary encoder for adjusting temperature
- Compact and low profile design
- Super fast heatup time (~4s)
- Hardware should be mostly 3D printable, and easy to build using regular tools

## Station
![](/mechanical/images/station_1.png)  
The station is built in a Hammond enclosure, with a center mounted transformer. The PCB is mounted to the custom front panel, and all 230VAC connections is in the rear of the chassis.
There is two connectors on the rear panel, one for connecting to the stand, and an optional ST-link connector for firmware upgrades.

## Stand
![](/mechanical/images/stand_1.png)  
The stand is made of 3D-printed parts, a genuine JBC tool holder, a tip remover made from aluminium angle and some fasteners. Most parts can be made from aluminium sheet if that is preferred.

## Schematic
This is the internal connections of the JBC C245 cartridge:  
![](/hardware/images/jbc_c245_connection_diagram.png)  

[__The schematic__](https://github.com/howie-j/OpenSolder/raw/main/hardware/schematic.pdf) is divided into three sheets: AC, Analog and MCU.

### AC
The 24VAC input is fused, then connected directly to the tip center pin (5). Switching is done with two N-channel mosfets, driven by a Si8751 isolated gate driver.

Zero cross detection is done with an AC optocoupler, and the current is limited by three 10k resistors (to reduce BOM items). This solution triggers the ZeroCross about 90uS prematurely, and that is compensated for in firmware.

The MCU and thermocouple frontend must be electrically isolated, and a 24V to 5V DCDC PSU module solves that problem. The DCDC-supply is preregulated with a 24V linear regulator, and post-regulated to 3.3V with a LDO. This power supply chain is in need of some simplification, but the components are few and cheap, and it works great for now.

### Analog
The thermocouple signal is amplified with a MAX4238 chopper opamp, with a fairly arbitrary amplification of 221, since temperature calibration is done in firmware anyway. No negative supply rail means the thermocouple can't sense lower temperatures than room temp, but that's not needed for a soldering station.

Long handpiece wires means lots of crosstalk, so a pair of diodes clamps the thermocouple input at Vf. The MCU pin _TIP_CLAMP_ is driven low when power is applied to the handpiece, to clamp noise on the opamp input. When its time to measure tip temperature, the pin is set to input, and a delay waits for the RC filter to stabilize.

To sense if there is a tip cartridge in the handpiece, _TIP_CHECK_ is driven high. The low impedance of the thermocouple + heater (a few Ω's) will create a very low voltage drop. If the tip is removed, the Vf of the input protection diode will drive _THERMOCOUPLE_ADC_ close to 3.3V.


### MCU
A STM32F072 is used since I have a few of them in stock, but almost any STM32 could be adapted to work. To sense PCB temperature (and for cold junction compensation), an I2C temp sensor is added. The MCU might have an internal temperature sensor, but  have not looked into that yet.

The rotary encoder and handpiece stand inputs are filtered using plain RC filters. This, combined with the input hysteresis, removes most input noise. The TIM encoder inputs also have additional filtering and FSM redundancy.
 

## PCB
[___Check out this interactive BOM___](http://htmlpreview.github.io/?https://github.com/howie-j/OpenSolder/blob/main/hardware/bom/interactive_bom.html)

![](/hardware/images/pcb_front.png)  
![](/hardware/images/pcb_rear.png)  
The PCB is quite low density, uses 0805 SMD components and is fully hand solderable with a microscope. Connections to the rear connectors (stand and ST-link) is done via a 6-pin IDC connector, so no crimping of tiny connector pins are necessary. The PCB is mounted to the front panel via the rotary encoder and the handpiece connector.


## Firmware
The firmware is simple and functional, and uses a modified hysteresis temperature controller that is very fast but has some initial overshoot. Currently all default values are set in _opensolder.h_, and are reset when the station is powered off.


### TODO
Nothing is currently missing for regular functionality. However some features would be cool to have:

- More accurate temp control (PID or other controller)
- User changeable default values
- Menu for changing default values
- Better looking UI

### B Broadbent TODO
- Pullup on zero cross line (external to STM32) (10k)


## Links and Sources
[The unisolder project](https://github.com/sparkybg/UniSolder-5.2)  
[Marco Reps](https://youtu.be/GYIiOkr6x9o)  
[foldvarid93's JBC Soldering Station](https://github.com/foldvarid93/JBC_SolderingStation)  
[johnmx's eevblog thread](https://eevblog.com/forum/testgear/jbc-soldering-station-cd-2bc-complete-schematic-analysis/)  
