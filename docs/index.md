
<h1>HNAP Manual</h1>

![HNAP Logo](https://hnap.de/assets/img/hnap_logo.png)


This documentation contains installation and configuration manuals for HNAP4PlutoSDR. It is split into two parts.
The first part will help you to get the system running and do some configuration as an end-user. The second part is
dedicated to developers who want to modify and build the whole system themselves.

## User guide
Start [here](prerequisites)!

## Developer's guide

There are 3 main things to build:

* Build the complete [firmware image](building_the_fw) that can be flashed to the pluto.
* Build the main basestation and client [applications](build_apps) that are run on the pluto without building the firmware.
   This is only useful if you already have flashed the firmware and want to develop on the client/basestation applications.
* Build a [simulation target](build_simulation) that does not require the Pluto Hardware. Useful to see how the PHY and MAC layer work when developing.

This documentation is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

![CC-BY-SA](assets/cc-by-sa.svg)
