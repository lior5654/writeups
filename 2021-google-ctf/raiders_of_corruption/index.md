## lior@wildboar:/2021-google-ctf/raiders_of_corruption# cd ..

[Go Back](../index.md)

---------

# Raiders of Corruption [52 solves, 197pt]

## Challenge Description

> Picked up these at a yardsale, there doesn't seem to be anything useful in there though!
>
> [Attachment](./original.zip)

## Solution

In the solution writeup, I include the terms that I've googled and that've helped me find the relavant resources and documentation for solving the challenge. In some of the sections, I quote major parts of the resources they lead to, you are free & encouraged to explore by yourself then skip certain background explanations I provided prior to explaining the insights which lead to the final solution of the challenge (for instance, what RAID and MD are).

### Background: Initial Analysis

We begin by downloading the `.zip` file attached in the challenge's description and extracting the challenge files from it.

```console
lior@wildboar:~$ wget https://storage.googleapis.com/gctf-2021-attachments-project/6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1c
--2021-07-24 09:15:27--  https://storage.googleapis.com/gctf-2021-attachments-project/6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1c
Resolving storage.googleapis.com (storage.googleapis.com)... 142.250.185.144, 142.250.185.80, 142.250.74.208, ...
Connecting to storage.googleapis.com (storage.googleapis.com)|142.250.185.144|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8377014 (8.0M) [text/plain]
Saving to: ā6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1cā

6eb79e92b1c4e9e8f85 100%[===================>]   7.99M  10.7MB/s    in 0.7s    

2021-07-24 09:15:29 (10.7 MB/s) - ā6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1cā saved [8377014/8377014]

lior@wildboar:~$ ls
6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1c
lior@wildboar:~$ file 6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1c 
6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1c: Zip archive data, at least v2.0 to extract
unzip 6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1c 
Archive:  6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1c
 extracting: chall.tar.gz    

lior@wildboar:~$ ls
6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1c
chall.tar.gz
```

Now, let's extract the files from the `.tar.gz` file that we extracted from the attached `.zip`.

```console
lior@wildboar:~$ tar xvzf chall.tar.gz
disk01.img
disk02.img
disk03.img
disk04.img
disk05.img
disk06.img
disk07.img
disk08.img
disk09.img
disk10.img
lior@wildboar:~$ ls
6eb79e92b1c4e9e8f85b70f497d7c1b93342487243ca8c8161a9061b2207c6c57006b6e02c9890523dd8ab68beb737b18f8961ca2869367f0dd502b29d5f7c1c
chall.tar.gz
disk01.img
disk02.img
disk03.img
disk04.img
disk05.img
disk06.img
disk07.img
disk08.img
disk09.img
disk10.img

```

As we can see, the `.tar.gz` file contains 10 `.img` files & we successfully extracted them.
We observe that their filename format is `disk<id>.img`. Let's analyze them.

```console
lior@wildboar:~$ file disk*.img
disk01.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
disk02.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
disk03.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
disk04.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
disk05.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
disk06.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
disk07.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
disk08.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
disk09.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
disk10.img: Linux Software RAID version 1.2 (1) UUID=ad89154a:f0c39ce3:99c46240:21b5e681 name=0 level=5 disks=10
```

All of our `.img` files give `Linux Software RAID version 1.2` as output. Let's explore that that means:

Googling this term leads us to [here](https://www.thomas-krenn.com/en/wiki/Linux_Software_RAID_Information):

> Linux Software RAID (often called mdraid or MD/RAID) makes the use of RAID possible without a hardware RAID controller.
> For this purpose, the storage media used for this (hard disks, SSDs and so forth) are simply connected to the computer as individual drives, somewhat like the direct SATA ports on the motherboard.

Furthermore, googling `RAID` leads us to the following [Wikipedia page](https://en.wikipedia.org/wiki/RAID):
> RAID (/reÉŖd/; "Redundant Array of Inexpensive Disks" or "Redundant Array of Independent Disks") is a data storage virtualization technology that combines multiple
>physical disk drive components into one or more logical units for the purposes of data redundancy, performance improvement, or both. This was in contrast to the previous concept of highly reliable mainframe disk drives referred to as "single large expensive disk" (SLED).

RAID, "Redundant Array of Inexpensive Disks", is essentially a technology which uses multiple disk components to store, manipulate & access data. Depending on the type of RAID, the data is spread in chunks across the disk drives the RAID consists of in a certain way.

The advantages of using a RAID array rather than a single disk drive are better performance, and most importantly, data recovery - if a single drive is damanged not all data is lost and sometimes it's possible to "fill in the gaps". It's important to note that some RAID types also store extra parity chunks which help significantly recovering from data corruption.

`Linux Software RAID`, or `MD` (Multiple Device Driver) essentially enables us to use RAID with a software driver that controlls the array, rather than hardware. As we'll see, Linux enables us to create a [device file](https://en.wikipedia.org/wiki/Device_file) for a raid driver, specifiy to it parameters such as what disk drivers to use and what type of RAID and then use it for storage. The [man page for md](https://linux.die.net/man/4/md), mentions that the `mdadm` command is used to manage md devices (creating a RAID, examining it, etc). We will use the `mdadm` command frequently in the upcoming sections.

Now that we did some research, let's relate it to the challenge files. We conclude from the output of the `file` command seen above that the 10 `disk*.img` files are [disk images](https://en.wikipedia.org/wiki/Disk_image) of disk drive components that together form a RAID in the format used for a linux `md` device (specifically, the version 1.2 in the linux kernel, available since kernel release v3.1.1). In addition, we can see (based on the output) that the type of RAID system they form is [RAID5](https://searchstorage.techtarget.com/definition/RAID-5-redundant-array-of-independent-disks) (no need to understand at this point how data is stored in this type of RAID, we will elaborate on that later on).

### The "Corruption" Part in The Challenge Name

Naturally, now that we understand what we are dealing with, our next step will be to try to construct a RAID5 device from these existing disk images.

Googling `"construct RAID5 md from disk images linux"` leads us to [here](https://askubuntu.com/questions/663027/create-raid-array-of-image-files):

We learn from the answer to the stackoverflow question that we can use the `mdadm` tool (that we have previously seen in the man pages of `md`) to perform the task. We firstly have to create for each disk image a [loop device](https://en.wikipedia.org/wiki/Loop_device) (which is essentially a "pseudo device" that allows us to access a computer file as a [block device](https://en.wikipedia.org/wiki/Device_file#Block_devices)), then pass the RAID level, the device count and the list of loop devices to `mdadm`.

When I personally solved this challenge, I firstly did this process manually. As later on we'll discover that in the process of exploring this challenge and solving it, one would probably have to do this lengthy & tedious process of constructing the md device multiple times, and as a good practice, we shall now automate this process.

The following python script creates loop devices for each of the disk images, removes the md device if it existed & then creates an md device using the `mdadm` linux command.

TODO: add the script and document it

TODO: finish this section

### Superblocks, Kernel Docs & Realising the Issue in Recovery

TODO

### Shakespear Comes To The Rescue!

TODO

<img src="images/challenge_page.png" width="50%" height="50%"/>
