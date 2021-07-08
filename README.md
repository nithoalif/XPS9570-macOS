![Screenshot](img/screenshot.png)

# Dell XPS 15 9570 Big Sur
A collection of all resources needed to run macOS (Big Sur as of 8 July 2021) on a Dell XPS 15 9570 4K model

## 🔍 Overview
This is more of a compilation of information and configs from various repositories and forums than a place where real development happens. This repository should contain everything needed to get Big Sur up and running on your specific Dell XPS 9570 configuration.

## ℹ️ Current Status

| Feature | Status | Notes |
| ------------- | ------------- | ------------- |
| **Intel iGPU** | ✅ Working | Fully supported. Both 2560x1440@144Hz over HDMI and 4k@60Hz over DisplayPort have been tested |
| **Trackpad** |  ✅ Working | Full gesture support. Probably the best trackpad experience on a non-mac.
| **iMessages and App Store** | 🔶 Partially working | Try to follow the  [this guide](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html) |
| **Speakers and Headphones** | ✅ Working | To permanently fix headphones follow the instructions [here](#-audio) |
| **Built-in Microphone** | ✅ Working |
| **Webcam** | ✅ Working | Fully working, is detected as Integrated Webcam |
| **SD Reader** | ✅ Working | Fully supported, but rather slow |
| **Handoff** | ✅ Working |
| **Unlock with Watch** | 🔶 Buggy | Works, but it tends to disable itself after sleep or reboot |
| **Wi-Fi/BT** | ✅ Working | Use the experimental AirportItlwm to enable wifi with intel wireless card |
| **Thunderbolt/USB-C** | 🔶 Partially working | Normal USB-C and charging work as intended. Thunderbolt works, but hotplugging is broken. Thunderbolt devices and docking stations have to be attached prior to boot to work properly. However, display over Thunderbolt seems to hotplug fine. |
| **Touchscreen** | 🔶 Working, but high power consumption | The touchscreen works fine and emulates a huge trackpad. This means you can do all native macOS gestures. However, power management isn't that great. [Battery drain](https://github.com/jaromeyer/XPS9570-Catalina/issues/1) is very high. If you don't need it, you can [disable](#-display) it completely. |
| **NVIDIA GPU** | ❌ Not working | Will never work because of Nvidia Optimus and Apple completely dropped Nvidia support beginning with Mojave. Thus it's completely disabled to save power. |
| **PM981 SSD** | ❌ Not working | Even with [NVMeFix](https://github.com/acidanthera/NVMeFix), which promises to fix Kernel Panics caused by the PM981, there are random shutdowns. Just replace it with a SATA M.2 drive or a supported NVMe one. |
| **Fingerprint reader** | ❌ Not working | Probably will never work, because proprietary Goodix drivers that only exist for Windows are needed. Disabled to save power. |


## 💾 Installation
Follow this guide if you have never set up a Hackintosh before.

### Creating a bootable installer
To start you need a USB flash drive with at least 12GB of available storage and a local copy of macOS. The installer for macOS Big Sur can be obtained from [here](https://apps.apple.com/us/app/macos-big-sur/id1526878132?mt=12).

Next, you want to format the USB flash drive using Disk Utility. Click on “View” in the toolbar and choose “Show All Devices” to see all physical disks instead of only partitions. Select your USB flash drive, name it “MyVolume” and format it HFS+/Mac OS Extended (Journaled) with GUID Partition Map.

Now you are ready to create the installation media. Use the following command to start the process. It may take a while depending on the USB flash drive you are using.

`sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume`

If your USB flash drive has a different name, replace ```MyVolume``` with the name of your volume.

After the installer says that it's done, the volume now contains the macOS Big Sur installer and is ready to boot on a real Mac. However, because we are building a Hackintosh, we have to take an additional step and install the OpenCore bootloader. To do this, you first have to mount the EFI partition. This is where OpenCore and all its configuration files go. Use the following command to list all disks.

`sudo diskutil list`

Find the EFI partition of your USB flash drive. Normally its entry 1 under /dev/disk2. Use the following command to mount the EFI partition - in this case, ```disk2s1```.

`sudo diskutil mount disk2s1`

Now that you have access to the EFI partition, the real fun starts.

### Configuring EFI
Clone this repository to get the base EFI folder as well as all additional kexts and patches. Now you will have to prepare the EFI folder for your exact hardware configuration. Read through the [hardware section](#-Hardware) to learn more about the different options. Once everything is configured properly, copy the folder into the EFI partition you have mounted in the previous step.

### Booting the installer
After having created the installer USB flash drive, you are ready to install macOS on your XPS. Make sure SSD mode is set to AHCI mode instead of RAID in BIOS otherwise, macOS won't be able to detect your SSD. Select your USB flash drive as boot media and go through the macOS installer like you would on a real mac. Once you have come to the desktop, advance to the next step.

### Post Installation
Congratulations! You have successfully booted and installed macOS. At this point, you just have to copy the EFI folder you have prepared in a previous step to the SSD. Mount the EFI partition of your SSD with

`sudo diskutil mount disk0s1`

and copy your customized EFI folder into the newly mounted EFI partition. You should now be able to boot your computer without the USB flash drive attached. If you're having issues with specific parts like Wi-Fi, Bluetooth, or Audio, have a look at the corresponding sections in this repository and open an issue if you are unable to solve them.

## 🛠 Hardware
This section talks about configuring the EFI folder for your exact hardware.

Almost all changes are done inside the OpenCore configuration file. I strongly recommend using either [ProperTree](https://github.com/corpnewt/ProperTree) or Xcode to edit `EFI/OC/config.plist`.

### 🔈 Audio
Without any modifications, the headphone jack is buggy. External microphones aren't detected and the audio output may randomly stop working or start making weird noises. Sometimes un- and replugging the headphones works, but that's pretty annoying and unreliable. To permanently fix this issue you will have to install [this fork of ComboJack](https://github.com/lvs1974/ComboJack).

### 🔋 Power management
Hibernation is not supported on a Hackintosh and everything related to it should be completely disabled. Disabling additional features prevents random wakeups while the lid is closed. After every update, these settings should be reapplied manually.

```
sudo pmset -a hibernatemode 0
sudo rm -f /var/vm/sleepimage
sudo mkdir /var/vm/sleepimage
sudo pmset -a standby 0
sudo pmset -a autopoweroff 0
sudo pmset -a powernap 0
sudo pmset -a proximitywake 0
sudo pmset -b tcpkeepalive 0 (optional)
```

For the best power management it's recommended to disable CFG lock and let macOS do the power management. Follow [this guide](https://github.com/jaromeyer/XPS9570-Catalina/issues/44#issuecomment-708540167) to do so. For more information about CFG lock, have a look [here](https://dortania.github.io/OpenCore-Post-Install/misc/msr-lock.html).

### ⚡️ Performance
CPU power management is done by `CPUFriend.kext` while `CPUFriendDataProvider.kext` defines how it should be done. `CPUFriendDataProvider.kext` is generated for a specific CPU and power setting. The one supplied in this repository was made for the i7-8750H. In case you have another CPU, you can use [one-key-cpufriend](https://github.com/stevezhengshiqi/one-key-cpufriend) to generate your own `CPUFriendDataProvider.kext`.

## 🔧 Tweaks
This section talks about various optional tweaks that enhance your experience

### ⤵️ Undervolting
Undervolting your CPU can reduce heat, improve performance, and provide longer battery life. However, if done incorrectly, it may cause an unstable system. My preferred method is using [VoltageShift](https://github.com/sicreative/VoltageShift).

Using `./voltageshift offset <CPU> <GPU> <CPUCache>` you can adjust the voltage offset for the CPU, GPU, and cache. Safe starting values are ```-100, -75, -100```. From there you can start gradually lowering the values until your system gets unstable.

## 🤔 Frequently Asked Questions

### I have a Samsung PM981 SSD, will it work?
The Samsung PM981 (or more precise the controller it uses) is known to cause random kernel panics in macOS. Up until now, there was no way to even install macOS on the PM981 and the only option was to replace it with either a SATA or a known working NVMe SSD. However, recently a new set of patches, namely [NVMeFix](https://github.com/acidanthera/NVMeFix) was released. It greatly improves compatibility with non-apple SSDs including the PM981. Thanks to those patches, you can now install macOS, but there is still a chance for kernel panics to occur while booting.

## Acknowledgments
- [acidanthera](https://github.com/acidanthera) for providing almost all kexts and drivers
- [alexandred](https://github.com/alexandred) for providing VoodooI2C
- [daliansky](https://github.com/daliansky) for providing the awesome hotpatch guide [OC-little](https://github.com/daliansky/OC-little/) and always up-to-date Hackintosh solutions in [XiaoMi-Pro-Hackintosh](https://github.com/daliansky/XiaoMi-Pro-Hackintosh)
- [RehabMan](https://github.com/RehabMan) for providing many laptop [hotpatches](https://github.com/RehabMan/OS-X-Clover-Laptop-Config/tree/master/hotpatch) and guides
- [knnspeed](https://www.tonymacx86.com/threads/guide-dell-xps-15-9560-4k-touch-1tb-ssd-32gb-ram-100-adobergb.224486) for providing ComboJack, well-explained hotpatches and a working USB-C hot plug solution
- [bavariancake](https://github.com/bavariancake/XPS9570-macOS) and [LuletterSoul](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave) for providing detailed installation guides and configurations for the XPS 9570
- [xxxzc](https://github.com/xxxzc/xps15-9550-macos) for providing OpenCore support for the XPS 9570
- [frbuccoliero](https://github.com/frbuccoliero) for PM981 related testing and extending the guide
- [mr-prez](https://github.com/mr-prez) for the Native Power Management guide
- [jaromeyer](https://github.com/nithoalif/XPS9570-macOS) for providing his OpenCore EFI folder
- Everyone else involved in Hackintosh development
