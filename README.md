######HackOver 2016 - 9soc writeup - zonmei

##thecard - forensics 9

#thecard
#We obtained a cyber terrorist's SD card, but it seems to be broken. Can you help us recover its content?
```
file thecard.img.xz-9835f59faf299b35d01a0bc39a9954ff42ba451659ac0f7c8ad954e270924600
```
```
thecard.img.xz-9835f59faf299b35d01a0bc39a9954ff42ba451659ac0f7c8ad954e270924600: XZ compressed data
```
```
cp thecard.img.xz-9835f59faf299b35d01a0bc39a9954ff42ba451659ac0f7c8ad954e270924600 thecard.img.xz (I like to get rid of chaff)
```
```
xz -dc thecard.img.xz > thecard.tar
```
```
file thecard.tar
thecard.tar: Linux rev 1.0 ext4 filesystem data, UUID=ce28a582-486d-41b1-8c66-1690e11cfcf2 (needs journal recovery) (errors) (extents) (huge files)
```
```
mkdir thecardmount

sudo mount -t ext4 thecard.tar thecardmount/
```

nothing else....

google lost files on ext4, find mke2fs

from man mke2fs
```
-n
    Causes mke2fs to not actually create a filesystem, but display what it would do if it were to create a filesystem. This can be used to determine the location of the backup superblocks for a particular filesystem, so long as the mke2fs parameters that were passed when the filesystem was originally created are used again. (With the -n option added, of course!)
```
```
mke2fs -n thecard.tar
mke2fs 1.42.9 (4-Feb-2014)
thecard.tar is not a block special device.
Proceed anyway? (y,n) y
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
4096 inodes, 16384 blocks
819 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=16777216
2 block groups
8192 blocks per group, 8192 fragments per group
2048 inodes per group
Superblock backups stored on blocks:
	8193
```
e2fsck will check the file structure of ext4 and attempt to mount
```
sudo e2fsck -b 8193 thecard.tar
e2fsck 1.42.9 (4-Feb-2014)
thecard.tar was not cleanly unmounted, check forced.
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Unattached inode 13
Connect to /lost+found<y>? yes
Inode 13 ref count is 2, should be 1.  Fix<y>? yes
Unattached inode 14
Connect to /lost+found<y>? yes
Inode 14 ref count is 2, should be 1.  Fix<y>? yes
Unattached inode 15
Connect to /lost+found<y>? yes
Inode 15 ref count is 2, should be 1.  Fix<y>? yes
Unattached inode 16
Connect to /lost+found<y>? yes
Inode 16 ref count is 2, should be 1.  Fix<y>? yes
Pass 5: Checking group summary information
Free blocks count wrong for group #0 (7597, counted=940).
Fix<y>? yes
Free blocks count wrong for group #1 (7102, counted=940).
Fix<y>? yes
Free blocks count wrong (14699, counted=1880).
Fix<y>? yes
Free inodes count wrong for group #0 (2037, counted=2032).
Fix<y>? yes
Free inodes count wrong (4085, counted=4080).
Fix<y>? yes

thecard.tar: ***** FILE SYSTEM WAS MODIFIED *****
thecard.tar: 16/4096 files (0.0% non-contiguous), 14504/16384 blocks
```
now cd into thecardmount/lost+found
```
thecardmount/lost+found$ lsa
total 13M
drwx------ 2 zonmei zonmei  12K Feb  6  2016 .
drwxr-xr-x 3 zonmei zonmei 1.0K Feb  6  2016 ..
-rw-r--r-- 1 zonmei zonmei  11M Feb  6  2016 #13
-rw-r--r-- 1 zonmei zonmei 526K Feb  6  2016 #14
-rw-r--r-- 1 zonmei zonmei 1.1M Feb  6  2016 #15
-rw-r--r-- 1 zonmei zonmei 273K Feb  6  2016 #16
```
#13 is the biggest and the first so I looked at it
```
strings \#13 | more
RIFF
WAVEfmt
dataD
...redacted....
```
Long story short, loaded this into VLC to listed to what might be a clue/flag only to hear men grunting.  My wife thought, oh wrestleing.  I thought gay porn.  30 minutes wasted.


Next #14
```
strings \#14 | more
JFIF
FXDXk%&;
hVIi
```
Oh yeah, a picture!    But looks like something is wrong.
![LF](/images/lost+found.png)

I use bless to look at data ![Bless](/images/Bless.png)

A quick google search shows magic number for JPG should be FF D8.  It's currently 00 D8 so a quick manipulation in Bless gives us this. ![Flag](/images/thecard.jpg)

FLAG: hackover16{DontWorryItsJustADoll!}
