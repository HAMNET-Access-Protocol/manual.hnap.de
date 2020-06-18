In order to run a HNAP system you will need:

<ul>
<li>At least 2 Adalm Plutos</li>
<li>UHF antennas. Do not use the original 2.4GHz antennas that ship with the Pluto, they do not work.</li>
<li>A TCXO with better accuracy than the original (25ppm). We strongly suggest that you switch the original TCXO, no stable operation
can be guaranteed with it.</li>
</ul>


### Antenna selection
The Pluto does not come with any analog filters after/before the mixing stage. We therefore produce strong intermodulation products (e.g. output with -10dBc @1.31GHz). These products will be mixed down to the baseband at the receiver and interfere with our signal. Use UHF antennas in order to suppress the intermodulation products. DO NOT use the original 2.4GHz antennas! They just recept the intermodulation products, but not the original signal at 440MHz.

### TCXO modification
The original TCXO in the Adalm Pluto has an accuracy of 25ppm. At 440MHz carrier frequency, this could result in frequency
offsets of up to 10kHz. Only up to +-2kHz can be detected by our system. Additionally, the frequency drifts over time introduced by temperature changes
can be too severe to be compensated.

We therefore strongly suggest to switch to a more accurate TCXO. We switched to a 0.5ppm TCXO.
