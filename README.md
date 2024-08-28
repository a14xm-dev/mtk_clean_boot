# Suppressing Bootloader Warnings on Samsung Devices

**Identify the Bootloader "Logo" Partition**
   - Connect the device to your computer and open terminal or command prompt.

Enter an adb shell:

![shell](.src/adbshell.png)

Access device shell with *elevated* priveleges

![alt text](.src/su.png)

Using Magisk, accept "SuperUser Request"

![SURequest](.src/superuserrequest.png)

List the *block device* partitions

```shell-session
$ a14xm:/ # ls -al /dev/block/by-name
```
   - Look for the entry similar to `up_param -> /dev/block/sdxY`. This is the partition we're interested in

This is our entry:
![list-partitions](.src/lspartition~2.png)

**Backup the Original Partition**
   - To copy out the original `up_param` partition, run:
     
![pullorigparam](.src/copyoriginal.png)
     ```
     dd if=/dev/block/sdc40 of=/sdcard/up_param.img
     ```
   - This will write a copy of the partition as a *tar* partition image named `up_param.img` in the root directory of our device Internal storage. **Backup this file** elsewhere if you mess with the image beyond repair.


The location is arbitrary, but move the file to your system. I would create a subdirectory in *Downloads/*

```plaintext
mkdir -p ~/Downloads/up_param && mv up_param.img ~/Downloads/up_param 
```


**Decompress and Edit the Images**
   - Decompress the `up_param.img` file to reveal a list of images 

![extractimg](.src/extraction.png)

```bash
user@ubuntu:~/Downloads/up$ tar xvf up_param.img 

```

   - The archive contents are decompressed in the *up_param/* directory

We want to edit the `booting_warning.jpg` and `svb_orange.jpg`
   - Use any image editor of your choice to modify these images. I used GIMP to fill `booting_warning.jpg` over the initial warning message 
   
![alt text](.src/originalbootwarn.jpg)

   - With GIMP *Fill Tool*: make sure to move the `tolerance` slider to maximum value. In order to completely fill the image black.
   
   - Since `svb_orange.jpg` is originally in 936x1800 aspect ratio, it doesn't match full screen resolution. 

To fix the modified `svb_orange.jpg` from appearing "too small" upon boot, You can copy out the `logo.jpg` and rename and overwrite to `svb_orange.jpg`

**Repack and Replace the Partition Image**
   - After editing, repack the modified images into back into a *tar* archive 

![repack](.src/repack.png)


```bash
user@ubuntu:~/Downloads/up_param$ tar -cvf up_param.tar *.jpg 

```
   - Linux uses the `mv` command to rename, from `up_param.tar` to `up_param.img`

![rename](.src/rename.png)

```bash
user@ubuntu:~/Downloads/up_param$ mv up_param.tar up_param.img

```

   - Place the modified `up_param.img` back into the `/sdcard` directory on your device. Where we first copied out the original `up_param.img`

![alt text](.src/modifiedparam.png)

**Restore the Modified Image**
- Enter back into an *root* adb shell
   - Write the modified image back to the original partition
 
![final step](.src/writebackmodified.png)

```shell-session
dd if=/sdcard/up_param.img of=/dev/block/sdc40
```
Reboot, and voila. 
