# System-Level Design

In this project the goal is to design a system with the following
characteristics:

- USB powered
- Single channel
- Low-frequency signal analyser and signal generator

This system can be used as a very basic and inexpensive test and measurement
equipment for hobbyists and students. For example:

- Freuqency response analysis of electronic circuits
- Arbitrary signal generator
- PC-based oscilloscope

## Initial System Requirements

Here we can refine the system requirements from the characteristics above:

- USB powered
  - The system should be powered from a USB port
  - The system should not draw more than 500mA from the USB port (nominally +5V
    bus voltage, that gives us max. 2.5W)
- Low-frequency
    - Audio frequency range (20Hz - 20kHz)
- Signal analyser
    - Need an single channel ADC
- Signal generator
    - Need an single channel DAC

### System Block Diagram

![System Block Diagram](/img/system_diagram.jpeg)

Futher considerations:

- USB Connector
  - USB 1/2/3?
  - USB Type A/B/C?
  - Is the system a USB device/host or both (OTG)?

-Power Supply:
    - Linear or switching?

- Processor:
    - Brand (STM32, TI, Microchip, etc)?
    - Specs (speed, memory, peripherals)?
    - Programmable (MCU) or fixed (FPGA)?

- Data Converters (ADC and DAC):
    - Resolution (8/10/12/16/24 bits)?
    - Speed (samples per second)?
    - Interface (SPI, I2C, parallel)?

- Analog sections:
    - ADC front-end (high-impendance, anti-aliasing filter,...)?
    - DAC circuitry (output buffer, ranti-aliasing filter,...)?

## Detailed System Requirements

### 1. USB 

This design operates within a frequency range of **20 Hz to 20 kHz**, with a
maximum frequency of interest of **20 kHz**. According to the **Nyquist
theorem**, the minimum required sampling frequency must be at least twice
the maximum frequency of interest. Therefore, the system requires a minimum
sampling frequency of **40 kHz**. To reduce quantization error, an ADC
resolution of at least 12 bits is necessary. This results in a minimum data
rate calculated as follows: 

$1 \text{ channel} \times 40\text{kHz} \times 12 \text{ bits} = 0.48 \text{ Mbit/s}$

The USB 2.0 specification defines two data rates: **Full-Speed (12 Mbit/s)** and
**High-Speed (480 Mbit/s)**. Given our system's requirements, the **Full-Speed
mode (12 Mbit/s) is more than sufficient** for reliable operation. Additionally,
the system is designed to be **bus-powered only**, without acting as a host. We
anticipate that its power consumption will remain well below the **500 mA
limit** imposed by USB 2.0. To ensure **modern compatibility and
userconvenience**, we have chosen the **USB Type-C connector**, which is
becoming increasingly popular due to its **reversible design and 
widespread adoption**.

### 2. Power Supply

It is good practice to split up digital and analog power supplies to prevent
digital noise from affecting the analog circuitry. 

For the digital section, a **switching power supply** (buck or boost converter)
is more efficient and is far more tolerant of noise. Our digital circuitry draws
100s of mAs maximum. For the analog section, a **linear regulator** (LDO) is
more suitable due to its improved noise performance. The analog circuitry draws
around 10s of mAs maximum.

The system is powered from a USB port, which usually contains a very noisy power
rail. To prevent this noise from affecting the analog circuitry, we need to
provide adequate filtering. The USB power rail is nominally +5V, but it can go
as low as +4.5V. Here we have to watch out for the regulator's dropout voltages.
The USB power rail can deliver up to 500mA, if negotiated with the host.
Otherwise it is limited to 150mA, so let's aim for a current draw of less than
150mA.

### 3. Processor

Since we are dealing with a low-frequency signal, the processor does not need to
be particularly powerful. Therefore, an FPGA or a high-performance
microcontroller is unnecessary. Instead, we can use a low-cost microcontroller
with a built-in USB interface to stream data to the PC.

Programming the microcontroller in C/C++ is simpler compared to programming an
FPGA with VHDL/Verilog. Additionally, debugging can be done using Serial Wire
Debug (SWD) or JTAG, making development more accessible. For ADC and DAC
communication, we can utilize SPI or I2C interfaces.

Regarding processing power, a low-power ARM Cortex-M0 microcontroller with a low
clock speed is more than sufficient for our needs. To further  reduce cost and
complexity, we aim to avoid BGA and QFN packages.

### 4.Data Converters

For data converters, the following basics facts are true:
- More bits the better the signal-to-noise ratio (SNR).
- Higher sampling rate results in higher Nyquist limit and less alaising.
- In both cases more data needs to be stored and processed.  

Additional features to consider:
- Oversampling: Increases resolution by averaging multiple samples.
- Anti-aliasing filter: Removes high-frequency noise before sampling.

For the ADC we might need a high-impedance (> 1 MOhm?) input to avoid drawing
too much current from the signal source. An input buffer with ESD protection is
also recommended, since we are probing and measuring external signals. RF
filtering might be necessary to prevent high-frequency noise from entering the
system. To avoid aliasing, an anti-aliasing filter is required with a cutoff
frequency of half the ADC's sampling frequency.

For the DAC we need a low-impedance (~50 Ohm?) with ESD protection output to
drive the signal to the load. Also an anti-aliasing filter with a cutoff
frequency of half the DAC's sampling frequency is required.

### 5. Miscellaneous Considerations

- Mechanical:
    - PCB dimension constraints, mounting holes, etc.

- Connectors:
    - Debugging interface(SWD, JTAG, UART), Input/Output connectors (SMA, BNC)

- Peripherals:
    - LEDs, buttons, etc.

- Other:
    - ESD protection, filtering for EMC
    - Timing (crystal frequency, types, etc.)
    - Biasing for single-supply analog circuitry

## Final Detailed System Diagram

With the above requirements in mind, we can now create a more detailed system
diagram.

![Detailed System Diagram](/img/detailted_system_diagram.jpeg)