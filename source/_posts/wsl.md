---
title: Windows Subsystem for Linux (WSL)
date: 2020-09-06 11:15:47
categories: Windows
tags: WSL
---

[WSL](https://docs.microsoft.com/en-us/windows/wsl/about) allow to run a GNU/Linux environment on windows without virtual machine/dualboot setup.

Once enabled, can run Linux CLI as a Windows app:

1. search in Windows start menu `Turn Windows feature on or off`, check `Windows Subsystem for Linux`, reboot computer.

2. search for Linux in Microsoft Store, can now download as Windows apps

    - e.g. download Ubuntu CLI as windows app --> opening the app launches the bash shell command line window.

Install homebrew on WSL:

1. install as Linuxbrew:
    - `sh -c "$(curl -fsSL [https://raw.githubusercontent.com/Linuxbrew install/master/install.sh](https://raw.githubusercontent.com/Linuxbrew install/master/install.sh))"`

2. add Linuxbrew to PATH:
    > **Note**: hoembrew on MacOS will install packages to `/user/local/bin`, Linuxbrew will install packages to `/home/linuxbrew/.linuxbrew/bin`(thus need to add this to PATH)

    - add to PATH:
        - `vi ~/.bashrc`
        - `export PATH=$PATH:/home/linuxbrew/.linuxbrew/bin`
