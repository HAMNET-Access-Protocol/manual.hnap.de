
Here is how to use the Client and Basestation application.

All applications are located in `/root/`
directory.

```
./client -g RXGAIN -t TXGAIN -f FREQ -u ULMCS -d DLMCS -c CONFIG-FILE -l LOGLEVEL
```

```
./basestation -g RXGAIN -t TXGAIN -f FREQ -c CONFIG-FILE -l LOGLEVEL
```

## Client Arguments
### Modulation and Coding scheme

The system uses QAM modulation and convolutional coding at the PHY layer.
By default, MCS0 (QPSK, 1/2rate code) is used. You can manually set other
MCS values for uplink and downlink as an argument to the client application.
Downlink MCS can be set with the `-d` or
`--dl-mcs` flag. Uplink MCS can be set with the `-u` or `--ul-mcs` flag.

The following MCS have been defined

| MCS value | modulation | conv-coding| Notes  |
|:----------|:-----------|:-----------|:-------|
|     0     |    QPSK    | k=7, r=1/2 | ~80kbps|
|     1     |    QPSK    | k=7, r=3/4 | ~120kbps|
|     2     |    QAM16   | k=7, r=1/2 | ~180kbps|
|     3     |    QAM16   | k=7, r=3/4 | ~280kbps|
|     4     |    QAM64   | k=7, r=1/2 | ~280kbps|
|     5     |    QAM64   | k=7, r=3/4 | currently unstable. sometimes not working|
|     6     |    QAM256  | k=7, r=1/2 | ~380kbps|

### Gain

By default the rxgain is automatically set during startup-phase and adapted
during runtime of the client. If you want to manually fix the gain, use the `-g` flag.
The gain can be set in the range of [0 73].
 
The application `client-calib` can be used to read the current signal level
of the application. It prints the absolute amplitudes of the I and Q path and the calculated
rssi. RXgain should be set to keep the RSSI at ~-15dB.

`./client-calib -g <rxgain>`

The txgain of the client is automatically set, assuming a symmetrical UL and DL link.
It is calculated from RX and TX gain broadcasts that the basestation sends and the
rxgain that the client calculated:

`txgain_client = bs_txgain + rxgain_client - bs_rxgain`

In the current implementation, neither the rxgain of the basestation nor the
txgain of a client are adaptive. This feature still has to be implemented.

### Carrier frequency

You can specify the carrier frequency of the downlink channel with the `-f` parameter.
The frequency is given in Hz. The uplink frequency is automatically calculated (4.8MHz shift).

### Log level
Define the log level of the application with the `-l` parameter.  
0=TRACE 1=DEBUG 2=INFO 3=WARN 4=ERR 5=NONE
### Configuration file
The OFDM system can be customized. You can define your own pilot allocation scheme, redefine the number
of data subcarriers and much more. See [Advanced config](advanced_config) for more information.


