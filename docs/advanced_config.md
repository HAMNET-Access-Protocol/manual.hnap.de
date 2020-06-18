
Both basestation and client may be configured via a config.txt file. We use library 
[libconfig](https://hyperrealm.github.io/libconfig/) and its syntax for the config file.
The configuration is split into currently three parts: PHY layer, Platform and log configuration.
The default configuration is given in config.txt in this repo.

The configuration file can be used by passing the `-c <config-file>` parameter to the applications.


### Frequency offset calibration

The default pluto TCXO has a bad accuracy and might need some initial
calibration. The client and the client calib tool perform an initial
cfo estimation and retune the transceiver to the correct carrier frequency.

However, the offset estimation only works in a range of +-2Khz, the default TCXO
 might give larger offsets. A simple method is to
recalibrate the XO in steps of 100Hz and test until the client finds sync.

The TCXO can be calibrated using the *fw_setenv* command:

`fw_setenv xo_correction <new frequency>`

The default frequency is 40Mhz, so try 39999800 39999900 etc.

Instead of tweaking the TCXO, you can also specify the frequency of the client with the
`-f` flag. The default DL carrier is located at 439.7 MHz, try frequencies nearby if your client
cannot get sync.


**NOTE:** in the first minutes after the pluto started, the frequency offset varies by somtimes
multiple Khz. Let the Pluto run and heat up for some minutes, if you cannot find a constant
offset. The offset will settle after a while.
