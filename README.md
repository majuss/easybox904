# Open your EasyBox 904 xDSL for every provider --- Without MIC (Modem Installation Code)
This is a step by step guide to open your easybox 904 xDSL for the usage with every provider (Telekom, 1&1 etc). This Guide is only supported for the 904 xDSL (the firmware of the 804 is protected by encryption and cannot be modified this easy). The Guide will also work with almost any vectoring connection.

This Guide is only for the manual configuration of the Easybox, if you want to turn your Easybox into an IoT-device see these two links: [Quallenauges openWRT-fork](https://github.com/Quallenauge/Easybox-904-XDSL)
 and [openWRT-forum](https://forum.openwrt.org/viewtopic.php?id=44676).

# [Debricking guide is now online (remove bootloop etc.)](https://github.com/majuss/easybox904/blob/master/debricking.md)

# Fast and easy installation

## Prerequisites:
- EasyBox 904 xDSL
- Working internet connection
- Computer with Ubuntu installed or [live stick](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-windows#0) and LAN cable. If you're experienced with Windows, look at the [Linux-Subsystem](https://docs.microsoft.com/en-us/windows/wsl/install-win10)

1. Downlaod this opened firmware image: [opened Firmware](https://github.com/majuss/easybox904/raw/master/fullimage.img)
2. Start a tftp-server on your computer:
	Linux: [tftp setup](https://www.cyberciti.biz/faq/install-configure-tftp-server-ubuntu-debian-howto/)
	Mac: [tftp setup](https://rick.cogley.info/post/run-a-tftp-server-on-mac-osx/)
3. Check your current network configuration with: `ifconfig` and get the ID of your LAN-port. Disable your network manager with `sudo stop network-manager` (if installed) so it will not set the IP automatically. Disable all other Network sources, like WLAN. Set the IP of your computer to: 192.168.2.100 with `sudo ifconfig ENTER_YOUR_LAN_ID 192.168.2.100 netmask 255.255.255.0` and copy the downloaded firmware image to the transfer directory of your tftp-server
4. Plug in the power of the Easybox and connect it via a yellow port with a LAN cable with your computer which is running the tftp server
6. Hold the reset-button of the Easybox and turn it on, release it after 5 seconds. A red rescue screen should appear and the Box should automatically download the image. The screen will prompt you to restart the Box, do it.
7. Run `nano .ssh/config` in a terminal and paste this:
```
Host easy
	Hostname 192.168.2.1
	User root
	KexAlgorithms diffie-hellman-group1-sha1
```
This will create a shortcut to connect to the Easybox. 

8. Run `ssh easy` and login with the default password: `123456` (if the above `nano` command failed, run: `ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@192.168.2.1`). Change your root password with the command `passwd root`. Now enter these commands according to your internet-connection:

## ADSL:
```
ccfg_cli set username@wan000=[Anschlusskennung][T-Onlinenummer]#0001@t-online.de    //example for Telekom customers, just enter your username provided by your ISP
ccfg_cli set password@wan000=[Password]
```
## VDSL:
```
ccfg_cli set username@wan050=[Anschlusskennung][T-Onlinenummer]#0001@t-online.de
ccfg_cli set password@wan050=[Password]
ccfg_cli set vlan_id@wan050=7
```
## VOIP:
```
ccfg_cli set lineEnable@sip_acc_1=1
ccfg_cli set userId@sip_acc_1=[AreaCode][PhoneNumber]
ccfg_cli set userId_area@sip_acc_1=[AreaCode]
ccfg_cli set userId_local@sip_acc_1=[PhoneNumber]
ccfg_cli set account_name@sip_acc_1=[AreaCode][PhoneNumber]
ccfg_cli set displayName@sip_acc_1=[AreaCode][PhoneNumber]
ccfg_cli set password@sip_acc_1=
ccfg_cli set useAuthId@sip_acc_1=0
ccfg_cli set authId@sip_acc_1=
ccfg_cli set realm@sip_acc_1=tel.t-online.de
ccfg_cli set sipdomain@sip_acc_1=tel.t-online.de
ccfg_cli set registrar@sip_acc_1=tel.t-online.de
ccfg_cli set proxy@sip_acc_1=tel.t-online.de
ccfg_cli set outboundProxy@sip_acc_1=tel.t-online.de
ccfg_cli set useOutboundProxy@sip_acc_1=0
ccfg_cli set useDNSSRV@sip_acc_1=1
ccfg_cli set dtmfTxMethod@sip_acc_1=1
```
## Activation:
```
ccfg_cli set ivr_mode@bootstrap=2
ccfg_cli set arcor_pinConf@bootstrap=1
ccfg_cli set arcor_customer@bootstrap=1
ccfg_cli set keep_in_act@bootstrap=0
ccfg_cli set FirstUseDate@tr69=2015-05-24T01:07:49
```
## Save Changes:
```
ccfg_cli commitcfg
```
9. Run the command `reboot` and enjoy your internet-connection

---

# Manual installation and own image creation (optional)

## Prerequisites:
- EasyBox 904 xDSL
- computer (with a Linux distribution, a Raspberry Pi is fine or a Live Linux on a thumb drive: https://www.ubuntu.com/download/desktop/create-a-usb-stick-on-windows)
- LAN cables
- working internet connection

## Step One

Everything in step one is performed on the **Linux computer** with a **terminal**.

Install all required software:
```bash
sudo aptitude install binwalk unzip gcc g++ liblzma-dev lzma-dev build-essential libtool automake
```

Create a working directory with:
```bash
mkdir easyboxhack
cd easyboxhack
```
Download the **original firmware image** from vodafone:
```bash
wget https://www.vodafone.de/downloadarea/fullimage_AT904X-03.17.01.16.zip
```
Then extract the zip with:
```bash
unzip fullimage_AT904X-03.17.01.16.zip
```
Download `squashfs`, a utility to extract the filesystem from the firmware (or manual from the website: https://sourceforge.net/projects/squashfs/):
```bash
wget https://master.dl.sourceforge.net/project/squashfs/squashfs/squashfs4.3/squashfs4.3.tar.gz
```
Extract the tar with:
```bash
tar -xzf squashfs4.3.tar.gz
```
Alter the Makefile of squashfs to include LZMA support:
```bash
cd squashfs4.3/squashfs-tools
nano Makefile
```
Scroll to the line with "LZMA_XZ_SUPPORT = 1" and delete the `#`on the beginning to uncomment it. Exit nano with `ctrl + x` Then compile squashfs with:
```bash
make
```
When the compiler was successful install with:
```bash
sudo make install
```
If you never created a ssh-key you need to do it with (you can leave everything at default):
```bash
ssh-keygen
```

To check if the step one was succesful you should check this list:

+ Type `unsquashfs` a list of options should be displayed.
+ Type `binwalk` a list of option should be displayed.
+ Check if there is a file named "fullimage_AT904X-03.17.01.16.bin" in your easyboxhack directory.


## Step two

After setting up everything we will analyse the firmware image and extract the filesystem to add the ssh key etc.

Inside of your `easyboxhack` directory type:
```bash
binwalk fullimage_AT904X-03.17.01.16.bin
```
The output should look like this:
```bash
binwalk fullimage_AT904X-03.17.01.16.bin

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             uImage header, header size: 64 bytes, header CRC: 0xD655816D, created: Fri Jun 12 03:17:21 2015, image size: 22192128 bytes, Data Address: 0x0, Entry Point: 0x0, data CRC: 0x554256C3, OS: Linux, CPU: MIPS, image type: Filesystem Image, compression type: lzma, image name: "LTQCPE RootFS"
64            0x40            Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 22189643 bytes,  2822 inodes, blocksize: 131072 bytes, created: Fri Jun 12 03:17:18 2015
22192643      0x152A203       PEM certificate
22198336      0x152B840       uImage header, header size: 64 bytes, header CRC: 0xDE8F55A1, created: Fri Jun 12 03:16:59 2015, image size: 1572800 bytes, Data Address: 0x80002000, Entry Point: 0x800061B0, data CRC: 0xD3A38B77, OS: Linux, CPU: MIPS, image type: OS Kernel Image, compression type: lzma, image name: "MIPS LTQCPE Linux-2.6.32.32"
22198400      0x152B880       LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 4608900 bytes
23771651      0x16ABA03       PEM certificate
```
You can see that the squashfs ranges from bit 64 to 22192643. This is what we need to extract.

To archive it type:
```bash
dd if=fullimage_AT904X-03.17.01.16.bin of=first.part count=64 bs=1
dd if=fullimage_AT904X-03.17.01.16.bin of=second.part skip=64 count=22192579 bs=1
dd if=fullimage_AT904X-03.17.01.16.bin of=third.part skip=22192643 bs=1
```
The second part contains the squashfs. We need to extract the filesystem inside with:
```bash
unsquashfs second.part
```
This creates the directory `squashfs-root`. cd into it with `cd squashfs-root`. 

Then delete the dropbear init-service with: `rm etc/init.d/dropbear`. And run `nano` on `etc/init.d/dropbear` now copy everything from: this [dropbear-file](https://raw.githubusercontent.com/majuss/easybox904/master/dropbear.txt) and paste it into nano. ctrl + x to save and exit and continue with the following commands.
```bash
cd ..
mksquashfs  squashfs-root  second.part.new  -comp lzma  -b 131072  -no-xattrs  -all-root
```
The new second.part (squashfs) will be smaller than the old one. We need to add a padding with the size of the difference between the new and the old second.part.
```bash
ls -la sec*
```
Now read the size in the third column. The old second.part should be 22192579 bits and the new something around 22052864. Now calculate the difference. In this case 139715 bit. Now create the padding with this size:

```bash
dd bs=1 count=139715 if=/dev/zero of=padding
```
Now we are gonna create the new binary image:
```bash
cat first.part second.part.new padding third.part > fullimage.img
```

Compare the filesize from fullimage.img and the original firmware.bin (with `ls -la`). They should have the same size.

## Step three

We're now gonna flash the new firmware onto the easybox with the help of a tftp server.
Grep your easybox and start it and immedeatly press the reset button for about 10 seconds. A red screen should appear on the display. It describes the recovery process and tells you which IP you should use and the correct filename of the new image.

Now setup the tftp server. You can find a guide for Ubuntu/Debian here: https://www.cyberciti.biz/faq/install-configure-tftp-server-ubuntu-debian-howto/.
Copy the fullimage.bin into the tftp root directory and set the TFTP_ADDRESS to 192.168.2.100.

Now connect the computer running the tftp server to the easybox. The easybox should now download and flash the firmware. Check the display and shutdown the easybox. Now start it again and run the following commands in your terminal:

Run:
```bash
nano .ssh/config
```
and paste:
```bash
Host easy
	Hostname 192.168.2.1
	User root
	KexAlgorithms diffie-hellman-group1-sha1
```
When you now type `ssh easy` the computer will ask for your easybox-admin password, it is by default `123456`. If the connection was successful you will see:
```
ssh easy
root@192.168.2.1's password:


BusyBox v1.16.2 (2015-06-12 10:30:01 CST) built-in shell (ash)
Enter 'help' for a list of built-in commands.

vr9 FW for VOX 2.0
root@easy:~#
```

Now you should change the password with: `passwd root`. After that we can add your DSL login credentials which you got from your provider.

See in the fast and easy guide.

## Tips

Get the complete config:
```
ccfg_cli showcfg
```
Read a specific configuration entry:
```
ccfg_cli get xxx
```

// Huge portions of this guide were taken from the openwrt forum. Please thank these guys for the affords!
