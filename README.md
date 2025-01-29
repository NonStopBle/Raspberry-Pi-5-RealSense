# Raspberry Pi 5 with RealSense on Ubuntu 22.04
# This wiki is modify from https://github.com/datasith/Ai_Demos_RPi/wiki/Raspberry-Pi-4-and-Intel-RealSense-D435 to can use with ubuntu 22.04 in raspberry pi 5  

## Pre-install Requirements
* Start by updating, upgrading, and installing dependencies and tools:
```bash
sudo apt-get update && sudo apt-get dist-upgrade
sudo apt-get install automake libtool vim cmake libusb-1.0-0-dev libx11-dev xorg-dev libglu1-mesa-dev
```
* Increase swap to 2GB by modifying the following file:
```bash
sudo vi /etc/dphys-swapfile
```
Change the line to:
```bash
CONF_SWAPSIZE=2048
```
Apply the change:
```bash
sudo /etc/init.d/dphys-swapfile restart
swapon -s
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
### Install `libtbb-dev`:
```bash
sudo apt-get install libtbb-dev
```
### Install RealSense SDK (`librealsense`):
```bash
cd ~/librealsense
mkdir build && cd build
cmake .. -DBUILD_EXAMPLES=true -DCMAKE_BUILD_TYPE=Release -DFORCE_LIBUVC=true
make -j$(nproc)
sudo make install
```
### Install RealSense SDK Python bindings (`pyrealsense2`):
```bash
cd ~/librealsense/build
cmake .. -DBUILD_PYTHON_BINDINGS=bool:true -DPYTHON_EXECUTABLE=$(which python3)
make -j$(nproc)
sudo make install
```
Modify the Python path:
```bash
export PYTHONPATH=$PYTHONPATH:/usr/local/lib/python3.12/dist-packages:/home/net/librealsense/build/Release
```
Persist the change:
```bash
echo 'export PYTHONPATH=$PYTHONPATH:/usr/local/lib/python3.12/dist-packages:/home/net/librealsense/build/Release' >> ~/.bashrc
source ~/.bashrc
```
### Install `OpenGL`:
```bash
sudo apt-get install python3-opengl
sudo -H pip3 install pyopengl
sudo -H pip3 install pyopengl_accelerate
```

## Remote Operation
* Enable `VNC`:
```bash
sudo apt install realvnc-vnc-server realvnc-vnc-viewer
```
Enable VNC server:
```bash
sudo systemctl enable vncserver-x11-serviced
sudo systemctl start vncserver-x11-serviced
```

Connect to the Raspberry Pi 5 using a VNC client such as [VNC Viewer by RealVNC](https://www.realvnc.com/en/connect/download/viewer/).

## Debugging
### Error: "Could not initialize offscreen context!"
If you encounter this error:
```bash
realsense-viewer
Could not initialize offscreen context!
```
Ensure you are running VNC correctly:
```bash
glxinfo | grep "OpenGL renderer"
```
Expected output:
```
OpenGL renderer string: V3D 4.2
```
If it shows `llvmpipe`, reconfigure VNC:
```bash
sudo raspi-config
# Navigate to Advanced Options -> GL Driver -> G2 GL (Fake KMS)
```

### Error: "GLFW Driver Error: GLX: GLX version 1.3 is required"
Ensure OpenGL is correctly installed and configured:
```bash
sudo apt-get install --reinstall xserver-xorg-core
sudo dpkg-reconfigure xserver-xorg
```

## Running RealSense Viewer
Start the viewer with:
```bash
realsense-viewer
```

For further debugging, refer to [Intel RealSense Issues](https://github.com/IntelRealSense/librealsense/issues).

