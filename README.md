## Set Up Keyboard Layout

1. 
   ```
   localectl list-keymaps
   ```
   list all avaably kayboard kayouts
2. 
   ```
   loadkeys de
   ```
   load the select keyboard layout (defautl is us)

## Check if bootet to UEFI

1.
  ```
  efivar -l
  ```
  if loits of wierd unkwon text appear on your screen that byou prorably doent unterstand thats means ou bootet into uefi mode

  ## Check Internet connection

    ```
    ping archlinux.org
    ```
    this will ping a website and show you an outout in bytes where you can check of you connectet to the internet

  ## Get disk information

  ```
  lsblk
  ```
  this wil show all informations about you instakkled disks on you pc (you can use it all the time after changing things on you disk to show infomrmations needet later)

  ## partition the disk

  ```
  cfdisk /dev/nvme0n1
  ```
  this will use the cfdiks tool to easaly partitioning you disk

  - make sure to select "gpt"
  - Create a New partition with the size of 1024M and give it the type "EFI SYSTEM"
  - then create ontoher partition with the size of the half of you ram or maximum 8GB (8GB is the recommenced but you can create larger like 16GB) make the Type a Linux Swap Partition
  - create ontoher parition with the size of 20-40GB and make it you root partitio nwith the type root system
  - Then create a last ontiher partition with the restr of you avaiable size and leave uit as default
  
  ## format the partitions

  ```
  mkfs.fat -F32 /dev/nvme0n1p1
  ```
  this will format the efi partition to Fat32 format

  ```
  mkswap /dev/nvme0n1p2
  ```
  this will format you swap partition

  ```
  swapon /dev/nvme0n1p2
  ```
  this will enable you formatet swap partition

  ```
  mkfs.ext4 /dev/nvme0n1p3
  ```
  this will format you root (20-40GB) disk to the linxu ext4 file system

  ```
  mkfs.ext4 /dev/nvme0n1p4
  ```
  this will format you main home partition where all you files will be sotred to the linux ext4 file system

## mount the partitions to install linux base system

  ```
  mount /dev/nvme0n1p3 /mnt
  ```
this will mount you root partition to: /mnt

  ```
  mkdir /mnt/boot
  mkdir /mnt/home
  ```
this will create a new derecorys for you home partition and you bootloader

  ```
  mount /dev/nvme0n1p1 /mnt/boot
  ```
this wil mount the efi partition to the boot partition

  ```
  mount /dev/nvme0n1p4 /mnt/home
  ```
this will moujnt you home directory

  ```
  
  ```


  
  
