# 2. Part Selection

## Part Selection Basics

1. **Select the main parts first:** In our case, the main parts are the
   microcontroller, ADC, and DAC. We need to make sure that the voltage
   requirements are same or similar for these parts.

2. **Choose the power supply after calculating the power requirements for the
   relevant parts:** Power supplies need to handle continuous and peak currents
   at given input and output voltages.

3. **Choose additional components (resistors, capacitors, etc.) for the surrounding
   circuitry:** These components are usually chosen based on the datasheets of
   the main parts.

4. **Choose specific connectors last:** General types should be of course be known
   early on, but the specific connectors can be chosen last.

## Part Selection

### 1. Microcontroller (MCU)

** Important Specs:**
- Programming tools and IDE
- Bus width (8/16/32 bits)
- Speed (8 MHz, 16 MHz, 48 MHz, etc.)
- Flash and RAM sizes
- Number of cores (usually 1), DMA and supply voltage
- Peripherals (ADC, DAC, SPI, I2C, etc.)
- Package type (DIP, QFN, BGA, etc.) and size
- Cost

**For this design:** [STM32F103CBT6](https://www.mouser.com/ProductDetail/STMicroelectronics/STM32F103CBT6?qs=WxFF5lh7QM3goh6GV5Ogig%3D%3D)
- Very common MCU, LQFP-48 package, +3.3V, 72 MHz, 128 KB Flash, 20 KB RAM
- Support for USB 2.0 Full-Speed natively and SPI


### 2. ADC and DAC

ADC and DAC are primarily chosen based on the **resolution** and **speed**
required. However, many other factors can influence the performance of these
parts.

**Important Specs:** 
- Single channel and single-ended (differential is also possible)
- At least 12-bit resolution and at least 40 kSPS sampling rate to cover the
  audio range of 20 Hz to 20 kHz
- SPI interface and +3.3V supply voltage
- Suitable package type and low cost

**For this design:** 
- [ADC141S626CIMM/NOPB](https://www.mouser.com/ProductDetail/Texas-Instruments/ADC141S626CIMM-NOPB?qs=7X5t%252BdzoRHCk2zqpP70%252BpA%3D%3D)
  :14-bits, 50 kSPS to 250 kSPS, differential input
- [DAC7563SDGSR](https://www.mouser.de/ProductDetail/Texas-Instruments/DAC7563SDGSR?qs=QtI0yD1FyOPUuM44rgyQjg%3D%3D)
  :12-bits, single-ended output

### 3. Power Supply

**Supply requirements:**
- +4V5 < V(USB) < +5V
- Digital: 100mA to 200mA (MCU, ADC, DAC, LEDs)
- Analog: 10mA to 50mA (ADC, DAC, OpAmps)

**Types of DC/DC converters:**
- Switching (buck, boost, buck-boost): High efficiency, higher power, more
  noise, more complex
- Linear (LDO): Low noise, low power, simple, less efficient

As a general rule, digital sections use switching power supplies, while analog
sections use linear regulators. 

**Switching regulator:** [TLV62569DBVR](https://au.mouser.com/ProductDetail/Texas-Instruments/TLV62569DBVR?qs=0C8XhJW8e4oaylc%252BzBrCmA%3D%3D)
- Up to 95% efficiency, 2A max current (which is overkill)
- +2V5 < Vin < +5V5
- 0V6 < Vout < Vin
- 1.5 MHz switching frequency
- Extra features: soft-start, over-current protection, thermal shutdown

![Switching Regulator](/notes/img/switching_regulator.png)

**Linear regulator:** [HT7533-1](https://www.digikey.com/en/products/detail/umw/HT7533-1/17635239)
- Very low dropout voltage (25mV typically)
- Max 100mA output current
- Small SOT package
- Only requires > 10uF input/output capacitors

LDO regulators typically require input/output capacitors to function.
![Linear Regulator](/notes/img/ldo_regulator.png)

### 4. Analog Circuitry

Here we need to target a low-noise design. It is important to aim for lower
resistance values since larger resistors can introduce more Johnson noise.

**OpAmp:** [MCP6001](https://www.mouser.de/ProductDetail/Microchip-Technology/MCP6001T-E-OT?qs=npqfqDPP3%2FwkX3GgGH%252BE4A%3D%3D)
- BJT vs. MOS -> Impedance? Bias current?
- Rail-to-rail -> Input and/or output?
- Single-supply vs. dual-supply -> Input and output voltage range? Biasing?
- Bandwidth
- Stability
- Decoupling
- Package
- Cost

**ADC and DAC Interface:**
- Single-ended vs. differential
- Drive strength
- Max. capacitance

**Filters:**
- Component values for required cutoff frequencies, impedance, etc.


### 5. Surrounding Circuitry

The requirements for the surrounding circuitry are typically given in the
datasheets of the main parts, the datasheets are located [here](/datasheet/).

**Very Important:**
- Voltage/Current/Power ratings
- Material/Process types (resistor: thin/thick film,..., capacitor: X5R,
  Y5V,...)
- Temperature coefficients and tolerances
- Package sizes (0402, 0603, 0805,...)

**ESD Protection and EMI filters:**
- Most IC s include some form of ESD protection
- Good practice to add additional ESD protection (TVS diodes, etc.)
- Filtering at connectors, and power supplies

**Even though the circuitry is secondary, it is important to choose the right
components to ensure the system works as intended.**

### 6. Final Points

**BOM considerations:**
- Try to keep the number of different components low. For example, instead of
  using 1k, 1k05 and 1k1 resistors, use only 1k resistors.

**Try to choose standard valued standard components:**
- For example E-series:
    - Resistors: 1k, 2k2, 4k7, 6k8, etc.
    - Capacitors: 1n, 2n2, 4n7, 6n8, etc.

**If possible, choose components that are easy to assemble and debug:**
- This means at least 0402 passives, no BGA and no QFN packages