# Commonly Asked Questions

## How to increase size of disk on LVM virtual machines
1. Shutdown the VM
2. Right click the VM and select Edit Settings
3. Select the hard disk you would like to extend
4. On the right side, make the provisioned size as large as you need it and confirm
5. Power on the VM and connect to it.
6. Identify your disk name with `sudo fdisk -l` for example /dev/sda
7. `sudo fdisk /dev/sda`
8. Enter `p` to print the partition table
9. Enter `n` to add a new partition
10. Enter `p` again to make it a primary partition
11. Enter the number of your new partition
12. Pick the first cylinder which will most like come at the end of the last partition (this is the default value)
13. Enter the amount of space (default is the rest of space available)
14. Enter `w` to save these changes
15. Restart the VM and log in
16. Type `sudo fdisk -l` and check that a new partition is present
17. Find your volume group with `df -h`.
 * Example: `/dev/mapper/ubuntu--vg-root 15G 4.5G ...`
 * Volume group is: `ubuntu-vg`
18. Extend the volume group : `sudo vgextend [volume group] /dev/sdaX`
 * Example: `sudo vgextend ubuntu-vg /dev/sda3`
19. Find the amount of free space available : `sudo vgdisplay [volume group] | grep "Free"`
20. Expand the logical volume : `sudo lvextend -L+[freespace]G /dev/[volgroup]/[volume]`
 * Example: `sudo lvextend -L+64G /dev/ubuntu-vg/root`
21. Expand the ext3 file system in the logical volume : `sudo resize2fs /dev/[volgroup]/[volume]`
 * Example: `sudo resize2fs /dev/ubuntu-vg/root`
22. You can now run the df command to very that you have more space `df -h`
