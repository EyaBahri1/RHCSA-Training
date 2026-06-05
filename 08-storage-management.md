
# Storage Management
## Introduction
👋 In this section, we will explore how to manage disks, partitions, LVM, and Stratis in a Red Hat Linux environment.
## Theory:
- **Partitioning**: creating one or more independent storage zones.
- **MBR Disk Structure**:
- Example:
  - SATA device:
    - `/dev/sda`: first SATA disk.
    - `/dev/sdb`: second SATA disk.
    - `/dev/sda1`: first partition of the first disk.
    - `/dev/sda2`: second partition of the first disk.
- **File System Types**: the way Linux organizes and stores its files on a device: ext2, ext3, ext4, jfs, xfs...
- **Mounting**: integrates a file system into the directory tree; once mounted, you can navigate its files like any other directory.
- **SWAP partition**: a temporary space on disk used to move inactive data from RAM.

### Commands:
- `lsblk` → view disks and partitions
- `fdisk /dev/disk` then `n` → create a partition
- `partprobe /dev/disk` → notify kernel of partition changes
- `mkfs.filesystem_type /dev/partition` → format the partition
- `mkfs.xfs -L mylabel /dev/partition` → format with a label
- `mkdir /mount_point` → create the mount point
- `blkid /dev/partition` → get UUID
- Add to `/etc/fstab`:
  ```
  UUID=<uuid>  /mount_dir  xfs  defaults  0 0
  LABEL=mylabel  /mount_dir  xfs  defaults  0 0
  ```
- `mount -a` → mount all entries from `/etc/fstab`
- `umount /partition`
- `fdisk /dev/disk` then `d`, then `w` → delete the partition

### SWAP Partition:
- `fdisk /dev/disk` → create partition, then `t`, then type `82`
- `mkswap /dev/partition`
- `free -m` → verify
- Add to `/etc/fstab`:
  ```
  UUID=<uuid> none swap defaults 0 0
  ```
- `swapon -a` → activate
- `swapoff -a` → deactivate

## Lab 07

### Q0. Create a new partition `/dev/sdb1` with 500MB  
### Q1. Format it as ext3  
### Q2. Mount it at `/mnt` during startup

```bash
fdisk /dev/sdb …
mkfs.ext3 /dev/sdb1
echo “UUID=uuid  /mnt  ext3  defaults 0 0” >> /etc/fstab
mount -a
```

### Q3. Create and activate a 1GB SWAP partition (`/dev/sdb2`) without affecting existing SWAP

```bash
fdisk /dev/sdb …
mkswap /dev/sdb2
echo “UUID=uuid  none  swap  0 0” >> /etc/fstab
swapon -a
```
## Logical Volume Management (LVM)

### Theory:
To obtain a logical volume, we must:
* Have a physical volume (PV) created from a partition.
* Create a volume group (VG) from physical volumes (PVs).
* Create a logical volume (LV).

*Why?* We need an 8G volume but only have partitions smaller than 8G.
→ LVM combines them into one pool and carves out any size you need.

### Commands:

#### Creation of an LV by giving you an exact size:
* `pvcreate /dev/partition_name` → create a physical volume
* `vgcreate vg_name /dev/partition_name1 /dev/partition_name2` → create a volume group
* `lvcreate -L <size> -n lv_name vg_name` → create the logical volume

Mount the logical volume:
* `mkdir /mount_point` → create the mount point
* `mkfs.xfs /dev/vg_name/lv_name` → format the LV
* `echo "/dev/vg_name/lv_name  /mount_point  xfs  defaults  0 0" >> /etc/fstab` then `mount -a` → mount the LV

We can also extend the LV: two cases — VG space sufficient, or insufficient.

##### If VG space is sufficient:
* `vgs` → view VG details (free size)
* `lvextend -r -L +<size> /dev/vg_name/lv_name` → extend the LV (`-r`: also extends the filesystem)
* `lvs` → verify

##### If VG space is insufficient: (extend VG then LV)
* `vgs` → view VG details (free size)
* `pvs` → check if a free PV exists, otherwise: `pvcreate /dev/partition_name`
* `vgextend vg_name /dev/free_pv_name` → extend the VG
* `lvextend -r -L +<size> /dev/vg_name/lv_name` → extend the LV (`-r`: also extends the filesystem)
* `lvs` → verify

#### Creation of an LV by giving number of PEs:

A **PE (Physical Extent)** is the smallest allocation unit in a VG.
Instead of specifying a size directly, you allocate a number of PEs.
→ size of LV = PE size × number of PEs (e.g. 4M × 3 = 12M)

* `vgdisplay` → view PE size and number of free PEs
* `vgcreate -s <pe_size> <vg_name> /dev/pv_name ...` → create a VG with a custom PE size (e.g. `-s 8M`)
* `lvcreate -l <nb_pe> -n <lv_name> <vg_name>` → create an LV using a number of PEs (`-l`) instead of a size (`-L`)
* `lvs` → verify: size should equal nb_pe × pe_size

*Note:* `-l` (lowercase) = number of PEs / `-L` (uppercase) = direct size (e.g. `-L 12M`)

#### Deleting an LV / VG:
* `umount /lv_name` → unmount the LV
* comment out its line in `/etc/fstab`, then `mount -a`
* `lvremove /dev/vg_name/lv_name` → delete the LV
* `vgremove vg_name` → delete the VG

## Lab 08
#### Q0. Create a Logical Volume partition. Below are the conditions: Volume Group is 510MB and named vol0; Logical Volume is 80MB and named lv0; File type is xfs and permanently mounted to the /cms file system.

**Problem:** default PE size = 4M (always)
510 / 4 = 127.5 → not divisible → LVM rounds up to 128 PE → 128 × 4 = 512M ≠ 510M ❌

**Solution:** change PE size to 2M
510 / 2 = 255 PE → 255 × 2 = 510M ✓

**But:** LVM always reserves 1 PE for its metadata (always)
→ create the partition as 512M (510M + 1 PE of 2M) to compensate

```bash
fdisk /dev/sda   # n → +512M → w
partprobe /dev/sda
pvcreate /dev/sda1
vgcreate -s 2M vol0 /dev/sda1        # PE size = 2M → VG = 510M usable
lvcreate -L 80M -n lv0 vol0
mkdir /cms
mkfs.xfs /dev/vol0/lv0
echo "/dev/vol0/lv0  /cms  xfs  defaults  0 0" >> /etc/fstab
mount -a
```
#### Q1. Create a Logical Volume Lvi with 60 extents; Volume Group Vgi with 16MB extent size. Mount it permanently under /record with file system ext3.

PE size = 16M (given) → LV = 60 × 16M = 960M
Partition = 960M + 1 PE (16M) = 976M

```bash
fdisk /dev/sda   # n → +976M → w
partprobe /dev/sda
pvcreate /dev/sda1
vgcreate -s 16M vgi /dev/sda1        # PE size = 16M
lvcreate -l 60 -n lvi vgi            # -l (lowercase) = number of PEs
mkdir /record
mkfs.ext3 /dev/vgi/lvi
echo "/dev/vgi/lvi  /record  ext3  defaults  0 0" >> /etc/fstab
mount -a
```

#### Q2. Resize the LV named lv0 = 152M so that it falls within the range of 200MB to 300MB.

**Creation of lv0:**
fdisk /dev/sda then +156M (152 + 4 PE) → vgcreate vg /dev/sda1 →
lvcreate -L 152M -n lv0 vg

*Correction:*
```bash
How many PEs are needed so that the max = 300:
lvextend -l ? /dev/vg/lv0

max - 152 = 300 - 152 = 148
PE = 4 \* ? = 148 (since PE = 4)
? = number of PEs = 148 / 4 = 37 PE

*Note:* partition → vgcreate / vgextend reduces one PE

To extend the VG, we must add a PE to the partition. → Create a partition:
fdisk /dev/sda then +152M (148 + 4) → we get /dev/sda2: 152M
vgextend vg /dev/sda2 → (VG is extended by 152 - 4 = 148M)

Then:
lvextend -L +148M /dev/vg/lv0 or lvextend -l +37 /dev/vg/lv0
```
