# Open your EasyBox 904 xDSL for every provider
This is a step by step guide to open your easybox 904 xDSL for the usage with every provider (Telekom, 1&1 etc). This Guide is only supported for the 904 xDSL however it should also work with the LTE version or the EasyBox 804 etc.

# Prerequisites:
- EasyBox 904 xDSL
- computer with a Linux distribution (a Raspberry Pi is fine)
- LAN cables
- FAT32 USB stick
- working internet connection

#Step One

Everything in step one is performed on the **Linux computer** with a **terminal**.

Install all required software:
```bash
sudo aptitude install binwalk unzip gcc g++ liblzma-dev lzma-dev
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
Download `squashfs`, a utility to extract the filesystem from the firmware:
```bash
wget https://heanet.dl.sourceforge.net/project/squashfs/squashfs/squashfs4.3/squashfs4.3.tar.gz
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
Scroll to the line with "LZMA_XZ_SUPPORT = 1" and delete the `#`on the beginning to uncomment it. Then compile squashfs with:
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
Save an empty file named: "sesame.txt" on the root of your FAT32 formated USB-stick. This file will initialize the ssh access.

To check if the step one was succesful you should check this list:

+ Type `unsquashfs` a list of options should be displayed.
+ Type `binwalk` a list of option should be displayed.
+ Check if there is a file named "fullimage_AT904X-03.17.01.16.bin" in your easyboxhack directory.
+ There should be a empty file on your thumb drive named "sesame.txt". Now put it aside we will need it later.


#Step two

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
dd if=fullimage_AT904X-03.17.01.16.bin of=first.part skip=0 count=64 bs=1
dd if=fullimage_AT904X-03.17.01.16.bin of=second.part skip=64 count=22192579 bs=1
dd if=fullimage_AT904X-03.17.01.16.bin of=third.part skip=22192643 bs=1
```
The second part contains the squashfs. We need to extract the filesystem inside with:
```bash
unsquashfs second.part
```
This creates the directory `squashfs-root`. cd into it with `cd squashfs-root`. Then run `nano` on `etc/init.d/rcS` and append the following lines on the bottom:
```bash
stty -F /dev/ttyS0 115200
enable_console.sh
(cd /tmp; nohup sh -c "sleep 200; exec dropbear" & )
```
This ensure that dropbear, the ssh server, is always running.
Now to add your ssh key first untar the config directory with:
```bash
tar -xzf etc/dftconfig.tgz
```
Now open a second terminal where you run:
```bash
cat ~/.ssh/id_rsa.pub
```
and copy the outputted ssh key.

Back in the squashfs-root run and paste your key at the end:
```bash
nano config/cert/authorized_keys
```
Now compress the config, delete the old package, copy the new and delete the extracted config directory.
```bash
tar -zcvf dftconfig.tgz config
rm -rf etc/dftconfig.tgz
cp dftconfig.tgz etc/
rm -rf config
```
After that we will repack the squshfs:
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

#Step three

We're now gonna flash the new firmware onto the easybox with the help of a tftp server.
Grep your easybox and start it and immedeatly press the reset button for about 10 seconds. A red screen should appear on the display. It describes the recovery process and tells you which IP you should use and the correct filename of the new image.

Now setup the tftp server. You can find a guide for Ubuntu/Debian here: https://www.cyberciti.biz/faq/install-configure-tftp-server-ubuntu-debian-howto/.
Copy the fullimage.bin into the tftp root directory and set the TFTP_ADDRESS to 192.168.2.100.

Now connect the computer running the tftp server to the easybox. The easybox should now download and flash the firmware. Check the display and shutdown the easybox. Stick your USB thumb drive into the easybox and start it.
Now you should skip the setup process on the easybox and restart it again after setting a login password.

Run:
```bash
nano .ssh/config
```
and paste:
```bash
Host easy
	Hostname 192.168.2.1
	User root
	IdentityFile ~/.ssh/id_rsa.pub
	KexAlgorithms diffie-hellman-group1-sha1
```
When you now type `ssh easy` the computer will ask for your easybox-admin password. If the connection was successful you will see:
```bash
ssh easy
root@192.168.2.1's password:


BusyBox v1.16.2 (2015-06-12 10:30:01 CST) built-in shell (ash)
Enter 'help' for a list of built-in commands.

vr9 FW for VOX 2.0
root@easy:~#
```

Now we can add your DSL login credentials which you got from your provider.
```bash
/usr/sbin/ccfg_cli set username@wan000=YOUR_USERNAME
/usr/sbin/ccfg_cli set username@wan001=YOUR_USERNAME
/usr/sbin/ccfg_cli set password@wan000=YOUR_PASSWORD
/usr/sbin/ccfg_cli set password@wan001=YOUR_PASSWORD
/usr/sbin/ccfg_cli commitcfg
```
And then run `reboot` to restart the EasyBox. If everything gone right your EasyBoy should running with a different provider now.