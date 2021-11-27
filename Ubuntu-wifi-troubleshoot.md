---
title: Ubuntu-wifi-troubleshoot
created: '2021-11-24T17:57:44.815Z'
modified: '2021-11-27T05:41:21.082Z'
---

# Ubuntu-wifi-troubleshoot
Date : 24-11-21

> `Killer Wifi 6 AX500-DBS` wireless network adapter
_Laptop_: Dell `XPS15-9500`
_Ubuntu `20 LTS Focal-fossa`_
[intel driver support AX500-DBS](https://www.intel.com/content/www/us/en/support/products/217613/wireless/intel-wireless-products/intel-killer-wireless-products/intel-killer-wireless-series/intel-killer-wi-fi-6-ax500-dbs.html)

Problem statement
> New laptop, dual booted with windows 10, worked well for 1 month and suddenly wifi adapter not found in both windows and ubuntu. The day this started, there were many back and forth switches between OSes; could have possibly been updates in both OSes but can't confirm. The only way internet works is if windows is at the top of the boot order and never goes into GRUB and you let it be shutdown for 20 hrs after setting the boot order.

## Hypothesis of what is going wrong
The problem is probably an interaction of multiple hypothesis written here, so more than one things might be required to fix things permanently 
### Fast boot theory
- Windows does fast-boot => does hibernation when you ask it to shutdown. 
- Ubuntu does not wake up Wifi properly when waking up from hibernation (either it;s own or windows'?)
- So Ubuntu messes up and has no WIFI.
- Something throws off when windows is opened back up again since Ubuntu tried to wake up from Window's hibernation changing the disk state somehow-- but corrupting only the wifi

### Certain linux kernels are unable to boot with wifi adapter connected
- Some answers indicate that older kernels help fix the problem, or going from safe-mode booting could fix
- Since grub boots first, this is affecting windows too?
- It could have been a kernel update in ubuntu that triggered this

### Intel's Killer 6 AX500 and other killer series could have internal low power states that are hard to wake up from
- So they could have problems in windows only situations too, could interact with hibernation, fast-boot etc. and show problems more frequently
- Keeping system off for 20 hrs time or removing battery for 15 mins etc. are draining the wifi adapter causing it to restart cleanly when computer is powered on again, causing it to work (unless one of the above problems interfere)

- The [dell][XPS13 dell]  [posts][XPS15 dell] suggest the problem is not unique to dual boot, occur in windows only XPS13,15 too, due to Kille r WIFI adapter? (but could be exacerbated..)


- Could be affected by the model of wifi adapter - having some internal switch off things or harder to wake up from hibernation without re-loading firmware or doing clean restart..


**Solutions**
- [x] Moving to older kernel and disabling powersave worked
- [x] (didn't work) Try disabling fast-boot, boot into windows as first option, force wifi to turn on once and see if it lasts after restarts and shutdowns
  - then boot into Ubuntu as first option and see if wifi survives
- [x] (didn't work) Use the ubuntu's [Boot repair](https://help.ubuntu.com/community/Boot-Repair) to prevent windows and Ubuntu from stepping on each other's hibernation disk images while starting up
- [ ] try kernel 5.14.2
- [ ] Follow linux [troubleshooting guide](https://help.ubuntu.com/community/WifiDocs/WirelessTroubleShootingGuide/Drivers)

## Fastboot from Windows is the problem?
- [ ] disable fastboot and see if that helps?
  - fastboot does some sort of a hibernate -- which we know the wifi card has problems recovering from; especially in ubuntu
  - Since windows bootloader is preceeded by GRUB, this means that both windows and ubuntu will have problems that are common to GRUB's wifi adapter issues of waking it up from hibernate/suspend
  [askubuntu](https://askubuntu.com/questions/816935/need-grub-for-windows-linux-dual-boot-no-internet-connection)
  - could also be happening because of Legacy vs UEFI BIOS or how the [dual boot](https://itsfoss.com/install-ubuntu-1404-dual-boot-mode-windows-8-81-uefi/) situation was installed. 
    - _Example:_ I see `Windows bootloader` uption on the GRUB menu, which says `Windows 7` with legacy BIOS in my other laptop (Thinkpad) - so you are calling a windows bootloader instead of directly windows. [long answer](https://askubuntu.com/questions/666631/how-to-get-grub-to-be-the-default-bootloader-instead-of-windows-boot-manager-on) that touches upon Ubuntu installation, fast boot etc. 
  - [Boot repair](https://help.ubuntu.com/community/Boot-Repair) might help?  
  - Fast boot on windows is completely disabled trough powershell with `powercfg -h off` [archlinux](https://bbs.archlinux.org/viewtopic.php?id=257079)
    - neat article on various [hibernate/sleep modes](https://docs.microsoft.com/en-us/windows/win32/power/system-power-states)

### Fast boot + chainloading causing it?
- [reddit/Ubuntu](https://www.reddit.com/r/Ubuntu/comments/lbl0l5/can_windows_dual_boot_be_messing_with_my_wifi_card/)
> The firmware is loaded by the OS, and some times after a reboot (especially after a Windows pseudo-shutdown and restart) some settings of the wifi persist to the new OS which cause the firmware to not load and hence the card to not work.
- [Superuser](https://superuser.com/a/1231011/911646) 
> On rare occasion, one boot manager causes complications when launching another. For instance, this bug renders Windows unbootable from GRUB 2 if Secure Boot is enabled. I've heard of other, more exotic problems, involving specific hardware devices that fail when a new boot manager is inserted into the boot sequence. Such problems are rare, though.
> The dual-boot drawback is that if you boot your other OS, the partitions used by Windows will appear to be in an inconsistent state, which can prevent shared partitions from being used
  - This does not talk about hardware loading.. which also seems to be saved to disk?  


## Ubuntu

- quick commands to check network status,
  - shows network hardware details `sudo lshw -C network` [askubuntu](https://askubuntu.com/questions/1347951/wireless-adapter-unclaimed-after-update-and-reboot)
  - Shows if network is hardblocked: `sudo rfkill list all`
  - shows the network hardware details `lspci -nnk | grep -A2 0280` | `lspci` shows all the hardware connected
  - restart kernel configuration - `sudo service network-manager restart`
- others
  - check kernel version `uname -r`
  - Check firmware version `sudo dmidecode | more` or `apt list | grep linux-firmware` [r/Ubuntu](https://www.reddit.com/r/Ubuntu/comments/ag65on/how_to_check_the_version_number_of_linuxfirmware/)
  - check BIOS version `sudo dmidecode -s bios-version`
  - list all installed kernels - `apt list | grep linux-image | grep installed`
  - (too long list) You can see a full list of available kernels with `apt list linux-image*` 

- more commands
  > If your wireless has worked on this install before, and you want to troubleshoot a connection:
      - nm-tool will give you a report on the state of your network manager and any attached devices.
      - rfkill will enable or disable network devices. rfkill --list will list any available devices and provide some clues to their state. You may need to run this as root.
      - ifdown and ifup will bring network interfaces up and down. If you've changed settings, stopping and starting (as root) with ifdown wlan0 and ifup wlan0 may help.
      - Not sure what your network interfaces are even called? Try ifconfig -a [askubuntu](https://askubuntu.com/a/425166/841719)
      - If you aren't sure you even have drivers installed, find out with lshw if your wireless card is built in, or lspci (for PCI cards -- less likely these days) or lsusb (for a USB wireless device). lshw -C network will display details of hardware in the "network" class, which is what you want. Again, you may need to run this as root. 
  - you can get more information by running `journalctl --follow` in a terminal window. Then, when your WiFi drops, look at the messages.
- [Ubuntu wireless troubleshooting guide](https://help.ubuntu.com/community/WifiDocs/WirelessTroubleShootingGuide)

Fixes to try

- [x] (fixes wifi once, didn't work after reboot or second safe-mode trip :angry:) Boot into ubuntu with advanced options for ubuntu (2nd option in the grub menu)
  - _(didn't work for me)_ Try to run with an older kernel if visible there, (going onto `5.11.0.27` instead of `5.11.0.40`) else
  - run in safe mode with any kernel (_older might be better_), brings the `recovery menu`. click on `network` (_I did notice something failed..message after running this although this didn't affect the outcome_)
    - `system-summary` didn't show any information under the `--detailed network configuration--` section
  - Now select `resume` to boot into ubuntu, the wifi works right now! _(notice the super tiny screen upon login - might be a indication of safe-mode booting, adjust scaling to 200% to get decent icons)_


- [x] (didn't help after reboots) install OEM or firmware with known support : [linux-oem-20.040-edge](https://packages.ubuntu.com/focal-updates/amd64/linux-oem-20.04-edge/download)
[askubuntu](https://askubuntu.com/questions/1280328/ubuntu-20-04-killer-ax500s-dbs-drivers-support)
  - Downloaded this on another ubuntu and onto a pendrive and installed it in the problematic computer
  1. on an ubuntu with internet, ensure that focal-fossa mirrors are accessed by apt-get. open `etc/apt/sources.list` as admin in gedit/text editor, add this line `deb http://mirrors.kernel.org/ubuntu focal-updates main` at the end _(remove this line after the task is done if your OS is not focal-fossa version)_ 
  2. Use this to download the package (not install) onto a local folder, `cd desired/folder` ; `apt-get -d -o dir::cache=`pwd` -o Debug::NoLocking=1 install linux-oem-20,04-edge` and copy it into a USB drive. _Source:_ [superuser](https://superuser.com/questions/588167/apt-get-d-download-to-a-specific-directory) 
  3. Plug USB into problem ubuntu and `mv the folder` into a local folder, `cd` into it and install using `dpkg -i archives/*.deb`
  - Can follow up by updating the firmware to [latest](https://launchpad.net/ubuntu/+source/linux-firmware/1.187.20)

  - starting produces new errors before entering grub menu
  > soundcard cannot be loaded : `snd_hda_intel_..`
  > something else I forgot..

- [ ] Some kernel changes/updates can break wifi/other hardware connections: v 5.10
  - [x] clone and install a dev version of the kernel and firmware for ath11k (wifi drivers) 
  > still has issues with WIFI not working after suspend [medium](https://medium.com/@tomas.heiskanen/dell-xps-15-9500-wifi-on-ubuntu-20-04-d5f1c218e78a)
  - [askubuntu](https://askubuntu.com/questions/1354440/wifi-adapter-not-working-on-xps-13-9310-ubuntu-20-04?rq=1) answer says other things to experiment with
  > update the firmware of the device with `fwupdmgr refresh --force && fwupdmgr upgrade`

  - (doesn't work) How to check the latest kernel :  `apt-cache policy linux-generic-*` 
    - (too long list) You can see a full list of available kernels with `apt list linux-image*`
- [ ] try specific kernel versions that are known to work
  - People say it works in [5.14.2](https://bugzilla.kernel.org/show_bug.cgi?id=214455); [5.11.0.36-generic](https://www.reddit.com/r/Dell/comments/pwpg1w/linux_kernel_511037generic_breaks_wifi_and_bt_on/)
  > [finding and installing kernels](https://linuxhint.com/update_ubuntu_kernel_20_04/#google_vignette)
- [ ] ath11k drivers wifi [wiki](https://wireless.wiki.kernel.org/en/users/drivers/ath11k/installation)

 other stunts
  > I had to disable the chip in bios, reboot, and re-enable before the card would be recognized by the system again. [bugzilla](https://bugzilla.kernel.org/show_bug.cgi?id=214455)
 - [x] Change wifi.powersave to 2 in `/etc/NetworkManager/conf.d/default-wifi-powersave-on.conf`
  > I recall doing this on Thinkpad or Surface's Ubuntu which likely fixed wifi dropping intermittently when idle for 10 mins or so


- [ ] `Fn + F2` seems to be a toggle switch for WIFI hardware [askubuntu](https://askubuntu.com/questions/176897/how-do-i-toggle-the-wifi-hardware-switch-for-a-dell-xps-17-l702x)




## BIOS-bootloader-general solutions

- [x] (?) BIOS defaults reset
> F12 -> hardware scan -> restart. F2 -> BIOS defaults. Network adapter shows up after this. Rep says drivers were OK, BIOS is causing the issue.


- [ ] update to the latest BIOS
> Dell support told me today to **update to the latest BIOS**.  I had 1.1.4.  Now 1.2.5. This does not help- the BIOS defaults seems to help (_temporarily?_)

[XPS13 dell][XPS13 dell]

- Some advanced hardware workaround byuing a WIFI adapter and components [reddit/DellXPS](https://www.reddit.com/r/DellXPS/comments/hx1dq9/comment/g93m5ul/?utm_source=share&utm_medium=web2x&context=3)

  - This above post has lots of information specific to Dell 9500 with all sorts of linuxes dual booted

- [ ] replace GRUB bootloader with [rEFInd](https://teejeetech.com/2020/09/05/linux-multi-boot-with-refind/)
> I came across rEFInd five years ago, and it was one of those things that solved a lot of headaches for me. Many years later, I’m often surprised by how few people are aware of it, and how many people still struggle with GRUB for booting their Linux systems. 
> EFI systems have not reason to use GRUB -- which is good for legacy BIOS compatibility
> GRUB is both a boot loader and a boot manager. Refind is only a boot manager -- cleaner separation of tasks ; no interference of GRUB in Windows I guess
> Does rEFInd work with Windows? [Superuser](https://superuser.com/questions/1133040/unable-to-boot-into-windows-10-with-refind-solved) troubleshooting guidelines


## Intel's troubleshooting guide
If wireless adapter on your system is not working or seems to be disabled, try the following recommendations:

- [ ] (trivial) Make sure that Wi-Fi is not disabled through a hardware toggle on your computer. Some laptops have a hardware Wi-Fi switch or the keyboard combination to turn the Wi-Fi on and off. To learn more about the hardware toggle options refer to the specification for your system on the official site of the system manufacturer.
- [x] (?) Try a clean installation of the Intel Wi-Fi driver. If you have Intel Killer Wireless adapter installer on your system, follow the Intel Killer Software clean-install guide here.
- [ ] Update BIOS and chipset driver from your system manufacturer's website, if available.
    Update firmware on your wireless access point's Wi-Fi modem, router, or extender. Old firmware can cause this issue as the adapter will disable itself if it receives a large number of bad frames from the access point.
- [ ] Change your Wi-Fi adapter's power settings. Click here for our guide on Wi-Fi power settings.
- [ ] Use the built-in Network Reset for Windows 10. You can do this by clicking Start and typing Network Reset. You may have to reinstall any VPN adapters and re-input any Wi-Fi or VPN passwords after using the Network Reset. You may also need to reinstall the network adapter drivers.

If you have tried all of the above recommendations, but still find that your Wi-Fi adapter is disabled, use the Contact support link in the blue banner below.

[Intel](https://www.intel.com/content/www/us/en/support/articles/000059081/wireless/intel-wireless-products.html)





## Windows fixes

- [x] (?) Updating killer control center and drivers [intel](https://www.intel.com/content/www/us/en/support/articles/000058920/ethernet-products.html)
> For Windows* OS: Before attempting a manual install, we recommend trying to install and update your drivers and software automatically using the Intel® Driver & Support Assistant tool. For Linux* OS: See Linux* Support for Intel® Wireless Adapters for more information about Linux drivers.

  - [ ] Killer driver version check/update manually
> This is seen after putting the PC in an idle or sleep state for several hours. These issues happen when the PC has the Killer driver 1.0.0.1428 installed.
XPS 15 9500/XPS 17 9700 users should uninstall **Killer driver** 1.0.0.1428 and install 1.0.0.1274. Restart the PC when done.  
  - latest version of Killer 500 is **[1.0.0.1606](https://www.dell.com/support/home/en-us/drivers/driversdetails?driverid=666pp)** - 1 Jun 2021

  - Instructions from the above page for manual installation
  > After you install this driver, the Killer 500 Wi-Fi driver version in Device Manager is v1.0.0.1606. Killer 500 drivers must be installed in the following sequence:
    1. Install Killer 500 Wi-Fi and Bluetooth driver.
    2. Install Killer Extension and Component driver.
    3. Install Killer Control Center application. 


- [ ] network adapter: turn off power save
> I resolved it by unchecking the Allow the computer to turn off this device to save power option.
The otpion can be found under Device Manager → Network Adapters → your network adapter → Properties → Power Management. I guess it's more of a Windows problem than Ubuntu.

[askubuntu][askubuntu1]
  - [ ] Can't find the Power Management tab
  > Add PlatformAoAcOverride (0) to HKLM\System\CurrentControlSet\Control\Power; same place as the old CsEnabled key. -- Seems to do more than just enabling the tab [reddit getting back S3 sleep](https://www.reddit.com/r/Dell/comments/h0r56s/getting_back_s3_sleep_and_disabling_modern/)
  > If i go to network adapters and open the properties of my wifi card (Intel(R) Wi-Fi 6 AX201 160MHz) there is **no power management tab**. Another post said to go to HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power in regedit, where I had to create a new DWORD called CsEnabled and set the value to 0, and restart, but that did not work. How do I fix this? [microsoft](https://answers.microsoft.com/en-us/windows/forum/all/power-management-tab-is-missing-in-device-manager/f84fe229-fb66-4aa1-a397-e9ac65f791e5)

  - [ ] _(in addition to above)_ General power management -> Disable USB selective suspend settings [demcrumbliesreviews](https://demcrumbliesreviews.com/how-to-fix-wifi-drops-issues-on-your-pc/)
  - [x] Extra property to disable: Mimo power saving
  > @Kvothe Mine is an Intel card, which had an extra property **MIMO Power Saving** or similar. Disabling this also fixed the problem. Just look through all settings tabs of your device and you will find something with energy. – Xerusial Feb 6 at 11:00 [askubuntu][askubuntu1]
  


- [ ] Windows update: 
> My problem resolved itself "magically" after Windows update **20H2**, which included **intelpep.sys** files and some other intel base drivers... And yesterday the last KB windows update patch had an **Intel Serial IO driver update**, pure coincidence I'm sure 
[XPS15 dell][XPS15 dell]
> I then unplugged the battery to the main board for 30 seconds and replugged it. F2 BIOS mennu, WIFI shows up now
- [ ] having a WIFI firmware backup on USB..

- [ ] (fixes a different problem) Manually download new wifi drivers - suggests a longer term fix (tested 8 days)
[reddit/Dell](https://www.reddit.com/r/Dell/comments/d7x0wt/new_drivers_solve_killer_wifi_problems_for_xps_15/)

### temporary fix to wake up Wifi once
use this once the long-standing probolem is fixed

- [ ] Reset network adapters using windows powershell. [Windowsreport](https://windowsreport.com/wifi-adapter-not-working-windows-10/)
>   netsh winsock reset
    netsh int ip reset
    ipconfig /release
    ipconfig /renew
> Winsock is a programming interface and supporting program in Windows operating system. It defines how Windows network software should access network services. If its data went wrong, this issue may occur. [drivereasy](https://www.drivereasy.com/knowledge/windows-10-wireless-adapter-missing-solved/)





[askubuntu1]: https://askubuntu.com/questions/984303/no-wifi-adapter-found-dual-boot-windows-10-and-ubuntu-17-10
[XPS15 dell]: https://www.dell.com/community/XPS/XPS-15-9500-Killer-AX500-WiFi-BT-WiFi-not-working/td-p/7728889
[XPS13 dell]: https://www.dell.com/community/XPS/XPS-13-9310-WiFi-randomly-shuts-down-AX500-DBS/td-p/7775525 

