{docsify-updated}
# RAID MDADM

- Récupérer facilement une grappe raid après réinstallation du système

### Installation
```bash
apt install mdadm
```
### Configuration
```bash
reboot
```
### Mount
- create /data
```bash
mkdir /data
```
- fstab
```bash
nano /etc/fstab
```
- Ajouter
```bash
/dev/md0      /data      ext4      defaults      0      0
```
### Other
- Check RAID status
```bash
cat /proc/mdstat
```
- Check RAID status in live
```bash
watch -n 1 cat /proc/mdstat
```
- RAID détails
```bash
mdadm --detail /dev/md0
```
### Source
https://wiki.debian-fr.xyz/Raid_logiciel_(mdadm)
