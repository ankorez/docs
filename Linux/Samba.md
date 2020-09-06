{docsify-updated}
# Samba

### Installation
```bash
apt install samba
```
### Configuration
```bash
nano /etc/samba/smb.conf
```
Modifier ou ajouter ces lignes
```bash
   # Configuration
   server role = standalone server
   security = user
   # My Shares
[share1]
    comment = share
    path = /data/share1
    browsable = yes
    guest ok = yes
    read only = no
    create mask = 0755
    write list = ankorez

[share2]
    comment = share
    path = /data/share2
    browsable = yes
    guest ok = yes
    read only = no
    create mask = 0755
    write list = ankorez
```
### Create user
```bash
adduser ankorez #Créer l'utilisateur
passwd ankorez  #Definir le password
usermod -aG sudo ankorez  #Ajouter l'utilisateur sudo
adduser ankorez users     #Ajouter l'utilisateur au groupe users qui sera aussi le groupe du share
```
### Mount
 - fstab
```bash
//ipduserveursamba/share1 /mnt/share1 cifs username=ankorez,password=******** 0 0
```
