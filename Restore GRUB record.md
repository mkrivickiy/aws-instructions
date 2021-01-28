The error "Unknown filesystem type ; entering into grub rescue", usually appears when the boot loader (grub) is not able be determine the location of the kernel image and initramfs. Now in order to troubleshoot this issue, kindly follow the below steps :

Troubleshooting Steps :
==========================

1. Create an ami of the instance for backup. (Important Recommendation)

Creating a Linux AMI from an instance - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html#how-to-create-ebs-ami 

2)  Launch a new recovery instance (t2.micro with the same AMI as impaired instance i.e. ubuntu 16 and name it as “Recovery instance” for easy understanding) and make sure it in the same availability zone i.e. eu-central-1b as your source impaired instance.

2)  Stop the impaired instance i.e.

3)  Detach the root volume (/dev/sda1 ) from the impaired instance and attach it to the new recovery instance as  /dev/sdf

4)  SSH/Login to the new recovery instance with the EBS volume attached and do the following: #sudo lsblk

	$ sudo lsblk            (Should show something like:)
	NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
	xvda    202:0    0    8G  0 disk
	└─xvda1 202:1    0    8G  0 part /
	xvdf    202:80   0  8G  0 disk     <—— xvdf is the EBS volume attached in step 3.
	└─xvdf1 202:81   0  8G  0 part


5)  Run the below commands to create a mount point for the EBS volume and to view the content of the attached EBS volume (xvdf1)

   - To mount the volume :     # sudo mount /dev/xvdf1 /mnt

6) Confirm that xvdf1 is your boot partition by running : # sudo ls -l /mnt/boot/

Here in the output you should be able to see kernal images (vmlinuz), grub directory and few other files.

7) Then chroot to /mnt by running the following commands :

# cd /mnt/
# sudo mount --bind /dev dev
# sudo mount --bind /proc proc
# sudo mount --bind /sys sys
# sudo chroot .

8) Commands to fix filesystem mappings : 

#sudo grub-install /dev/xvdf

Note: The output should display "- Installation finished. No error reported."

#sudo update-grub

9) Run the command - $exit

10) sudo umount /mnt/{proc,sys,dev,run}

11) Once you are done with the above steps, you can now detach the volume from the recovery instance and attach it to back to the original impaired instance as /dev/sda1.

7) Finally start the instance and monitor the system logs.

I hope the above information was helpful to you. Please feel free to write back if there are any further queries or if you are struck at any point following the given steps. If any, I will happy to assist you.

Looking forward for your response. 
