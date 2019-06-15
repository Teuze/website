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
 - **Shipping** : Copy your sytem into the target machine(s)

In this article we'll try to get a high-level solution for this install problem.
The desired solution should be **system-agnostic**, **automatic**, **modular** and **scalable**.
In other words, any distro with the adequate configuration file should be able to install on any
target machine(s).

An Infrastructure Architect's dream ! </br>
Spoiler alert : it's not straightforward.

</br>

## 1. Bootstrapping

Bootstrapping is the first stage of a system install.
Traditionally done using a bootable medium (drive or network card),
it consists of the following steps.

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

Bootstrapping can be automated using [Packer][B1] from HashiCorp. <br/>
You can find good tutorials here and there.

I also happen to have a [repo][B2] about debian bootstrapping using Packer.

[B1]: https://packer.io/
[B2]: https://github.com/teuze/prologue

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
Thankfully, the whole file is doumented on [Microsoft Docs][BW1], and you can also use [Windows AFG][BW2]
(online) as well as [WADK][BW3], the Windows Assessment and Deployment Kit to help you create it.

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

[BW1]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/automate-windows-setup
[BW2]: https://windowsafg.com
[BW3]: https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install

### 1.2 Debian and derivatives

Preseeding on Debian and derivatives is done using **preseed files**.
Those preseed files leverage `debian-install` commands and act as answers in the install process.
They therefore allow any type of configuration you could have with the graphical installer.
Please note however that some options are non-trivial to find and to apply.
You might have some testing to do !

[Here][BD1] is a preseed example from Debian. <br/>
[Here][BD2] is a preseed example from Ubuntu. <br/>
Almost identical, aren't they ?

Preseed files need to be explicitly given to the bootloader (`grub` or `isolinux` mostly).
You can find on the [official Debian install guide][BD3] a chapter on preseeding.
<!-- I also have a [GitHub repository][7] about unattended Ubuntu Server 18.04LTS ISO creation,
check it out ! -->

[BD1]: https://www.debian.org/releases/stable/example-preseed.txt
[BD2]: https://help.ubuntu.com/lts/installation-guide/example-preseed.txt
[BD3]: https://www.debian.org/releases/stretch/amd64/apb.html
[BD4]: https://github.com/teuze/isomaker

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

**Note** : Ubuntu distros seem to partially support kickstart files starting since 18.04LTS, which is good news considering how easier it is to preseed that way ! 

### 1.4 Other operating systems

I should further investigate the preseeding possibilities for ArchLinux and BSD systems, it's a longer-term goal. As I understand, Arch has built-in support for shell-scripted bootstrapping, but I definitely need to dig a little deeper.

</br>

## 2. Provisionning

Provisionning is the second stage of the install process and consists in two steps. First, supplemental content such as packages, updates, and add-ons are installed or copied onto the system. Then configuration files are adjusted if necessary in order to adjust to security and usage requirements.

At this point, system scripting tools have become available, so it's easier and quicker to develop and test provisonning routines than it was to create preseeding files. However, system-specific scripts can still be a hassle to maintain, and you have to develop a new set of scripts for each new environment.

This is why orchestration tools are popular : they describe system configuration in a static and system-agnostic file, turning infrastructure into human-readable code. Without that you will have to rely heavily on your local package managment system.

### 2.1 Provisonning scripts

#### 2.1.1 A UNIX shell example

```shell
#!/bin/bash
# Simple provionning script for a featherlight graphical desktop
# Needs to be launched with administrative priviledges
# Author : Richard Jarry
# Licensed under WTFPL

set -euo pipefail

# 1. Software to install (by theme)

CORE="bspwm sxhkd lemonbar dmenu"
GUI="mupdf feh xterm"
CLI="fish git vim stow tmux"

apt update
apt install $CORE $GUI $CLI

# 2. Configuration files for custom desktop and shell environment

git clone https://github.com/teuze/private-repo.git $HOME/.dotfiles

cd .dotfiles

stow desktop
stow fish
stow tmux
stow vim
```
By the way if you want to build your own desktop environment on Linux or Mac,
I highly recommend the [unixporn channel on reddit](https://reddit.com/r/unixporn/).
Oh, and I use [GNU Stow](https://www.gnu.org/software/stow/) in this script in order
to track symbolic links and keep my `$HOME` tidy (**#MariKondo**).

#### 2.1.2 On Windows

Tweaking the environment on Windows doesn't look easy.
Instead of changing system values and breaking stuff I really have no clue about,
I decided to install additional software providing a higher level of comfort and
usability to the platform. I chose :

 - [Chocolatey][P1] as a package manager
 - [Rainmeter][P2] for Desktop customization
 - [AutoHotkeys][P3] for keyboard shortcuts 
 - [Clover][P4] to (finally) get tabs on the Explorer

[P1]: https://chocolatey.org/
[P2]: https://www.rainmeter.net/
[P3]: https://www.autohotkey.com/
[P4]: http://en.ejie.me/

I also use Powershell and WSL (Linux on Windows) to script things when I need to.</br>
Below, a Powershell script to install and put on the `PATH` Chocolatey.

```powershell
$chocoExePath = 'C:\ProgramData\Chocolatey\bin'

if ($($env:Path).ToLower().Contains($($chocoExePath).ToLower())) {
  echo "Chocolatey found in PATH, skipping install..."
  Exit
}

# Add to system PATH
$systemPath = [Environment]::GetEnvironmentVariable('Path',[System.EnvironmentVariableTarget]::Machine)
$systemPath += ';' + $chocoExePath
[Environment]::SetEnvironmentVariable("PATH", $systemPath, [System.EnvironmentVariableTarget]::Machine)

# Update local process' path
$userPath = [Environment]::GetEnvironmentVariable('Path',[System.EnvironmentVariableTarget]::User)
if($userPath) {
  $env:Path = $systemPath + ";" + $userPath
} else {
  $env:Path = $systemPath
}

# Run the installer
iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))

```

### 2.2 Orchestration tools

The infrastructure's prodigal sons.

**Upsides**

 - Orchestration tools are distribution-agnostic up to a certain limit
 - They are also concurrent (you can configure several machines at once)
 - Correctly implemented, they always the deliver the same (accurate) result

**Downside**

Orchestration tools tend to only fulfill the specific use-case where you configure
and manage machines remotely. Notable exceptions are [Ansible][O1] and [SaltStack][O2] (but
SaltStack is a meanie because you need to register on their website to get their software).

Also, some orchestrators are agentless, while others are not. </br>
For example, [Puppet][O3] and [Chef][O4] need some software to be installed on the target machines
before you can start configuring them, while [cdist][O5] only needs SSH access and a shell.

 [O1]: https://www.ansible.com/
 [O2]: https://www.saltstack.com/
 [O3]: https://puppet.com/
 [O4]: https://www.chef.io/
 [O5]: https://www.cdi.st/

</br>

## 3. Staging

Staging is the third step in the install process and is very often overlooked.
It consists in checking everything is installed correctly and runs as expected.
Additionally, it can include manual steps that cannot be easily reproduced,
such as direct user interface interaction for example.

The only way to make this kind of manipulations reproducible is to make a snapshot of your system.
On physical machines, this means copying your disk one-on-one and compressing
the resulting file to avoid filling up too much space. On virtual machines,
this is made easier thanks to snapshotting utils, extensible virtual disks,
and bundles such as OVA Files and Vagrant Boxes.
In containers, you can directly stage your system by running it with a persistent volume mount.

Once your snapshot is ready, put it on a server. This can be your local backup system, Vagrant Up, Docker Hub, or really any cloud service suiting your needs.

### 3.1 Physical Machines

```shell
# For saving entire disks (with boot sector)
dd if=/dev/sdX status=progress | gzip -c > systemname-version-arch-date.img.gz 

# For saving only a specific partition 
dd if=/dev/sdXN status=progress | gzip -c > systemname-version-arch-date.img.gz 

# Save the snpashot checksum for future use
sha256sum systemname-version-arch-date.gz >> SHA256SUMS

# Backup using rsync, borg, scp, or wadevz
```

**Note** : It is possible to adjust `dd` options to make the snapshot less error-prone, but it also makes it slower. Check out the blocksize options in the man pages.
Also, dismount before copy, or you will get copy errors for sure.

### 3.2 Virtual Machines

On [VirtualBox][S1], staging is quite straightforward. You can list and edit snapshots
from the command-line, and you can also export the whole machine as an OVA bundle.

```shell
# Managing VM snapshots
VBoxManage snapshot list
VBoxManage snapshot take snapname
VBoxManage snapshot restore snapname
VBoxManage snapshot delete snapname

# Export your VM (three possible formats)
VBoxManage export machine-name exportname.ova
VBoxManage export machine-name exportname.ovf
VBoxManage export machine-name exportname.tar.gz
```

On [QEMU/KVM][S2], since virtual drives and virtual machines are treated separately,
this looked just a little bit more complicated. Hopefully there will be soon
a tool allowing for automatic conversion (maybe there already is?)

```shell
# 1. Convert your native QEMU disk file to VDI or VMDK format
qemu-img convert -O vdi input.qcow2 output.vdi
qemu-img convert -O vmdk input.qcow2 output.vmdk

# 2. Get the machine settings and put it on an OVF
virsh dumpxml machinename output.xml
vim output.xml # Change disk path if necessary
mv output.xml output.ovf

# 3. Bundle the lot as an archive
tar cvzf machinename.tar.gz ./*
mv machinename.tar.gz machinename.ova
```
Lastly you can also use [Hashicorp Vagrant][S3] to manage your machines.
It supports a lot of formats including cloud images which sounds pretty awesome !

[S1]: https://www.virtualbox.org/
[S2]: https://www.qemu.org/
[S3]: https://www.vagrantup.com/

### 3.3 Containers

Staging on Docker is pretty easy with Dockerfiles.</br>
Just read my [previous article](../docker) lulz

</br>

## 4. Shipping

Now that we have a virtual or physical disk ready to ship, we just need to bitwise-copy
it on our target. You first need to plug the target hard drive(s) somewhere visible so that
you can issue your final command (here, from an OVA file) : 

```shell
tar xvf machinename.ova # Should give you machinename in VDI or VMDK
VBoxManage clonehd machinename.vdi machinename.img --format RAW
dd if=machinename.img of=/dev/sdX # replace X with target drive 
rm *.{vdi,ovf,img} # cleanup
```
</br>


## Conclusion

Automation sounds like a paradox because it requires a massive time investment.
In our case, I'd say this is worth the time spent, because our jobs will likely
make us deploy and reinstall systems quite frequently.

It's also a way to reduce electronical waste, because it allows to repurpose old
or infected computers with a lightweight distribution of your choice.
