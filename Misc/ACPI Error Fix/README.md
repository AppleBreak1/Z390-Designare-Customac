# ACPI Error Fix

Due to ACPI bug in Gigabyte's firmware, we see this ACPI Error [\_TZ.TZ10._STA] in the boot log.  TZ10(Thermal zone for temp/fan monitoring) _STA method checks for operating system. However, there is no check for macOS, resulting in an ACPI error.

To fix this issue, we can rename _OSI to XOSI and inject SSDT-XOSI.aml

