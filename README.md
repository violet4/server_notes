# server_notes

this is where notes go to get my filesystems bootstrapped before i can run my programs/servers/services/docker/etc.

most of my applications/data/etc are run from docker containers on a hard drive running zfs with automatic incremental snapshots that can be pushed over the network, but we recently (2025-04-20) updated our server setup and the motherboard the original server was on was misbehaving. so now i have disembodied HDDs/SSDs. the SSD is running the OS on LVM, the docker data is on zfs HDD. then we need notes outside of those two systems that we can use to get ourselves bootstrapped into those.

# LVM

SSD/OS

```bash
vgscan
vgchange -ay vg0
lvs
mount /dev/vg0/...
```

```bash
borealis / # vgscan
  Found volume group "vg0" using metadata type lvm2
borealis / # vgchange -ay vg0
  3 logical volume(s) in volume group "vg0" now active
borealis / # lvs
  LV        VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-0      vg0 -wi-a----- 100.00g                                                    
  lv-1      vg0 -wi-a----- 400.00g                                                    
  lv-varlog vg0 -wi-a-----  50.00g

borealis /mnt/msi # mount /dev/vg0/lv-0 0
borealis /mnt/msi # mount /dev/vg0/lv-1 1
borealis /mnt/msi # mount /dev/vg0/lv-varlog varlog/
```


# ZFS

dedicated `zfsbackup` user with cron job in `/var/spool/cron/crontabs/zfsbackup`:
```
# backup local dockerz/docker dataset to dockerz2 pool on host 192.168.1.121
43 * * * * syncoid --no-resume dockerz/docker 192.168.1.121:dockerz2
```
