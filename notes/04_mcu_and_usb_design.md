# MCU and USB Design

## STM32F103C8T6 Microcontroller

### Power Pins

The various VDD as well as VBAT pins of the STM32F103C8T6 microcontroller are connected to the 3.3V digital power supply. Per VDD and VBAT pin, ST recommends a 100nF decoupling capacitor to be connected to the ground. Additionally, a 10uF capacitor is connected as a bulk coupling capacitor to the 3.3V digital power supply for stabilization.

For the analog power supply, the VDDA pin of the microcontroller is connected to the 3.3V analog power supply. To provide additional filtering, a 10nH inductor is placed in series with the 3.3V analog power supply, followed by a 1uF capacitor. ST further recommends using a 1uF and a 10nF bypass capacitor, both connected to the ground.

Regarding grounding, the VSS and VSSA pins are connected to a common ground, which is shared between the digital and analog power supplies.

### Configuration Pins

- **NRST**: Active low reset pin, usually a pulled-up resision is connected to this pin. But we are going to use the SWD interface to connect to
this pin.
- **BOOT0**: Boot mode selection pin, pulled down to ground with a 10k resistor. If BOOT0 is low it will boot from the main flash memory, if it is
  high it will listen for debug signals on various ports.
- **HSE_IN & HSE_OUT**: External high-speed clock input and output pins. Wee will rely on external clock source for the microcontroller, since they are
  more accurate and stable than the internal RC oscillator.


### High-Speed External Clock

For more detailed information about the high-speed external clock, refer to the [ST AN2867 Application Note](/datasheet/AN2867.pdf).

![STM32F103C8T6 Clock Configuration](/notes/img/hse_design.jpeg)

We need to select the appropriate crystal (Q) for the microcontroller, the required values for the load capacitors ($C_L$) and the feed resistors
($R_{Ext}$). Given the following crystal parameters:

- $f_0$ = 16 MHz
- $C_0$ = 10 pF
- ESR = 80 Ohm

An an estimate stray capacitance ($C_{S}$) of typically 3-5 pF, we can calculate the required values for the load capacitors.

$$
C_{L} = C_{L1} = C_{L2} = 2 \cdot (C_0 - C_S) = 2 \cdot (10 - 4) \quad \rightarrow \quad C_{L} = 10 pF \text{ to } 14 pF
$$

We need a feed resistor ($R_{Ext}$) to limit drive level, which prevents distortions. Also with the load capacitors, it forms a low-pass filter
to suppress harmonics. The value of the feed resistor is calculated as follows:

$$
R_{Ext} = 0.1 \cdot \frac{1}{2\pi \cdot 16MHz \cdot 10pF} = 100 \Omega \quad.
$$

As a rule of thumb we use $C_{L}$ and $f_{0}$ for calculating $R_{Ext}$, using the RC filter formula. Also we reduce $R_{Ext}$ by a factor of 10.
In the end, we choose a 0 Ohm resistor for $R_{Ext}$, since the crystal has a built-in ESR of 80 Ohm. If the crystal does not work properly we
can change the resistor to a higher value.

### Peripherals

Where these pins needs to be connected o can be found using the STM32CudeIDE, where the pinout is shown. The following peripherals are used:

- **SPI1 & SPI2**: Used for serial communication with other devices. The NSS pins are pulled up to the 3.3V digital power supply with a 10k
  resistor.
- **TIM_CH1 & TIM_CH2 & TIM_CH3**: Timer channels are used for PWM generation. These are connected to the RGB LED of the board.
- **USB**: The USB pins are connected to the USB connector. The D+ pin is pulled up to the 3.3V digital power supply with a 1.5k resistor and the D-
  pin is connected to the data lines of the USB connector. See the [ST USB Guidelines](/datasheet/AN4879.pdf) for more information.
- **SWD**: The SWD pins are used for programming and debugging the microcontroller. The three important pins are SWO, SWIO and SWCLK. 


## USB Connector

### USB Type C Connector

The MCU requires a 1.5k pull-up resistor on the D+ line of the USB connector. The D- line is connected to the data lines of the USB connector.
For type C  connectors, the [TA0357 Application Note](/datasheet/TA0357.pdf) from ST provides detailed configuration information. To tell the
host device to enable USB VBUS (+5V) at up to 500 mA current, we need to connect the CC1 and CC2 pins (CC = configuration channel) to a 5k1
resistor each (sepereate resistors). 

### ESD Protection

The supply might be very noisy due to the long cable, which can pickup high-frequency noise. To protect the USB lines from ESD, we use a common
USSBLC6-2SC6 ESD protection diode. The diode is connected between the D+ and D- lines of the USB connector. The diode has a low capacitance of
0.6 pF and a low clamping voltage of 6V. The diode is connected to the 3.3V digital power supply. and the common mode choke.

## Serial Wire Debug

The SWD (Serial Wire Debug) is an in-circuit debugging and programming protocol, similar to JTAG. It consists of the SWDIO and SWCLK signals. The SWDIO is a bidirectional data line used for communication, while the SWCLK is a clock signal that synchronizes data transfer. Additionally, the SWO pin is used for real-time debugging, allowing variable monitoring, and the NRST pin is used to reset the microcontroller.

For ESD protection, bidirectional TVS diodes will be connected to ground to protect the MCU from voltage spikes. A 49.9kΩ resistor will be placed between the connector and the MCU to prevent short circuits. To filter out high-frequency noise and prevent sporadic resets, a 100nF capacitor will be placed next to the NRST pin.
