# lsi-sas92xx-m1015-flash-firmware
The full bootable filesystem needed to reset & flash an LSI SAS92xx M1015 SAS controller card (eg, to IT mode), bootable in both DOS & UEFI


## Acknowledgements

When putting this together, for the most part I didn't plan for it to be reproducible, so I've done my best to retrace my steps and the provinence of the difference components.

- I chose to license this project as GPLv3, based on the assumption that at least one of the binaries in this stack is GPL licensed, and will infect this distribution.
  - I haven't bothered checking most licenses too closely, I'll address any license disputes if they come up but I'd be pretty surprised
- I got the original fileset for flashing to P20 from [Spearfoot on TrueNas](https://www.truenas.com/community/threads/ibm-serveraid-m1015-and-no-lsi-sas-adapters-found.27445/post-301617)
  - I found this via [Philip and Dakota Schneider's blog](http://codefromabove.com/2017/03/crossflash-ibm-m1015-to-lsi-9220-8i-it-mode-for-freenas/)
- The final combination of binaries I needed came from [this stack overflow comment](https://serverfault.com/questions/679175/failed-to-initialize-pal-while-upgrading-an-lsi-9211-8i-to-it/679176#comment1313992_679176)
  - The [./efi/boot/bootx64.efi] is from [tianocore/edk2](https://github.com/tianocore/edk2/blob/8afe7c9ae565f3369b8fa46af70ccb7c2f05ce01/EdkShellBinPkg/FullShell/X64/Shell_Full.efi), 
    - the specific version I am using plays well with the other binary utils, particularly ( `sas2flash.efi`)
    - `tianocore/edk2` is BSD licensed, which should 
  - [sas2flash.efi](./sas2flash.efi) is [from broadcom](https://www.broadcom.com/site-search?q=Installer%20for%20UEFI) specifically [version P20 for UEFI](https://docs.broadcom.com/docs/12350820)
- [GeekGoneOld's comment on truenas](https://www.truenas.com/community/threads/ibm-serveraid-m1015-and-no-lsi-sas-adapters-found.27445/post-221154) was a solid comprehensive reference explaining the process.
- The very first walkthrough I used was a 10 year old guide from [servethehome](https://www.servethehome.com/ibm-serveraid-m1015-part-4/) which also gives a solid reference for **what IT and IR modes are** and **why** you might want them.