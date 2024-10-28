## Installing the ArchLinux .iso
---
The first step in creating the ArchLinux vm is to navigate to the ArchLinux wiki page, which can be found by following this [link](https://wiki.archlinux.org/title/Main_page), from there navigate over to the [installation guide](https://wiki.archlinux.org/title/Installation_guide) and install the .iso file via HTTP under one of the mirror sites. I used the 0xem.ca site under the Canada tab.

![A screenshot showcasing the Canada mirror sites available to download from](https://github.com/PcKaPnCh/PcKaPnCh.github.io/blob/main/Screenshot%202024-10-20%20222537.png)

## Verifying the Image Signature
___
Next, install the latest full version of GnuPG from their [website](https://gnupg.org/download/), then proceed to select the Gpg4win download version, if you use a windows OS but make sure to select the correct version for your working OS. From there, download the PGP signature, which can be found at the bottom of this section. Then open up the windows terminal and using the command ```gpg --keyserver-options auto-key-retrieve --verify .\Downloads\archlinux-2024.10.01-x86_64.iso.sig .\Downloads\archlinux-2024.10.01-x86_64.iso ```  to verify the file's signature and make sure the image signature is correct, which it was for me.

> [!NOTE] 
> Make sure to install the GnuPG installer (.exe file) and not the source code, which is at the very top, make sure to scroll down and install the .exe file, not the files at the top of the page.

## Booting up The VM
___
Using VirtualBox as a medium for the VM, add the ArchLinux .iso file onto a new VirtualBox VM, I titled mine "Archtic Ocean". Next allocate system resources for however much RAM, virtual disk space, and cores necessary for the system. For myself, 20 GB for the virtual disk, 2 GB for RAM, and 2 cores was enough for my purposes. After that, simply start the VM and let the machine boot up.

## Basic Setup
___
Verify a connection to the internet by using the ```ping google.com``` command, as long as there are messages being generated and no error messages being generated, your configuration should be alright, if not, refer to 1.7 on the [installation guide](https://wiki.archlinux.org/title/Installation_guide) for ArchLinux. After that, ensure that the system clock is correct by using ```timedatectl```, for me it was correct and I had no further steps to take in regards to ensuring the clock was on time. 

## Verifying the Boot Mode
___
After using the ```cat /sys/firmware/efi/fw_platform_size``` command in the ArchLinux vm, I was returned not with an expected value of 64 or 32, but rather with a 'no such file or directory'. This means the machine is booted in BIOS, which I would like to change to boot using UEFI. To amend this, I firstly enabled EFI usage in the settings for the vm in VirtualBox. After I checked the EFI box, the command returned 64, which means that the vm now boots in UEFI like I would prefer.

![A screenshot showing the available customizable settings for the vm](https://github.com/PcKaPnCh/PcKaPnCh.github.io/blob/main/Screenshot%202024-10-22%20130027.png)

>[!NOTE]
>Enabling EFI can be found in the setup before making the VM, if you would like to have the machine boot in BIOS, leave it unchecked.
## Partition Disks
___
For this part of the project, I used the command ```fdisk /dev/sda``` command in order to make the two partitions necessary for the virtual machine. Firstly, after using the initial gdisk command I typed 'g' to create a new partition table, then I typed 'n' for a new partition, then 'enter' for the default partition number, then 'enter' again for the first sector, then I used ```+500M``` as the amount of memory recommended for the EFI system partition, and then the code '1' after pressing 't' to denote the partition as an EFI system partition. I then repeated this process for the root partition with the only difference being that I made its memory store the rest of the memory.

![A screenshot of the gpt partition table after making an EFI and root system partition](https://github.com/PcKaPnCh/PcKaPnCh.github.io/blob/main/Screenshot%202024-10-24%20134136.png)

After this, I continued to follow the instructions set before me on the ArchLinux wiki, which stated that formatting was the next item on the checklist. I followed their instructions and used ```mkfs.ext4 /dev/sda2``` for the root partitionw and ```mkfs.fat -F 32 /dev/sda1``` for the EFI partition. From there, I was then instructed to mount each of these partitions to their designated spaces. I followed their instructions and used ```mount /dev/sda2 /mnt``` to mount the root partition to /mnt, ```mount --mkdir /dev/sda1 /mnt/boot``` to mount the EFI partition to /mnt/boot.
>[!Warning]
>ArchLinux wiki states that if there was already a pre-existing EFI system partition, DO NOT REFORMAT IT! It can destroy boot loaders!

## Installing Essential Packages
___
I started by using ```pacstrap -K /mnt base linux linux-firmware intel-ucode vim networkmanager man-db man-pages texinfo``` to install the essential kernel packages for the Stable linux kernel.

# Configuration
___
I used ```genfstab -L /mnt >> /mnt/etc/fstab``` to create an fstab file. Then I used the ```arch-chroot /mnt``` command to change the root into the new system. After that, it was time to configure the time for the machine which required using ```ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime``` to configure the time and then running ```hwclock --systohc```. Then I ran ```locale-gen``` and ```echo "LANG=en_US.UTF-8 > /etc/locale.conf```. After that,  I used ```echo '*hostname*' > /etc/hostname``` for network configuration and used ```pacman -Sy xorg-server```. I then also used ```passwd``` to reset the password for the root user. After that was the boot loader. For this project, I used GRUB. Firstly, I used ```pacman -Sy grub efibootmgr``` to install the packages necessary for the bootloader. Then, I navigated to /boot, which is where I mounted my EFI system partition, and created the directory ```EFI```, then I used ```grub-install --target=x86_64-efi --efi-directory=boot --bootloader-id=GRUB``` to install GRUB and make the main GRUB directory.
>[!Warning]
>Follow every instruction given above, if actions taken above were incomplete, you will have to start over, make sure EVERYTHING is completed before you `reboot` 
# Post Installation:
___
Firstly, I logged into my root L0G1N account using the associated login and password for the account. Then I started on the DE (desktop environment) by first using ```pacman -Syu``` to check for available updates for packages and automatically upgrade them before continuing to install others. I used GNOME as my chosen desktop environment, so I started by using ```pacman -Sy gnome``` to install the necessary gnome packages, then I had to enable and start it upon startup. This task required me to use ```systemctl enable gdm.service``` and ```systemctl start gdm.service``` in order to achieve that. After the last command, the desktop environment started immediately.

>[!Note]
>For the login field, use root as the input, and THEN use your root password to login, that took a while to figure that one out

## Creating New Users
___
To first create the three new users (justin, codi, and my personal account), I had to use the ```useradd -md /home/*username* *username*``` to create the accounts, set the locations of their home directories, and create their directories. Then I set the passwords of account with ```passwd *username*``` to "GraceHopper1906" and then expired the password.  From there, I created the sudo group and added each user to the group with ```usermod -aG sudo *username*```. Then I used ```vim /etc/sudoers``` to edit the sudo group permissions. From there, I uncommented the line involving sudo group permissions and saved the file. Then I used ```pacman -Sy zsh``` to install the zsh shell. Then I ran the ```zsh``` command and installed the framework for zsh through wget and ```sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"```. Afterwards, I installed ssh using ```pacman -Sy openssh```. Due to my previous installation of zsh, I also had .zshrc installed as well, which I have been using to change the format of the shell. Finally, in order to customize the shell a bit more, I edited the .zshrc file to change the theme to "random" so that the theme is never stagnant.

>[!NOTE] 
>There are other ways of installing the framework, but installing wget and oh-my-zsh was easiest 
## Customization
___
	
For some customization, I used ```vim ~/.zshrc``` as a method of editing the file, from there I edited the theme line and make it completely random every time I logged in. I then changed some settings such as aliases in my personal account like using ```c``` as the alias for ```clear```. I also used the same method for using ```ll``` as the alias for ```ls -la```.  I also added one of my favorite packages as well ```sl```, which spawns a steam locomotive that scrolls across the screen. I also used ```pacman -Sy firefox``` to install the Firefox browser.
# Extras
___
>[!Note] 
>I had trouble getting the ls and some other commands to give a proper output, so I reinstalled coreutils package using `pacman -S coreutils --overwrite '*'` to only reinstall without uninstalling coreutils which restored the commands to their rightful place

>[!Note] 
>There was a bit of trouble with the sudo permissions at first, but after reinstalling coreutils, I realized that the problem was with the coreutils package rather than the sudo permissions setup. IF THE COMMAND DID NOT OUTPUT CORRECT FIRST CHECK ALIASES!

