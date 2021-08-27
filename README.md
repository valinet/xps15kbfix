# xps15kbfix
A simple Windows PS/2 filter driver based on the [sample from Microsoft](https://docs.microsoft.com/en-us/samples/microsoft/windows-driver-samples/keyboard-input-wdf-filter-driver-kbfiltr/) that remaps some keys on the Dell XPS 15 7590's keyboard to my personal preference.

To run in a secure manner, [try this]().

To install, do this:

```
cd bin_folder
inf2cat /driver:. /os:10_19H1_X64
signtool sign /fd sha256 /ac C:\Certificates\localhost-root-ca.der /f C:\Certificates\localhost-km.pfx /p password /tr http://sha256timestamp.ws.symantec.com/sha256/timestamp xps15kbfix.cat
devcon install kbfiltr.inf ACPI\VEN_DLLK&DEV_0905
```



