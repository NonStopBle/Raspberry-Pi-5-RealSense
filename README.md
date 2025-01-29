# Raspberry Pi 5 with RealSense on Ubuntu 24.04
# This document is modify from https://github.com/mathklk/realsense_raspberry_pi4 to can use with ubuntu 24.04 on raspberry pi 5  


## Hardware
  - Raspberry Pi 5 Model B 8GB
  - Intel Realsense D435

## Operating System
  - Ubuntu 24.04 

## Pre-install Requirements
* Start by updating, upgrading, and installing dependencies and tools:
```bash
sudo apt-get update && sudo apt-get dist-upgrade
sudo apt-get install automake libtool vim cmake libusb-1.0-0-dev libx11-dev xorg-dev libglu1-mesa-dev
```

```bash
sudo apt-get install clang
sudo apt-get install libtbb-dev
```

```
* Create a new `udev` rule:
```bash
cd ~
git clone https://github.com/IntelRealSense/librealsense.git
cd librealsense
sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/
```
* Apply the change:
```bash
sudo udevadm control --reload-rules && udevadm trigger
```
* Modify the library path by adding the following line to the `.bashrc` file:
```bash
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```
Apply the change:
```bash
source ~/.bashrc
```

## Installation
### Install `protobuf`:
```bash
cd ~
git clone --depth=1 -b v3.10.0 https://github.com/google/protobuf.git
cd protobuf
./autogen.sh
./configure
make -j$(nproc)
sudo make install
cd python
export LD_LIBRARY_PATH=../src/.libs
python3 setup.py build --cpp_implementation
python3 setup.py test --cpp_implementation
sudo python3 setup.py install --cpp_implementation
export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=cpp
export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION_VERSION=3
sudo ldconfig
protoc --version
```

## Change default C/C++ compiler to clang ([#9962](https://github.com/IntelRealSense/librealsense/issues/9962))
```
export CC=/usr/bin/clang
export CXX=/usr/bin/clang++
```

### Install RealSense SDK (`librealsense`):
```bash
cd ~/librealsense
mkdir build && cd build
sudo cmake .. -DBUILD_EXAMPLES=true -DCMAKE_BUILD_TYPE=Release -DFORCE_LIBUVC=true
sudo make -j4
sudo make install
```
### Install RealSense SDK Python bindings (`pyrealsense2`):
```bash
cd ~/librealsense/build
sudo cmake .. -DBUILD_PYTHON_BINDINGS=bool:true -DPYTHON_EXECUTABLE=$(which python3)
sudo make -j4
sudo make install
```

- Change the default compiler back to gcc
```
unset CC
unset CXX
```

- Modify the Python path:
```bash
export PYTHONPATH=$PYTHONPATH:/usr/local/lib/python3.12/dist-packages:/home/$(whoami)/librealsense/build/Release
```
- Persist the change:
```bash
echo 'export PYTHONPATH=$PYTHONPATH:/usr/local/lib/python3.12/dist-packages:/home/$(whoami)/librealsense/build/Release' >> ~/.bashrc
source ~/.bashrc
```
### Install `OpenGL`:
```bash
sudo apt-get install python3-opengl
sudo -H pip3 install pyopengl
sudo -H pip3 install pyopengl_accelerate
```
### Launch `Realsense Viewer`:
* Require display and desktop user to access realsense-viewer
```bash
realsense-viewer
```

### Test Realsense Library on Python :
( Script : https://github.com/mathklk/realsense_raspberry_pi4 )
- Download the example python script from this repo
```bash
cd ~
wget https://raw.githubusercontent.com/mathklk/realsense_raspberry_pi4/master/example.py
chmod +x example.py
```
- Run the script (requires a desktop environment to show the image)

