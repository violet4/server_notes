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

dedicated `zfsbackup` users on both sending and receiving machines:
```bash
# we need a home folder to set up ssh keys
useradd --system --create-home zfsbackup
```

cron job in `/var/spool/cron/crontabs/zfsbackup`:
```
# backup local dockerz/docker dataset to dockerz2 pool on host 192.168.1.121
43 * * * * syncoid --no-resume dockerz/docker 192.168.1.121:dockerz2

# useful setting?
# --keep-sync-snap - don't delete (local? remote? both?)
```

extra zfs goodies:

* `zfs list -r dockerz` recursive listing, including sub-datasets

# podman

podman compose syntax docs: https://github.com/compose-spec/compose-spec/blob/main/spec.md#compose-file

used to use docker, now podman for better security and control. this 

by default, podman has restrictions or odd behaviors that aren't compatible with how i usually use `docker-compose.yml`.

in `docker-compose.yml` we typically have:

```yaml
services:
  someservice:
    environment:
      # inside someservice container, it will attempt to reach somedatabase container via DNS hostname `somedatabase`
      - BACKEND_HOST_NAME=somedatabase
    ...
  somedatabase:
    ...
```

however, by default, podman won't let the containers communicate with each other. we have to explicitly set up the networks:

```yaml
services:
  someservice:
    environment:
      - BACKEND_HOST_NAME=somedatabase
    networks:
      - my-compose-stack-network
    
  somedatabase:
    networks:
      - my-compose-stack-network

networks:
  my-compose-stack-network:
    name: my-compose-stack-network
    external: false
```

old hacky workaround for networking issues:

```yaml
# https://github.com/containers/podman-compose/pull/964/files
x-podman:
    in_pod: false
```
