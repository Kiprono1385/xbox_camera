# Kinect v2 Setup on Ubuntu 20.04

This guide explains how to connect and use the **Microsoft Kinect for Xbox One (Kinect v2)** on **Ubuntu 20.04** using **libfreenect2**.

> **Important:** Kinect v2 is different from the original Kinect for Xbox 360 (Kinect v1). Kinect v2 requires **libfreenect2**, a USB 3.0 connection, and an official Kinect Adapter when connecting it to a PC.

---

## Requirements

### Hardware

* Microsoft Kinect for Xbox One (Kinect v2)
* Official Kinect Adapter for Windows/PC
* USB 3.0 port
* Ubuntu 20.04
* Compatible USB 3.0 controller

The Kinect v2 uses a proprietary connector. When connecting it to a PC, an **official Kinect Adapter** is required.

### Software

* Ubuntu 20.04
* CMake
* GCC/G++
* libusb
* libturbojpeg
* GLFW
* OpenNI2

---

## 1. Install Dependencies

Update the package list:

```bash
sudo apt update
```

Install the required dependencies:

```bash
sudo apt install build-essential cmake pkg-config \
  libusb-1.0-0-dev \
  libturbojpeg0-dev \
  libglfw3-dev \
  libopenni2-dev
```

---

## 2. Clone libfreenect2

Clone the official OpenKinect `libfreenect2` repository:

```bash
git clone https://github.com/OpenKinect/libfreenect2.git
```

Enter the repository:

```bash
cd libfreenect2
```

---

## 3. Build libfreenect2

Create a build directory:

```bash
mkdir build
cd build
```

Configure the project:

```bash
cmake .. -DENABLE_CXX11=ON
```

Build the project:

```bash
make -j$(nproc)
```

Install libfreenect2:

```bash
sudo make install
```

---

## 4. Configure USB Permissions

To allow the Kinect to be accessed without running programs as root, install the provided udev rules:

```bash
sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/
```

Reload the udev rules:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### Reconnect the Kinect

After applying the udev rules:

1. Disconnect the Kinect from USB.
2. Wait a few seconds.
3. Reconnect the Kinect.
4. Make sure the Kinect power adapter is connected.

---

## 5. Check Kinect Detection

Check whether Ubuntu detects the Kinect:

```bash
lsusb
```

The Kinect normally appears as something similar to:

```text
Microsoft Corp. Xbox NUI Sensor
```

You can also search specifically for Microsoft devices:

```bash
lsusb | grep -i microsoft
```

If the Kinect does not appear, check the USB connection, power adapter, and USB 3.0 port.

---

## 6. Test the Kinect

`libfreenect2` includes a test application called `Protonect`.

From the build directory:

```bash
cd bin
```

Run:

```bash
./Protonect
```

If the Kinect is working correctly, a window should open displaying the available Kinect streams.

Depending on the configuration, you should be able to see:

* RGB
* Infrared (IR)
* Depth

A successful `Protonect` test confirms that `libfreenect2` can communicate with the Kinect.

---

## 7. USB 3.0 Requirements

Kinect v2 is particularly sensitive to USB 3.0 compatibility.

It is strongly recommended to connect the Kinect directly to a native USB 3.0 port.

Avoid using:

* USB 2.0 ports
* USB hubs
* Low-quality USB adapters
* Long or poor-quality USB extension cables

Some USB 3.0 controllers can also cause problems with Kinect v2.

If the Kinect appears in `lsusb` but `Protonect` cannot initialize it, USB 3.0 controller compatibility should be one of the first things checked.

---

## 8. Troubleshooting

### Kinect is not detected

Run:

```bash
lsusb
```

If no Kinect device appears:

1. Check that the Kinect power adapter is connected.
2. Check the USB connection.
3. Try another USB 3.0 port.
4. Avoid USB hubs.
5. Try another USB 3.0 controller if available.

---

### Permission denied

Check whether the udev rule exists:

```bash
ls /etc/udev/rules.d/90-kinect2.rules
```

If necessary, reload the rules:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Then disconnect and reconnect the Kinect.

The goal is to use the Kinect without requiring:

```bash
sudo ./Protonect
```

---

### Protonect cannot find the Kinect

First check whether Ubuntu detects the Kinect:

```bash
lsusb
```

If the Kinect appears in the USB list, verify that it is connected to a USB 3.0 port.

Then run:

```bash
cd ~/libfreenect2/build/bin
./Protonect
```

If your `libfreenect2` repository is located somewhere else, change the path accordingly.

---

## 9. Using Kinect v2 with ROS

After confirming that the Kinect works correctly with `Protonect`, it can be integrated into a robotics or computer vision system.

For **ROS 1**, one commonly used package is `iai_kinect2`, which provides ROS interfaces for Kinect v2 through `libfreenect2`.

A typical Kinect-based robotics pipeline looks like:

```text
Kinect v2
    |
    v
libfreenect2
    |
    v
Kinect ROS Driver
    |
    +---- RGB
    |
    +---- Depth
    |
    +---- IR
    |
    +---- Point Cloud
              |
              v
        ROS Processing
              |
              +---- Computer Vision
              |
              +---- Object Detection
              |
              +---- 3D Perception
              |
              +---- SLAM
```

For robotics applications, Kinect RGB-D data can be used for:

* RGB image processing
* Depth perception
* Point cloud generation
* Object detection
* 3D mapping
* SLAM
* Robot perception

> **Note:** `iai_kinect2` is primarily a ROS 1 package. For ROS 2, a compatible ROS 2 Kinect v2 driver or bridge should be selected based on the ROS 2 distribution being used.

---

## 10. Kinect v1 vs Kinect v2

| Feature           | Kinect v1        | Kinect v2                   |
| ----------------- | ---------------- | --------------------------- |
| Original platform | Xbox 360         | Xbox One                    |
| Linux library     | libfreenect      | **libfreenect2**            |
| USB               | USB 2.0          | **USB 3.0**                 |
| Depth technology  | Structured light | Time-of-flight              |
| PC connection     | Kinect adapter   | **Official Kinect Adapter** |
| Protonect         | ❌                | ✅                           |

**Do not follow Kinect v1 installation guides when working with Kinect v2.**

---

## 11. Complete Installation

The complete installation can be performed with:

```bash
sudo apt update

sudo apt install build-essential cmake pkg-config \
  libusb-1.0-0-dev \
  libturbojpeg0-dev \
  libglfw3-dev \
  libopenni2-dev

git clone https://github.com/OpenKinect/libfreenect2.git

cd libfreenect2

mkdir build
cd build

cmake .. -DENABLE_CXX11=ON

make -j$(nproc)

sudo make install

sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/

sudo udevadm control --reload-rules
sudo udevadm trigger
```

Disconnect and reconnect the Kinect, then run:

```bash
cd ~/libfreenect2/build/bin
./Protonect
```

---

## References

* [OpenKinect/libfreenect2](https://github.com/OpenKinect/libfreenect2)
* [iai_kinect2](https://github.com/code-iai/iai_kinect2)

---

## License

This documentation is provided for educational and development purposes. Refer to the respective project repositories for their individual licenses.
