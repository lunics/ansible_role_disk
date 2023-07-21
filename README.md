# Ansible role: DISK

Manage disk partitions at every level:
- gpt
- luks
- lvm
- btrfs

Only available on Archlinux.

## Usage
see [default]()
```yaml
gpt:
  - device: /dev/nvme0n1
    partition:
     - number: 1
       start:  0%
       end:    300MiB
       flags:  esp
     - number: 2
       start:  300MiB
       end:    100%

luks:
  - partition: /dev/nvme0n1p2
    name:      luks
    password:  "{{ vault.luks }}"

lvm:
  - vgname: lvm
    disks:
      - /dev/mapper/luks
    volumes:
      - lvname:     swap
        size:       16g
      - lvname:     btrfs
        size:       100%FREE

fs:
  - fstype: vfat            # UEFI System Partition (ESP) must be in FAT variant vfat/FAT32
    dev:    /dev/nvme0n1p1
    force:  true
    label:  boot
    mount:
      point: /boot
  - fstype: swap
    dev:    /dev/lvm/swap
    label:  swap
  - fstype: btrfs
    dev:    /dev/lvm/btrfs
    opts:   --nodiscard --checksum xxhash
    label:  btrfs
    mount:
      point: /

btrfs:
  - @
  - @var
  - @var_log
  - @usr
  - @opt
  - @snapshots
  - @home
```
