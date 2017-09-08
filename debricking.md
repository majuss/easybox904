# Guide to debrick your EasyBox (get rid of the bootloop)

1. Go on Amazon and buy a serial to USB adapter: https://www.amazon.de/SODIAL-USB-TTL-Konverter-Modul-eingebautem-CP2102/dp/B008RF73CS/
2. Open the plastic housing of the EB. It can be very difficult and you have to use flat metal tools. Look at this image to detect where are the plastic nibbets. : http://www0.xup.in/exec/ximg.php?fid=50621562
3. Boot your PC running Debian (I will support no other OS for this process!!!) and connect the serial console according to these pictures: INSERT PICS and the USB into the PC
4. Shortcut the PINS on the EB according to these INSERT PIC for example by soldering two wires to the pins and simply connect them while starting, or use a tweezer etc.
5. Start screen on your debian with the command `screen /dev/ttyUSB0 115200`
6. Now start the EB while you shortcut the pins. After it started you don't have to shortcut anymore.
7. In the screen on your debian you should the see UART greeting message.
8. Open another Terminal windows and download the opened u-boot image with `wget LINK`.  INSERT DL
9. Transfer the opened image via: `cat u-boot.asc > /dev/ttyUSB0` in a fresh terminal window.
10. The EB will now restart automatically and you will get into the open u-boot which requires no password. No we will reflash the closed u-boot which requires a password, this is necasary since the opened u-boot can't flash firmware images successfull!
11. 



1. First buy Serial to USB Adapter
2. Open up the easybox
3. connect serial to easybox and PC
4. start screen on PC
5. Shortcut Pin XY (RX and TX)
6. Start easybox
7. CAT the opened UBOOT to easybox
8. EB will restart with no password UBOOT.
9. Start loady on EB
10. transfer closed UBOOT with sb (`sb ubootfrommtd </dev/ttyUSB0 >/dev/ttyUSB0`) to EB
11. Flash closed UBOOT to NAND
12. Start EB in recovery mode and transfer image via tftp