Once you have installed the application and performed the initial system configuration, you can test the system.
Your basestation should start automatically once it is connected to power. In order to start you client log
into the system via SSH and do:
```
./client
```

The client should start up and connect to the basestation. Once this is done,
open another SSH connection and test the HNAP connection with *ping* or *iperf*.

Every Pluto starts an iperf3 server by default, so simply run:
```
iperf3 -c 192.168.123.1 -R
```

## Troubleshooting

If the client is not able to connect to the basestation, the following might help:

**Make sure the second CPU core is active**

Verify it with `fw_printenv maxcpus` and set up according to [here](../installation#cpu-setup).
Without the second core, no realtime operation is possible.

**Use correct antennas for the used band**

The Pluto does not come with any analog filters after/before the mixing stage.
We therefore produce strong intermodulation products (e.g. output with -10dBc @1.31GHz).
These products will be mixed down to the baseband at the receiver and interfere with our signal. 
Use UHF antennas in order to suppress the intermodulation products. DO NOT use the original 2.4GHz antennas! They just
recept the intermodulation products, but not the original signal at 440MHz.

**Adjust the carrier frequency**

The default TCXO on the Adalm Pluto has an accuracy of 25ppm. At 440MHz, this can result
in a frequency offset of up to 10kHz. The application can only detect and compensate +-2kHz, we therefore
strongly suggest to switch to a more accurate TCXO (more details here TODO!).

If you do not want to swap the TCXO immediately, try the following:
1. Let the Pluto heat up a bit. The frequency drift is very high in the first minutes of operation. Wait some minutes for the offset to settle.
2. Adjust the frequency offset manually. Run `./client -f 439702000` to tune to higher and lower carrier frequencies. Do this in 2kHz steps 
around the default frequency of 439700000. There is a calibration tool *client-calib* that only syncs to the Downlink and 
estimates the carrier offset that might help: `./client-calib -f <DL-FREQ>`

## What's next?

Get higher data-rates by changing the modulation coding scheme: [here](system_config)

Read how to build the firmware yourself [here](https://manual.hnap.de/building_the_fw/)
