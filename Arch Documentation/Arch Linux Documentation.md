## Arch Linux Documentation

### Downloading the ISO
I went to United States on the download page and scrolled to mit.edu iso download. 

### Creating the VM
Afterwards, I opened VMware Workstation and on the home page clicked "create a new machine." I used the downloaded ISO to make the machine. After the machine is created, I went to the "Edit virtual machine settings" page.

![alt text](<Screenshot 2024-10-25 131306.png>)

Ensure that you allocate 2GB of memory and 20GB of HDD space at this step.

### Booting into UEFI mode

As I am on a Windows machine, I had to edit the .vmx file to allow it to boot into UEFI mode. To get here, I navigated to files -> documents -> virtual machines -> arch linux.vmx -> open in notepad. I inserted "firmware="efi"" as the 2nd line.

![alt text](<Pasted image 20241025132549.png>)

I tried to turn on the VM at this point, but I encountered this error:

![alt text](<Pasted image 20241025132947.png>)

Moving the firmware="efi" down to line 3 instead of line 2 in the notepad file seemed to fix this error and it allowed the VM to boot as it normally would. This is the screen that is reached when you officially enter the installation environment:

![alt text](<Screenshot 2024-10-23 154936.png>)

I entered the following command to see if it booted in UEFI mode like it was supposed to:
`cat /sys/firmware/efi/fw_platform_size`
It was unable to return a file or directory, which meant that I performed a step incorrectly. 

I shut down the VM and navigated to VM -> Settings -> Options -> Advanced -> Firmware type and changed the firmware manually to UEFI. Secure boot should NOT be enabled as the installation image does not support it.

![alt text](<Pasted image 20241025141338.png>)

I powered the VM back on, and huzzah! The installation medium is UEFI.

![alt text](<Pasted image 20241025141710.png>)

I am now back at the installation environment. To test the UEFI mode, I again use the command:
`cat /sys/firmware/efi/fw_platform_size`

This time, it returned "64"--beautiful!

### Partition the Disks

Moving on, it is time to partition the disks.
Use the following commands:
`fdisk /dev/sda`
`g` (*this will create a GPT partition table*)
`n` (*this makes a new partition, we will make sda1 here*)
`1`
Hit enter
`+500M` (*makes the size 500MB*)
`t` (*switches partition to EFI*)
`1` (*corresponds EFI system with sda1*)
`n` (*making a new partition, here we create sda2*)
`2`
Hit enter 3 times to use the remaining disk space.
`w` (*this accepts all of the changes and exits fdisk*)

At this point, I must format the new partitions.
`mkfs.fat -F 32 /dev/sda1`
`mkfs.ext4 /dev/sda2`

![alt text](<Pasted image 20241027121056.png>)

I now mount the root partition (/dev/sda2) with the command:
`mount /dev/sda2 /mnt`
And mount the EFI partition with the command:
`mount --mkdir /dev/sda1 /mnt/boot`

The official install is then initiated with the command:
`pacstrap -K /mnt base linux linux-firmware`
`genfstab -U /mnt >> /mnt/etc/fstab`
`arch-chroot /mnt`
`ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`
`hwclock --systohc`
`sed -i '/en_US.UTF-8 UTF-8/s/^#//' /etc/locale.gen`
`echo "LANG=en_US.UTF-8" > /etc/locale.conf`
`echo "KEYMAP=us" > /etc/vconsole.conf`
`echo "arch" > /etc/hostname`
`passwd`

### Installing a Bootloader

Finally, I will install GRUB as the bootloader:
`pacman -S grub efibootmgr`
`mkdir -p /boot/efi`
`mount /dev/sda1 /boot/efi`
`grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`
`grub-mkconfig -o /boot/grub/grub.cfg`

### Installing Desktop Environment (Plasma)

Everything appeared to run successfully; I'll implement the DE before I reboot the system.
I did some research on desktop environments, and I really like how Plasma looks.

`pacman -S xorg-server xorg-apps xorg-xinit`
- Hit enter to accept the default, then type y to accept

`pacman -S plasma kde-applications`
- Hit enter to accept the default for all of the inquiries, then type y when prompted
- It will take a few minutes to install 

`pacman -S sddm`
- Type y to install

`systemctl enable sddm`

`pacman -S sddm-kcm`
- Type y to install

`exit`
`reboot`

Now that the reboot is complete, the VM now looks like this:

![alt text](<Pasted image 20241027153129.png>)

But alas...I cannot select a user. Oops. Because of this, I switched to the terminal interface by pressing CTRL + ALT + F3 and I logged back into the root user.

### Adding users

Following along with the assignment instructions, I created a user account for myself, justin, and codi via the following commands:
`useradd -m -G wheel justin`
`useradd -m -G wheel codi`
`echo "justin:GraceHopper1906" | chpasswd`
`echo "codi:GraceHopper1906" | chpasswd`
`chage -d 0 justin`
`chage -d 0 codi`
`useradd -m -G wheel atley`
`passwd atley`
- I put in my own password here.

At this point, I had a bit of problems trying to ensure sudo access for all users in the group wheel. Because of this, I decided to backup the current mirror list for safety and then I went on to try to update the mirror list.

`cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup`
`pacman -S reflector`

Alas, I ran into another problem here. 

![alt text](<Pasted image 20241027161125.png>)

### Fixing network problems

Because of this, I knew something was up with the network connectivity.
I rebooted the VM and logged in as myself, and then logged in as the root user once more in the DE terminal.
Once that was done, I did:
`pacman -S networkmanager`
Huzzah! It worked.
`systemctl enable NetworkManager`
`systemctl start NetworkManager`
`systemctl status NetworkManager`

![alt text](<Pasted image 20241027165654.png>)

I ran `ip a` to check the network status, and thankfully it resulted with "state UP" under ens33.

### Installing nano and implementing sudo permissions

Now that the network is back up and running, I can install nano:
`pacman -S nano`
`nano /etc/sudoers.tmp`
`root ALL=(ALL:ALL) ALL`
`%wheel ALL=(ALL:ALL) ALL`

![alt text](<Pasted image 20241027174108.png>)

CTRL + O, Enter, CTRL+X
`mv /etc/sudoers.tmp /etc/sudoers`
`chmod 440 /etc/sudoers`
`pacman -S sudo`
`visudo`

After doing this...I had a lot of problems with root permissions. I switched over to user atley to test sudo permissions, but when I typed in my password for atley, it did not accept the correct password. I tried switching back to root by typing su root, but it had an authentication failure.

After messing around with some commands, I eventually just restarted the VM and edited the boot parameters to allow me to enter as a single user in grub. I changed the root password, and then restarted the VM once more. Everything went back to normal after that.

I then used sudo pacman -S vi to see if I could get visudo to work as it should. It did, at last.

![alt text](<Pasted image 20241027182824.png>)

### Installing a new shell

Since I got that to work, I moved on to the next part of the assignment, which required me to install a different shell. I chose to install zsh.

`sudo pacman -S zsh`

The assignment did not specify that the new shell needed to be set as the default, so I kept bash as the default and the user can switch to zsh if need be.

### Installing ssh

I then installed ssh:
`sudo pacman -S openssh`

### Adding color coding

And at this point I added color coding:
`nano ~/.bashrc`
`source ~/.bashrc`

![alt text](<Pasted image 20241027184146.png>)

### Adding aliases

`nano ~/.bashrc`
`alias c='clear'`
`alias ls='ls --color=auto'`
`source ~/.bashrc`

All done~
