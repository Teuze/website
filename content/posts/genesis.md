+++
date = "2019-05-10"
description = "A memo on system install workflow."
language = "en-us"
title = "Automatic System Genesis"
slug = "genesis"
+++

## Introduction

Operating systems for personal computers have drastically evolved over the past few decades.
They became modular, thread-aware, architecture-agnostic, capable of handling graphics and virtual environments as well as networking, and much more !

They also branched and forked into thousands of flavors, creating a plethora of compatibility problems. One of these problems is the system installation process.

While relying on different installation tools, all systems follow the same logical steps :

 - **Bootstrapping** : Using a system to install another one
 - **Provisionning** : Customizing your freshly installed OS
 - **Staging** : Checking your system's behavior and compliance
 - **Shipping** : Copy your sytem in the target machine(s)

In this article we'll try to get a high-level solution for this install problem.
The desired solution should be **system-agnostic**, **automatic**, **modular** and **scalable**.
In other words, any distro with the adequate configuration file should be able to install on any
compatible hardware target(s) simultaneously, indempotently and without any human intervention.<br/>

An Infrastructure Architect's dream !

## 1. Bootstrapping

Bootstrapping is the first stage of a system install.<br/>
Traditionally done using a bootable medium, it consists of the following steps.

 - Product registration (if applicable)
 - Language, keyboard and locale setup
 - Date, time and timezone setup
 - Network interfaces setup
 - Disk partitionning
 - System install

Bootstrapping can usually be automated using preseeding files,
but the preseeding file format depends on the target environment.
The preseeding file path needs to be explicitly given to the bootstrapping program,
or must correspond to the default one.

### 1.1 Windows environments

Windows environments from Windows XP onwards can be preseeded.

On Windows XP, the file was called `winnt.sif` and followed a simple ini structure.
Under different sections were only specified arguments to the install procedure.

```ini
[Data]
Autopartition=1
UnattendedInstall=Yes
AutomaticUpdates=Yes

[Unattended]
UnattendMode=FullUnattended
OemSkipEula=Yes
FileSystem=NTFS
Repartition=Yes

[...]

[UserData]
ProductKey=
ComputerName=
FullName=
OrgName=
```

Since Windows 7, the file is called `Autounattend.xml` and is a little bit more complex.
There are four phases in the install procedure, and XML verbosity make them untrivial to understand.
Thankfully, the whole file is doumented on [Microsoft Docs][1], and you can also use [Windows AFG][2]
(online) as well as [WADK][3], the Windows Assessment and Deployment Kit to help you create it.

For the sake of comparison, here is what this XML file looks like :

```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
<settings pass="windowsPE">
<component name="Microsoft-Windows-Setup" processorArchitecture="x86" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<Diagnostics><OptIn>false</OptIn></Diagnostics>
<DiskConfiguration>
<WillShowUI>OnError</WillShowUI><Disk wcm:action="add"><DiskID>0</DiskID>
<WillWipeDisk>true</WillWipeDisk><CreatePartitions>
<CreatePartition wcm:action="add"><Order>1</Order><Type>Primary</Type><Size>100</Size>
</CreatePartition><CreatePartition wcm:action="add"><Order>2</Order><Type>Primary</Type>
<Extend>true</Extend></CreatePartition></CreatePartitions>
<...>
</component>
<...>
</settings>
```

`Autounattend.xml` is automatically used when it's on the root folder of the install media.
Otherwise, it's apparently possible to pass it as an argument from `setup.exe`, but I personally
never tried it, so you might want to look for a tutorial focusing specifically on this.

[1]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/automate-windows-setup
[2]: https://windowsafg.com
[3]: https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install

### 1.2 Debian and derivatives

Preseeding on Debian and derivatives is done using **preseed files**.
Those preseed files leverage `debian-install` commands and act as answers in the install process.
They therefore allow any type of configuration you could have with the graphical installer.
Please note however that some options are non-trivial to find and to apply.
You might have some testing to do !

[Here][4] is a preseed example from Debian. <br/>
[Here][5] is a preseed example from Ubuntu. <br/>
Almost identical, aren't they ?

Preseed files need to be explicitly given to the bootloader (`grub` or `isolinux` mostly).
You can find on the [official Debian install guide][6] a chapter on preseeding.
<!-- I also have a [GitHub repository][7] about unattended Ubuntu Server 18.04LTS ISO creation,
check it out ! -->

[4]: https://www.debian.org/releases/stable/example-preseed.txt
[5]: https://help.ubuntu.com/lts/installation-guide/example-preseed.txt
[6]: https://www.debian.org/releases/stretch/amd64/apb.html
[7]: https://github.com/teuze/isomaker

### 1.3 Fedora and derivatives

Preseeding on Fedora and derivatives (RHEL, CentOS et al.) is done using **kickstart profiles**.
Kickstart profiles follow a rather basic structure with commands for each bootstrapping step
(`network`, `lang`, `keyboard`, `timezone`, `part` and so on).

```shell
# OEL7 kickstart baseline installation file
# Simplified, stripped-out version from Bendler's Gist
# Author:       Thomas Bendler <project@bendler-net.de>
# Date:         Fri Feb  5 12:12:46 CET 2016

# Generic installation options
cdrom
text
firstboot --disable
logging --level=info
reboot --eject

# Security settings
auth --enableshadow --passalgo=sha512
firewall --enabled --service=ssh
selinux --permissive

# Network settings
network  --bootproto=dhcp --device=enp0s3 --ipv6=auto --activate
network  --hostname=localhost.localdomain

# Disc settings
zerombr
clearpart --all --initlabel
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
autopart --type=lvm
ignoredisk --only-use=sda

# User settings (password: hamburg1)
rootpw --iscrypted $6$WYx8jpVu/jAyvXEl$n2aYNqFCFvMWebHVjS.MrDbheQ7AE4zOpfZnWpXq0tnT43MSnMIDzANW8MqltNHfbRaebLlMPodfdObwTbh5g/
user --groups=wheel --homedir=/home/sysdeploy --name=sysdeploy --password=$6$CmofRJck/r0rcYck$De8OqS.OqrFS3BDpnmQsE88aj93KZZtcTdlniXpeGr4HRPUMr9Vl8zBml3JxrabBJiY4a1LGcEv6PSo5bfIwI1 --iscrypted --gecos="System Deploy"

# Package group and packages that will be installed
%packages
@core
%end

# Final sync
echo "Sync and finalize kickstart installation"
sync
exit 0
%end
```

**Note** : Ubuntu 18.04LTS and up seem to partially support kickstart files, which is good news considering how easier it is to preseed that way ! 

### 1.4 Other operating systems

I should further investigate the preseeding possibilities for ArchLinux and BSD systems, it's a longer-term goal. As I understand, Arch has built-in support for shell-scripted bootstrapping, but I definitely need to dig a little deeper.


## 2. Provisionning

### 2.1 Using shell scripts
### 2.2 Infrastructure as code

## 3. Staging

### 3.1 Physical Machines
### 3.2 Virtual Machines
### 3.3 Containers

## 4. Shipping

### 4.1 Bundle burdens
### 4.2 Transcendence

## Conclusion
