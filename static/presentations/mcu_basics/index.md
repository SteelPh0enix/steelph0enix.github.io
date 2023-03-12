---
presentation:
  width: 1280
  height: 720
  controls: true
  progress: true
  slideNumber: true
  keyboard: true
  overview: true
  fragments: true
  touch: true
  hideAddressBar: true
  enableSpeakerNotes: true
---

<!--
Add this to the presentation HTML, just before the </head>:

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Fira+Mono&family=Fira+Sans&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="/presentations/style.css">

-->

<!-- slide -->

# Digital electronics and microcontrollers in applications

<_the cruel consequences of enslaving the sand_>

Wojciech Olech

<div style="font-size:50%">press <code>?</code> for navigation, and <code>s</code> to see the notes</div>

<!-- slide -->

## Table of content

- **Digital electronics basics** - how can we enslave the sand to do what we want?
- **Digital signals basics** - How does the spicy sand communicate, and how do we understand it?
- **Microcontrollers basics** - What happens when we put a lot of spicy sand together, and how to cope with it?
  - **What makes an MCU?** - How are they built?
  - **How to use an MCU?** - How do we tell it what we want it to do?
  - **Peripherals** - From digital brain, to digital senses

<!-- slide data-notes="These properties apply to every electric component (or circuit), and they can be modeled based on them." -->

## Digital electronics basics

Five primary properties in electronics are

- **current** ($I$ [$A$ - amper]) - amount of electric charge carriers flowing in conductor,
- **voltage** ($V$ [$V$ - volt]) - electric potential between two points (_"pressure"_),
- **resistance** ($R$ [$\Omega$ - ohm]) - opposition to current flow (makes it harder to push current throught the conductor),
- **capacitance** ($C$ [$F$ - farad]) - ratio of amount of electric charge to voltage (how much charge can be stored between two points)
- **inductance** ($L$ [$H$ - henry]) - ability to store electrical energy in magnetic field

<!-- slide class="img-70" data-notes="At the beginning of 19th century, some (now famous) scientists started noticing strange things happening to some elements and compounds under the influence of electricity. Some (Michael Faraday) noticed that the resistance of some materials decreases, when they are heated (while conventional conductors, like copper, increased their resistance with temperature). Others noticed photovoltaic effect, which induces current flow in material upon exposure to light. In 1878 Edwin Herbert Hall discovered the Hall effect, which causes the production of voltage difference across an conductor when magnetic field was applied to it. All this eventually lead to discovery of electron in 1897 by J.J. Thomson, which later would lead to creation of p-n junction in 1939 by Russell Ohl of Bell Laboratories, which is the base of all modern digital electronics." -->

### Semiconductors

@import "../../img/pres_mcu_basics/Pn-junction-equilibrium.png"

<!-- slide class="img-90" data-notes="Simplest thing we can make out of p-n junction is an electrical diode. The general principle of operation of the diode is as follows: when the p-n junction is polarized, it's resistance changes, depending on how it's polarized (either in forward bias, where p-type is connected to positive terminal, and n-type to negative, or in reverse bias)". -->

### P-N Junction: a diode

@import "../../img/pres_mcu_basics/PN_diode_with_electrical_symbol.svg"

Diodes have many applications and characteristics, but the basic principle of operation is that they conduct the electricity well only in one direction.

<!-- slide class="img-50" data-notes="Following image shows the typical rectifier diode characteristics. When polarized in forward bias, at some point it hits forward bias voltage, it's resistance drops to almost zero (in this chart it's indicated by rapidly rising current) and the current can freely move throught the diode. However, in reverse bias, the resistance is close to infinite and current cannot flow, until the voltage rises to breakdown voltage which results in permanent damage (and sometimes smoke, or worse). Just before it hits the breakdown voltage, there is a small area where the diode actually conducts some current, and this effect is used in some types of diodes (for example, Zener diode)" -->

#### Rectifier diode characteristics

@import "../../img/pres_mcu_basics/diode_characteristics.jpeg"

<!-- slide class="img-80" -->

@import "../../img/pres_mcu_basics/anim_diode.gif"

<!-- slide -->

#### Types of diodes

@import "../../img/pres_mcu_basics/types_of_diodes.gif"

<!-- slide class="img-50" -->

### Transistors

@import "../../img/pres_mcu_basics/bipolar_transistor_scheme.png"

Transistors act like electrical switches that can be opened and closed with relatively small amount of current.

<!-- slide data-notes="When proper electrical potential is applied to the base of the transistor, it begins to open, allowing two junctions to diffuse and conduct current. When the potential is taken away, the transistor closes and junctions return to the default state, as can be seen on the animation" -->

#### How does transistor work?

@import "../../img/pres_mcu_basics/Threshold_formation_nowatermark.gif"

<!-- slide -->

#### Types of transistors

@import "../../img/pres_mcu_basics/transistor_types.jpg"

<!-- slide -->

#### Symbols of transistors

@import "../../img/pres_mcu_basics/transistor_symbols.gif"

<!-- slide -->

#### What can we make out of them?

- Switches
- Power amplifiers
- Logical gates
- Logical level converters
- Memories
- And much more...

<!-- slide data-notes="Simplest transistor circuit - if no current flows throught the base, NPN transistor will remain close. As soon as we apply enought voltage to force current flow, it will open and start conducting the signal from collector to emitter." -->

#### So, how it's made?

##### Transistor switch

@import "../../img/pres_mcu_basics/transistor_switch.png"

<!-- slide class="img-50" -->

@import "../../img/pres_mcu_basics/anim_switch.gif"

<!-- slide data-notes="This circuit is the simplest power amplifier, made using NPN bipolar transistor, bias resistor (to help stabilize the transistor and prevent too much current from flowing), and capacitor to filter out DC signal. When a signal comes to the base, transistor will work as power amplifier and will drive the speaker. Therefore, with small amount of current (usually micro/milliampers) required to open and close (drive) the transistor, we can drive something that requires much more current (9V/8Ohm = 1.125A)" -->

##### Power amplifier

@import "../../img/pres_mcu_basics/power_amp.png"

<!-- slide class="img-90" -->

@import "../../img/pres_mcu_basics/anim_amp.gif"

<!-- slide data-notes="Transistors are the smallest building block of every digital circuit. The second smallest block is a logical gate - circuit usually made out of 1-2 transistors, that performs some logical operation according to Bool algebra." -->

##### Logical gates

@import "../../img/pres_mcu_basics/gates_transistors.png"

<!-- slide -->

@import "../../img/pres_mcu_basics/logical_gates_tables.png"

<!-- slide -->

@import "../../img/pres_mcu_basics/anim_logic.gif"

<!-- slide data-notes="Logical level converters are often used in circuits where there's multiple different chips that work on different voltage levels (for example 1.8V and 3.3V or 3.3V and 5V)" -->

##### Logical level converter

@import "../../img/pres_mcu_basics/level_converter.png"

<!-- slide data-notes="This is one of the most popular level converter designs. In here, we have a single N-type MOSFET transistor, two resistors pulling each side of converter to wanted voltage levels - 3.3V (HIGH_LS) and 5V (HIGH_HS) in this case. When both sides are turned off, the transistor is also turned off because voltage on gate and source is equal to 3.3V. On the drain side, it's being pulled up to 5V via resistor so it stays high. When 3.3V device pulls the low side of the converter to the LOW level (0V), the source of the MOSFET is also being pulled down, but gate stays at HIGH_LS. Vgs rises above the threshold required to open the transistor, and MOSFET starts conducting. Since the MOSFET is open, and source is LOW, drain also gets pulled to LOW. Note that the LOW level doesnt have to be 0V, it just has to be high enough that HIGH_LS - LOW is bigger than Vgs threshold voltage of the transistor, so it will be able to open. The last possible state is when the high side pulls the line to LOW. In that case, the MOSFET is opening due to drain-substrate diode, which starts conducting (as it's getting polarized), pulling the drain to LOW level and hence opening the transistor." -->

@import "../../img/pres_mcu_basics/anim_converter.gif"

<!-- slide data-notes="Flash memory uses MOSFET transistors with two gates instead of one, separated with oxide layers that normally does not allow them to pass throught. When positive voltage is applied to wordline and bitline (gates and drain of the transistor), electrons start flowing from source to drain and some of them manage to get throught the oxide layer of gate, where they are entrapped. They can be flushed out by applying negative voltage to wordline, clearing the information. However, these transistors tend to degrade over time with write cycles, so you can usually check how durable is the disc by checking the amount of guaranteed write cycles." -->

##### Semiconductor memory (flash)

Flash memory uses characteristic transistors that can "trap" the electrons inside them, effectively storing a bit of information

@import "../../img/pres_mcu_basics/memory_mosfet.png"

<!-- slide data-notes="Czochralski process is the oldest and most common way of creating metal/metalloid or alloy monocrystals. It starts with melted material (for example, silicon polycrystals, sometimes additionally doped with metals to change the properties of final material), into which a single seed crystal is placed. The molten substance begins to crystallize on seed crystal, forming big monocrystal - sometimes this process is supported mechanically, for example by rotating the seed crystal or crucible." -->

### How do we put a lot of transistors in very dense, small space?

@import "../../img/pres_mcu_basics/Czochralski_Process_en.jpg"

Chochralski Process (crystal pulling) is a crystal growth technique that's most commonly used in process of making a silicon wafer.

<!-- slide data-notes="To create integrated circuit, we first need a silicon wafer. Silicon wafers are made by slicing silicon monocrystals into thin plates. However, the wafer is only a base - it has to be treated to make final circuit out of it." -->

#### Silicon wafer - base of any integrated circuit

@import "../../img/pres_mcu_basics/wafer.jpg"

Silicon wafer is made by slicing silicon monocrystal into thin plates. After that, it's put throught several manufacturing steps that create semiconductive structures on it.

<!-- slide class="img-50" data-notes="Raw, silicon wafer goes from foundry to front-end facility, where it's first deposited with layer of metals and other compounds out of which the transistors and other semiconductors will be formed, and also with photoresistive layer. Then, the wafer is etched to form the structures and connections. This process creates a single layer of semiconductors, and can be repeated until the chips are finished. After that, the silicon wafer is tested, cut into chips (which are tested again), and transported to back-end facility which will put them in casings, so they can be placed on PCBs." -->

#### How to make a circuit on silicon wafer?

@import "../../img/pres_mcu_basics/Basic-Semiconductor-Process-Chart.jpg"

<!-- slide data-notes="What is a signal? Take your voice for example - it is an analog signal, represented by change of pressure in the air. This signal carries information - the message, encoded with words in specific language. It's analog, because it's continuous in time and value - we can linearly change the tone and volume of the voice, and we speak continuously (we don't add gaps between letters). Digital signals are not continuous neither in time nor value. They have fixed values and they happen in fixed time intervals." -->

## Digital (and analog) signals

@import "../../img/pres_mcu_basics/What-are-Analog-and-Digital-Signals.png"

<!-- slide -->

### How can we create an electric signal?

Signal is a change of voltage in time. You can plot voltage between any two points of electric circuit, and it will be a signal.

In practice, digital signals are created mostly by switching transistors on and off, and analog signals in digital circuits are generated using DACs (Digital-to-Analog converter)

<!-- slide data-notes="draw some signals and explain the terms" -->

### Most important signals characteristics

- **Frequency / bandwidth** _[Hz] (Hertz)_ - How fast does the signal change?
- **Amplitude** _[V] (Volt)_ - How strong is the signal?

If the signal is periodic (it's "looped", constantly repeating itself) it also has **Phase** _[$\phi$] (Radian/Degree)_, which defines the starting position of signal compared to it's value at starting point (defined by it's formula)

<!-- slide class="img-80" -->

### How do we measure signals?

@import "../../img/pres_mcu_basics/1280px-Sampled.signal.svg.png"

<!-- slide data-notes="draw a diagram presenting what happens if we measure an x Hz signal with frequency (0, x), x and (x, 2x)" -->

### Nyquist–Shannon sampling theorem

_If a function $x(t)$ contains no frequencies higher than $B$ hertz, it is completely determined by giving its ordinates at a series of points spaced $\frac{1}{2B}$ seconds apart._

In other words, to correctly measure a signal we have to check it's value at least twice as often as it changes. Otherwise, we won't be able to measure it correctly.

<!-- slide -->

## Microcontroller - the brain of most modern devices

Microcontroller is a singular piece of silicon that contains stuff commonly found in personal computers, like

- Memory - usually at least two of them: RAM and Flash (permanent)
- Processing core - the core of the chip, that does the actual math
- Peripherals - everything else, like power supplies, communication interfaces and more

<!-- slide -->

### Microcontroller vs Processor

There are few major differences between microcontrollers (MCU) and (micro)processors (CPU)

<!-- slide class="img-40" -->

MCU is a singular piece of silicon that (usually) contains everything it requires to work inside it. So, you can just power it up and in most cases it will work as intended. Think of it like of a computer, integrated into single chip.

@import "../../img/pres_mcu_basics/stm32_mcu.jpg"

<!-- slide class="img-40" -->

CPU contains only the processing cores and fastest/core peripherals (like PCIe and memory controllers, along some other stuff). It doesn't have any permanent memory inside (at least not one accesible to user). CPUs integrated with memories and other peripherals usually not found in traditional CPUs are called Systems-on-a-Chip (SoC's)

@import "../../img/pres_mcu_basics/ryzen.jpg"

<!-- slide data-notes="8051 is based on 3.5um litography process, popularly used during mid 1970s. It had 4kB of internal ROM memory, 128B of RAM and 8-bit Intel core running from 3.5 to 12MHz, along with few timers and I/O peripherals" -->

### Microcontroller's architecture

Below, you can see architecture of Intel's 8051 microcontroller from 1980, one of the oldest MCUs on the market

@import "../../img/pres_mcu_basics/8051-microcontroller.png"

<!-- slide class="img-height-fit" data-notes="Here we can see block diagram of modern and relatively simple STM32G0 32-bit MCU, from 2018, made using 90nm litography process. This specific MCU can run with core clock up to 64MHz (while some timers can run twice this speed), it has up to 36KB RAM and 128KB Flash memory. It also uses propertiary ARM Cortex-M0+ core. It costs around 2-3USD per unit" -->

@import "../../img/pres_mcu_basics/mcu_block_diagram.png"

<!-- slide class="font-80" -->

#### The core

Every MCU has a core. The core is the element that executes the program, so we can say that it's the most important part of the MCU. There are few different architectures of cores, for example:

- ARM Cortex-M - One of the most popular architectures novadays, very flexible, low-cost and energy efficient. Lots of manufacturers moved from their propertiary architectures to ARM.
- Atmel AVR - very popular, due to the fact that most popular Arduino boards run on Atmel AVR MCUs. Also pretty old and not very fast, but still relevant
- Xtensa - found in Espressif MCUs (ESP32/ESP8266)
- Texas Instruments MSP - found in Texas MCUs
- Microchip PIC - found in Microchip PIC MCUs

And many more... But the rest of this presentation will be focused mostly on STM32 ARM MCUs!

<!-- slide class="font-80" -->

The core, besides the computing units, also has few interfaces that connects it to the rest of the microcontroller, or outside world, or other elements that require direct access to CPU. Notably, they usually are

- Debugging interfaces - in case of ARM MCUs, it's usually SWD (Single-Wire Debug) interface, other MCUs have other popular or propertiary standards (like JTAG). This allows to debug the program in real-time on the physical hardware.
- Bus connections - In order for CPU to do anything meaningful, it has to be connected to the rest of the MCU. It's done using a bus, that interconnects all MCU elements togeher.
- Core peripherals - like interrupt controller, for example, that requires direct access to the core.

<!-- slide -->

#### Memories

Every MCU has at least two types of memory.

- SRAM - Static RAM memory. The same type is used for CPU cache. It's the fastest type of semiconductor memory available.
- Flash - Permanent programmable memory. Usually used to store the program (one, or many) that microcontroller is supposed to run

Sometimes, the MCU also has interfaces for external memories, that allows to extend the existing memory. Advanced MCUs can have multiple Flash and/or RAM memories. There are also MCUs with additional cache memory inside the core, that boosts performance up.

<!-- slide -->

#### Buses

Every CPU and MCU has some buses that interconnect the peripherals inside them, allowing communication between core, peripherals and memories. In case of most ARM MCUs, the bus consist of:

- Bus multiplexer - this part connects multiple different buses togeher, and controls how they communicate with each other.
- Advanced High-Performance Bus (AHB) - connects the most important periperhals of MCU, like memories, DMA controllers, core, clock controllers, interrupt controllers and so on.
- Advanced Peripheral Bus (APB) - connects the peripherals to AHB via ABH-to-APB bridge

<!-- slide -->
#### Interrupt controller

Interrupts are used to tell the MCU core that something very important has happened, and it has to be handled right now. Most periperhals have their interrupts that can be used to notify the core of something happening, the core itself also has some interrupts that can tell us if something went very wrong. The mechanism of interrupts is similar to exceptions, althought they are not exactly the same thing.

Interrupt controller allows us to configure the interrupts and manage them.

<!-- slide -->

#### Direct Memory Access controller

Direct Memory Access controller (DMA) is a controller that can copy the data from one place in memory (or peripheral) to another, without involving MCU core. By default, when we write code that copies some data from location A to location B, it's done by the core that has to manually copy every word from A to B. However, since it can take a long time, and core has probably better things to do in meantime, we have a DMA controller that can do it instead.

<!-- slide -->

#### GPIO controller

GPIO stands for General Purpose Input/Output, and this controller manages the physical inputs and outputs of microcontroller.

<!-- slide -->

#### Reset and Clocks Controller

Reset and Clocks Controller (RCC) is used to handle the reset of MCU (which happens either when we pull one of the inputs to ground, or force it via software) and generate clocks for the core, buses and peripherals of MCU. It usually consists of some clock dividers and PLLs (Phase-Lock Loops) that can increase or decrease the clock coming either from internal MCU oscillator, or external one.

<!-- slide -->

#### Power supply and supervisor

Power supply section of microcontroller manages the voltage levels of it's internal components. It consists of programmable voltage converters and stabilizers, that generate proper voltages for some MCU components, like core (which usually requires much lower voltage than the rest of MCU). It also has some monitoring blocks, which check if the voltage is stable, and notify the core if something bad happens. It also can put the MCU into low-power, sleep or shutdown state.

<!-- slide -->

#### Watchdogs

Watchdogs are used to reset the microcontroller in case when it halts. Usually, when we enable the watchdog, we have to notify it once a while to prevent it from restarting the MCU (and our program). In case when our program crashes or halts, it will stop notifying the watchdog, and then it will automatically restart the microcontroller.