Hardware and setup of machine

- CPU: AMD THREADRIPPER 3960x
- Motherboard: Asrock Taichi trx40
- Power Supply: xxxxxxxxxxxxxxxxxxxxx
- GPU: NVIDIA 1080ti
- RAM: Kingston 3200 UECC - 4x 32 gb
- Storage: Western Digitial sn850 - 1tb | 2x Samsung 860 EVO - 1 tb
- Backup: Western Digital Gold - 6 tb
- OS: Fedora minimal

- Storage:
  - mdadm chosen since it handles raid even though lvm can now be used directly
  - lvm for expandability chosen since it was easier for me to understand
  - ext4 use for filesystem at the moment unless noted otherwise
  - btrs and zfs and lvm are able to take some or all functionality but it is not my main focus
  - whole disk and paritions were used but it can be split further or consolidated
  - Steps (Mapping can sometimes be used in place of direct paths and some additional steps might be required for raid or luks to work properly):
    1. Created disk partitions: `sudo fdisk /dev/<disk/ssd/nvme>`
    2. Created mdadm raid0: `sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdx1 /dev/sdx1`
    3. Create physical volumes: `sudo pvcreate /dev/< sdx1 | raid0 >`
    4. Create volume groups: ` vgcreate <name> /dev/<physical volume from before> `
    5. Create logical volume: `lvcreate -n <name> -L <size><B | M | G | T> <volume group name from before>`
    6. Create luks2 (i let the installer handle this point onward for `/` `/home` and `/var` and used the same password for ease):
      - Create luks2: `sudo cryptsetup --type luks2 /dev/<path>`
      - Get luks id: `sudo cryptsetup luksUUID /dev/<volume group>/<logical volume>`
      - Unlock and map: `sudo cryptsetup open /dev/<volume group>/<logical volume> luks-<luksUUID>`
    7. Create filesystem: `sudo mkfs.ext4 /dev/mapper/<luks2 mapped name>`
    8. Modify `/etc/crypttab` and map luks-uuid to unlock at startup
    9. Modify `etc/fstab` and add mapped location to mount

| SSD1   | SSD2   | notes |
|:-----:|:-----:|:-----:|
| partition1 | partition1 | |
| mdadm raid0 | -----> | raids partitions, not required but can use different sized disk and might protect against `Partition misalignment` |
| phyiscal volume | -----> | spans whole raid |
| volume group | -----> | spans whole volume |
| logical volume1 | logical volume2 | LVs for VMs are passed to libvirt for storage pool creation or direct use |
| luks2          | luks2          | encryption at this level allows for easier management after setup and allows for VM storage to be encrypted and moveable |
| file system    |                | must be unencrypted before use and will be mapped under `/dev/mapper/<name>` |
| `/home`        |                | |

| nvme   | -----> | -----> | notes | 
|:-----:|:-----:|:-----:|:-----:|
| partition1 | partition2 | partition3 | |
| filesystem | efi | physical volume = volume group | |
| `/boot` | `/boot/efi` | logical volume 1 - logical volume 2 | |
|         |             | luks2 - luks2 | |
|         |             | file system - file system | |
|         |             |`/` - `/var` | |

- Software:
  - I want to run VMs for web browsing/torrenting and containerization/docker for development workloads
  - cockpit
  - libvirt
  - virt-manager
  - firefox
  - git
  - qbittorrent
  - wireguard-tools : client tunnel to connect networks and allow ssh/remote desktop to minimize laptop resource usage. Should be forwarded through proxy server when used in public to minimize exposure of public ip. Additional work required to tunnel all traffic with wireguard as I need to either use [VRF](https://networkengineering.stackexchange.com/questions/30596/vrfs-vlans-and-subnets-difference) using [routing and network namespacing](https://www.wireguard.com/netns/) or [create tunnels](https://discourse.nixos.org/t/route-all-traffic-through-wireguard-interface/1480/7) to forward traffic properly.

Issues:

- ECC was not properly detected so I am running it since it should be more reliable anyways
- Installers would not detect my keycron keyboard



