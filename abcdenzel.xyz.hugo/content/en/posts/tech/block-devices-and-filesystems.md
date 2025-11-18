+++
date = '2025-11-16T12:00:00-04:00'
draft = false
title = 'Block Devices and Filesystems: A Learning Journey'
tags = ['linux', 'filesystems', 'storage', 'tutorial']
categories = ['tech']
+++

I bought a new NVMe drive and decided this was as good a time as any to actually understand what I was doing instead of just copying commands from Stack Overflow until something worked.

Here's what I learned about block devices, partition tables, and why everyone still pretends sectors are 512 bytes when they haven't been for years.

## Finding the Drive

First step: figure out where the hell the new drive actually is.

```bash
lsblk
```

There it was at `/dev/sda`, advertising itself as a 1.8TB device with no filesystem and no partitions. Clean slate.

```
NAME          FSVER FSSIZE FSAVAIL FSTYPE   LABEL      MOUNTPOINTS                  STATE   PATH                  LOG-SEC PHY-SEC VENDOR
sda                                                                                 running /dev/sda                  512    4096 JMicron
```

That "JMicron" vendor name is the USB 3.2 adapter that came in the box, not the actual drive (PNY CS2241 M.2, if you care).

## The 512e Rabbit Hole

Notice those sector sizes: 512 bytes logical, 4096 bytes physical. This is called "512e" (512-byte emulation), and it's one of those backwards compatibility things that seemed like a good idea at the time.

**Logical vs Physical Sector Sizes**

| Logical Sector Size | Physical Sector Size |
|---------------------|---------------------|
| 512 bytes          | 4096 bytes          |

The **logical sector size** is what the OS thinks it's working with. It's the smallest unit of data it can read or write.

The **physical sector size** is what the hardware is actually using.

When these don't match, the drive's firmware has to do translation. Want to write 512 bytes to the third logical sector? The firmware has to:

1. Load the entire 4096-byte physical sector (sector 0) into a buffer
2. Modify the appropriate 512-byte chunk (position 3 × 512)
3. Write the whole 4096-byte sector back to disk

This is called a read-modify-write cycle, and it's slower than just writing directly to native 4K sectors.

### Can We Fix This? Should We?

Technically yes—you can format some drives to use 4Kn (4K native) instead of 512e. Practically? The internet consensus is: "This is one of those high risk, low return 'fixes' that I don't think are worth it."

So I left it at 512e and moved on to partitioning.

## Partition Tables: The Map Your Drive Needs

Before you can use a drive, you need a partition table which is basically a map that tells the system "these bytes are partition 1, these bytes are partition 2" and so on.

### GPT vs MBR

The modern standard is **GPT** (GUID Partition Table), which is part of UEFI. It replaces the older MBR (Master Boot Record) scheme. Just know that modern systems prefer GPT partition tables and you would not need the older scheme unless you already know what you're doing.

**Where GPT Lives:**

- **Primary GPT header**: First sector of the disk (LBA 1)
- **Backup GPT header**: Last sector of the disk (LBA -1)
- **Partition entries**: Following the primary header, with backups before the backup header

This redundancy means if the primary table gets corrupted, you can recover from the backup. Nice.

**GPT Features:**

- Supports disks larger than 2TB (unlike MBR)
- Up to 128 partitions by default (expandable)
- Each partition identified by a unique GUID
- No concept of "primary" vs "logical" partitions—all partitions are equal

### Creating a Partition Table

Check what's currently on the drive:

```bash
sudo parted /dev/sda print
```

```plaintext
Error: /dev/sda: unrecognised disk label
Model: JMicron Tech (scsi)
Disk /dev/sda: 2000GB
Sector size (logical/physical): 512B/4096B
Partition Table: unknown
```

No partition table yet. Let's create one.

```bash
sudo parted /dev/sda mklabel gpt
```

This creates a GPT partition table. You could use `msdos` for MBR if you hate yourself or need compatibility with something ancient.

## Creating Partitions

Now we can create actual partitions. I decided to split the drive: 512GB for large files I'm currently working with, like my AI model checkpoints, rest for backups.

### Understanding `mkpart`

```bash
sudo parted /dev/sda mkpart primary ntfs 2s 25%
```

Let's break this down:

- `mkpart`: Creates a partition (doesn't format it—that comes later)
- `primary`: Technically meaningless in GPT (legacy term from MBR days), but required by `parted`'s syntax
- `ntfs`: Sets the partition type GUID to NTFS—this is metadata, not an actual filesystem yet
- `2s`: Start at sector 2 (sector 1 is the GPT header)
- `25%`: End at 25% of the disk

**Partition Types in GPT:**

Unlike MBR's "primary/logical/extended" mess, GPT uses Partition Type GUIDs to identify intended use:

- EFI System Partition (ESP) for UEFI booting
- Microsoft Reserved Partition (MSR) for Windows
- Linux filesystems (ext4, btrfs, etc.)
- Windows NTFS
- Linux swap
- And many more

The filesystem type you specify in `mkpart` just sets this GUID, it doesn't actually create a filesystem. I spent at least 10 minutes wondering what actually happened.

## Formatting: Actually Making It Usable

Now we have partitions, but they're empty. Time to format them with actual filesystems.

I went with NTFS for cross-platform compatibility (Linux and Windows both handle it fine).

```bash
sudo mkfs.ntfs -f /dev/sda1
```

The `-f` flag does a fast format (doesn't zero every byte). For a new drive, this is fine.

**Other filesystem options:**

- `mkfs.ext4` for Linux-only use (better performance, better features)
- `mkfs.btrfs` if you want snapshots and advanced features
- `mkfs.xfs` for large files and high performance
- `mkfs.fat -F 32` for maximum compatibility but limited features

## What I Learned

1. **512e is everywhere** and it's probably fine. Chasing 4Kn formatting isn't worth the hassle for most use cases, unless your drive's firmware has already ditched its old 512e ways.

2. **GPT is simple once you understand it**: Primary and backup headers, partition entries between them, GUIDs to identify partition types. That's it.

3. **Partition tables and filesystems are separate**: The partition table is just a map. Filesystems are what actually organize data on those partitions.

4. **`parted` syntax is weird**: It inherits MBR terminology ("primary") even for GPT, where it's meaningless. Don't overthink it.

5. **NTFS is the pragmatic choice** for cross-platform drives, even though ext4 would be faster on Linux.

## Useful Commands

```bash
# List block devices and current state
lsblk

# Interactive partitioning tool (text UI)
sudo fdisk /dev/sda

# Scriptable partitioning tool (what I used)
sudo parted /dev/sda

# Format as NTFS
sudo mkfs.ntfs -f /dev/sda1

# Format as ext4
sudo mkfs.ext4 /dev/sda1

# Show detailed block device info
sudo blkid

# Mount a filesystem
sudo mount /dev/sda1 /mnt/mydrive
```

## References

If you want to go deeper:

- [NVMe Specification](https://www.nvmexpress.org/specifications) - The actual NVMe standard (for masochists)
- [Why do hard drives still use 512e sectors?](https://superuser.com/questions/1667441/) - Good Stack Overflow thread on 512e vs 4Kn
- `man parted` - Actually useful docs, surprisingly

---

Next time I'll probably forget half of this and have to look it up again. That's what these notes and blogposts are for ;)
