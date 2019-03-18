# Benutze deine EasyBox mit jedem Provider -- Ohne MIC!

## Voraussetzungen:
- EasyBox 904 xDSL
- Funktionierende Internetverbindung (zB. übers Handy, es werden nur ca. 30 MB Daten benötigt)
- Computer mit Windows 10, Linux oder macOS

1. Downloade die geöffnete [geöffnete Firmware](https://github.com/majuss/easybox904/raw/master/resources/fullimage_AT904X-04.13.img)
2. Starte die EasyBox and verbinde dich mit ihr über LAN/WLAN. Browse zum Webinterface: `192.168.2.1` und stelle sicher das sie erreichbar ist. Nun halte den Resetknopf 5 Sekunden lang gedrückt. Die Box startet neu und löscht alle Konfigurationen. Nun gehe wieder auf das Webinterface und wähle den letzten Button "Aktualisierung", wähle das gedownloadete image und bestätige den Vorgang.
3. Erstelle eine leere Datei auf deinem USB-Stick mit dem Namen "sesame.txt". Benutze dafür [Notepad++](https://notepad-plus-plus.org/download/v7.6.4.html) (der normale Editor funktioniert nicht!) oder unter Linux/macOS das shell-Programm `touch`. Wenn du dir bei dem Schritt nicht sicher bist, downloade die Datei von [hier](https://raw.githubusercontent.com/majuss/easybox904/master/resources/sesame.txt). Stecke den Stick in die Box und starte sie neu.
4. Jetzt verbinden wir uns mit der Box über ein Programm namens `ssh`. Wähle nun entsprechend deinem Betriebssystem:

    **macOS / Linux**

    In einem Terminal führe diesen Befehl aus:
    ```
    ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@192.168.2.1
    ```
    Das Passwort ist: `123456`. Nun bist du zur Box verbunden und kannst mit dem Konfigurieren beginnen (siehe unten).

    **Windows 10**
    
    Öffne `Einstellungen` -> `Apps` -> `Optionale features verwalten` -> `Feature hinzufügen` und installiere den openSSH Client.
    
    Danach öffne eine CMD und füge ein:
    ```
    ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@192.168.2.1
    ```
    Antworte mit `yes` und tippe das Passwort ein: `123456`. Nun bist du zur Box verbunden und kannst mit dem Konfigurieren beginnen (siehe unten).

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
6. Führe als letztes das Kommando `reboot` aus und erfreue dich an deiner Internetverbindung.
