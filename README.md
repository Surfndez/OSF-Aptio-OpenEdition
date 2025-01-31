# ** Aptio OpenEdition for TiogaPass **

The Minimum Platform is a software architecture that guides uniform delivery of Intel platforms enabling firmware solutions for basic boot functionality with extensibility built-in.  This project incorporates support for the OCP derived Tioga Pass platform.

Package maintainers are listed in Maintainers.txt.

## Overview
The key elements of the architecture are organized into a staged boot approach where each stage has requirements and
functionality for specific use cases. The generic control flow through the boot process is implemented in the
[`MinPlatformPkg`](https://github.com/tianocore/edk2-platforms/tree/devel-MinPlatform/Platform/Intel/MinPlatformPkg).
The generic nature of the tasks performed in MinPlatformPkg lends to reuse across all Intel platforms with no
source modification. Details for any particular board are made accessible to the MinPlatformPkg through a well-defined
statically linked board API. A complete platform solution then consists of the MinPlatformPkg and a compatible board
package.

## Board Naming Convention
The board packages supported by Intel follow the naming convention \<xxx\>OpenBoardPkg where xxx refers to the
encompassing platform name for a particular platform generation. For example, the [`KabylakeOpenBoardPkg`](https://github.com/tianocore/edk2-platforms/tree/devel-MinPlatform/Platform/Intel/KabylakeOpenBoardPkg) contains the
board code for Intel Kaby Lake reference systems. Intel uses the moniker "OpenBoardPkg" to indicate that this package
is the open source board code. A closed source counterpart may exist which simply uses "BoardPkg". Both directly use
the MinPlatformPkg from edk2-platforms.

## Stage Selection
Stage selection is controlled via the PCD `gMinPlatformPkgTokenSpaceGuid.PcdBootStage` in [`MinPlatformPkg.dec`](https://github.com/tianocore/edk2-platforms/blob/devel-MinPlatform/Platform/Intel/MinPlatformPkg/MinPlatformPkg.dec).
The stage should be configured in the board package DSC file to the appropriate value. For example, a board may disable
all advanced features by setting this value to 4 instead of 6. This may be used to improve boot time for a particular
use case. Decrementing the stage can also be used for debug since only the actions required for that stage objective
should be executed. As an example, ACPI initialization is not required for a Stage 3 boot.

The stages are defined as follows:

| Stage  | Functional Objective         | Example Capabilities                                                                               |
| -------|------------------------------|----------------------------------------------------------------------------------------------------|
| I      | Minimal Debug                | Serial port output, source debug enabled, hardware debugger enabled                                |
| II     | Memory Functional            | Basic hardware initialization necessary to reach memory initialization, permanent memory available |
| III    | Boot to UI                   | Simple console input and output to a UI, UEFI shell                                                |
| IV     | Boot to OS                   | Boot an operating system with the minimally required features                                      |
| V      | Security Enable              | UEFI Secure Boot, TCG measured boot, DMA protections                                               |
| VI     | Advanced Feature Enable      | Firmware update, power management, non-essential I/O                                               |

## Minimum Platform Firmware Solution Stack
A UEFI firmware implementation using MinPlatformPkg is constructed using the following pieces.

|                                    |
|------------------------------------|
| [EDK II](https://github.com/tianocore/edk2)                                                                              |
| [Intel(r) FSP](https://github.com/IntelFsp/FSP)                                                                            |
| [Minimum Platform (`MinPlatformPkg`)](https://github.com/tianocore/edk2-platforms/tree/devel-MinPlatform/Platform/Intel/MinPlatformPkg)                        |
| [Board Support (\<xxx\>OpenBoardPkg)](https://github.com/tianocore/edk2-platforms/tree/devel-MinPlatform/Platform/Intel)  |


## Board Support
* The `PurleyOpenBoardPkg` contains board implementations for Purley systems.

## Board Package Organization
The board package follows the standard EDK II package structure with the following additional elements and guidelines:
* Only code usable across more than one board at the root level.* Board-specific code in a directory. The directory name should match that of the board supported.
* Features not essential to achieve stage 5 or earlier boots are maintained in a Features folder at the appropriate
  level in the package hierarchy.

Shared resources in the package root directory can include interfaces described in header files, library instances,
firmware modules, binaries, etc. The UEFI firmware implementation is built using the process described below from the
board-specific directory.

A board package must implement the board APIs defined in the MinPlatformPkg even if a "NULL" implementation is used to
return back to the minimum platform caller.

## **Windows Build Instructions**

### Pre-requisites

* [GIT client](https://git-scm.com/downloads): Available from https://git-scm.com/downloads
* [VS2015 Build Tools from Microsoft Visual Studio 2019.](https://nam12.safelinks.protection.outlook.com/?url=https%3A%2F%2Fvisualstudio.microsoft.com%2Fdownloads%2F%23build-tools-for-visual-studio-2019&data=04%7C01%7Cjoshuas%40ami.com%7Cc5cd5c9371da416b07b308d9101cc01a%7C27e97857e15f486cb58e86c2b3040f93%7C1%7C0%7C637558537940708425%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwiLCJXVCI6Mn0%3D%7C1000&sdata=ZDuDUYUBS10Eoi9niUM6opxVFvrrdLdXbOf4aRBRGeA%3D&reserved=0)
  - Visual Studio 2015 build tools from Visual Studio 2019.  See image below for recommended install configuration.
    - **Please note that the VS2015 Build Tools are not enabled by default in the Visual Studio 2019 install.**
  - Visual Studio 2015 can be used instead.
  ![Visual Studio 2019 Installation](/Readme_VisualStudioInstall.png)
* [ASL compiler](https://www.acpica.org/downloads/binary-tools): iasl.exe available from http://www.acpica.org
  - Install into ```C:\ASL``` to match default tools_def.txt configuration.
  - The validated version of iasl compiler that can build MinPurley is 20180629.
* [NASM assembler](https://www.nasm.us/):  nasm.exe available from: http://www.nasm.us/
  - Install into ```C:\NASM``` to match default tools_def.txt configuration.
* [Python 2.7.6](https://www.python.org/download/releases/2.7.6/):  Available from: https://www.python.org/download/releases/2.7.6/
  - Install into ```C:\Python27``` to match default tools_def.txt configuration.
  - Add C:\Python27 to your path
  - Other versions of 2.7 may also work fine.

  
### Download the required components

 To download the project, clone the repository along with all the submodules and checkout required TAG using the following command:
 git clone --recurse-submodules https://github.com/opencomputeproject/Aptio-OE.git -b (need to be replaced with TAG name)

### Build

**Building with the batch scripts**

For PurleyOpenBoardPkg
1. Open command window, go to the workspace directory, e.g. c:\Purley.
2. Type "cd edk2-platforms\Platform\Intel\PurleyOpenBoardPkg\BoardTiogaPass".
3. Type "GitEdk2MinTiogaPass.bat" to setup GIT environment.
4. Type "bld" to build Purley Tioga Pass board UEFI firmware image, "bld release" for release build, "bld clean" to
   remove intermediate files.
5. Final BIOS image will be Build\PurleyOpenBoardPkg\BoardTiagoPass\DEBUG_VS2015x86\FV\PLATFORM.fd or Build\PurleyOpenBoardPkg\BoardTiagoPass\RELEASE_VS2015x86\FV\PLATFORM.fd, depending on bld batch script input.
6. This BIOS image needs to be merged with SPS FW.

The validated version of iasl compiler that can build MinPurley is 20180629. Older versions may generate ACPI build
errors.

**`Users with access to the Intel proprietary FITC tool and ME ingredients can build full images for flash  (BIOS + ME +
DESC).`**

**`Users can also flash the UEFI firmware image to the highest area of the flash region directly.`**

### **Binary and Reference Code Details**

* [EDK2](https://github.com/tianocore/edk2) source based on commit hash 9abc60f9f722d891b3cedb0205fd6c9c0e2af54b.
* FSP binaries derived from Purley Refresh RC version RC610D02.
* AST2500 UEFI option ROM v1.11.03 (released 12/29/2020).
* [EDK2-Platforms](https://github.com/tianocore/edk2-platforms) source based on commit hash 4b8483b74963b63312a2b08fd1c307cea47307b6.
* [EDK2-Non-OSI](https://github.com/tianocore/edk2-non-osi) source based on commit hash de63002b5de696f6d6ebc91dc80fa8fd6e47c507.
* [FSP](https://github.com/IntelFsp/FSP) source based on commit hash f5ce887fb45279f2b6557107c819e35a12547c87.

### **Validation Details**

* All firmware projects can only build on Windows with the validated configuration below.

**PurleyOpenBoardPkg**
* This firmware project has only been tested on the Tioga Pass hardware.
* This firmware project build has only been tested using the Microsoft Visual Studio 2015 build tools.
* Booted to UEFI shell.
* Installed and booted to UEFI Windows 2016 on M.2 NVME slot.
* Installed and booted to UEFI Windows 2019 on M.2 NVME slot and with SATA HDD.
* Installed and booted to UEFI RHEL 7.3 on SATA HDD.
* Installed and booted to Ubuntu 18.04 on M.2 NVME slot.
* Verified Mellanox card detection during POST and OS.


### **New Features**
* Add Linux boot support

### **Planned Activities**
* Sync with latest EDKII platforms master

### **Additional Support and Customizations**
*	To get dedicated support or additional features or customizations for Aptio OpenEdition, feel free to email sales@ami.com
