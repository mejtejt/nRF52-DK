
### 6LoWPAN over Bluetooth low energy
This example from [nRF5 SDK](http://developer.nordicsemi.com/nRF5_SDK/nRF5_SDK_v15.x.x/) illustrates the use of Local network in communication with devices through IPV6.
The aim of this project is to establih connection between desktop computer and nRF52 DK over Raspberry Pi as a Linux border router.

### HW Requirements
* nRF2 Development Kit
* Raspberry Pi
* Desktop Computer

### SW Requirements
* nRF5 SDK v15.0.0
* Latest version of Segger Embedded Studio
* PuTTY

### IDE/Toolchain Support
* Segger Embedded Studio

### Installing a 6LoWPAN enabled Linux kernel and required modules
```
sudo apt-get install bluez radvd libcap-ng0
sudo dpkg -i bluez_4.99-2_armhf.deb libcap-ng0_0.6.6-2_armhf.deb radvd_1.8.5-1_armhf.deb
git clone https://github.com/raspberrypi/tools.git
export CCPREFIX=/path/to/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf-
/raspbian/kernel/linux$ make mrproper
/raspbian/kernel/linux$ ARCH=arm CROSS_COMPILE=${CCPREFIX} make bcm2709_defconfig
/raspbian/kernel/linux$ ARCH=arm CROSS_COMPILE=${CCPREFIX} INSTALL_MOD_PATH=${MODULES_TEMP} make –j5
/raspbian/kernel/linux$ CONCURRENCY_LEVEL=5 DEB_HOST_ARCH=armhf fakeroot make-kpkg --append-to-version –name_of_your_choice --revision `date +%Y%m%d%H%M%S` --ARCH=arm --cross-compile ${CCPREFIX} kernel_image kernel_headers
```
### Installing the kernel on Raspberry Pi

```
sudo dpkg –i linux-header*.deb linux-image*.deb
pi@raspberry /boot $ ls | grep vmlinuz
vmlinuz-3.18.11-rpi2-v7+
pi@raspberry /boot $ sudo sh –c ‘echo “kernel=vmlinuz-3.18.11-rpi2-v7+” >> config.txt’
```

### Router Advertisement Daemon

```
root@raspberrypi:/etc# nano radvd.conf
```
```
interface bt0
{
    AdvSendAdvert on;
    prefix 2001:db8::/64
    {
        AdvOnLink off;
        AdvAutonomous on;
        AdvRouterAddr on;
    };
};
```
Enable the module to load during boot by adding bluetooth_6lowpan and radvd to /etc/modules.

```
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.

bluetooth_6lowpan
6lowpan
radvd
```
```
echo 1 > /sys/kernel/debug/bluetooth/6lowpan_enable
sudo echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
ifconfig bt0 add 2001:db8::1/64
sudo service radvd restart
echo "connect 00:96:50:36:9f:1f 1" > /sys/kernel/debug/bluetooth/6lowpan_control
```

### MQTT Client - Publisher
The MQTT publisher example is an MQTT client that connects to the broker identified by the broker address configured in the example at compile time. If the connection succeeds, it is ready to publish the LED state information under the topic "led/state".

### MQTT broker - Raspberry Pi

```
pi@raspberrypi:~$ mosquitto -d
pi@raspberrypi:~$ mosquitto_sub -v -t 'led/state'
```

