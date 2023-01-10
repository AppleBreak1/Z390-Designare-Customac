# Z390-Designare-Customac
macOS Ventura, OpenCore 0.8.3


![03023952](https://user-images.githubusercontent.com/97265013/188255392-44718af6-9b02-44c1-8539-27a90679f42f.png)



# Hardware Specification

CPU : Intel i7-9700K

GPU: Sapphire Rx 580 Nitro+ Special Edition

RAM: Corsair 3200MHz DDR4 16GB x 4 (64GB)

Motherboard: Z390 Designare 

BIOS revision: F8

Wifi/BT Card: Fenvi FV-T919

WebCam: Logitech C920

Keyboard/Mouse: Space Gray Magic Keyboard/Mouse/Trackpad2

NVMESSD: Samsung 970 Pro 500GB (macOS Mojave)

SSD: Samsung 860 Evo 250GB (macOS Ventura)

SSD: Samsung 850 Evo 250GB (Windows 10)

HDD: Western Digital Blue 4TB x 2

Power Supply: Corsair RM850x White

CPU Cooler: NH-D15S Dual NF-A15 Chromax.Black

Case: NZXT H440 White Mid Tower Case

# Working

- Restart/Sleep/Shutdown

- Wake on Lan / Wake for network access

- iMessage / Facetime (WebCam Required)

- AirDrop/Handoff/Continuity (Requires natively supported WiFi/BT card such as FV-T919)

- AirplaytoMac/Sidecar/Universal Control (Requires natively supported WiFi/BT card such as FV-T919)

- UHD-630(IQSV Encoding/Decoding)

- Thunderbolt 3 / USB-C to HDMI / Ethernet (i211/i219(V7))

- Boot time from Apple logo to desktop: 20 sec.

- Working Mac OS: Sierra ~ macOS Ventura 13.0

Note 1: Config.plist changes for Sierra(10.12.6) ~ Catalina

    - SecureBootModel -> Disabled
    - UEFI -> APFS -> MinDate and MinVersion -> -1

Note 2: For Sierra(10.12.6) and High Sierra(10.13.6), 

    - Change SMBIOS to iMac18,3 
    
    - iGPU DeviceProperty -> AAPL,ig-platform-id -> 03001259 (For headless mode)  
                                                 -> 00001259 (To drive a display)     
                                            
                          -> rps-control instead of igfxfw (For Sierra Only)    
                          
    - Spoof to i7-7700K CPUID (0x0906E9) for (Sierra 10.12.6)   
    
    - Boot-args -> -disablegfxfirmware (For Sierra only. This prevents firmware load)
    
    
# Not Working

- DRM in Safari and TV+ (For workaround, [force AMD ENC/DEC](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.Chart.md))


# BIOS Settings

**Run Load Optimized Defaults**

- VT-d -> Enabled

- CSM Support -> Disabled

- X.M.P -> Profile 1

- Internal Graphics -> Enabled

- Onboard WiFi/BT -> Disabled

- Above 4G Decoding -> Enabled

- Thunderbolt

- Security Level -> No security

- Thunderbolt USB support -> Enabled

- GPIO3 Force Pwr -> Enabled

- XHCI Hand-Off -> Enabled

- Windows 8/10 WHQL

Note: F8 BIOS does not have CFG_Unlock option. MSR 0xE2 has been unlocked following the [guide](https://www.tonymacx86.com/threads/success-gigabyte-designare-z390-thunderbolt-3-i7-9700k-amd-rx-580.316533/post-2092710) written by CaseySJ.

# OpenCore 0.8.3 Config.plist

- ACPI -> Delete -> Drop OEM DMAR Table

- ACPI -> Patch -> SAT0 to SATA rename


- Device Properties

    - Intel i211 model name injection 
    - ALC 1220-VB Audio layout-id injection
    - UHD-630 Property Injection 
      
      - ig-platform-id (For iGPU acceleration)
      - device-id (Fixes KBL Unknown to UHD-630)
      - igfxfw (Load Apple's GuC Firmware)


- Kernel -> Quirks -> CustomSMBIOSGuid -> Yes


- Misc -> Security -> ScanPolicy -> 0 (Set it to 2,687,747 if you wish to hide unnecessary entries)


- NVRAM -> Boot-args -> Nothing. (For AMD Rx 5xxx/6xxx GPU, agdpmod=pikera is required)


- PlatformInfo -> Generic -> SystemProductName -> iMac19,1
- PlatformInfo -> Generic -> UpdateSMBIOSMode -> Custom


# SSDT

- SSDT-PLUG.aml (Injects Plugin Type 1; No longer required beginning with Monterey 12.3) 

- SSDT-NVRAM.aml (Required for native NVRAM)

- SSDT-EC-USBX.aml (Injects Fake Embedded Controller - High Power Charging for iPhone, iPad, etc)

- SSDT-DMAR.aml (Modifed to remove Reserved Memory Regions in OEM DMAR Table is required to load AppleVTD and to fix issues with ethernet/WiFi when VT-d is enabled. It is important to drop OEM DMAR table in ACPI section of the OpenCore config.plist)

- SSDT-DESIGNARE-Z390-NO-CNVW.aml (Disables Onboard WiFi/BT)

- SSDT-MCHCSBUS-DTGP.aml (Injects MCHC, SBUS, and DTGP method)

- SSDT-DESIGNARE-Z390-TB3HP-V4.aml (Required for functional Thunderbolt ports - Hotplug Fix)

- SSDT-COSMETIC.aml (Optional; injects fake or missing devices and rename devices)

    - Injected Devices: PNLF, THRM, SRAM, XSPI 
    - RP02.PXSX to GIGE  -> Intel i211 Ethernet
    - RP04.PXSX to ARPT  -> FV-T919 in 2nd PCIEx1 slot
    - RP09.PXSX to SSD1  -> 2nd M.2 slot (M2P) * Disables SATA 1 port when occupied by NVMe SSD
    - RP17.PXSX to SSD0  -> 1st M.2 slot (M2M) * Disables SATA 4 and 5 ports when occupied by NVMe SSD

- SSDT-LPCB.aml (injects DMAC, ALS0, FWHD, and ARTC devices under LPCB)

- SSDT-USBW.aml (One hit key wake from sleep; associated with USBWakeFixup.kext)

# Kexts

- VirtualSMC and plugins

- Lilu.kext

- AppleALC.kext

- WhateverGreen.Kext

- CPUFriend.kext

- CPUFriendDataProvider.kext (For i7-9700K)

- NVMeFix.kext

- USBPorts.kext (Ports: HS03, HS04, HS05, HS06, HS07, HS08, HS09, HS10, HS11, HS12, SS03, SS04, SS05, SS06, SS07)
   
   - Works on iMac18,3, iMac19,1, iMacPro1,1 and MacPro7,1

- IntelMausi.kext (i219V7 ethernet)

- SmallTreeIntel82576.kext (i211 ethernet; this kext is no longer required since Monterey 12.3; is natively supported)

   Note: Since Monterey 12.3.0, AppleEthernetE1000 driver kit natively attaches to i211 ethernet. However, if the ethernet port is occupied without having AppleVTD enabled, the system will experience freeze, crash, and etc. To avoid having these issues, we need to enable AppleVTD.
    
    Enabling AppleVTD
    
        - Enable VT-d in BIOS, 
        - Set DisableIoMapper to false
        - Drop OEM DMAR Table in config.plist
        - Inject modified DMAR Table(Reserved Memory Regions removed) in Config.plist

- USBWakeFixup.kext (Works with SSDT-USBW. Causes Bluetooth issue in Monterey or Higher; thus set minkernel to 19.0 and max kernel to 20.9.9) 
        
   Note: One may wish to continue using USBWakeFixup.kext on Monterey or higer. If so, you may use [Bluesnooze](https://github.com/odlp/bluesnooze) as a workaround for the bluetooth issue caused by this kext.


# Drivers

- HfsPlus.efi

- OpenCanopy.efi

- OpenRuntime.efi

- CrScreenshotDxe.efi

- AudioDxe.efi (Causes a good 4~5 second delay at splash screen thus disabled)
  
  To enable boot chime in Opencore Boot Picker, make the following changes.
  
        - UEFI -> Audio -> AudioSupport -> Yes
        - UEFI -> Drivers -> Enable AudioDxe.efi

# ACPI USB Port Mapping
I have made different versions of [USB Port Mapping](https://github.com/AppleBreak1/Z390-Designare-Customac/tree/main/USB%20Ports) SSDTs. 


# macOS Update Notes

macOS Sierra 10.12.6

- Change SMBIOS to iMac18,3 (Required)
- Spoof to Kaby Lake CPUIDs (Required)
- Inject ig-platform-id: 03001259 device-id: 12590000 (Necessary for iGPU acceleration) 
- Inject rps-control property instead of igfxfw (Necessary for iGPU acceleration and improvement) 
- Add boot-arg -disablegfxfirmware (To prevent firmware load. Necessary for iGPU acceleration) 
- SecureBootModel -> Disabled (Required)
- UEFI -> APFS -> MinDate and MinVersion -> -1 (Required)

macOS High Sierra 10.13.6

- Change SMBIOS to iMac18,3 (Required)
- Inject ig-platform-id: 03001259 device-id: 12590000 (Necessary for iGPU acceleration) 
- SecureBootModel -> Disabled (Required)
- UEFI -> APFS -> MinDate and MinVersion -> -1 (Required)

macOS Mojave ~ Catalina

- SecureBootModel -> Disabled (Required)
- UEFI -> APFS -> MinDate and MinVersion -> -1 (Required)
- Added NVMeFix.kext
- Added USBWakeFixup.kext, SSDT-USBW.aml (For Catalina ~ Big Sur)

macOS Big Sur

- Injecting AAPL,slot-name for iGPU or dGPU breaks certain encoding functions(H264) in FCPX.

macOS Monterey 

- USBWakeFixup.kext causes bluetooth issue after sleep/wake cycle.
- SmallTreeIntel82576.kext no longer works for Intel i211 ethernet.
- Beginning with Monterey 12.3, Intel i211 ethernet is natively supported. 
         ** AppleVTD is required for this to work **
- Beginning with Monterey 12.3, SSDT-PLUG.aml is no longer required. 

macOS Ventura

- As of Ventura 13.1, 9 out of 10 times my bluetooth Apple Magic Keyboard and Trackpad do not respond on startup unless bluetooth or the devices itself has been toggled off and on again with USB mouse. This also seems to be a problem with real Macs as well, as decribed [here](https://discussions.apple.com/thread/254330209?answerId=258137456022#258137456022)   
   
     For a workaround, I am temporary using utility called [Blueutil](https://github.com/toy/blueutil) to automatically toggle bluetooth "off" and "on" at login with a help of Automator.
     
    - Build blueutil from the source and install(copy) it to /usr/local/bin folder 
    - Create Automator to run shell script below
        
      ```shell
      export PATH=/usr/local/bin:$PATH
      blueutil -p0 && blueutil -p1
      ```
    - Save the Automator and name it (BluetoothToggle)
    - Add the Automator(BluetoothToggle) under Login Items to run at login.
    
    ** You may use a similar method to create service to run the script with keyboard shortcut when waking from sleep. 
    
# Geekbench 5

<img width="973" alt="Screen Shot 2020-07-14 at 1 09 02 AM" src="https://user-images.githubusercontent.com/97265013/185341359-56b1625c-2ea8-495e-a606-db66cc1d8bc2.png">





# Resources

- [Apple Inc](https://www.apple.com/) for MacOS.
- [Acidanthera Team](https://github.com/acidanthera) for OpenCore Bootloader, its documentation, and KEXTS.
- [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/) for thorough explanation.
- [CaseySJ's Golden Build Guide](https://www.tonymacx86.com/threads/success-gigabyte-designare-z390-thunderbolt-3-i7-9700k-amd-rx-580.316533/) for IO layout findings and Thunderbolt 3 hotplug fix SSDT.


Big thanks to Hackintosh community and those who contributed to make hackintosh possible from the beginning to the end!






