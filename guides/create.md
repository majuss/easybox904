# Manual image creation

## Prerequisites:
- EasyBox 904 xDSL
- computer (with a Linux distribution, a Raspberry Pi is fine or a Live Linux on a thumb drive: https://www.ubuntu.com/download/desktop/create-a-usb-stick-on-windows)
- LAN cables
- working internet connection

## Step One

Everything in step one is performed on the **Linux computer** with a **terminal**.

Install all required software:
```bash
sudo apt install binwalk unzip gcc g++ liblzma-dev lzma-dev build-essential libtool automake
```

Create a working directory with:
```bash
mkdir easyboxhack
cd easyboxhack
```
Download the **original firmware image** from vodafone:
```bash
wget https://www.vodafone.de/downloadarea/fullimage_AT904X-04.13.zip
```

> You can use an old firmware if the above results in trouble: `wget https://www.vodafone.de/downloadarea/fullimage_AT904X-03.17.01.17.zip`

Ignore the .zip suffix, it's not zipped.

Download `squashfs`, a utility to extract the filesystem from the firmware (or manual from the website: https://sourceforge.net/projects/squashfs/):
```bash
wget https://sourceforge.net/projects/squashfs/files/squashfs/squashfs4.5/squashfs4.5.tar.gz
```
Extract the tar with:
```bash
tar -xzf squashfs4.5.tar.gz
```
Alter the Makefile of squashfs to include LZMA support:
```bash
cd squashfs4.5/squashfs-tools
sed -i 's/#LZMA_XZ_SUPPORT/LZMA_XZ_SUPPORT/g' Makefile
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
+ Check if there is a file named "fullimage_AT904X-04.13.zip" in your easyboxhack directory.


## Step two

After setting up everything we will analyse the firmware image and extract the filesystem.

Inside of your `easyboxhack` directory type:
```bash
binwalk fullimage_AT904X-04.13.zip
```
The output should look like this:
```bash
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             uImage header, header size: 64 bytes, header CRC: 0x51AF5503, created: 2018-03-09 07:01:22, image size: 23265280 bytes, Data Address: 0x0, Entry Point: 0x0, data CRC: 0x552EDC4F, OS: Linux, CPU: MIPS, image type: Filesystem Image, compression type: lzma, image name: "LTQCPE RootFS"
64            0x40            Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 23262920 bytes, 2892 inodes, blocksize: 131072 bytes, created: 2018-03-09 07:01:19
23265795      0x1630203       PEM certificate
23271488      0x1631840       uImage header, header size: 64 bytes, header CRC: 0x133D2257, created: 2018-03-09 07:00:58, image size: 1572800 bytes, Data Address: 0x80002000, Entry Point: 0x800061B0, data CRC: 0xC0DDC2D8, OS: Linux, CPU: MIPS, image type: OS Kernel Image, compression type: lzma, image name: "MIPS LTQCPE Linux-2.6.32.32"
23271552      0x1631880       LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 4617284 bytes
24844803      0x17B1A03       PEM certificate
```
You can see that the squashfs ranges from bit 64 to 23265795. This is what we need to extract.

To archive it type:
```bash
dd if=fullimage_AT904X-04.13.zip of=first.part count=64 bs=1
dd if=fullimage_AT904X-04.13.zip of=second.part skip=64 count=23265731 bs=1
dd if=fullimage_AT904X-04.13.zip of=third.part skip=23265795 bs=1
```
The second part contains the squashfs. We need to extract the filesystem inside with:
```bash
unsquashfs second.part
```
This creates the directory `squashfs-root`. cd into it with `cd squashfs-root`. 

Then delete the dropbear init-service with: `rm etc/init.d/dropbear`. And run `nano` on `etc/init.d/dropbear` now copy everything from: this [dropbear-file](https://raw.githubusercontent.com/majuss/easybox904/master/resources/dropbear.txt) and paste it into nano. ctrl + x to save and exit and continue with the following commands.
```bash
cd ..
mksquashfs  squashfs-root  second.part.new  -comp lzma  -b 131072  -no-xattrs  -all-root
```
The new second.part (squashfs) will be smaller than the old one. We need to add a padding with the size of the difference between the new and the old second.part.
```bash
ls -la sec*
```
Now read the size in the third column. The old second.part should be 23265731 bits and the new something around 23142400. Now calculate the difference. In this case 123331 bit. Now create the padding with this size:

```bash
dd bs=1 count=123331 if=/dev/zero of=padding
```
Now we are gonna create the new binary image:
```bash
cat first.part second.part.new padding third.part > fullimage.img
```

Compare the filesize from fullimage.img and the original firmware.bin (with `ls -la`). They should have the same size.

## Step three

Follow the quick and easy [guide](https://github.com/majuss/easybox904/blob/master/guides/simple.md) to flash your own image.
