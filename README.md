# Aztec Platforms
## AP-XIAO-PB-3V3-V1
In this Documentation we will explore the Aztec Platforms AP-ESP32C6-PB-1V0. This board is designed to be a versitile project board that can be used in many ways. The main reason I design these boards is to reduce the amount of wiring for simple tasks like voltage reduction, common pinout placement, PSU jankery, etc.

Designed to get your projects off the ground faster, and, suitable for use in long term and permanent projects.

# Design Overview

### Board Size and hole spacing

<p align="center">
<img src="https://github.com/user-attachments/assets/cf06c168-8b13-44a1-a11b-242e8b214617" alt="Board and Hole Size" width="600">
</p>


---

### Power Rails

This board provides three usable voltage rails — 12 V, 5 V, and 3.3 V — making it versatile for a wide range of projects and connected devices.

Power can be supplied in two ways:
⚠️ Important: If you’re powering through USB-C, make sure your charger supports the 12 V profile. Not all USB-C bricks do — some only provide 9 V or 15 V. For higher-power use cases, connect 12 V directly through the VIAs to take advantage of the full 5 A capacity. 

Note - if a 12v profile is not availbale on the on your USB-C brick, it will usually deliver one voltage lower (9v). allowing you to still use the brick if you do not need the 12v power rail.
- USB-C female connector — The onboard USB-C port negotiates with the charger and requests 12 V. Long story short - your power supply must explicitly state that it can deliver 12v. It will not work if a PSU delivers power within a voltage range, a range of voltage typically indicates that the PSU is PPS capable and will not work with this board.
- Direct 12 V input — Near the USB-C connector, you’ll see two larger VIAs. These allow you to feed the board directly with 12 V at up to 5 A. The onboard power system is designed for this: the power supply, traces, and components are all rated for 5 A and can safely deliver that if needed.


Once 12 V is available on the board, it is stepped down in two stages:
- 12 V → 5 V
- 12 V → 3.3 V

The ESP32-C6 runs on 3.3 V, which is also the voltage used by most sensors and peripherals. The 5 V rail is handy for certain modules, and the raw 12 V input remains available as well.

This design keeps the board simple, flexible, and safe while giving you multiple voltage rails to power different devices.

---

### Level Translator (Level Shifter) and Pin Voltages

Another key feature of this board, made possible by the 3.3 V and 5 V rails, is the level shifter. This component translates signals between 3.3 V (from the ESP32-C6) and 5 V, enabling you to control devices that require 5 V inputs. For example, the PWM input on a computer fan expects a 5 V signal, so you couldn’t drive it directly from the ESP32 alone.

The level shifter also works in reverse: it safely steps down 5 V input signals to 3.3 V, protecting the ESP32-C6 pins. This is particularly useful when working with sensors that output only 5 V, as connecting them directly could damage the microcontroller.

For each level-shifted signal, you’ll notice duplicate pins: one for the 3.3 V side and one for the 5 V side. ⚠️ Important: these pairs are electrically connected. You can use either the 3.3 V or the 5 V version of a pin (e.g., D0), but not both at the same time.

<p align="center">
<img src="https://github.com/user-attachments/assets/c4ed8c7b-f77f-4b1b-9a6b-b081895d08f2" alt="Board and Hole Size" width="600">
</p>


---

### I2C Pinout
The Seeed Studio ESP32-C6 has a limited number of pins, so this board places D4 and D5 side by side as the dedicated I²C pins (SDA and SCL). These are arranged in a row with 3.3 V and GND on standard 2.54 mm pin spacing, allowing a JST-XH female plug to be soldered directly to the board. This makes it simple to plug in I²C expansion boards and start adding peripherals right away.

For example, if you’re building an air quality sensor, you can connect an I²C expander board to add multiple sensors while still using just the SDA/SCL lines. This gives you more inputs and outputs without tying up other GPIO pins.

I²C expander boards are also very affordable—typically under $5. For reliability, I recommend picking one up from Adafruit or another trusted supplier.

<p align="center">
<img src="https://github.com/user-attachments/assets/9970cbf6-c3b2-4ac9-9345-adf1c72c1f29" alt="Board and Hole Size" width="600">
</p>


---

### N-Channel Mosfet

The output highlighted in the image below is controlled by an N-channel MOSFET. Think of it like an electronic switch that turns on and off very quickly. When you send a PWM signal to the MOSFET, you can control how much power gets through to your device.

This is especially useful for things like motors or fans that only have two wires, voltage and ground. Normally, they would just run at full speed whenever power is applied. By using a MOSFET, your controller can rapidly switch the power on and off, making it look to the device like it is receiving less power. For example, if the PWM signal is set to 50%, the motor will only draw about half the power and run at roughly half speed, even though it is still being supplied with the full voltage during each “on” cycle.

You can safely drive 3A through the Mosfet Power.

<p align="center">
<img src="https://github.com/user-attachments/assets/ef97ba51-b275-46ed-947f-041ad46cfe2e" alt="Board and Hole Size" width="600">
</p>


---

### D0 Madness

Why are there so many D0 pins on my board?

It might look confusing at first, but D0 can actually be used in three different ways. The most important rule is this: you should only use one version of D0 in your build. That means you can’t use the 5-volt D0, the 3.3-volt D0, and the MOSFET D0 all at the same time. You need to pick just one.

Here’s how the different versions of D0 work:

- 5V GPIO D0 pins
On the far left side of the board, you’ll see two D0 pins at 5 volts. These go through a level shifter and connect back to the main controller, which means they can be used as normal GPIO pins. This makes them great for general-purpose control of small devices like LEDs, fans, or little motors.

- MOSFET D0
D0 also drives the onboard MOSFET. In this case, the PWM signal from D0 switches the 12-volt and ground pins on and off very quickly. This is perfect for dimming 12-volt LEDs, since the rapid on/off switching makes them appear dimmer. You can also use it for motors, pumps, or other two-pin 12-volt devices where you need speed or power control.

- 3.3V GPIO D0 pin
Just to the right of the MOSFET section, you’ll find the standard 3.3-volt GPIO D0 pin. This is a direct general-purpose I/O pin, just like you’d expect on a microcontroller.

So while D0 shows up in multiple places, each one serves a specific purpose. The key is to only choose one version of D0 for your project. If you try to tie them together, you risk damaging the electronics or the sensors you’re connecting.

<p align="center">
<img src="https://github.com/user-attachments/assets/37f790b2-a57f-4fcd-a10c-cc08207973ca" alt="Board and Hole Size" width="600">
</p>



---

## License
This repository is licensed under a custom Source-Available License.  
- ✅ Personal, hobbyist, and educational use of the code is allowed.  
- ❌ Commercial use, redistribution, and modification of images/docs are prohibited.  
- ℹ️ See [LICENSE](./LICENSE) for details. 
