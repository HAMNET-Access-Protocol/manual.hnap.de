## Build the simulation application

The PHY and MAC layer can be simulated over a channel, in order to test BER performance. Here is 
how to build the target on your PC.

First, install libfftw:

`apt install libfftw3-dev`

Then, compile and install libfec

```
cd libfec
./configure
make install
```

Now we have the library dependencies for liquid-dsp installed. Install liquid-dsp:

More detailed installation instruction is here: https://github.com/jgaeddert/liquid-dsp

```
cd liquid-dsp
apt install automake autoconf
./bootstrap.sh
./configure
make install
```
When running `./configure` make sure that the libfec and libfftw libraries are detected!

Finally, the simulation target can be compiled. The transceiver project uses cmake, the build target
for the simulation is called test_mac.

```
cd transceiver
mkdir cmake-build-debug
cd cmake-build-debug
cmake -DCMAKE_BUILD_TYPE=Debug ..   # or Release, as you wish
make test_mac
```

The simulation can be configured by setting variables in `runtime/test.h` and `runtime/test_mac.c`.
