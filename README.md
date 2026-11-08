# dnj28.github.io
Arch Installation Documentation

# Largest Heading
## Medium Heading
### Smallest Heading
**bold**
*Italcs*
- Bullet

# Part 1 - Pre-installation and Virtual Machine Preperation:
---
## Step 1. Installing an Image:
- First, aquire the installation image from [The Arch Linux Downloads Page](https://archlinux.org/download/).
- Image chosen for this assignment: mirror.arizona.edu > archlinux2026.11.01-x86_64.iso

## Step 2. Verify iso Image credibility using checksum verification:
- From the chosen mirror, I downloaded its iso file and verified its credability using SHA-256 checksum.
- Once the iso and SHA-256 files are downloaded, ensure the files are in the same download folder.
- Next, navigate to the windows terminal and enter the command: **Get-FileHash "FILE_PATH" -Algorithm SHA256**
- In my case, the command is as follows: **Get-FileHash "$env:USERPROFILE\Downloads\archlinux-2026.11.01-x86_64.iso" -Algorithm SHA256**.
- To verify the credability of the iso file the checksum of the Arizona mirror must match the checksum presented in the terminal upon typing the command.
- In this case, the hashes were identicle which proves the iso file credible.

## Step 3. Prepare the Installation Medium:


## Step 4. Configure the Virtual Machine Settings:


## Step 5. Attach the Arch iso file to the VM

# Part 2 - Booting into the Arch Installation Environment.
---
## Step 6. Boot into the Arch linux installation operating system:
- Power on the Virtual Machine.
- Once powered on, a prompt should appear with various options to choose from.
- Select the option that resembles **Boot Arch Linux (x86_64)**.
- Upon clicking the Boot Arch Linux option an **Arch Linux Installation environment** should appear and the user should be set to **root**.   

## Step 7. Set the console keyboard layout and font:
- Per the assignment and instructions, I set the keyboard layout to US using the command **loadkeys us**.
- Other available layouts can be referenced with the command **localectl list-keymaps**

## Step 8. Verify the Boot Mode.
- verify the bootmode using the command **cat /sys/firmware/efi/fw_platform_size**
- If the system is booted in UEFI mode and has a 64-bit x64 UEFI, the output will be 64.

## Step 9. Connect to the internet.
- In the VM settings I set the **Network Adapter** to **NAT** to share my Devices internet conenction with my VM. 
- Per the assignemnt instructions, when typed, the command **ip link** provides the network interface **ens33**.

## Step 10. Verify the system clock:
- Use the command **timedatectl** to verify the systemd-timesyncd service is active.
  

# Part 3 - Partitioning the virtual hardrive.
---

## Step 11. identify the virtual hardrive and CD/DVD:
- Type the command **lsblk** to preview the systems Block Device.
- The comands output presented 3 disks, ending in /dev/loop0, /dev/sda, and /dev/sr0.
- Per the wiki instillation guidlines, the /dev/loop0 may be ignored.
- Hence, the virtual hardrive is presented as /dev/sda, while the CD/DVD containing the Arch iso is presented as /dev/sr0.

## Step 12. Create the partitions:
- Refer to the formatted command to activate the partition modification tool **fdisk /dev/the_disk_to_be_partitioned**.
- The command to be entered in my environment is as follows: fdisk /dev/sda
- The creation of a new GPT partition is as follows.

Create partition 1.
1. g - Creates new GPT partition table
2. n - Creates a new partition
3. Enter - Accepts partition number 1
4. Enter - Accepts the default sector 1
5. +1G - Assigns the ammount of storage

Create partition 2 and confirm the changes.
1. t - changes partition type
2. 1 - Selects the EFI System partition type
3. n - Creates another partition
4. Enter - Accepts partition number 2
5. Enter - Accepts the default first sector
6. Enter - Uses the remaining disk space
7. w - writes the changes

- To verify the changes enter the following command: **fdisk -l**

## Step 13. Format the partitions:

Start by formatting the EFI partition.
- Given that the partition is an EFI system, enter the command **mkfs.fat -F 32 /dev/sda1** to create a FAT32 filesystem for UEFI booting.

Format the rood partition.
- To create an ext4 filesystem for the Arch operating system, enter the command **mkfs.ext4 /dev/sda2**.

Verify the partitions format.
- Enter the command **lsblk -f** to view th partitions layout.
- **/dev/sda1** should contain **FAT32**
- **/dev/sda2** should contain **ext4**

## Step 14. Mount the file systems:  
- Mount the **root** partition to **/mnt** using the command **mount /dev/sda2 /mnt**. 
- Mount the **EFI** partition using the command **mount --mkdir /dev/sda1 /mnt/boot**.
- Note: **--mkdir** creates a specified mount point.
- Verify the partition formats using the command **lsblk**
- Upon typing the command, **sda1** should contain **/mnt/boot** and **sda2** should contain **/mnt**.

# Part 4. Install Arch Linux:
---
## Step 15. Select Package mirrorr: 
- Given that I am using a live ISO, HTTPS mirrors should already be enabled. 
- However, enter the command **/etc/pacman.d/mirrorlist** if the mirrors need to be inspected or modified.

## Step 16. Install essential packages:
- For the sake of simplicity, the following command installs the essential Arch Linux packages, linux kerel, hardware firmware, network manager, sudo, nano, an alternative shell, SSH, AUR downloads, AUR tools, manual page management, and linux documentation in one instance.
- The command is as follows: **pacstrap -K /mnt base linux linux-firmware networkmanager sudo nano zsh openssh git base-devel man-db man-pages**

# Part 5 - Configure the system.
---
## Step 17. Generate the filesystem table (fstab)
- To generate the fstab table, enter the command **genfstab -U /mnt >> /mnt/etc/fstab**.
- Check the resulting file using the command **cat /mnt/etc/fstab** and ensure that entires for the root and EFI partitions are displayed.

## Step 18. Enter the installed Arch environment using chroot.
- To enter the installed Arch environment enter the command **arch-chroot -S /mnt**.

## Step 19. Time zone
- The format for setting a time zone is as follows: **ln -sf /usr/share/zoneinfo/Area/Location /etc/localtime**.
- In my case, I will enter the command **ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime**
- Next, enter the command **hwclock --systohc** to connect the hardware clock to the system time.

## Step 20. Localization:
- Enter the command **nano /etc/locale.gen** to open the various regional format options.
- Next, for this VM, locate **#en_US.UTF-8 UTF-8** and remove the #. The desired layout may vary from user to user.
- Generate the locale using the command **locale-gen**.
- After generating the locale, create the locale configuration by entering the command **echo "LANG=en_US.UTF-8" > /etc/locale.conf**.
- Confirm the configuration by entering the command **cat /etc/locale.conf**.
- For this VM, the expected output is the **LANG=en_US.UTF-8** layout that was commented out in the previous steps.

## Step 21. Create hostname.
- For this assignment, I will be using the hostname arch-vm.
- Create a hostname file by entering the command **echo "arch-vm" > /etc/hostname**
- Verify the hostname using the command **cat /etc/hostname**.

## Step 22. Network Configuration:
- Given that networkmanager was installed in the previous steps, enable it for automatic statup by entering the command **systemctl enable NetworkManager**.
- Following the command, NetworkManager will manage the VM's connection through VMware NAT.

## Step 23. Set root password:
- Enter the command **passwd** and create a secure password.
  
# Part 6 - Install and configure the bootloader.
---
## Step 24. choose a bootloader to install:
- For this VM I will be choosing **systemd-boot**.

## Step 25. Install systemd-boot:
- First, ensure the EFI partition is mounted by entering the command **findmnt /boot**.
- Next, install the bootloader using the command **bootctl install**.

## Step 26. Locate the root partition UUID: 
- Enter the command **blkid /dev/sda2**
- Note, remember the UUID value that appears upon entering the command because it will be needed in the next step.

## Step 27. Configure the bootloader.
- open the loader configuration using the command **nano /boot/loader/loader.conf**.

Enter the following text, save it using Ctrl+o, and exit using Ctrl+x: 

default arch.conf

timeout 4

console-mode max

editor no
  
- Problem occured when attempting to save changes: Error - /boot/loader/loader.conf could not be located.

## Fixing Error. 
- Upon entering the command **ls -la /boot** the output showed **no such file or directory**.
- Check the bootloader by entering the commands **bootctl status** and **findmnt /** to check if systemd-boot was properly installed and is in the root filesystem /dev/sda2.
- The commmands output for my VM confirms my Arch root filesystem is mounted correctly to /dev/sda2, and that the systemd-boot was not successfully insatalled on my EFI patition.
- The output that conveys the issue is **"systemd-boot not installed in ESP"**.

## Step 27 continued...
- Open the loader configuration using the command **nano /boot/loader/loader.conf**.

Enter the following text, save it using Ctrl+o, and exit using Ctrl+x: 

default arch.conf

timeout 4

console-mode max

editor no

- Next, create the Arch boot entry using the comand **nano /boot/loader/entries/arch.conf**.

Enter the following, while including the UUID from step 26. Then save and exit the nano file. 

title Arch Linux

linux /vmlinuz-linux

initrd /initramfs-linux.img

options root=UUID=**ROOT-UUID** rw

- Now, verify the bootloader by entering the command **bootctl status**. Information on the systemd-boot should appear.

# Part 7 - Reboot into Arch Linux
---
## Step 28. Exit the Arch installation environment, unmount the installed filesystems, and reboot the VM:
- Exit the installation environment using the command **exit**.
- Unmount the installed filesystems using the command **umount -R /mnt**.
- Enter the command **power off** to power off the virtual machine.
- In the VM setting remove the installation medium by unchecking the **Connect** box in the CD/DVD category. 
- Next, power on your VM to reboot into the newly installed Arch environment.

# Part 8 - Installing the graphical desktop environment.
---
## Step 29. Update Arch Linux:
- Enter the command **pacman -Syu** to update Arch Linux

## Step 30. Install the (DE) of choice.
- For this project LXQt is the desktop environment of choice.
- Enter the command **pacman -S lxqt sddm xorg-server firefox**. This command will install the lxqt, sddm, xorg-server, and firefox packages.
- Next, it would also be helpful to install VMware integration tools by typing the command **pacman -S open-vm-tools**.

## Step 31. Enable the desktop environment.
- Configure the Simple Desktop Display Manager to start automatically when arch is boots by entering the command **systemctl enable sddm**.
- To enable VMware integration use the command **systemctl enable vmtoolsd**.

# Part 9 - Creating User Accounts (Personal).
---
## Step 32. Creating personal account:
- User accounts can be created using **useradd -m -G wheel -s /usr/bin/zsh username**. This adds the user account to the **wheel group**.
- For my user account my command will be **useradd -m -G wheel -s /usr/bin/zsh dylan**
- Give the user a password by entering the command **passwd username**. In my case, it will be **passwd dylan**.

## Step 33. Creating User Accounts (Other)
- Essentially, repeat all the steps from **Step 32**, making sure to replace the username with the specified user.
- Per the assignment instructions the command for adding the user codi would be **useradd -m -G wheel -s /usr/bin/zsh codi**

## Step 34. Adding sudo permissions to the wheel group:
- Open the configuration for sudoers by typing the command **EDITOR=nano visudo**.
- Find **# %wheel ALL=(ALL:ALL) ALL**, remove the **#**, save the change, and exit nano. The users in the wheel group should now have sudo permissions enabled.

# Part 10. Shell Installation / Color Coded Terminal
---
## Step 35. Enter the Zsh shell as personal user:
- In the previous steps, I chose the Zsh shell for my installtion.
- To log into the shell, enter the command **su - yourusername** to switch to log in as the personal user, considering there are now 2 user accounts.
- Next, enter the command **echo $SHELL** to confirm the Zsh shell is in use.

## Step 36. Customizing the terminal colors:
- First, enter the command **nano ~/.zshrc** to open the Zsh file with nano.

Enter the following text in the file as follows:
- To enable zsh color support type:

autoload -Uz colors

colors

- To configure a colored terminal prompt type:

PROMPT='%F{cyan}%n@%m%f:%F{green}%~%f %# '

- To enable colored output type:

alias ls='ls --color=auto'

alias grep='grep --color=auto'

- Finally, save the file and view the newly added color coded terminal by entering the command **source ~/.zshrc**.

## Step 37. Adding shell aliases
- Again, enter the command **nano ~/.zshrc** to return to the Zsh file with nano.

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

- Next, save and exit the Zsh file and confirm it using the command **source ~/.zshrc**.

# Part 11 - SSH.
---
## Step 38. Enable SSH
- For this guide SSH was installed in the previous steps.
- To enable its server service use the command **sudo systemctl enable sshd**.
- Next, to start the service, enter **sudo systemctl start sshd**.
- To verify if the SSH is running, enter **sudo systemctl status sshd**.

# Part 12 - Generating the automatic graphical startup.
---
## Step 39. Set the default system target.
- Set the system target using the command **sudo systemctl set-default graphical.target** and chaeck it by entering the command **systemctl get-default**.

## 13 - Installing an AUR package.
---
# Step 40. Install an AUR helper
- considering that git and base-devel were installed in the previous steps, we only need to download the para AUR helper.
- To download the paru AUR helper enter the command **git clone https://aur.archlinux.org/paru.git**.
- Now, to enter the paru directory, enter **cd paru**.
- Once in the paru directory, create and install the package by entering the command **makepkg -si**.

** Side Notes:**
- ERROR oocurred upon enering the command **makepkg -si**: Output = error: could not compile 'paru' (bin "paru").

**Fixing the Paru installtion error**

Commands entered: 
- makepkg -si 2>&1 | tee 'error' ~/paru-build.log
- grep -n -B 5 -A 10 'error' ~/paru-build.log
- Output: signal: 9, SIGKILL: kill
- The output is likely related to a lack of available memory error.

- **free -h**
- **sudo dmesg -T | grep -i -E 'out of memory|killed process'**
- Upon entering the commands the output revealed that my available memory was 3.8Gi which is not enough for the installation process to complete.
- To fix the issue I powered off my VM, and adgusted the VM's settings to have an available memory of 6144 mb.
- I saved the changes, powered on the VM, and then re-typed the installation commands to successfully complete the Paru installation process. 



## Step 41. Istall a package from the AUR (Google chrome):
- To begin installing Google Chrome enter the command **paru -S google-chrome**.
- Verify the installation by entering the command **google-chrome-stable --version**.


## Part 14. - Final configuration and reboot.
- For the final step, enter the command **sudo reboot** to restart the VM and ensure the LXQT desktop evironment and applications are working properly. 


  




































