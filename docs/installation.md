---
title: Environment set up
nav_order: 2
---

## SDDP development environment set up
This guide was created to teach how to install and compile the SDDP project for the first time.

### Prerequisites

- Accounts (with permission to clone SDDP)
  - Github account
  - BitBucket account

- Files
  - Working SmartGit license for development
  - Xpress working license (xpauth.psr)
  
- Programs
  - SmartGit
  
- Basic functionality such as: make and rm

### Step by step set up procedure:

- Installation:
    - Install Microsoft Visual Studio Community 2022
    - Install Intel oneAPI 2022.1 BaseKit (last know to work with SDDP)
    - Install Intel oneAPI 2022.1 HPCKit (last know to work with SDDP)
    - Intel Installer: http://psr.me/cmsw89
    - Preferable to set up both Github and BitBucket accounts as hosting providers inside SmartGit
    - ‘make’ dir in ‘C:\Sys ‘ path: http://psr.me/1trrgd1
- Steps:
    - Get the MSVC version 14.30.30705 inside the folder: “C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC”.
        - If desirable, it should be possible to change the compiling version inside the SDDP make_userconfig. The installed versions are in the path mentioned before.
    - Check if intel oneAPI installed the compiler version correctly by checking the environment variables “IFORT_COMPILER21” is properly defined.
    - Clone SDDP using SmartGit and choose the SDDP project.
    - Copy the Xpress working license to the respective folders:
        - “./sddp/ncplite/xpressmp/win64/bin/”
        - “./sddp/thirdparty/xpressmp/bin/win64/”
    - Open the Microsoft Visual Studio Community 2022 and open the project “./sddp/sddp.sln”
    - Change the build configuration to Debug | x64 and check if the “sddp” project in the Solution Explorer is set as default project.
    - Build the project (Ctrl+Shift+B).
    - Run the project (F5).
    Obs: The default path is set to the example case. To change the case path, you will need to add a file called path.dat and have the structure below and substitute the string “$PATH$” by the correct path to the case folder.