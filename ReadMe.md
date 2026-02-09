 # Morrow DISK JOCKEY 2D CP/M 2.2 "SYSGEN" Recompile From Source Files (For Exidy Sorcerer)

This repo carefully documents how to rebuild a CP/M 2.2 "SYSGEN" image, from scratch, for the Morrow DISK JOCKEY 2D S-100 8" floppy disk drive controller card. In particular, the image will be set up for the Exidy Sorcerer variation whereby the DJ2D's ROM firmware is located at D000H instead of the standard E000H. Nonetheless, you could still follow the same process outlined herein to recompile the image for the SOL and other target machines if you can find an appropriate CBIOS.ASM for your machine. 

<div style="text-align:center">
<img src="/Images/CPM 2.2 recompile for Morrow DJ2D.jpg" alt="" style="width:50%; height:auto;">
</div>

The benefits of being able to regenerate such SYSGEN images are several:

- You can swap in a new CCP. In my case, and for this tutorial, that would be ZCPR v1.2.

- You can modify/improve/patch your CBIOS with ease, and/or add changes/patches to the stock BDOS code, as I have done to both in this tutorial.

- You don't need to go mentally crazy trying to hand-patch the image using DDT or SID, as was the norm back in the day.

- Using my associated Hex-File-Overlay-Tool, you can regenerate the image endlessly at ease from the source .hex files.

<div style="text-align:center">
<img src="/Images/Morrow DJ2D S-100 controller card.jpg" alt="" style="width:45%; height:auto;">     <img src="/Images/Morrow DJ2D Shugart 800 8in floppy drive.jpg" alt="" style="width:45%; height:auto;">
</div>

The photos above shows my Morrow DJ2D controller card (Model B Rev 2) and its associated "DISCUS 2D" Shugart 801R 8" Floppy Drive. Purchased July 1981 for US$899 from Mini Micro Mart, Syracuse NY. The firmware ROM is located at D000H and its RAM at D400H. The system is still functional and in active use today.

## A Short Historical Oveview

45 years ago all of the following steps were second nature to me, as common everyday knowledge, and as such I had not documented the process which I had last used to create my 48k and 52k SYSGEN images. Fortunately I had imaged all of my 8" diskettes 15 years ago from which I was able to piece together the necessary boot, CCP, BDOS and CBIOS source files then confirm them against the original images. 

The small challenges for this project was to start with no source files, no idea what original files were needed (if they actually still existed in my collection), how Sysgen.com works, how the various files related to the SYSGEN images and (most importantly) how to recreate a 1:1 perfect bit-level reproduction to my original 1983 images. Other than learning of the memory layout, figuring out how ABOOT's multiple cold boot code segments mapped to the various 128-byte sectors was most interesting to unravel in my mind; I had not been aware of that during my time as an active DISCUS 2D user in the early 1980s. 

The end result of this work was to create a 1:1, exact bit-level replica, of my 1982-era SYSGEN images (of known good and valid quality) against the source files I had on hand + the compilers that I used to recompile all of the old code. It was quite the "software hacker adventure" to get everything aligned and made bit-level identical to the 45 year old images.

## An Overview of the SYSGEN Memory Layout for CP/M 2.2

There wasn't any easily accessible documentation which I needed for this project at first (that explained what I needed to know). Hence, I chose to derive everything from first principles (ie. reverse engineering).

Normally, back in the 1980's, you might do something like this:

- Optionally use DDT.com or SID.com to patch your SYSGEN image with revised boot, CCP, BDOS and/or CBIOS compiled code.

- Use Movcpm.com to relocate your CP/M image to a specific memory configuration, such as 48k or 52k. CP/M naturally resides at the top of memory and with fixed memory addressing. 

- Then use Sysgen.com to write that image to the system tracks of your floppy disk. For the DJ2D, the first system track is single density (128 * 26 sectors) and the second system track is double density (1024 * 8 sectors).

The memory layout of the SYSGEN image is as follows:

|Offset from 0100H|Segment Length|Description|
| :-----: | :---: | :--- |
| | |
|0H| 800H| This is where Movcpm.com and/or Sysgen.com reside (at TPA = 0100H)|
|0800H|800H| DJ2J cold boot, warm boot and firmware| 
|1000H|800H| The CCP (command control processor)|
|1800H|E00H| The BDOS|
|2600|Variable| The CBIOS, up to the end of memory|

The CBIOS2.ASM file has a chart at the beginning which explains the layout for the first system track, of which there are 16 sectors (at 128 bytes) allocated to the various cold boot, warm boot and DJ2D firmware, with the remainder of the first track being allocated to the CCP. Track 1 then accomodates the remaining CCP, BDOS and CBIOS. Hence, all of CP/M 2.2 must fit within the confines of the first two tracks. That differs from CP/M 3+ for which the system components can be files on the diskette. 

If you may ask, why do I have a 52k system and not a 48k system? Well, one of my past projects in the eartly 1980s was to change my BASIC RAM/Pack module into a RAM/Pack offering another 4K of memory to the system. 

## Setting the Memory Size in the Source Files

Most importantly you will need to edit all 4 of the .asm and .mac files (in the [Src](/Src) directory to reflect your current memory size. The files are presently set up for 52k.

In the world of CP/M 2.2 the DRI source files are set up based on a 'bias' value relative to a stock 20k CP/M system of 2D00H. You need not be too concerned about that. Hence, for a 24k system, the following chart shows the absolute memory locations of the CCP, BDOS and CBIOS:

|Start of Memory|Description|
| :---: | :--- |
 | |
|2D00H| CCP (command control processor)|
|3500H| BDOS = CCP + 800H|
|4300H| CBIOS = BDOS + E00H|

Note: the CP/M manual talks about the CCP being loaded at 3400H instead of 2D00H as used by Morrow and the Exidy Sorcerer. This would lead to a BIOS being limited to 1536 bytes compared to the current 3328 bytes for the Exidy Sorcerer. 

## BDOS File Changes

Some changes have been made to the stock BDOS22.MAC file by myself while trying to make that file bit-level identical to the original 1983 SYSGEN images. I reverse engineered & compared the old/new images to find out where and why there were some small discrepancies. These are marked by 'RCL9' in the file.

- The 48 byte stack had been relocated from just-before 'USRCOD' to 70H bytes before the end of memory. That would be CF90H for a 52k system. 

- CONBRK - compare against SPACE rather than Control-C

- Backspace vs. Control-08 change for a better user experience 

- The official DRI "Patch #1' for the blocking/deblocking algorithm usage at 'DSKW09'

- Some other minor changes

## The Recompilation Process

- The recompilation process outlined herein assumes that you have a CP/M runtime environment set up and available. For this exercise we will be using the Yaze-AG emulator running CP/M 2.2 on Microsoft Windows. I have been very happy with using Yaze-AG to run CP/M except that it can run out of memory if you mount too many disk images. 

- The following discussion will assume that the '.yazerc-z3plus-cpm3' start-up disk assignment looks like the following, whereby we'll be placing the source code on Drive F0:
  
  ```
    mount a disks_rcl_z3plus/rcl_boot_disk.ydsk
    mount b disks_rcl_z3plus/rcl_cpm3_sys.ydsk
    mount c disks_rcl_z3plus/rcl_utils.ydsk
    mount d disks_rcl_z3plus/rcl_cypher_src.ydsk
    mount e disks_rcl_z3plus/rcl_big_utils_collection.ydsk
    mount f disks_rcl_z3plus/rcl_scratch.ydsk
    go
  ```

- The source directory also contains the necessary 8080 and Z80 compilers and linkers. So, you do not need to fish them out of your utility collection. 

- Start-up the Yaze-AG CP/M emulator and change to drive F0: which will need to be large enough to accept all of the source files + resulting HEX and related output files. 

- In the following explanation the files to be copied to/from the Windows environment will be first copied to the "tmp" sub-directory residing within the Yaze-AG home directory folder.

- Copy the files from the GitHub [Src](/Src) directory to the "tmp" directory on your Windows machine which has been made available to Yaze-AG. Then copy them into the CP/M virtual drive F0: via the Yaze-AG CP/M command "a:r tmp/*.*"

- Read over the info at the top of the submit *rcl-cpm2.sub* file for general reference. 

- While on drive F0: execute the command "*A:submit rcl-cpm2.sub*" on the CP/M command line. That will compile, link and create .HEX files for the source files of ABOOT.ASM, BDOS22.MAC, ZCCP12.MAC and CBIOS2.MAC. The CP/M batch file will copy these HEX files back over to the Windows "tmp" sub-directory in the Yaze-AG main directory.
  
- On the Windows side of things, copy the 4 resulting HEX files into the root directory containing my Hex-File-Overlay-Tool.

- Copy "Sorcerer_zcpm52k-9.bin" from the GitHub Sysgen_images directory to the Hex-File-Overlay-Tool root directory. You can also use Sorcerer_cpm48k.bin if you desire as it won't make a lot of difference to the final outcome. 

- On a DOS command line, execute the Hex-File-Overlay-Tool in a manner similar to the following:

```
hex_file_overlay_of_sysgen_image.exe -a -c -d -b Sorcerer_zcpm52k-9.bin new_sysgen_image.bin
```

- You can also override the command line arguments to the Hex-File-Overlay-Tool to provide different filenames for the Boot, CCP, BDOS and/or CBIOS source .hex files. In addition, you can leave out some of the command line arguments to prevent some of these HEX files from being used and hence overlaid on top of the original SYSGEN image. 

- Thereafter you can copy the resulting *new_sysgen_image.bin* to your CP/M machine and run Sysgen.com to copy the image to the boot tracks of a floppy diskette.


## Cursory Notes

Floppy systems diskette (drive A:) has to have 1024 byte sectors in order for the cold and warm boot loaders to work.  Be sure to format all new system diskettes with 1024 byte sectors.  The system diskette can be either single or double sided.  The sector size on normal (non A: drive) diskettes is not restricted.  Thus if you have  a diskette with software that is supposed to run on the A: drive then you should mount the diskette in the B: drive and then PIP it over to a 1024 byte sector system diskette.		

## See Also

[Hex File Overlayer Utility for CP/M 2.2 SYSGEN Image Files](https://github.com/rcl9/Hex-File-Overlayer-of-CPM-Sysgen-Image)
