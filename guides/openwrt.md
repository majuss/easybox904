# Step by step guide for openWrt installation

### Prerequisities:
- Computer with LAN port
- working Internet connection (for example via mobile)
- If you don't have a Linux computer an USB-stick (at least 8 GB)

If you already have a computer with Linux and a working internet connection with it, jump to step 4.

1. Create a working Ubuntu Live-stick according to [this guide](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-windows#0) and shutdown your computer when finished.
2. Start your computer with the stick plugged in. Ubuntu should now start. Click on "try Ubuntu" never ever install Ubuntu on your machine! You will loose all your data!
3. Test you Internet connection in Ubuntu by opening a browser and go to github.com/majuss/easybox904.
4. First of all we need to install the needed software. Open a terminal and enter: `sudo add-apt-repository universe`. Then run: `sudo apt install git autoconf g++ pkg-config libz-dev gcc-multilib g++-multilib make automake gawk libtool libtool-bin bison flex libncurses5-dev imagemagick libacl1-dev libcap-dev tftp-hpa tftpd-hpa net-tools ssh bsdiff p7zip-full -y`.
6. Then clone the freetz repository: `git clone https://github.com/Freetz/freetz.git freetz-devel`.
7. Change your working directory into freetz-devel with: `cd freetz-devel` and run `make tools`. If the make command does fail some requirements are missing, just google for the error message or open an issue. The compiling should take some time.
8. After the tools were successfully build keep the terminal open and go to your browser. Visit the website: `https://ftp.avm.de/fritzbox/fritzbox-7490/deutschland/fritz.os/` and download the latest image.
9. We now extract the download image with multiple tools. First of all with `7zip`: `7z e ../Downloads/FRITZ.Box_7490.*.image -r filesystem.image` and once again: `7z e filesystem.image filesystem_core.squashfs`. Now we are using the unsquash utility from freetz: `tools/unsquashfs4-avm-be filesystem_core.squashfs -e lib/modules/dsp_vr9/`. You can now find the modem-firmware in `squashfs-root/lib/modules/dsp_vr9`. We only need to patch it with: `cd squashfs-root/lib/modules/dsp_vr9/` then: `bspatch vr9-B-dsl.bin vr9-A-dsl.bin vr9-A-dsl.bin.bsdiff` and `bspatch vr9-B-dsl.bin release-vr9-B-dsl.bin release-vr9-B-dsl.bin.bsdiff`. Congratulations, the hard part is done.
10. Now we can flash the OpenWrt starter-image to the EasyBox. Download the latest starter-image (fullimage) from [here](https://app.box.com/s/tjeobifjb8ohj90m5k2u7g1efgq8308y)
11. Also download the most recent firmware image from [here](https://app.box.com/s/hvqg535dnubt4r2ontpmtodpvt6ydf00/folder/36913951101) (last confirmed working is 2019-02-07).
12. Now pull your LAN cable from your linux machine and disable WIFI. Click the grid icon in the bottom left corner of your desktop and enter `settings` to open the system settings. Then in the network tab click the gear icon behind your `eno1` interface. Then go to IPV4 and choose manual. Enter `192.168.2.100` as IP and `255.255.255.0` as netmask, leave everything else blank. Turn the interface off and on to reload the settings.
13. Now run: `sudo cp Downloads/fullimage.img /var/lib/tftpboot` in a new Terminal window to copy the firmware image to the tftp-directory (close the old one).
14. Connect your EasyBox (yellow port) with your PC with a LAN cable. Turn the box on while pressing the reset button. Release the button after 5 seconds. You now should see the red recovery screen. When it states that rescue process is complete, turn it off and on.
15. Change the IP addres of your computer to `192.168.1.2` (see step 12 on howto) and the gateway to `192.168.1.1`. Always check in the terminal if the IP was really set successfull with: `ifconfig`. If it wasn't close and open the settings menu again and turn off the LAN and turn on again til the IP was set.
16. Now open a browser and browse to: `http://192.168.1.1` (mind the http!) click the *Browse* button and choose the `openWrt-sysupgrade.bin_smp` from your downloads directory. Click *Flash Image* and wait. Then on the new site click *Proceed*. The router will now flash the firmware image. Wait 5 minutes and then open the website `https://192.168.1.1` in a private window of your browser to avoid caching problems. He will tell you that the connection is not secures. Just hit *Advanced* -> *Add exception* -> *Confirm Security Exception*.
17. You're now on the openWrt webinterface. Just hit the *Login* button.
19. Open a terminal and run: `scp freetz-devel/squashfs-root/lib/modules/dsp_vr9/release-vr9-B-dsl.bin root@192.168.1.1:/lib/fw.bin` this copies the modem-firmware which we extracted from the Fritz!Box image to the EasyBox. Now run: `ssh root@192.168.1.1` type *yes* and hit enter. You now are connected to the EasyBox via terminal.
20. Run: `vim /etc/config/network` and scroll with the arrow keys to the section: *config interface 'wan'*. If you're not already in INSERT-mode just press `i`. Enter your ISP-login to *option username*. For Telekom customers it is: `[Anschlusskennung]#[T-Onlinenummer]#0001@t-online.de` and your password into `option password`. Also set the ipv6 option at the bottom to 0, and the `ifname` to `dsl0.7`. Then comment out the complete section *config interface 'wan6'* like so:
```
#config interface 'wan6'
#   option ifname '@wan'
#   option proto 'dhcpv6'
```
This disables IPV6 completely which is not needed in private networks. Then scroll to the bottom and paste the following switch config:
```
config switch_vlan
        option device 'switch0'
        option vlan '7'
        option ports '4t'
        option vid '7'
```
Add ths line to `config dsl 'dsl'`:
```
        option firmware '/lib/fw.bin'
```

Exit vim by pressing esc and then `:wq` and enter. Then type the command `passwd root` and enter a new password for your box which is also used in the webinterface. Now simply type `reboot` and hit enter.

21. Your easybox is now successfully setted up. You can reach it's webinterface at *https://192.168.1.1*


*Since the firmware for the EasyBox modem (lantiq xrx200) is closed source and licensed I'am not able to upload it here. We will extract the firmware from a firmware image of a Fritz!Box 7490 (If you want another firmware, check this page: https://xdarklight.github.io/lantiq-xdsl-firmware-info/)*
