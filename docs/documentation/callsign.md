## Callsign broadcasting

All amateur radio stations are required to transmit their callsign regulary.

We comply with this requirement by using the Link Layer Discovery Protocol (LLDP).

Every Adalm Pluto hosts a lldp daemon that is configured to broadcast a LLDP packet every 60 seconds. This packet
contains the Callsign of the station.

The configuration takes place in `/etc/init.d/S90transceiver.sh`. We evaluate the variable *hnap_callsign*,
which can be set via the command:
```
fw_setenv hnap_callsign <YOUR-CALLSIGN>
```

You can view the current LLDP configuration with `lldpcli show config`. All received LLDP information from other stations
can be displayer with `lldpcli show neigh`.

For more info, view the [lldpd](https://manpages.ubuntu.com/manpages/trusty/man8/lldpd.8.html) 
and [lldpdcli](https://manpages.ubuntu.com/manpages/trusty/man8/lldpcli.8.html) manual
