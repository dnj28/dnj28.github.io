# Arch Installation Documentation
---
Dylan Jones

Professor West

System Administration: 353-01

8 Nov 2026

# Abstract
---

The purpose of this guide is to document the steps taken to install Arch Linux in a Virtual Machine using VMware Workstation for a Windows device. This guide is in reference to the [Arch Wiki Installation Guide]( https://wiki.archlinux.org/title/Installation_guide). In addition, **problems and solutions** encountered along the way can be referenced at the bottom of this document. 

## Virtual Machine Specifications 
- Virtualization software: VMware workstation
- Operating sytem: Arch Linux
- Chosen Desktop Environment: LXQT
- Chosen Shell: Zsh
- Users: Dylan, Codi

## Virtual Machine Settings:
- RAM: 6GB
- Processors: 2
- Hard Disk: 40GB
- Network Adapter: NAT


---

# Part 1

# Pre-installation and Virtual Machine Preparation

---
## Step 1 - Downloading the Arch Linux ISO:
1. Reference the official [Arch Linux Downloads]( https://archlinux.org/download/) Website and choose your desired Arch Linux installation ISO. Note: mirror selected for this installation process was mirror.arizona.edu.
2. Download the ISO and save it to a memorable folder. This ISO download will be needed to serve as the installation medium for the VMware virtual machine.

## Step 2 - Verify if the ISO is secure:
1. From the chosen mirror, download its iso file and verify its credibility using SHA-256 checksum.
2. Once the iso and SHA-256 files are downloaded, ensure the files are in the same download folder.
3. Next, navigate to the windows terminal and enter the command: **Get-FileHash "FILE_PATH" -Algorithm SHA256**
4. In my case, the command is as follows: **Get-FileHash "$env:USERPROFILE\Downloads\archlinux-2026.11.01-x86_64.iso" -Algorithm SHA256**.
5. To verify the credibility of the iso file the checksum of the Arizona mirror must match the checksum presented in the terminal upon typing the command.
6. In this case, the hashes were identical which proves the iso file credible.

## Step 3 - Preparing VMware Workstation:
1.	Open VMware Workstation.
2.	Choose the option to **create a new virtual machine**.
3.	Select the option to install the operating system from an ISO image. 
4.	Locate the downloaded Arch ISO and select it. 
5.	Customize the virtual machines firmware settings according to the Virtual Machine Settings found in the **abstract**. 
6.	Save the virtual machine. **Do not start the VM yet**.  


## Step 4 - Setting up UEFI
1.	From the virtual machine settings, navigate to **firmware and boot options**. 
2.	Select the UEFI option and confirm that the CD/DVD drive is connected at power on.
3.	Save the setting and continue to **Part 2** to start the VM. 

# Part 2

# Booting the Arch Installation Environment

---
## Step 5 - Boot into the Installation Environment:
1.	Start the VM
2.	Choose the installed Arch Linux ISO instillation medium option and give the VM time to boot into it. The option should closely resemble **Boot Arch Linux (x86_64)**.
3.	Upon clicking the Boot Arch Linux option an **Arch Linux Installation environment** should appear and the user should be set to **root**.   

## Step 6 - Set up the Keyboard Layout and font:
1. Set the keyboard layout to US using the command **loadkeys us**.
2. Other available layouts can be referenced with the command **localectl list-keymaps**

## Step 7 - Verify the boot mode:
1. Verify the bootmode using the command **cat /sys/firmware/efi/fw_platform_size**. If the system is booted in UEFI mode and has a 64-bit x64 UEFI, the output will be 64.

## Step 8 - Connect to the internet
1. In the VM settings we previously set the **Network Adapter** to **NAT** to share our devices internet connection to our VM.
2. Ensure your network interface is listed and enabled by entering the command **ip link**.
3. Next, verify the connection by entering **ping ping.archlinux.org**.

## Step 9 - Update the system clock
1. Enter the command **timedatectl** to verify the system clock is synchronized.

# Part 3 

# Disk Partitioning
---
## Step 10 - identify the virtual hard drive and CD/DVD:
1. Type the command **lsblk** to preview the systems Block Device.
2. The commands output presented 3 disks, ending in /dev/loop0, /dev/sda, and /dev/sr0.
3.  Per the wiki instillation guidelines, the /dev/loop0 may be ignored. Hence, the virtual hard drive is presented as /dev/sda, while the CD/DVD containing the Arch iso is presented as /dev/sr0.

## Step 11 - Create the partitions
1. Refer to the formatted command to activate the partition modification tool **fdisk /dev/the_disk_to_be_partitioned**.
2. The command to be entered in my environment is as follows: fdisk /dev/sda
3.  Create of a new GPT and EFI partition:

Create partition 1 (GPT).
1. g - Creates new GPT partition table
2. n - Creates a new partition
3. Enter - Accepts partition number 1
4. Enter - Accepts the default sector 1
5. +1G - Assigns the amount of storage

Create partition 2 and confirm the changes (EFI).
1. t - changes partition type
2. 1 - Selects the EFI System partition type
3. n - Creates another partition
4. Enter - Accepts partition number 2
5. Enter - Accepts the default first sector
6. Enter - Uses the remaining disk space
7. w - writes the changes

- To verify the changes, enter the following command: **fdisk -l**

## Step 12 - Format the Partitions:
1. Start by formatting the EFI partition.
2. Given that the partition is an EFI system, enter the command **mkfs.fat -F 32 /dev/sda1** to create a FAT32 filesystem for UEFI booting.
3. Format the root partition. To create an ext4 filesystem for the Arch operating system, enter the command **mkfs.ext4 /dev/sda2**.
4. Verify the partitions format. Enter the command **lsblk -f** to view th partitions layout.
- **/dev/sda1** should contain **FAT32**
- **/dev/sda2** should contain **ext4**

## Step 13 - Mount the Filesystems:
1. Mount the **root** partition to **/mnt** using the command **mount /dev/sda2 /mnt**.
2. Mount the **EFI** partition using the command **mount --mkdir /dev/sda1 /mnt/boot**. Note: **--mkdir** creates a specified mount point.
3. Verify the partition formats using the command **lsblk**
4. Upon typing the command, **sda1** should contain **/mnt/boot** and **sda2** should contain **/mnt**.

# Part 4
# Installing Arch Linux
---
## Step 14 - Select Package mirror: 
1. Given that this system is using a live ISO, HTTPS mirrors should already be enabled.
2.  However, enter the command **/etc/pacman.d/mirrorlist** if the mirrors need to be inspected or modified.

## Step 15 - Install essential packages:
1. For the sake of simplicity, the following command installs the essential Arch Linux packages, Linux kernel, hardware firmware, network manager, sudo, nano, an alternative shell, SSH, AUR downloads, AUR tools, manual page management, and linux documentation in one instance.
2.  The command is as follows: **pacstrap -K /mnt base linux linux-firmware networkmanager sudo nano zsh openssh git base-devel man-db man-pages**.

# Part 5
# Configure the system
---

## Step 16 - Generate the filesystem table (fstab)
1. To generate the fstab table, enter the command **genfstab -U /mnt >> /mnt/etc/fstab**.
2. Check the resulting file using the command **cat /mnt/etc/fstab** and ensure that entires for the root and EFI partitions are displayed.

## Step 17 - Enter the installed Arch environment using chroot.
1. To enter the installed Arch environment, enter the command **arch-chroot -S /mnt**.

## Step 18 - Time zone
1. The format for setting a time zone is as follows: **ln -sf /usr/share/zoneinfo/Area/Location /etc/localtime**.
2. In my case, I will enter the command **ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime**
3. Next, enter the command **hwclock --systohc** to connect the hardware clock to the system time.

## Step 19 - Localization:
1. Enter the command **nano /etc/locale.gen** to open the various regional format options.
2.  Next, for this VM, locate **#en_US.UTF-8 UTF-8** and remove the #. The desired layout may vary from user to user.
3.   Generate the locale using the command **locale-gen**.
4.    After generating the locale, create the locale configuration by entering the command **echo "LANG=en_US.UTF-8" > /etc/locale.conf**.
5. Confirm the configuration by entering the command **cat /etc/locale.conf**.
6.  For this VM, the expected output is the **LANG=en_US.UTF-8** layout that was commented out in the previous steps.

## Step 20 - Create hostname.
1. For this assignment, I will be using the hostname arch-vm.
2.  Create a hostname file by entering the command **echo "arch-vm" > /etc/hostname**
3.   Verify the hostname using the command **cat /etc/hostname**.


## Step 22 - Network Configuration:
1. Given that NetworkmManager was installed in the previous steps, enable it for automatic statup by entering the command **systemctl enable NetworkManager**.
2.  Following the command, NetworkManager will manage the VM's connection through VMware NAT.

## Step 24 - Set root password:
1. Enter the command **passwd** and create a secure password.

## Step 25 - Install and configure the bootloader: 
1.  Choose a bootloader to install: For this VM I will be choosing **systemd-boot**.
2.  Install systemd-boot: First, ensure the EFI partition is mounted by entering the command **findmnt /boot**.
3. Next, install the bootloader using the command **bootctl install**.
4. Locate the root partition UUID: Enter the command **blkid /dev/sda2**. Note, remember the UUID value that appears upon entering the command because it will be needed in the next step.
5.  Open the loader configuration using the command **nano /boot/loader/loader.conf**. Enter the following text in the editor, save it using Ctrl+o, and exit using Ctrl+x.
default arch.conf

timeout 4

console-mode max

editor no
 
4. Next, create the Arch boot entry using the command **nano /boot/loader/entries/arch.conf**. Enter the following, while including the UUID from the previous steps. Then save and exit the nano file. 

title Arch Linux

linux /vmlinuz-linux

initrd /initramfs-linux.img

options root=UUID=**ROOT-UUID** rw

5. Now, verify the bootloader by entering the command **bootctl status**. Information on the systemd-boot should appear.


# Part 6
# Reboot into Arch Linux
---
## Step 26 - Exit the Arch installation environment, unmount the installed filesystems, and reboot the VM:
1. Exit the installation environment using the command **exit**.
2. Unmount the installed filesystems using the command **umount -R /mnt**.
3. Enter the command **power off** to power off the virtual machine.
4. In the VM setting remove the installation medium by unchecking the **Connect** box in the CD/DVD category.
5. Next, power on the VM to reboot into the newly installed Arch environment.

# Part 7 (Post-installation)
# Installing graphical desktop environment.
---
## Step 27 - Update Arch Linux:
1. Enter the command **pacman -Syu** to update Arch Linux

## Step 28 - Install the (DE) of choice.
1. For this project LXQt is the desktop environment of choice.
2.  Enter the command **pacman -S lxqt sddm xorg-server firefox**. This command will install the lxqt, sddm, xorg-server, and firefox packages.
3.   Next, it would also be helpful to install VMware integration tools by typing the command **pacman -S open-vm-tools**.

## Step 29 - Enable the desktop environment.
1. Configure the Simple Desktop Display Manager to start automatically when arch is boots by entering the command **systemctl enable sddm**.
2.  To enable VMware integration, use the command **systemctl enable vmtoolsd**.

# Part 8
# - Creating User Accounts with sudo Permissions
---
## Step 30 - Creating personal account:
1. User accounts can be created using **useradd -m -G wheel -s /usr/bin/zsh username**. This adds the user account to the **wheel group**.
2. For my user account my command will be **useradd -m -G wheel -s /usr/bin/zsh dylan**
3. Give the user a password by entering the command **passwd username**. In my case, it will be **passwd dylan**.

## Step 31 - Creating User Accounts (Other)
1. Essentially, repeat all the steps from **Step 32**, making sure to replace the username with the specified user.
2. Per the assignment instructions the command for adding the user Codi would be **useradd -m -G wheel -s /usr/bin/zsh codi**

## Step 32 - Adding sudo permissions to the wheel group:
1. Open the configuration for sudoers by typing the command **EDITOR=nano visudo**.
2. Find **# %wheel ALL=(ALL:ALL) ALL**, remove the **#**, save the change, and exit nano. The users in the wheel group should now have sudo permissions enabled.

# Part 9
# Shell Installation / Color Coded Terminal
---
## Step 33. Enter the Zsh shell as personal user:
1. In the previous steps, I chose the Zsh shell for my installation.
2. To log into the shell, enter the command **su - yourusername** to switch to log in as the personal user, considering there are now 2 user accounts.
3. Next, enter the command **echo $SHELL** to confirm the Zsh shell is in use.

## Step 34. Customizing the terminal colors:
1. First, enter the command **nano ~/.zshrc** to open the Zsh file with nano.

Enter the following text in the file as follows:
- To enable zsh color support type:

autoload -Uz colors

colors

- To configure a colored terminal prompt type directly below the previous command:

PROMPT='%F{cyan}%n@%m%f:%F{green}%~%f %# '

- To enable colored output type these commands below the previous command:

alias ls='ls --color=auto'

alias grep='grep --color=auto'

2. Finally, save the file and view the newly added color coded terminal by entering the command **source ~/.zshrc**.

## Step 35. Adding shell aliases
1. Again, enter the command **nano ~/.zshrc** to return to the Zsh file with nano, and type the following. The bullet pointed headings may be included as comments.

Aliases used for this VM:

- Directory shortcuts
  
alias ll='ls -lah'

alias la='ls -A'

alias ..='cd ..'

alias ...='cd ../..'

- System management

alias update='sudo pacman -Syu'

alias clean='sudo pacman -Sc'

- Networking
  
alias myip='ip addr'

alias ports='ss -tuln'

- Terminal
  
alias cls='clear'

alias grep='grep --color=auto'

2. Next, save and exit the Zsh file and confirm it using the command **source ~/.zshrc**.

# Part 10
# SSH
---
## Step 35. Enable SSH
1. For this guide SSH was installed in the previous steps.
2. To enable its server service, use the command **sudo systemctl enable sshd**.
3. Next, to start the service, enter **sudo systemctl start sshd**.
4. To verify if the SSH is running, enter **sudo systemctl status sshd**.

# Part 11
# Generating automatic graphical startup
---
## Step 36. Set the default system target.
1. Set the system target using the command **sudo systemctl set-default graphical.target** and check it by entering the command **systemctl get-default**.

# Part 12
# Installing an AUR Package
---
# Step 37. Install an AUR helper
1. considering that git and base-devel were installed in the previous steps, we only need to download the para AUR helper.
2. To download the paru AUR helper enter the command **git clone https://aur.archlinux.org/paru.git**.
3. Now, to enter the paru directory, enter **cd paru**.
4. Once in the paru directory, create and install the package by entering the command **makepkg -si**.

## Step 38. Install a package from the AUR (Google chrome):
1. To begin installing Google Chrome enter the command **paru -S google-chrome**.
2. Verify the installation by entering the command **google-chrome-stable --version**.

# Part 13
# Final Configuration and Reboot
---
1. For the final step, enter the command **sudo reboot** to restart the VM and ensure the LXQT desktop environment and applications are working properly. 


# Problems and Solutions
---
## Problem 1: Keyboard and Terminal Access
- During the initial installation process I first ran into the problem of accessing the terminal to allow me to enter commands inside VMware.

Solution:
- To to solve this issue, I just entered the virtual machine settings and ensured the keyboard input option was checked. As a result, I was then able to use the terminal.
- 
## Problem 2: Sudo Permisions
- When giving sudo permissions to the users in the wheel group, I encountered confusion on determining which entries to comment out. For context, there were 2 similar entries for the wheel group: **%wheel ALL=(ALL:ALL) ALL** and **%wheel ALL=(ALL:ALL) NOPASSWD: ALL**.
- For my VM I wanted the users to enter a password to gain sudo access, so I commented out the first option **%wheel ALL=(ALL:ALL) ALL**.
- However, upon saving the changes, uncommenting the wheel group entry did not enable sudo permissions.
- It was not until I reopened the /etc/sudoers file with nano a second time, that I noticed an extra space before the commented out wheel entry.
- After deleting the space, I saved the changes and I used the **groups** command to verify users in the wheel group.
- Finally, I entered **sudo whoami** and I was able to return to root.

## Problem 3: Shutting Down the Virtual Machine
- Initially, I was also uncertain about whether powering off the VM would erase all of the progress completed in the earlier steps. However, I found that the virtual machines virtual disk saves the installation progress.

## Problem 4: Paru Installation Error
- The first attempt to install Parue failed during the Rust compilation. The command used was **makepkg -si**.
- The failure was relayed when the terminal displayed **ERROR: A failure occurred in build(). (signal: 9, SIGKILL: kill)

Investigating the problem:
- First, I checked the available memory using the command **free -h**.
- Then, I checked the system logs using the command -T | grep -i -E 'out of memory|killed process'.
- Upon entering these commands, the output confirmed that the Rust termination happened due to a lack of available memory needed for the Paru installation process.

Solution
- To fix this problem, I powered off the VM and increased the memory in the VM settings from 4 GB to 6 gb.
- Next, I re-entered the command **makepkg -si**.
- Then, I verified the Paru installation by checking it with **paru --version**

## Problem 5: Restarting Arch Linux
- When all the steps were complete, and it was time to reboot, I entered the **sudo reboot** command in my terminal and nothing happened. The terminal prompt disappeared and the terminal appeared frozen.

Solution
- To bypass this issue, I ended up opening another tab for the terminal, and it was in this second terminal tab that I was successfully able to reboot my system. 


