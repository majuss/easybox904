# Guide to debrick your EasyBox (get rid of the bootloop)

1. Go on Amazon and buy a serial to [USB adapter](https://www.amazon.de/SODIAL-USB-TTL-Konverter-Modul-eingebautem-CP2102/dp/B008RF73CS/)
2. Open the plastic housing of the EB. It can be very difficult and you have to use flat metal tools. Look at this [image](http://www0.xup.in/exec/ximg.php?fid=50621562) to detect where are the plastic nibbets.
3. Boot your PC running Debian (I will support no other OS for this process) and connect the serial console according to this [picture](https://github.com/majuss/easybox904/blob/master/serial.JPG)  and the USB into the PC 
4. Shortcut the 2 pins with the label R148 next to the screen on the EB according to this [picture](https://wiki.openwrt.org/_media/media/arcadyan/eb904xdsl_pcb_r148_uart.jpg?w=500&tok=318f3d) for example by soldering two wires to the pins and simply connect them while starting, or use a tweezer etc.
5. Start screen on your Debian with the command `screen /dev/ttyUSB0 115200`
6. Now start the EB while you shortcut the pins. After it started you don't have to shortcut anymore.
7. In the screen on your Debian you should see the short UART greeting message.
8. Open another Terminal windows and download the opened u-boot image with `wget https://github.com/majuss/easybox904/blob/master/uboot_open.asc`.
9. Transfer the opened image via: `cat uboot_open.asc > /dev/ttyUSB0` in a fresh terminal window.
10. Switch to the window with screen opened. The EB will now restart automatically and you will get into the open u-boot which requires no password. Now we will reflash the closed u-boot which requires a password, this is neccesary since the opened u-boot can't flash firmware images successfull!
11. Now hit enter when it asks for a password. You will now see a prompt like: `VR9#` enter `loady`. A program will start which allows the recieving of a new u-boot via y-modem.
12. In another terminal window install the package `lrzsz`. Download the closed u-boot image: `wget https://github.com/majuss/easybox904/raw/master/uboot_closed.lq`, and transfer it with: `sb uboot_closed.lq </dev/ttyUSB0 >/dev/ttyUSB0`.
13. Switch to the screen-window, when the transferring is finished, loady will close automatically. Erase the partition of the persistent memory which contains the bootloader: `nand erase 0 d0000` and write the transferred file with: `nand write 80800000 0 40000`.
14. Turn off the box, hold the reset button, and start it. Hold the button ~10 seconds. Now prepare the tftp-server and serve the opened-firmware image (see the quick and easy guide). When flashing is finished your EB is debricked.