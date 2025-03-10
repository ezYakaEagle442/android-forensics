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
- []()
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

ls -al /etc/udev/rules.d/51-android.rules
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

<span style="color:red">**/!\ IMPORTANT WARNING: **</span>

On your Android device, choose the 'MTP  File transfer' USB preference AND go to Parameters / System / Developpers options :

- enable Developpers options
- USB debugging MUST BE switched OFF

```bash
ls -al /mnt/d

mtp-detect
sudo jmtpfs -l
adb devices

```

Once you are done with the device, unmount it like this:
```bash
usbipd detach --busid $BUS_ID
```


## xxxx

Read :
- []()
- []()
- []()
- []()
- []()

```bash

```
```console

```