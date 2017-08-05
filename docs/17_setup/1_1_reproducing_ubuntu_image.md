# Reproducing the image

These are the instructions to reproduce the Ubuntu image that we use.

Please note that the image is already available, so you don't need to do this manually.

However, this documentation might be useful if you would like to port the software to a different distribution.

We organize this in three steps:

- Step 1: Downloading and loading the raw ubuntu image
- Step 2: Installation of ros and other dependencies and other configurations

## Step 1: From blank to minimal setup

Resources necessaries:

- Internet connection to download the packages.
- A PC running any Linux with an SD card reader.
- Time: about 20 minutes.

Results:

-  A baseline Ubuntu Mate 16.04.2 image with updated software.

### Download and uncompress the Ubuntu Mate image

Download the image from the page https://ubuntu-mate.org/download/.

The file we are looking for is:

    filename: ubuntu-mate-16.04.2-desktop-armhf-raspberry-pi.img.xz
        size: 1.2 GB
      SHA256: dc3afcad68a5de3ba683dc30d2093a3b5b3cd6b2c16c0b5de8d50fede78f75c2

Run the command `sha256sum` to make sure you have the right version:

    laptop $ sha256sum ubuntu-mate-16.04.2-desktop-armhf-raspberry-pi.img.xz
    dc3afcad68a5de3ba683dc30d2093a3b5b3cd6b2c16c0b5de8d50fede78f75c2

If the string does not correspond exactly, your download was corrupted.
Delete the file and try again.

Then decompress using the command `xz`:

    laptop $ xz -d ubuntu-mate-16.04.2-desktop-armhf-raspberry-pi.img.xz

### Finding your device name for the SD card an unmount it

    laptop $ df -h

Inspect the output for something like `/dev/mmcblk0`, you may see `/dev/mmcblk0pX` of a couple of similar entries for each partition on the card, where `X` is the partition number.
If you don't see anything like that, take out the sd card and run the command again and see what disappeared.

Next unmount all the partitions associated with the device probably:

    laptop $ sudo umount /dev/mmcblk0p1
    laptop $ sudo umount /dev/mmcblk0p2

### Burn the image to an SD card

Then burn to disc using the command `dd`:

    laptop $ sudo dd of=DEVICE if=IMG status=progress bs=4M

where `IMG` is the `.img` file you unzipped, and `DEVICE` is the device
that represents your SD card reader (NB without partitions. ie, `/dev/mmcblk0` not `/dev/mmcblk0pX`)

### Verify that the SD card was created correctly

Remove the SD card and plug it in again in the laptop.

Ubuntu will mount two partitions, by the name of `PI_ROOT` and `PI_BOOT`.


### Installation

Boot the disk in the Raspberry PI.

I chose the following options:

    language: English
    username: ubuntu
    password: ubuntu
    hostname: duckiebot

(LP: I also chose the option to log in automatically)

Then I rebooteed.

### Update installed software

The WiFi was connected to airport network `duckietown`
with password `quackquack`.

Afterwards I upgraded all the software preinstalled with these
commands:

    duckiebot $ sudo apt update
    duckiebot $ sudo apt dist-upgrade

(Expect dist-upgrade to take quite a long time - e.g. 2hrs)

## Part 2: Dependencies and Configurations

### Raspi Config

The raspi is not sshable by default, the camera is disabled, and the I2C bus is disabled. We need to fix those things:

    duckiebot $ sudo raspi-config
     choose 3. Interfacing Options,
     P2 SSH
     change no to yes
     back
     P1 camera
     change no to yes
     back
     P3 I2C
     change no to yes
     back
     finish

things to be installed:

### ROS

first commands are copied from [http://wiki.ros.org/kinetic/Installation/Ubuntu](http://wiki.ros.org/kinetic/Installation/Ubuntu)

    duckiebot $ sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
    duckiebot $ sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
    duckiebot $ sudo apt update
    duckiebot $ sudo apt install ros-kinetic-desktop-full
    duckiebot $ sudo apt install ros-$ROS_DISTRO-{tf-conversions,cv-bridge,image-transport,camera-info-manager,theora-image-transport,joy,image-proc,compressed-image-transport,phidgets-drivers,imu-complementary-filter,imu-filter-madgwick}


### Install packages

    duckiebot $ sudo apt install -y emacs vim byobu build-essential git python python-dev ipython libblas-dev liblapack-dev libatlas-base-dev gfortran libyaml-cpp-dev python-sklearn i2c-tools python-smbus
    duckiebot $ sudo pip install scipy --upgrade

(may need to do the following but might be done already through raspi-config):

    duckiebot $ sudo usermod -a -G i2c ubuntu
    duckibot $ sudo udevadm trigger

### Optional user preferences

To automatically boot into byobu:

    duckiebot $ byobu-enable

This can be disabled with byobu-disable.

### Wireless configuration

There are the two key to edit files:

* `/etc/network/interfaces` should look like this:

```
 interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
#source-directory /etc/network/interfaces.d

auto wlan0

# The loopback network interface
auto lo
iface lo inet loopback

# Wireless network interface
allow-hotplug wlan0
iface wlan0 inet dhcp
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
```
* `/etc/wpa_supplicant/wpa_supplicant.conf` should look like this:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="duckietown"
psk="quackquack"
proto=RSN
key_mgmt=WPA-PSK
pairwise=CCMP
auth_alg=OPEN
}
network={
   key_mgmt=NONE
}
```


### SSH config

Add `.authorized_keys` in the image so that we can all do passwordless ssh.

Create the `.authorized_keys` files.

On the PI, download the official key:

    duckiebot $ cd ~
    duckiebot $ mkdir -p .ssh
    duckiebot $ chmod g-rwx,o-rwx .ssh
    duckiebot $ wget -O .ssh/authorized_keys https://www.dropbox.com/s/pxyou3qy1p8m4d0/duckietown_key1.pub?dl=1


### Swap Space

Do the following:

Create an empty file using the dd (device-to-device copy) command:

    duckiebot $ sudo dd if=/dev/zero of=/swap0 bs=1M count=512

This is for a 512 MB swap space.

Format the file for use as swap:

    duckiebot $ sudo mkswap /swap0

Add the swap file to the system configuration:

    duckiebot $ sudo vi /etc/fstab

Add `/swap0 swap swap` at the bottom

Activate the swap space:

    duckiebot $ sudo swapon -a

(Can probably do something similar through `raspi-config`.)