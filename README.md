# Peak CAN usb setup

1. Install usbip on WSL (if it's not already installed):
```
sudo apt install linux-tools-generic hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*-generic/usbip 20
```

2. List all of the USB devices connected to Windows using PowerShell in administrator mode:
```
usbipd wsl list
```

3. Select the bus ID of the device you’d like to attach to WSL and run:
```
usbipd wsl attach --busid <busid>
```

4. Open Ubuntu and check if the peak can appears in the list of attached USB devices:
```
lsusb
```

5. Push some can driver:
```
modprobe can
modprobe can-dev
modprobe can-raw
```

6. Set up the can0 device with 250000 Kb/s bus rate:
```
sudo ip link set can0 up type can bitrate 250000
```

7. check if something it's going on on the can0 device we just setup:
```
candump can0
```

# Virtual CAN setup

1. Check if vcan Kernel Module is Loaded:
```
lsmod | grep vcan
```

2. If you see no output, load it manually:
```
sudo modprobe vcan
```

3. Create a vCAN interface using the ip command:
```
sudo ip link add dev vcan0 type vcan
```

4. Bring Up the vCAN Interface:
```
sudo ip link set up vcan0
```
