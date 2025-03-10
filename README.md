# android-forensics

# Pre-req

Get some [Markdown knowledge](https://learn.microsoft.com/en-us/contribute/content/markdown-reference)

## SetUp Hyper-V and WSL2 on Windows

If you need to Setup Hyper-V + WSL2 with Ubuntu 24.04 on Windows 11, you can read [https://github.com/ezYakaEagle442/sec102?tab=readme-ov-file#pre-req](https://github.com/ezYakaEagle442/sec102?tab=readme-ov-file#pre-req)

## Install Docker

Read :
- [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
- [https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script) 

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# check docker version
docker -v

sudo groupadd docker
sudo usermod -aG docker $USER
```

## Install Android Debug Bridge

Read :
- [https://developer.android.com/tools/adb](https://developer.android.com/tools/adb)
- []()
- []()
- [https://www.debugpoint.com/how-to-access-android-devices-internal-storage-and-sd-card-in-ubuntu-linux-mint-using-media-transfer-protocol-mtp/](https://www.debugpoint.com/how-to-access-android-devices-internal-storage-and-sd-card-in-ubuntu-linux-mint-using-media-transfer-protocol-mtp/)
- [https://www.frandroid.com/android/rom-custom-2/403222_comment-telecharger-les-outils-adb-et-fastboot-sur-windows-macos-et-linux](https://www.frandroid.com/android/rom-custom-2/403222_comment-telecharger-les-outils-adb-et-fastboot-sur-windows-macos-et-linux)

```bash
lsb_release -a
hostnamectl

sudo apt update && apt upgrade -y

sudo apt-get install android-tools-adb --yes
sudo apt-get install android-tools-fastboot --yes

# https://packages.ubuntu.com/search?suite=noble&section=all&arch=any&keywords=libmtp&searchon=names
sudo apt-get install go-mtpfs --yes
sudo apt-get install libmtp-common libmtp-dev libmtp-doc libmtp-runtime libmtp9t64 --yes
sudo apt-get install jmtpfs mtp-tools --yes

mtp-detect
sudo jmtpfs -l
adb devices

ls -al /etc/udev/rules.d/
```

At this stage, if you do not see your Android device on WSL2, check it is correctly mount on Windows.
You can check running Windows Devcie Manager / you should see your phone under "Mobile devices"

```bash
USB_IPD_WIN_VERSION=4.4.0
USB_IPD_WIN_URL="https://github.com/dorssel/usbipd-win/releases/download/v$USB_IPD_WIN_VERSION/usbipd-win_$USB_IPD_WIN_VERSION.msi"
echo "Download $USB_IPD_WIN_URL and run it"

sudo apt-get install linux-tools-common --yes

# This doesn't work on Windows Subsystem for Linux because the kernel version name is non-standard (e.g. 5.15.167.4-microsoft-standard-WSL2+).
# sudo apt install linux-tools-$(uname -r) hwdata

# sudo apt-get install linux-azure-tools-6.8.0-1007 --yes
# sudo apt-get install linux-oem-6.8-tools-6.8.0-1005 --yes
# sudo apt-get install linux-tools-6.8.0-31 --yes
# sudo apt-get install linux-tools-6.8.0-1007-azure --yes
# sudo apt-get install linux-tools-6.8.0-31-generic --yes

sudo apt install linux-tools-generic hwdata --yes
# sudo apt-get install usbipd --yes
```

Run PowserShell as admin
```bash
usbipd list
$BUS_ID="7-2" # example in my case, take the right one for you
usbipd bind --busid $BUS_ID
usbipd attach --wsl --busid $BUS_ID
```

```console
Connected:
BUSID  VID:PID    DEVICE                                                        STATE
1-3    0489:e0f6  MediaTek Bluetooth Adapter                                    Not shared
1-4    0b05:19b6  Périphérique d’entrée USB                                     Not shared
1-5    0b05:193b  Périphérique d’entrée USB                                     Not shared
4-1    047f:02e9  Périphérique d’entrée USB                                     Not shared
5-1    3277:0051  USB2.0 FHD UVC WebCam, USB2.0 IR UVC WebCam, Camera DFU D...  Not shared
7-1    046d:c077  Périphérique d’entrée USB                                     Not shared
7-2    2a70:f003  ONEPLUS A6013                                                 Not shared
```

Now you can go to your wsl session and use the mounted device, mine was located at /mnt/d


Plug in your Android device using a USB cable connected to your laptop.
On your Android device, swipe down from above on the home screen and click Touch for more options.
In the following menu, select the option “Transfer File (MTP)“.

<span style="color:red">**/!\ IMPORTANT WARNING: **</span>
go to Parameters / System / Developpers options :

- enable Developpers options
- USB debugging MUST BE switched OFF

```bash
ls -al /mnt/d

mtp-detect
sudo jmtpfs -l
```

![mtp-detect](img/mtp-detect.png)

![jmtpfs](img/jmtpfs.png)

```bash

# https://www.debugpoint.com/how-to-access-android-devices-internal-storage-and-sd-card-in-ubuntu-linux-mint-using-media-transfer-protocol-mtp/

sudo touch /etc/udev/rules.d/51-android.rules
sudo chmod g+w /etc/udev/rules.d/51-android.rules
sudo chmod a+r /etc/udev/rules.d/51-android.rules

ls -al /etc/udev/rules.d/
sudo vim /etc/udev/rules.d/51-android.rules
```


Type the below line using your device’s VID and PID in the 51-android.rules file (which you note down in previous step). 

```bash
'SUBSYSTEM=="usb", ATTR{idVendor}=="2a70", ATTR{idProduct}=="f003", MODE="0666"' >> /etc/udev/rules.d/51-android.rules
```

```bash
sudo udevadm control --reload-rules
sudo service udev restart

sudo adb kill-server
sudo adb start-server

adb tcpip 5555
```

to find your phone ip:
    Open your phone wifi-settings
    Click on Advanced
    You will see your ip. It will be something like: 192.168.x.y


Open the Settings app, and select System & updates -> Developer Options.
Toggle Wireless debugging to on.
Click Allow on the pop-up menu to activate debugging on the connected network.
Open the Wireless Debugging option via Settings -> System & updates -> Wireless debugging, and tap Pair device with pairing code

```bash
# adb get-state
adb pair 192.168.x.y:37109
adb connect 192.168.x.y:5555

sudo adb kill-server
sudo udevadm control --reload-rules
sudo service udev restart
sudo adb start-server

adb devices -l
adb version
adb get-state
adb get-serialno
# adb pull
# adb root
# adb usb

adb shell; su;
```


# Manage SMS

- [https://stackoverflow.com/questions/43325303/reading-sms-from-android-device-using-adb-shell-commands](https://stackoverflow.com/questions/43325303/reading-sms-from-android-device-using-adb-shell-commands)
- [https://android.stackexchange.com/questions/114437/backup-restore-sms-mms-via-adb-on-a-non-rooted-device](https://android.stackexchange.com/questions/114437/backup-restore-sms-mms-via-adb-on-a-non-rooted-device)
- []()


```bash
adb shell bu help
```
```console

```


# Backup your photos / videos

If you have a Synology NAS server, you can install the Synology Drive mobile App from the Play Store to Backup your photos / videos from your mobile, read :
- [ https://play.google.com/store/apps/details?id=com.synology.dsdrive&hl=fr&pli=1]( https://play.google.com/store/apps/details?id=com.synology.dsdrive&hl=fr&pli=1)

Other options: you can use a Dual Drive USB Key which can be plugged either to your mobile phone, or to your laptop:
[https://www.fnac.com/SearchResult/ResultList.aspx?Search=cl%C3%A9+USB+SanDisk+dualdrive&sft=1&sa=0](https://www.fnac.com/SearchResult/ResultList.aspx?Search=cl%C3%A9+USB+SanDisk+dualdrive&sft=1&sa=0)

```bash

```
```console

```

# Erase your data

If you need to purge your phone, you can find this [App from the Play Store](https://play.google.com/store/apps/details?id=com.projectstar.ishredder.android.standard), however be aware that such kinf of App can collect some data (ex: your name and email address)

# Install /e/OS on a OnePlus 6T - “fajita”

Read :
- [Check if your device is supported by /e/OS](https://doc.e.foundation/devices)
- [https://doc.e.foundation/devices/fajita/install](https://doc.e.foundation/devices/fajita/install)
- [https://images.ecloud.global/community/fajita/](https://images.ecloud.global/community/fajita/)
- [https://doc.e.foundation/devices/fajita](https://doc.e.foundation/devices/fajita)
- [If you have a FairPhone, Get you BootLoader unlocking code](https://www.fairphone.com/en/bootloader-unlocking-code-for-fairphone-3/)

```bash

```
```console

```

#  

Read :
- [https://forum.ubuntu-fr.org/viewtopic.php?id=2071952](https://forum.ubuntu-fr.org/viewtopic.php?id=2071952)
- []()
- [https://doc.e.foundation/devices/fajita/install](https://doc.e.foundation/devices/fajita/install)
- [https://images.ecloud.global/community/fajita/](https://images.ecloud.global/community/fajita/)
- []()

## 
```bash

sudo apt install hashalot
sudo apt install checksums
wget https://images.ecloud.global/community/fajita/e-2.8-a14-20250221470208-community-fajita.zip.sha256sum
wget https://images.ecloud.global/community/fajita/e-2.8-a14-20250221470208-community-fajita.zip
FILE_PATH="/mnt/c/github/android-forensics/e-2.8-a14-20250221470208-community-fajita.zip" # "e-2.8-a14-20250221470208-community-fajita.zip.sha256sum"
HASH="a93d6a9cdb7f431f150c181f92f7839d2f549aec3a8216bd879d71ad2137f7ef"

#echo "$HASH  $HASH $FILE_PATH" | sha256sum -c 
#echo `cat $HASH $FILE_PATH` | sha256sum -c
# checksums sha256sum $HASH $FILE_PATH $HASH

echo "$HASH" | tr '[:upper:]' '[:lower:]' > ORIGINAL_HASH
cat ORIGINAL_HASH
shasum -a 256 $FILE_PATH | awk '{print $1}' > ecloud_OnePlus_6T_Fajita_SHA256
cat ecloud_OnePlus_6T_Fajita_SHA256
diff -qs ecloud_OnePlus_6T_Fajita_SHA256 ORIGINAL_HASH


```
```console

```




# Post-Install Clean-Up

Once you are done with the device, unmount it like this:
```bash
sudo adb kill-server
adb devices -l
usbipd detach --busid $BUS_ID
```
