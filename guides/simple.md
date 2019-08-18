# Quick and easy installation

## Prerequisites:
- EasyBox 904 xDSL
- Working internet connection (for example from a mobile)
- Computer with Windows 10, Linux or macOS
- FAT 32 formatted USB-Stick

1. Download this opened firmware image: [opened Firmware](https://github.com/majuss/easybox904/raw/master/resources/fullimage_AT904X-04.13.img)
2. Start the EasyBox normally and connect via LAN or WLAN to it. Browse to the webinterface at: `192.168.2.1` and confirm it's responding. Now press the reset button on the back for at least 5 seconds. The Box will reset completely and restart. Once again browse to the webinterface.
3. On the interface you are presented with the buttons, choose the last "Firmware Update" and select the downloaded fullimage file. Press install then confirm to flash.
4. Create an empty file on your USB-Stick with [Notepad++](https://notepad-plus-plus.org/download/v7.6.4.html) (normal notepad will not work!) or Linux `touch` called sesame.txt. If your not sure if it was correctly created, download it from [here](https://raw.githubusercontent.com/majuss/easybox904/master/resources/sesame.txt). Now put the stick into the box and restart it.
5. We now have to connect to the box via a program called `ssh` choose the appropriate point according to your operating system:

    **macOS / Linux**

    In a terminal run this command:
    ```
    ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@192.168.2.1
    ```
    Type the password `123456`. You're now connected and can set the box up (see below).

    **Windows 10**
    
    Open `Settings` -> `Apps` -> `Manage optional features` -> `Add Feature` and then click install on `openSSH Client`. Open a CMD and enter:
    ```
    ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@192.168.2.1
    ```
    Answer with `yes` and type the password: `123456`. You're now connected and can set the box up (see below).

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
6. Run the command `reboot` and enjoy your internet-connection
