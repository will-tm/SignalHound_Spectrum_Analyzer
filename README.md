Spike Spectrum Analysis Software
=====

This is the code in Qt which I took from https://github.com/SignalHound/BBApp .

It is the official Github page for Signal Hound's spectrum analyzer software Spike. 
See *www.signalhound.com* for more information, and for the users manual for this software.

This fork ports the application to **Linux** for the **SA44/SA124** analyzers, which the
current Linux release of Spike does not support. The BB60 backend and the Windows build
were removed. It also adds a *One Dark* program style (Edit > Program Style).

Tested on Arch Linux (Qt 5.15, SA API 3.0.11) with an SA44B: device detection and
swept spectrum. Real-time, zero-span/IQ, audio and tracking generator modes build but
have not been exercised on hardware.

Building on Linux
-----

### 1. Dependencies

Qt 5 (core, gui, widgets, opengl, printsupport), PulseAudio client library and a C++11 compiler.

```sh
# Arch
sudo pacman -S --needed base-devel qt5-base libpulse

# Debian/Ubuntu (untested)
sudo apt install build-essential qtbase5-dev libqt5opengl5-dev libpulse-dev
```

### 2. Signal Hound SA API for Linux

Download the SA44/SA124 API for Linux from Signal Hound
(SA44B/SA124B downloads page) and install `libsa_api.so.<version>` and `libftd2xx.so`
into `/usr/local/lib`. The application links against the soname `libsa_api.so.1`:

```sh
sudo cp libsa_api.so.3.0.11 libftd2xx.so /usr/local/lib/
sudo ln -sf libsa_api.so.3.0.11 /usr/local/lib/libsa_api.so.1
sudo ln -sf libsa_api.so.3.0.11 /usr/local/lib/libsa_api.so
sudo ldconfig
```

`BBApp/src/lib/sa_api.h` is the matching 3.x header.

### 3. USB access

The SA44B/SA124B enumerate as an FTDI FT2232 (`0403:6010`). The kernel `ftdi_sio` driver
must not claim it (the API talks to it through `libftd2xx`), and your user needs access to
the device. Create `/etc/udev/rules.d/50-signal-hound-sa.rules`:

```
# Signal Hound SA44B/SA124B: let libftd2xx use the device instead of ftdi_sio
ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6010", ATTR{manufacturer}=="Signal Hound", RUN+="/bin/sh -c 'for f in /sys/bus/usb/drivers/ftdi_sio/$kernel:*; do [ -e \"$f\" ] && echo $(basename $f) > /sys/bus/usb/drivers/ftdi_sio/unbind; done'"
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6010", MODE="0666"
```

Then reload the rules and re-plug the analyzer:

```sh
sudo udevadm control --reload-rules
```

### 4. Build

```sh
mkdir build-linux && cd build-linux
qmake-qt5 ../BBApp/BBApp.pro CONFIG+=release   # "qmake" on distributions where it is Qt 5
make -j$(nproc)
```

### 5. Run

```sh
QT_QPA_PLATFORM=xcb ./BBApp
```

The OpenGL views use the legacy `QGLWidget`; the application has only been tested under X11/XWayland,
hence `QT_QPA_PLATFORM=xcb` on Wayland desktops. The analyzer is opened automatically on start-up if a
single device is connected (the SA44B takes a few seconds to initialize).

Settings are stored in `~/.config/SignalHound/`, the same location used by Signal Hound's own Spike.

Original Windows notes
-----

Spike is built using the 64-bit Qt 5.2.1 Desktop OpenGL libraries. 
Spike has also been built using the 32-bit Qt 5.3.0 Desktop OpenGL libraries.
Other versions are likely to be compatible *(OpenGL only)* but untested. 
VS2012 or a later compiler is required, for the use of C++11 features. 
The original application was Windows only as it relied on the Signal Hound product APIs, which are C++ DLLs for operating Signal Hound's spectrum analyzers.
