# ntv2switchbitfile - Introduction

ntv2switchbitfile is a command-line utility for dynamically switching the firmware “personality” of NTV2 boards that support Xilinx “Partial Reconfig”, without re-flashing the board or power-cycling the machine. The boards capable of dynamic bitfile switching are the KONA 5 and Corvid44-12G.

Typically, NTV2 boards are flashed with a full bitfile using ntv2firmwareinstaller. Firmware for all boards is available from the [ntv2-firmware](https://github.com/aja-video/ntv2-firmware) repository. For boards capable of dynamic bitfile switching, the board's firmware directory contains two subdirectories:

- **tprom** — the traditional full bitfiles, flashed with ntv2firmwareinstaller. Flashing a TPROM bitfile requires a power-cycle after installation.
- **reconfig** — partial and clear bitfiles used by ntv2switchbitfile to perform so-called “fast bitfile switching”. These must be installed with the ntv2switchbitfile utility.

**NOTE: Firmware personality changes made with ntv2switchbitfile are not persistent — after a reboot or power-cycle, the board reverts to the originally-flashed (TPROM) bitfile.**

# Dynamic Bitfile Switching HOW-TO

Before using ntv2switchbitfile, two prerequisites must be met:

- The board must be flashed with a TPROM file (i.e. the main bitfile) from the same set of firmware files you plan to use. The easiest way to ensure this is to download your board's entire firmware directory from the ntv2-firmware repository.
- ntv2switchbitfile must be run from a directory containing both the partial and clear bitfiles you are switching to (i.e. the “reconfig” directory).

The steps below use the Corvid44-12G as an example; substitute the directory for your own board.

1. Download all of the firmware files for your board, e.g. https://github.com/aja-video/ntv2-firmware/tree/github-mirror/corvid44-12g.
Either clone the entire repository with git (very large — not advised over slow connections), perform a sparse checkout of just your board's directory, or download the files by hand from the GitHub website as “raw” downloads.
If cloning with git, make sure git-lfs is installed on your system, as the files in the repository are stored with Git Large File Storage (LFS).

2. Flash one of the bitfiles from the “tprom” directory to your board with ntv2firmwareinstaller, then power-cycle the machine after a successful flash.

3. Change to the corvid44-12g/reconfig directory of the ntv2-firmware checkout and run ntv2switchbitfile from there.
Running ntv2switchbitfile with the --info argument lists the firmware personalities available for the selected board.
In the example below the board is at device index 1, and the current firmware is the 2x4K personality:
```
firmware/corvid44-12g/reconfig$ ../../../build/libajantv2/demos/ntv2switchbitfile/ntv2switchbitfile -d1 -i
Active device is Corvid44-12G-2x4K
Corvid44-12G-2x4K    DevID 0x10832402  DesID 0x02  DesVer 0x17  BitID 0x02  BitVer 0x12  Active
Corvid44-12G-2x4K    DevID 0x10832402  DesID 0x02  DesVer 0x17  BitID 0x02  BitVer 0x12  Partial
Corvid44-12G-2x4K    DevID 0x10832402  DesID 0x02  DesVer 0x17  BitID 0x02  BitVer 0x12  Clear
Corvid44-12G-8KMK    DevID 0x10832400  DesID 0x02  DesVer 0x17  BitID 0x00  BitVer 0x20  Partial
Corvid44-12G-8KMK    DevID 0x10832400  DesID 0x02  DesVer 0x17  BitID 0x00  BitVer 0x20  Clear
Corvid44-12G-8K      DevID 0x10832401  DesID 0x02  DesVer 0x17  BitID 0x01  BitVer 0x19  Partial
Corvid44-12G-8K      DevID 0x10832401  DesID 0x02  DesVer 0x17  BitID 0x01  BitVer 0x19  Clear
Corvid44-12G-PLNR    DevID 0x10832403  DesID 0x02  DesVer 0x17  BitID 0x03  BitVer 0x14  Partial
Corvid44-12G-PLNR    DevID 0x10832403  DesID 0x02  DesVer 0x17  BitID 0x03  BitVer 0x14  Clear
```

4. Run ntv2switchbitfile with the --load argument (no index) to list the firmware personalities and their associated indices for switching:
```
firmware/corvid44-12g/reconfig$ ../../../build/libajantv2/demos/ntv2switchbitfile/ntv2switchbitfile -d1 -l
Active device is Corvid44-12G-2x4K
4 Device(s) for dynamic loading:
 1: Corvid44-12G-8KMK
 2: Corvid44-12G-8K
 3: Corvid44-12G-2x4K
 4: Corvid44-12G-PLNR
```

5. Run ntv2switchbitfile with the --load argument again, this time specifying the index of the desired firmware personality:
```
firmware/corvid44-12g/reconfig$ ../../../build/libajantv2/demos/ntv2switchbitfile/ntv2switchbitfile -d1 -l1
Active device is Corvid44-12G-2x4K
Can load device: Corvid44-12G-8KMK
Device: Corvid44-12G-8KMK loaded successfully
```

At this point, the selected firmware personality is loaded. Running ntv2switchbitfile again with the --info argument should reflect the change — in this example, the board has switched to the 8KMK personality:
```
firmware/corvid44-12g/reconfig$ ../../../build/libajantv2/demos/ntv2switchbitfile/ntv2switchbitfile -d1 -i
Active device is Corvid44-12G-8KMK
```
