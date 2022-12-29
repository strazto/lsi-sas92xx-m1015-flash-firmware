# lsi-sas92xx-m1015-flash-firmware
The full bootable filesystem needed to reset & flash an LSI SAS92xx M1015 SAS controller card (eg, to IT mode), bootable in both DOS & UEFI


## Reminder - Find the card's SAS address printed on a sticker on the card.

You're going to need this later, so whenever you get a chance to pull the card out, find a sticker with a 16 digit hex code and take a photo.
It probably looks something like: `500605Bxxxxxxx`

## How to use

In order to reliably flash one of these cards, there a few different tools we need to use, and a few different bootable environments needed to run these tools.

I've set up this project to make it so the steps to get the environments you need are as straightforward as possible.
- You will just need to copy the same filesystem to both:
  - A `DOS` compatible USB
  - A `UEFI` compatible USB
  - the only difference is the way you've formatted the USB.

### Tools / Environments

#### Tools:

- `megarec` utility
  - We need this to wipe the previous firmware and vendor information from the card.
  - (this only works in a `DOS` shell)
- `sas2flash.efi`
  - We need this to flash the new firmware and SAS address onto the card
  - (this *probably* only works in an `EFI` shell)
- [Rufus](https://rufus.ie) or a similar tool, for making bootable USBs.

#### Environments:

- `DOS` shell
   - We need this to run `megarec`
- `EFI` shell
   - We need this to run `sas2flash.efi`

#### Images

- `sbrempty.bin`
  - We write this to a device using `megarec` to help clear existing information from it
  - We use `megarec` to write this to the device.
- `2118it.bin` :
  - The target firmware image we're going to flash to the device.
  - This firmware allows direct disk passthrough to the host machine.
    - An alternative target firmware is `2118ir.bin`, which also supports disk passthrough, but with more hardware RAID features.
  - We use `sas2flash.efi` to flash this to the device.

#### Physical peripherals

- You need a USB stick to boot from.

### Steps

#### A: Wipe the existing firmware (`DOS` Shell)

Set up the USB:

1. Use rufus to make a `DOS` USB
   - For "Boot Selection" choose **"FreeDos"**
   - There's no need to use a disk or ISO image
   - Choose `MBR` for the partition scheme.
2. Copy everything from the root of this project to the root of the USB.

Wipe the card:

1. Plug the USB and boot from it
2. Wipe the card:
   You can either type:
   ```
   megarec -writesbr 0 sbrempty.bin
   megarec -cleanflash 0
   ```
   Or try running the batch script `TOITMD1.BAT` (does the same thing)

Shut down the system.

#### B: Flash the new firmware (`EFI` Shell)

Set up the USB

1. Use rufus to make a "Non-Bootable" USB
   - This is misleading, since the USB will be bootable, but that's because the files you copy from this project is what makes it bootable.
   - Use `GPT` for the partition scheme
2. Copy everything from the root of this project to the USB.
   - The part that makes the USB bootable is the special filepath `/efi/boot/bootx64.efi`.
   - This is the binary for the `EFI` shell we're booting to.

Flash the card:

1. Plug in the USB and boot from it
2. Navigate to the USB filesystem
   - The shell should give you a list of filesytems on the device on boot, but you can also use `map` to list them.
     - You can use PageUp / PageDown keys to scroll
   - The actual filesystem ID may vary (It's usually `fs0`, but for me it was `fs1`), but it seems to appear first on the list.
   - To change filesystems
     ```shell
     # The colon at the end is important
     fs1:
     ```
   - Confirm you're in the correct place by listing the contents:
     ```
     dir
     ```
     You should be able to see the files from this project
3. Flash the card
   You can type it out by hand:
   - Flash the image
     ```
     sas2flash -o -f 2118it.bin -b mptsas2.rom
     ```

     Re-add the card's address
     ```
     sas2flsh -o -sasadd 500605Bxxxxxxx
     ```
     Substituting your card's actual address for `500605Bxxxxxxx`
   Or you can try the batch script `TOITMD2.BAT`

If all goes well, you should get some output confirming the flash operation and address write.

Shutdown the system, unplug the USB, and boot normally.

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