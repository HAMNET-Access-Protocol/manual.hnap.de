
In order to install the system, you just have to flash a precompiled firmware.

## Get the firmware
Download the HNAP4PlutoSDR firmware from [here](https://github.com/HAMNET-Access-Protocol/HNAP4PlutoSDR/releases).
Connect one Adalm Pluto to your PC and copy the 
image to the Pluto's mass storage device. Eject the Pluto and wait until
the flashing process is completed (LED1 stops blinking).

## Initial system configuration
Now, connect to the Pluto via SSH to initially configure the system. This will set some variables in the
permanent memory of the Pluto, so you only have to do this once. The variables will even persist if you flash the
firmware again.

At first, we have to activate the second core of the ARM CPU. We also
isolate core #1 so that no applications will be executed there. The core is dedicated
for buffer transfers and some signal processing in our application:
```
fw_setenv maxcpus 2 isolcpus=1
```

Next, specify which IP address this Pluto will use within the HNAP Network. You can define any IP address as long 
as the IP addresses you assign to other Plutos are within one subnet:
```
fw_setenv hnap_tap_ip 192.168.123.1
```

If this Pluto will act as a basestation, set this variable
to enable autostart of the basesation. If it is going to be a client, just skip this.
```
fw_setenv hnap_bs_autostart 1
```

Set this station's callsign. It will be broadcasted via LLDP:
```
fw_setenv hnap_callsign <YOUR-CALLSIGN>
```
That's it. Now reboot the Adalm Pluto. Then do the same steps for your second Pluto.

If you you want to connect both Plutos to the same Host PC for testing, make sure that they use
different IP addresses on the Ethernet via USB interface.
See [here](https://wiki.analog.com/university/tools/pluto/users/customizing)
for more info. We suggest that you configure your first Pluto with the
IP addresses 192.168.2.1 (pluto) 192.168.2.10 (host PC) and the second with 
IP address 192.168.**3**.1 (pluto) and 192.168.**3**.10 (host PC)

Now you are ready to run a [first test](usage) of the system!
