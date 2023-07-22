# Ansible role: DISK

Manage disk partitions at every level:
- gpt
- luks
- lvm
- btrfs

Only tested on Archlinux.

## Usage
See [defaults](https://github.com/lunics/ansible_role_disk/tree/main/defaults/main)
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
  - fstype:   vfat            # ESP (UEFI System Partition) must be in FAT variant vfat/FAT32
    dev:      /dev/nvme0n1p1
    force:    true
    label:    boot
    path:     /boot
    mnt_opts: noauto,relatime
    dump:     1
    passno:   2
  - fstype:   swap
    dev:      /dev/lvm/swap
    label:    swap
  - fstype:   btrfs
    dev:      /dev/lvm/btrfs
    mkfs_opts: --nodiscard --checksum xxhash
    label:    btrfs
    path:     /

btrfs:                # btrfs subvolumes
  - name: "@"
    mnt_opts: relatime,errors=remount-ro
  - name: "@home"
    mnt_opts: rw,relatime,nodev,nosuid
  - name: "@logs"
    path: /var/log
    cow:  false       # disable Copy-On-Write
    mnt_opts: rw,relatime,nodev,nosuid,noexec
  - name: "@cache"
    path: /var/cache/
    cow:  false
    mnt_opts: rw,relatime,nodev,nosuid,noexec
  - name: "@packages"
    path: /var/cache/pacman
    cow:  false
    mnt_opts: rw,relatime,nodev,nosuid,noexec
  - name: "@libvirt"
    path: /var/lib/libvirt
    cow:  false
    mnt_opts: rw,relatime,nodev,nosuid,noexec
  # - name: "@var_tmp"
  #   path: /var/tmp
  #   opts: noatime,nodiratime,nodev,nosuid
  #   cow:  false
  - name: "@opt"
  - name: "@tmp"
    mnt_opts: rw,nodev,nosuid,noexec
  - name: "@root"
  - name: "@snapshots"
  - name: "@srv"
  - name: "@usr"
    mnt_opts: ro,noatime,nodev
```
Result:
```
nvme0n1         259:0    0 238.5G  0 disk
├─nvme0n1p1     259:2    0   299M  0 part
└─nvme0n1p2     259:5    0 238.2G  0 part
  └─luks        254:1    0 238.2G  0 crypt
    ├─lvm-swap  254:2    0    16G  0 lvm   [SWAP]
    └─lvm-btrfs 254:3    0 222.2G  0 lvm   /usr
                                           /srv
                                           /snapshots
                                           /root
                                           /tmp
                                           /opt
                                           /var/lib/libvirt
                                           /var/cache/pacman
                                           /var/cache
                                           /var/log
                                           /home
                                           /
```

TODO
- change path chroot by another variable
