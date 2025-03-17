# ADC & Analog Circuitry

The following sections will cover the analog-to-digital converter (ADC), and its analog-front-end (AFE) circuitry. It also has a differential
input stage, which is used to measure the differential voltage between two input signals. Although we will be using the ADC in single-ended mode.

## Analog-to-Digital Converter

![ADC IC](/notes/img/adc_schematic.png)

The chosen ADC for this design is the ADC141S626 from Texas Instruments. This is a 14-bit ADC with a variable sampling rate that can range from 50 kSPS to 250 kSPS.

### Power Pins
For the power input (VA and VIO), we are using a 3.3V digital power supply, with the VA pin connected to the 3.3V analog supply. Both of these power inputs are decoupled with bypass capacitors: each input is paired with a 10µF and 100nF capacitor. These capacitor values were selected with Bill of Materials (BOM) consolidation in mind. The ground pins of the device are connected to a common ground. Although splitting the ground into separate analog and digital sections is a common practice, in this design, the shared ground approach is chosen to minimize potential ground loops and interference, which will be further addressed in the PCB routing section.

### ADC Input Pins
The ADC141S626 features a differential input stage, allowing it to measure the differential voltage between two input signals. These differential inputs are provided by the analog front end (AFE), which is responsible for conditioning the input signals for optimal performance in the ADC.

### Reference Voltage
To ensure accurate and stable measurements, a reference voltage is required for the ADC. While we could use the output of the regulator as the reference, this voltage can fluctuate due to load changes, is not particularly stable, and is often noisier than a dedicated reference source. To improve performance, we are using the REF3033 from Texas Instruments, which provides a stable 3.3V reference voltage.

The REF3033 is powered by the +5V from the USB connector, but due to the high noise present on the USB power, we apply filtering before powering the device. A low-pass filter is used, consisting of two 10µF capacitors in parallel, with a 49.9Ω resistor between them. This configuration will help reduce high-frequency noise and ensure a clean reference voltage for the ADC.

### SPI Serial Interface
The ADC uses the SPI interface solely for outputting data. Therefore, the SPI interface is used only for reading data from the ADC. The DOUT pin is connected to the MCU’s MISO pin, the SCLK pin is connected to the MCU’s SCK pin, and the NCS pin is connected to the MCU’s NSS pin. A 0Ω resistor is placed between the clock pins, which is useful for high-speed SPI communication to reduce signal reflection and improve signal integrity.

In cases where a high-speed SPI interface is required, a series resistor can be added between the ADC and the MCU to prevent signal ringing and
mitigate electromagnetic interference (EMI). This resistor helps to maintain signal quality at higher data rates.

## Analog Front-End

![ADC Front-End](/notes/img/analog_front_end.png)

The Analog Front-End (AFE) is designed to provide impedance conversion and an anti-aliasing filter for the ADC. The ADC itself has a relatively
low input impedance (typically a few kΩ), but to avoid loading the signal source, we require a much higher input impedance in the AFE.

### 1st Stage

![ADC Analog Front-End 1st Stage](/notes/img/analog_front_end_1st_stage.png)

The AFE starts with a BNC connector, which allows for easy connection of the input signal to the board. Following the BNC connector is a TVS
diode for ESD  protection. It's important to note that the input range of this design is limited to ±1.65V, which corresponds to half of the 3.3V
supply voltage. To filter out high-frequency noise and prevent artifacts or aliasing in the ADC measurements, an RF filter with a 10kΩ resistor
and a 150pF capacitor is used. This combination creates a low-pass filter with a cutoff frequency of 1.1 MHz, ensuring that only the desired
signal frequencies are passed to the ADC.

This AFE design is not suitable for DC measurements because we are running on a single supply and biasing the operational amplifier (op-amp) to
half of the supply voltage. To prevent any DC bias on the input from interfering with the op-amp's input bias, an AC coupling capacitor (C404,
100nF) is included. Without this capacitor, any DC component on the input could negatively affect the op-amp's operating point.

The input impedance of the op-amp is defined by resistors R401 and R407, each valued at 2.2MΩ. At AC, the C404 capacitor acts as a short circuit,
and the two resistors combine to provide a 1.1MΩ AC input impedance. The bias voltage for the op-amp is supplied by a bias voltage generator
(VCOM) and is connected to R401. Due to the high resistance, no significant current flows, ensuring a stable bias voltage. Furthermore, the
configuration of C404, R401, and R407 forms a high-pass filter with a cutoff frequency of 1.4Hz. This allows for accurate measurements down to
approximately 2Hz without significant attenuation.

The first op-amp (U401) is configured as a voltage follower, which provides a high input impedance and low output impedance. This configuration
serves as a buffer between the input signal and the anti-aliasing filter, preventing any loading effects that could distort the signal. The
anti-aliasing filter is designed as a Sallen-Key low-pass filter and, in this case, is a 3rd-order Butterworth filter. The filter has a cutoff
frequency of 25kHz to ensure that the upper frequency limit of the range of interest (20kHz) is not attenuated significantly. This ensures the
ADC can sample signals accurately without distortion at the frequencies of interest.

### 2nd Stage

![ADC Analog Front-End 2nd Stage](/notes/img/analog_front_end_2nd_stage.png)

The filter is active because it involves an active component—the second op-amp (U9). If only passive components were used, impedance variations
at different frequencies and loading effects could result in a degraded frequency response. The active op-amp helps to maintain a stable and
accurate frequency response regardless of input conditions and load variations. The order of the filter is determined by the number of capacitors
in the circuit, namely C401, C406, and C407. The filter’s design can be optimized by selecting appropriate values for capacitors and resistors.
Tools such as this [online filter calculator](http://sim.okawa-denshi.jp/en/Fkeisan.htm) can assist in choosing the correct component values.

Using the ADC in differential input mode would be advantageous if we were dealing with a differential signal, as it would improve the
signal-to-noise ratio (SNR). However, in this design, we are using the ADC in single-ended mode, which is sufficient for our needs, as we are
targeting a low-precision instrument. At the output of the second stage, we place a feed resistor (R406, 1kΩ). This ensures that, in the event of
a short circuit at the output, the op-amp will not be damaged. Additionally, we include a capacitor (C408, 150pF), which, in combination with the
feed resistor, forms another low-pass filter to further reduce high-frequency noise. Before the signal is fed into the ADC, we perform a
'balanced' conversion to ensure that both ADC inputs are at the same voltage level. This "balanced" conversion means that both inputs are driven
by equal resistances. Specifically, ADC_IN+ will oscillate around VCOM, while ADC_IN- remains fixed at VCOM. This process effectively transforms
the signal from a single-ended to a pseudo-differential format, which helps improve the performance of the ADC in single-ended mode.


