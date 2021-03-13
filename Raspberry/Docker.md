# Docker avec Home Assistant, Plex et Transmission
{docsify-updated}
v1.0

Bye Bye Proxmox. Welcome Docker. Je vais utiliser Docker pour installer Home Assistant, Plex et Transmission sur mon Raspberry Pi 4. 

**Prérequis :**
- 1 RPi 4
- 1 HDD ou SSD Externe

*Sources*
https://forum.hacf.fr/t/installer-home-assistant-sur-rpi-ou-autres-sbc-debian-methode-docker-supervisor/676
https://hub.docker.com/r/linuxserver/plex/tags
https://github.com/linuxserver/docker-plex/issues/247
https://hub.docker.com/r/linuxserver/transmission

## Steps
- Installer Raspbian sur la carte micro SD
- Installer Docker
- Installer image Docker Home Assistant
- Installer image Docker Plex
- Installer image Docker Transmission
- Installer HACS dans Home Assistant
- Configurer les intégrations (Samba, MyEnedis, Philips Hue, etc...)

### Installation Raspbian OS
- Télécharger la dernière version de Raspbian Lite (sans interface graphique). https://raspberry-pi.fr/telechargements/
- Flasher l'image Raspbian Lite sur la carte micro SD avec https://www.balena.io/etcher/
- Insérer la carte micro SD dans le Raspberry et booter.
- Login `pi`
- Password `raspberry` (attention par défaut le clavier est en QWERTY).
- Taper la commande `raspi-config` pour changer la disposition du clavier et activer le SSH
- Se connecter dessus en SSH.
```bash
ssh root@ipdurpi
sudo apt update && sudo apt upgrade -y
passwd
newpassword
confirmnewpassword
```
### Installation Docker
```
sudo apt-get install -y software-properties-common apparmor-utils apt-transport-https avahi-daemon ca-certificates curl dbus jq network-manager
sudo systemctl disable ModemManager 
sudo systemctl stop ModemManager
curl -fsSL http://get.docker.com/ -o get-docker.sh && sh get-docker.sh
sudo usermod -aG docker pi 
sudo systemctl enable docker 
sudo systemctl start docker
```
Note : si il y'a une erreur à l'installation de Docker il faut juste rebooter le RPi et relancer l'installation. (J'ai remarqué que si je ne reboot pas le RPi après le `sudo apt update && sudo apt upgrade -y` Docker ne s'installe pas).

### Installation Home Assistant
```
curl -sL « https://raw.githubusercontent.com/home-assistant/supervised-installer/master/installer.sh » >> hassio_install.sh
sudo bash hassio_install.sh -m raspberrypi4
```
A la fin de l'installation on lance `sudo docker ps` pour vérifier que les containers home assistant sont bien lancés. On peut maintenant s'y connecter http://192.168.x.x/8123 et suivre les instructions.

### Installation Plex
```
docker pull linuxserver/plex:bionic
sudo docker create --restart=always --name plex -v /mnt/usb-drive:/media -v /home/pi/plex/config:/config --net=host linuxserver/plex:bionic
sudo docker start plex
```
L'argument `-v /mnt/usb-drive:/media` permet de monter un disque USB connecté et monté  sur le Host (le Raspberry)
L'argument `/home/pi/plex/config:/config` permet de sauvegarder la config du container et ainsi garder les settings lorsque je mettrai à jour l'image du container. J'ai créé des folders séparés dans le home de pi pour chaque container.
Il faut choisir le tag `linuxserver/plex:bionic` car avec les version `armv7` ça ne fonctionnait pas.
On vérifie que le container Plex est bien lancé `sudo docker ps -a`
On se connecte sur http://192.168.x.x:32400/web/index.html et on suit les instructions.
Si le first setup de Plex bloque au tout debut il faut lancer `sudo apt install libseccomp2`

### Installation Transmission
```
docker pull linuxserver/transmission
sudo docker create --name=transmission --restart=always -v /home/pi/transmission/config:/config -v /mnt/usb-drive:/downloads -v /home/pi/transmission/watch:/watch -e PGID=1000 -e PUID=1000 -e TZ=Europe/Paris -p 9091:9091 -p 51413:51413 -p 51413:51413/udp linuxserver/transmission
sudo docker start transmission
```
L'argument `-v /mnt/usb-drive:/downloads` permet de monter un disque USB connecté et monté sur le Host (le Raspberry) 
L'argument `/home/pi/transmission/config:/config` permet de sauvegarder la config du container et ainsi garder les settings lorsque je mettrai à jour l'image du container. J'ai créé des folders séparés dans le home de pi pour chaque container.
L'argument `-e PGID=1000 -e PUID=1000` correspond au user pi pour vérifier les valeurs on tape la commande `id pi` sur le RPi.

### Installation HACS

TODO

### Intégrations Home Assistant

TODO