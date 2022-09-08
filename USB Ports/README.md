# Important Notes

This method modifies OEM USB Table SSDT-6 A M I(where the USB Ports are declared). Modification includes limiting 11 ports using GUPC method and enabling 15 ports using TUPC method in Mac OS only. To make it work, first remove previously configured USB port mapping method, then drop OEM SSDT-6-A M I table in ACPI section of config.plist and add patched SSDT-6-A M I which is renamed to SSDT-USBPorts.aml after the modificiation. Ensure that the tablelength matches the length indicated in your OEM SSDT-6-A M I, as different BIOS revisions may have different table length. Otherwise, this USB port mapping won't work due to failure in dropping OEM SSDT-6-A M I table.

Below example is based on F8 BIOS

<img width="1310" alt="Screenshot 2022-08-25 at 10 56 46 AM" src="https://user-images.githubusercontent.com/97265013/186736714-6773c3cb-e7a0-438c-a6b0-6025f75ecb0a.png">

Config.plist

- ACPI -> Delete -> Drop OEM USB Table (SSDT-6 A M I)
- ACPI -> Add - > SSDT-USBPorts.aml




The advantage of using this USB port mapping method.

   - Kextless
   - Will work on any SMBIOS
   - Does not affect other types of operating systems.

# Patches

Diable USB port

<img width="1030" alt="Screenshot 2022-08-25 at 12 31 08 PM" src="https://user-images.githubusercontent.com/97265013/186759663-d4de3add-d1fc-4d7f-9b71-31c41cec7ce6.png">

Enable USB port

<img width="953" alt="Screenshot 2022-08-25 at 12 45 21 PM" src="https://user-images.githubusercontent.com/97265013/186757334-912eced2-8fd2-45a6-956f-c90a9a476dab.png">

For USB port connector types please refer to [Dortania's USB Mapping Guide](https://dortania.github.io/OpenCore-Post-Install/usb/intel-mapping/intel.html)
   
# Versions

Version 1 Ports: HS03, HS04, HS05, HS06, HS07, HS08, HS09, HS10, HS11, HS12, SS03, SS04, SS05, SS06, SS07 

- In the absence of front USB-C type port and when USB2.0 C-Type connection is needed; enables HS08 port.

Version 2 Ports: HS03, HS04, HS05, HS06, HS07, HS09, HS10, HS11, HS12, SS01, SS03, SS04, SS05, SS06, SS07 

- Front USB-C type port is present; enables SS01 port.

Version 3 Ports: HS03, HS04, HS05, HS06, HS07, HS09, HS10, HS11, HS14, SS01, SS03, SS04, SS05, SS06, SS07 

- In case of using Onboard CNVI Bluetooth; enables HS14 port.

IO Layout Credit ([CaseySJ](https://www.tonymacx86.com/threads/success-gigabyte-designare-z390-thunderbolt-3-i7-9700k-amd-rx-580.316533/#BIOSModifications))


![Designare-Z390-IO-Layout 1-1 (dragged)](https://user-images.githubusercontent.com/97265013/185357545-ba766504-8352-4f0c-87ba-42b86423c15f.png)
![Designare-Z390-IO-Layout 1-2 (dragged)](https://user-images.githubusercontent.com/97265013/185357716-8dccc81a-354a-43ff-b105-aabd2ca614d4.png)
