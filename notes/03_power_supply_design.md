# Power Supply Design

The power supply consist of an input filter, analog and digital power supplies, and a biasing circuit. The input filter
is a simple LC filter that filters out high frequency noise from the input power supply. The analog power supply is a
linear regulator that provides a stable 3.3V supply for the analog components. The digital power supply is a buck
converter that provides a stable 3.3V supply for the digital components. The biasing circuit provides a stable 1.2V
supply for the biasing circuit.

The following sections will describe the design considerations for the placements and values of the secondary components
of the power supply.


![PSU Schematic](/notes/img/psu_schematic.png)


## Input Filter

The inductor has a DC current rating of 500 mA, which matches the maximum current that a USB connection can provide. It is important to specify an inductor with a higher current rating than the expected operating current to ensure reliability and minimize saturation effects. Although the inductor will typically draw much less current (100 mA – 200 mA) during normal operation, the higher rating provides a safety margin, ensuring stable performance under varying load conditions.

The inductor has a value of **10 µH** and a **DC resistance of 0.15 Ω**. Without additional damping, the filter essentially forms a **resonant circuit**, which can introduce an undesirable peak at the resonant frequency. The **resonant frequency** is given by:  

$$f_0 = \frac{1}{2\pi\sqrt{LC}}$$

where:  
- $f_0$ is the resonant frequency,  
- $L$ is the inductance,  
- $C$ is the capacitance.  

To suppress resonance and prevent excessive peaking, **damping** can be introduced in two ways:  
- **Adding a series resistor** with the inductor to dissipate energy and reduce the Q-factor of the circuit.  
- **Relying on the inductor's inherent DC resistance** (0.15 Ω) to provide some degree of damping.  

The choice between these approaches depends on the required trade-off between damping effectiveness and power loss. If additional damping is necessary, selecting an appropriately sized resistor can help achieve a critically or optimally damped response without significantly increasing losses.

To determine the values for the input filter, we can use this [online tool](http://sim.okawa-denshi.jp/en/Fkeisan.htm), which provides various resources for visualizing the frequency response of different filter types. For our specific case, we will use the [RLC filter calculator](http://sim.okawa-denshi.jp/en/RLCtool.php).  

Using the given component values:  

- **Inductance**: $L = 10 \,\mu\text{H}$  
- **Capacitance**: $C = 10 \,\mu\text{F}$  
- **Resistance**: $R = 0.15 \,\Omega$  

The resulting filter has a **cutoff frequency of approximately 1.6 MHz**. The low-pass filter's **passband extends from 0 Hz to roughly 1 MHz**, effectively attenuating higher frequencies beyond this range.

![RLC Magnitude Spectrum](/notes/img/magnitude_rlc_filter.png)

This filter attenuates high-frequency noise from the input power supply. Since our design does not operate at high
frequencies, this filter is sufficient, as it effectively removes unwanted noise while maintaining the necessary
bandwidth for proper operation.

![RLC Magnitude Spectrum Peak](/notes/img/magnitude_rlc_filter_peak.png)

If we design the filter without any damping, we can observe the resonant peak in the frequency response as seen above. This peak can
cause instability and unwanted oscillations in the system. 

## Analog Power Supply (LDO)

According to the [HT7533-1 datasheet](/datasheet/ht75xx.pdf) , both the input and output capacitors must have a minimum capacitance of **10 µF**. In our design, we use **22 µF** capacitors for both, ensuring a margin above the requirement. Given a typical **20% tolerance**, the actual capacitance could drop to **18 µF**. Additionally, **DC bias effects** can further reduce the capacitance, reinforcing the need for a higher nominal value.  

To further improve stability, we place a **10 Ω series resistor (1/8 W)** at the LDO input. With a maximum current of **100 mA**, the power dissipation is:  

$$P = I^2 R = (0.1 A)^2 \times 10 Ω = 0.1\text{ W}$$  

This resistor, together with the **input capacitor**, forms a **low-pass filter** that attenuates high-frequency noise from the shared digital power supply. The filter’s **cutoff frequency** is given by:  

$$f_c = \frac{1}{2\pi R C}$$  

Substituting the values **R = 10 Ω** and **C = 22 µF**, we obtain:  

$$f_c \approx 723 \text{ Hz}$$

This cutoff frequency effectively suppresses high-frequency noise, ensuring a clean power supply for the **low-frequency analog circuitry**, where noise sensitivity is critical.  

## Digital Power Supply (Buck Converter)  

The components required for the buck converter can be found in the [TLV62569 datasheet](/datasheet/tlv62569.pdf) and are summarized below:  

- **Input capacitor(s)**  
- **Output filter** (inductor and output capacitor(s))  
- **Feedback network** (resistor divider)  

### Inductor Selection  
The inductor value is calculated using the formula:  

$$
L = \frac{V_{OUT} \times (V_{IN} - V_{OUT})}{f_s \times V_{IN} \times \Delta I_L}
$$

where:  
- $L$ = Inductance  
- $V_{OUT}$ = Output voltage  
- $V_{IN}$ = Input voltage  
- $f_s$ = Switching frequency
- $\Delta I_L$ = Inductor ripple current (typically 20% to 30% of the output current)

From the formula, we can deduce several important relationships. Increasing the switching frequency reduces the required inductance, allowing for a smaller inductor. Conversely, a lower output current necessitates a larger inductor to maintain stable operation. Additionally, the inductor ripple current is typically chosen to be between 20% and 30% of the output current to balance efficiency and performance.

Using the following values:  
- **Input voltage:** $V_{IN} = 4.5\text{ V} - 5\text{ V}$ (worst case: $V_{IN} = 5\text{ V}$)  
- **Output voltage:** $V_{OUT} = 3.3\text{ V}$
- **Switching frequency:** $f_s = 1.5\text{ MHz}$
- **Inductor ripple current:** $\Delta I_L = 0.25 \times I_{OUT}$ (we choose 0.2, this represents the worst case scenario)  

![Inductor Plot](/notes/img/plot_inductor_buck_sw.png)

From the plot above we can see that for lower currents the required minimum inductance is around **70 µH** for lighte loads. Since we are choosing standard values, we can select an inductor with a value of **68 µH**.

### Input & Output Capacitors Selection

According to the [TLV62569 datasheet](/datasheet/tlv62569.pdf), the input capacitor must have a minimum capacitance of **4.7 µF** and the output
capacitor must be in the range of **10 µF** to **47 µF**. We will use a **22 µF** capacitor for both the input and output capacitors, since we
already use the same value for the analog power supply (BOM consolidation).

### Feedback Network Resistor Selection

To archive the desired output voltage, we need to select the feedback resistors. The output voltage is given by the formula, taken from the [TLV62569 datasheet](/datasheet/tlv62569.pdf):

$$ V_{OUT} = V_{REF} \times \left(1 + \frac{R_1}{R_2}\right) = 0.6 \times \left(1 + \frac{R_1}{R_2}\right) $$

where:
- $V_{OUT}$ = Desired output voltage
- $V_{REF}$ = Reference voltage (given with 0.6 V)
- $R_1$ and $R_2$ = Feedback resistor 1 & 2

To maintain a stable output voltage, it's essential to keep the reference voltage steady. To achieve this, we will use resistors with a tolerance
of **1% or better**. The chosen values for the feedback resistors are **R1 = 100 kΩ** and **R2 = 22 kΩ**, which will provide a stable output
voltage of **3.3 V**, as shown in the schematic. The datasheet also recommend placing a **Do not place (DNP)** capacitor parallel to the first
resistor to leave room for a possible compensation capacitor.

### Power LED

At the output of the buck converter, we have a power LED that indicates when the board is powered on. For a debug LED like this, we aim for a
current of **2 mA**, which is sufficient to provide a visible indication without consuming excessive power. The required forward voltage of the
LED for the desired current is roughly **2.8 V**, see the graph below.

![LED Voltage Plot](/notes/img/led_voltage.png)

To calculate the resistor value, we use Ohm's Law:

$$ R = \frac{V_{OUT} - V_{LED}}{I} = \frac{3.3 V - 2.8 V}{2 mA} = 250 \Omega $$

For this particular case, we will use a **1 kΩ** resistor, which is a standard value that provides an even lower current of **0.5 mA**. This

## Bias Generator

We are operating the analog components with a **3.3V** supply and ground, but the OpAmps do not perform well with a single supply. To address this, we create a reference voltage at half the supply voltage. This is achieved using a voltage divider, a simple circuit that splits the input voltage into two equal parts with two identical **1 kΩ** resistors. To further enhance the stability of the voltage, a **22 µF** capacitor is added in parallel with the second resistor, effectively filtering out any noise present in the supply voltage. With this setup, the biased voltage will oscillate between **0V** and **3.3V** around the **1.65V** reference voltage.

The bias generator is configured in an OpAmp follower (buffer) arrangement, meaning that the output voltage will follow the input voltage. This behavior stems from the ideal properties of the OpAmp:

- $V_+ = V_-$ (the non-inverting and inverting inputs are equal)
- Infinite input impedance
- Zero output impedance

The OpAmp follower provides several advantages:
- Stable bias voltage
- Strong filtering of the bias voltage, reducing noise from the power supply
- High input impedance and low output impedance

Additionally, a decoupling capacitor of **100nF** is placed close to the OpAmp’s power supply pins. This capacitor acts as a local low-impedance energy reservoir, helping to prevent voltage dips during current spikes. It ensures the OpAmp operates reliably by stabilizing the power supply.
