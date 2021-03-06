In order to run a HNAP system you will need:

* At least 2 Adalm Plutos
* UHF antennas. Do not use the original 2.4GHz antennas that ship with the Pluto, they do not work.
* A TCXO with better accuracy than the original (25ppm). We strongly suggest that you switch the original TCXO, no stable operation
can be guaranteed with it.

### Antenna selection
The Pluto does not come with any analog filters after/before the mixing stage. We therefore produce strong intermodulation products (e.g. output with -10dBc @1.31GHz). These products will be mixed down to the baseband at the receiver and interfere with our signal. Use UHF antennas in order to suppress the intermodulation products. DO NOT use the original 2.4GHz antennas! They just recept the intermodulation products, but not the original signal at 440MHz.

### TCXO modification
The original TCXO in the Adalm Pluto has an accuracy of 25ppm. At 440MHz carrier frequency, this could result in frequency
offsets of up to 10kHz. Only up to +-2kHz can be detected by our system. Additionally, the frequency drifts over time introduced by temperature changes
can be too severe to be compensated.

We therefore strongly suggest to switch to a more accurate TCXO. We switched to a 0.5ppm TCXO.

### Amplifier selection

If you run the system for the first time, its best to start without any amplifier. If you want to try one out, here are some basic requirements:

The system is OFDM based, so a linear amplifier is required. The clients in the system operate in half-duplex
mode, with a short guard time between the slots for switching the amplifier.
The maximum allowed TX-RX switch time of the amplifier is 265us.
Currently a PTT signal is generated at the MIO0 Pin of the Adalm Pluto which can be used to drive the amplifier. The pin has
a voltage level of 1.8V.

We could not yet perform tests including amplifiers, we will update as soon as we have first results.

