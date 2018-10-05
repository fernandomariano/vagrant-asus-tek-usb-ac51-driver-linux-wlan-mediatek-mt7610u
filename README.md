# Vagrant: Asus USB AC51 (mediatek-mt7610u) - Linux driver

## About this project:

This vagrant file will compile a new Linux Kernel and an open-source Linux driver for USB AC 51 Asus available on GitHub or Bitbucket. Although this wireless device is advertised as Linux supported on its box, it will absolutely not run in modern Linux distributions. According to Asus, this wifi should work on Kernel 2.4 and 2.6, but I couldn't make it work.

There are a lot of mt7610u drivers on GitHub designed to compile in modern kernel versions (3.x and 4.x) for CentOS, Debian, Raspberry Pi. This project is intended to help you to compile these drivers without breaking your host operating system.

The virtual machine created by Vagrant will work as a NAT gateway for your physical machine (laptop or desktop) providing high-speed internet in computers where it is not possible to upgrade the internal network device (PCI express), so you must use an USB wireless device in order to use this Vagrant project.

## Documentation:

- connect your USB dongle (Asus AC51) in any available USB port on your computer
- rename and change the values in wifi-credentials.yml.example providing your SSID and passphrase
  - `mv wifi-credentials.yml.example wifi-crendentials.yml`
  - `vi wifi-credentials.yml`
- `vagrant up centos` (you need to run this command at least 3 times)
  - 1) it will build a new virtual machine, download and compile a new Linux Kernel (It can take a few hours. Be patient.)
  - 2) it will compile the Linux driver for your wireless device on the new running Kernel (it will take 5 minutes)
  - 3) it will set up wifi files and try to connect to your wifi network
- to make sure that your virtual machine is able to connect to your wifi network:
  - `vagrant ssh centos`
   - `ping 8.8.8.8 -n 4`
   - `ping google.ca -n 4`
- if both ping commands work, change your default gw (in your physical computer) to use the virtual machine as your default gateway
  - `sudo route del default gw 192.168.0.1 && sudo route add default gw 192.168.2.20`
  - replace 192.168.0.1 by your gateway address
- Now, you are good to enjoy a high-speed internet on your computer using the Asus AC51

## Tested OS and Github projects:
- CentOS 7 + likwidoxigen Linux driver + kernel 3.16.57 = very good, but it might freeze a few times after some days of use
- Ubuntu 14.04 + likwidoxigen Linux driver + kernel 3.16.57 = unstable - not recommended so far

## Product info:
- https://www.asus.com/ca-en/Networking/USBAC51/

## Thanks:
- to developers on Github to provide a way to use an unsupported Asus device on Linux

## References:
- https://www.vagrantup.com/docs/virtualbox/configuration.html
- https://sonnguyen.ws/connect-usb-from-virtual-machine-using-vagrant-and-virtualbox/
- https://wiki.centos.org/HowTos/Laptops/WpaSupplicant
- https://stackoverflow.com/questions/17154532/is-there-a-way-the-to-secure-proxy-user-passwords-for-vagrant-configs
- https://github.com/search?q=mt7610u
