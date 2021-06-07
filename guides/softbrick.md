# Revert to original Firmware and get rid of various problems

## Prerequisites:
- Computer with a LAN Port running Linux ([e.g. Ubuntu Live](https://tutorials.ubuntu.com/tutorial/try-ubuntu-before-you-install#0)) or a Macintosh with macOS, choose the appropriate guide
- LAN cable
- working internet connection (e.g. via mobile)

### Linux guide

1. Download the original Vodafone firmware with: 
    ```
    wget https://www.vodafone.de/downloadarea/fullimage_AT904X-04.13.zip
    ```
2. Run `sudo apt install tftp-hpa tftpd-hpa` to install the TFTP-Server.
3. Now pull your LAN cable from your linux machine and disable WIFI. Click the grid icon in the bottom left corner of your desktop and enter `settings` to open the system settings. Then in the network tab click the gear icon behind your `eno1` interface. Then go to IPV4 and choose manual. Enter `192.168.2.100` as IP and `255.255.255.0` as netmask, leave everything else blank. Turn the interface off and on to reload the settings.
4. Now run: `sudo cp fullimage_AT904X-04.13.zip /var/lib/tftpboot/fullimage.img` to copy the firmware image to the tftp-directory.
5. Connect your EasyBox (yellow port) with your PC with a LAN cable. Turn the box on while pressing the reset button. Release the button after 5 seconds. You now should see the red recovery screen. When it states that rescue process is complete, turn it off and on.

---

### macOS guide

1. Download the original Vodafone firmware with: 
    ```
    curl https://www.vodafone.de/downloadarea/fullimage_AT904X-04.13.zip --output fullimage_AT904X-04.13.zip
    ```
2. Now pull your LAN cable from your machine and disable WIFI. Open the System Preferences and got to Network. Choose your LAN-Connector and switch the IP setting from DHCP to manual. Enter `192.168.2.100` as IP and `255.255.255.0` as netmask, leave everything else blank. And hit apply.
3. Now run: `sudo cp fullimage_AT904X-04.13.zip /private/tftpboot/fullimage.img` to copy the firmware image to the tftp-directory.
4. Connect your EasyBox (yellow port) with your Mac with a LAN cable. Turn the box on while pressing the reset button. Release the button after 5 seconds. You now should see the red recovery screen.
5. Start the TFTP daemon: `sudo launchctl load -F /System/Library/LaunchDaemons/tftp.plist`. When the box states that the rescue process is complete, turn it off and on.

From here you can choose another guide to install the [opened Vodafone Firmware](https://github.com/majuss/easybox904/blob/master/guides/simple.md) or [openWrt](https://github.com/majuss/easybox904/blob/master/guides/openwrt.md).
