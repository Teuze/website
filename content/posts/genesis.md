+++
date = "2019-05-10"
description = "A memo on system install workflow."
language = "en-us"
title = "Automatic System Genesis"
slug = "genesis"
+++

## Introduction

### Software and hardware prerequisites

### Specifications and deliverables

### System genesis overview

## 1. Bootstrapping

Bootstrapping is the first stage of a system install.<br/>
Traditionally done using a bootable medium, it consists of the following steps.

 - Language, keyboard and locale setup
 - Date, time and timezone setup
 - Network interfaces setup
 - Disk partitionning
 - System install

Bootstrapping can usually be automated using preseeding files,
but the preseeding file format depends on the target environment.

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

You can find on the [official Debian install guide][6] a note on preseeding.
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

### 1.4 FreeBSD, NetBSD, and OpenBSD
### 1.5 Other operating systems

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
