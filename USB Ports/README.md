# Important Notes

This method modifies the OEM table where the USB Ports are declared(SSDT-6 A M I.aml for F8 BIOS). Modification includes limiting 11 ports using its GUPC method and enabling 15 ports using its TUPC method in Mac OS only.

Before proceeding with this method, you'll first need to have discovered which ports to enable and to disable for [limiting the USB port counts to 15 per controller](https://dortania.github.io/OpenCore-Post-Install/usb/#macos-and-the-15-port-limit). You may use scripts like [USBMap](https://github.com/corpnewt/USBMap) or discover the ports [manually](https://dortania.github.io/OpenCore-Post-Install/usb/manual/manual.html#usb-mapping-the-manual-way). 

Note that different BIOS versions may have different names for the table where the USB ports are declared.

# Requirements

- Modify the OEM table for USB(SSDT-6 A M I.aml for F8 BIOS) to satisfy the 15 port limit
- Inject the modified USB table (Preferably renamed after the modification to make it more easily identifiable)
- Remove the previously configured USB port mapping method
- Drop the OEM table for USB (SSDT-6 A M I.aml for F8 BIOS) in ACPI section of the config.plist

   Note: When configuring to drop the table, you may also need to fill in TableLength for the table as there may be more than one ACPI tables that use same OEM Table ID.

  
___

Below example is based on F8 BIOS

<img width="1310" alt="Screenshot 2022-08-25 at 10 56 46 AM" src="https://user-images.githubusercontent.com/97265013/186736714-6773c3cb-e7a0-438c-a6b0-6025f75ecb0a.png">

Config.plist

- ACPI -> Delete -> Drop the OEM table for the USB (SSDT-6 A M I.aml for F8 BIOS)
- ACPI -> Add - > SSDT-USBPorts.aml (Renamed from SSDT-6 A M I.aml after the modification)


The advantage of using ACPI USB port mapping method.

   - Kextless
   - USB ports are defined at the ACPI level and is unlikely to break because of future macOS updates
   - Will work on any SMBIOS
   - Does not affect other types of operating systems

         Note that if the changes are made to the OEM table due to BIOS updates, steps to
         modify and drop the OEM table for USB may need to be redone accordingly. 

# Modification

<img width="1243" alt="3" src="https://github.com/AppleBreak1/Z390-Designare-Customac/assets/97265013/93f4d5be-c4b6-459d-b6d9-6c7ba9ff35c4">
<br>
<br>

Looking at the above two methods used for _UPC object in OEM table for USB, we can see the following.


   - GUPC and TUPC both have one argument (Arg0)
     
   - GUPC(Arg0=PCKG[Zero]) controls Enable/Disable port (By default, GUPC disables(Zero) USB port and defines connector type as 0xFF(Internal)
     
   - TUPC(Arg0=PCKG[One]) controls USB connector type (By default, TUPC enables(One) USB port and defines connector type as Zero(USB1/USB2)

___
For its intended function of each USB port on macOS, limiting the number of USB ports to 15 per USB controller and correctly defining the USB connector type for each enabled USB port are necessary. 

As we can see, both of the written GUPC and TUPC method above only take one argument(GUPC=Enable/Disable USB port; TUPC=USB connector type). Without rewriting the method and with what's given, we can use both GUPC and TUPC to map the USB ports as below.

   - Use GUPC to disable USB port (If GUPC is used to enable USB port, we cannot control the connector type being defined for USB port individually. Instead, connector type of the enabled USB port will always be defined as 0xFF(Internal), a default for GUPC method. Thus, use GUPC for the purpose of disabling USB port)

      ~~~
      Return (GUPC (Zero)) -> Disable USB port
      ~~~
                 
   - Use TUPC to enable USB port and define USB connector type (By default, TUPC enables USB port and we can selectively and individually define USB connector type for each of its enabled USB port. However, it cannot be used to disable USB port)

      ~~~
      Return (TUPC (0x03)) -> Define USB connector type as 0x03 (Type-A USB2/USB3)
      ~~~
      
   - Add If (_OSI (" Darwin ")) statement to allow above method to be used in Mac OS only and leave everything else intact for other OSes.

<br>

**Example**

- Disable USB port: GUPC 

<img width="1030" alt="Screenshot 2022-08-25 at 12 31 08 PM" src="https://user-images.githubusercontent.com/97265013/186759663-d4de3add-d1fc-4d7f-9b71-31c41cec7ce6.png">

- Enable USB port: TUPC

<img width="953" alt="Screenshot 2022-08-25 at 12 45 21 PM" src="https://user-images.githubusercontent.com/97265013/186757334-912eced2-8fd2-45a6-956f-c90a9a476dab.png">


The _PLD (Physical Location Description) object is not very important for USB port mapping. For the USB ports to be enabled, may leave it as is and for the USB ports to be disabled, modify the first variable (Arg0) of GPLD or TPLD to Zero. You can ignore the second variable (Arg1) as well.

     Return (GPLD (Zero, 0x03))     -> Port not visible (Zero)

   - Do note that modifying _PLD and variables for GPLD and TPLD may not be necessary at all. May as well leave them untouched.
___

USB connector types for each port

- HS01/SS01 = 0x09

- HS03/SS03/HS04/SS04/HS05/SS05/HS06/SS06/HS07/SS07 = 0x03

- HS08/HS13 = 0x0A

- HS09/HS10 = 0x00

- HS11/HS12 = 0xFF (Define as Internal(0xFF) if Bluetooth is connected. Otherwise, Zero(USB2))

- HS14= 0xFF  (Onboard Intel CNVI module, disable if not used)


For more details of USB port connector types, please refer to [Dortania's USB Mapping Guide](https://dortania.github.io/OpenCore-Post-Install/usb/intel-mapping/intel.html)




   
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


# Activating more than 15 USB ports

Activating more than 15 USB ports require XhciPortLimit quirk to be enabled.  Normally when XhciPortLimit quirk is used with OEM USB table, all the USB ports returning (GUPC (One)) will be activated with connector type 0xFF as this is the default for GUPC method. However, with the modified USB table, we can manually activate only the needed USB ports beyond 15 USB ports using TUPC method and correctly define USB connector type. The USB ports to be disabled will need to return (GUPC (Zero))

Notes:

- OpenCore 0.9.3+ is required for XhciPortLimit quirk to be fully functional in Big Sur 11.3+
- While using modified USB table addresses most of the concerns listed [here]( https://dortania.github.io/OpenCore-Post-Install/usb/), the potential data corruption issue from using XhciPortlimit still remains which I have not yet experienced.

# Resources

https://learn.microsoft.com/en-us/windows-hardware/drivers/install/using-acpi-to-configure-usb-ports-on-a-computer

https://www.macos86.it/topic/9-mappatura-porte-usb/

https://dortania.github.io/OpenCore-Post-Install/usb/
