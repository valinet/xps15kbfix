# kbfiltr

WDF-based keyboard filter kernel-mode driver based on the [sample from Microsoft](https://docs.microsoft.com/en-us/samples/microsoft/windows-driver-samples/keyboard-input-wdf-filter-driver-kbfiltr/) that is able to remap keys for each individual keyboard connected to the PC. There are no settings - edit [the source code](https://github.com/valinet/xps15kbfix/blob/a9b39ecce7d07f6e3a01456481f376afd33538cd/sys/kbfiltr.c#L517) and customize to your liking. A list of scan codes (make codes) is available [here](https://learn.microsoft.com/en-us/windows/win32/inputdev/about-keyboard-input#scan-codes). Works for PS/2 and HID (USB, Bluetooth) keyboards.

## How to?

First of all, obviously, compile the driver on your machine (I have tested it solely on `x64` architecture). You need to have installed the [WDK](https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk) for this.

To run in a secure manner, I recommend [this](https://github.com/valinet/ssde), which is implied in the installation procedure below. So, to start, ensure ssde runs with Secure Boot on (`Confirm-SecureBootUEFI` in PowerShell); no need to enable test signing or Test Mode.

Remove the test signature from the driver:

```
signtool remove /s kbfiltr.sys
```

Sign the driver with the certificate created for ssde. Replace `password` with your certificate's password, and `17cf4521f162442bf61d3a09ec8c4455456eaf54` with the SHA1 of your `localhost-km.der` file.

```
signtool sign /fd sha256 /ac .\localhost-root-ca.der /f .\localhost-km.pfx /p password /tr http://sha256timestamp.ws.symantec.com/sha256/timestamp /td SHA256 /sha1 17cf4521f162442bf61d3a09ec8c4455456eaf54 kbfiltr.sys
```

Copy driver to `C:\Windows\System32\drivers`.

```
del C:\Windows\System32\drivers\kbfiltr.sysO
ren C:\Windows\System32\drivers\kbfiltr.sys kbfiltr.sysO
xcopy /Y kbfiltr.sys C:\Windows\System32\drivers
```

Install driver in the system.

```
sc create kbfiltr binpath=%windir%\system32\drivers\kbfiltr.sys type=kernel start=demand error=normal
```

Register driver as filter for the keyboard device class: in the Windows Registry, go to `HKLM\SYSTEM\CurrentControlSet\Control\Class\{4d36e96b-e325-11ce-bfc1-08002be10318}`, edit `UpperFilters` and add `kbfiltr` on a new line before `kbdclass` (`UpperFilters` is of type `REG_MULTI_SZ`).

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e96b-e325-11ce-bfc1-08002be10318}]
"UpperFilters"=hex(7):6b,00,62,00,66,00,69,00,6c,00,74,00,72,00,00,00,6b,00,62,\
  00,64,00,63,00,6c,00,61,00,73,00,73,00,00,00,00,00
```

Reboot the system end enjoy.

## Previous version

A simple Windows PS/2 filter driver based on the [sample from Microsoft](https://docs.microsoft.com/en-us/samples/microsoft/windows-driver-samples/keyboard-input-wdf-filter-driver-kbfiltr/) that remaps some keys on the Dell XPS 15 7590's keyboard to my personal preference.

To run in a secure manner, [try this](https://github.com/valinet/ssde).

To install, do this (use elevated "x64 Native Tools Command Prompt for Visual Studio 2019"):

```
cd bin_folder
"C:\Program Files (x86)\Windows Kits\10\bin\x86\Inf2Cat.exe" /driver:. /os:10_19H1_X64
signtool sign /fd sha256 /a /ac C:\Certificates\localhost-root-ca.der /f C:\Certificates\localhost-km.pfx /p password /tr http://sha256timestamp.ws.symantec.com/sha256/timestamp xps15kbfix.cat
devcon install kbfiltr.inf ACPI\VEN_DLLK&DEV_0905
```
