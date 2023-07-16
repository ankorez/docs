# Ubuntu 22.04 AutoInstall

## 1. Preseed YAML

### 2. Tiny PXE Menu Entry

### 3. Custom-startup.sh

### 4. custom-startup.service

## Quick documentation with new preseed

Source: [Ubuntu Autoinstall Reference](https://ubuntu.com/server/docs/install/autoinstall-reference)

### 1. Preseed YAML

#### Ubuntu Desktop with LVM Encryption and  Apps

**user-data**

We use YAML format for preseed now. We use two files:

1. user-data
2. meta-data

**meta-data must be present even empty**. We use the user-data YAML file for preseed. To access these two files in the iPXE menu entry, we need to add "/" at the end of the URL.

Put the ISO `ubuntu-22.04-live-server-amd64.iso` on Tiny PXE in the "iso" folder.

Extract `vmlinuz` and `initrd` into the casper folder from this ISO on Tiny PXE in the "ubuntu" folder.

User root is admin, and the password is encrypted. To encrypt the password, type this command in any SSH console and paste the result in the preseed:

    mkpasswd --method=sha-512 yourpassword

Passphrase is visible in preseed...

1. Preseed YAML Ubuntu Desktop with LVM encryption and Apps
   
file: user-data

    ```yaml
    #cloud-config
    autoinstall:
      apt:
        preserve_sources_list: false
        primary:
          - arches: [amd64]
            uri: "http://archive.ubuntu.com/ubuntu"
          - arches: [default]
            uri: "http://ports.ubuntu.com/ubuntu-ports"
        geoip: true
        sources:
          docker:
            source: "deb [arch=amd64] https://download.docker.com/linux/ubuntu jammy stable"
            key: |
              -----BEGIN PGP PUBLIC KEY BLOCK-----
              ...
              -----END PGP PUBLIC KEY BLOCK-----
          chrome:
            source: "deb [arch=amd64] https://dl.google.com/linux/chrome/deb/ stable main"
            key: |
              -----BEGIN PGP PUBLIC KEY BLOCK-----
              ...
              -----END PGP PUBLIC KEY BLOCK-----
    
    ## Identity:
    
    - **Hostname**: ubuntu-desktop
    - **Password**: $6$NnfCZ/UDKHoNot$q4cdmmQ/RoSl1FxEBif8vnDtgMBOZ5Os/5C5vPWleNWq2EPw/07vZsqn0.arhsKJjJIuDWh8YyuDRRZ
    - **Realname**: Admin
    - **Username**: admin
    
    ## Interactive Sections:
    
    - **Keyboard**:
      - Layout: fr
      - Toggle: null
      - Variant: ''
      - Locale: en_US.UTF-8
      - Timezone: Europe/Paris
    
    ## Storage:
    
    - **Grub**:
      - Update_nvram: true
      - Remove_duplicate_entries: true
      - Probe_additional_os: false
      - Reorder_uefi: false
    
    - **Swap**:
      - Filename: swap.img
      - Maxsize: 2GB
    
    - **Config**:
    
      - **Disks**:
        - id: disk0
        - type: disk
        - ptable: gpt
        - wipe: superblock
        - grub_device: false
        - match:
          - ssd: yes
          - size: largest
    
      - **Partitions**:
        - id: bios
        - type: partition
        - device: disk0
        - size: 1MB
        - flag: bios_grub
    
        - id: esp
        - type: partition
        - device: disk0
        - grub_device: true
        - size: 512MB
        - flag: boot
    
        - id: boot
        - type: partition
        - device: disk0
        - size: 1GB
    
        - id: pv
        - type: partition
        - device: disk0
        - size: -1
    
        - id: client_encrypted
        - type: dm_crypt
        - preserve: false
        - key: '12345678'
        - volume: pv
    
        - id: volumegroup
        - name: ubuntu-volumegroup
        - type: lvm_volgroup
        - devices: [client_encrypted]
        - preserve: false
    
        - id: lv_root
        - name: root
        - volgroup: volumegroup
        - size: 100%
        - type: lvm_partition
    
      - **Filesystems**:
        - id: esp_filesystem
        - type: format
        - volume: esp
        - fstype: fat32
        - label: EFI
    
        - id: boot_filesystem
        - type: format
        - volume: boot
        - fstype: ext4
    
        - id: root_filesystem
        - type: format
        - fstype: ext4
        - volume: lv_root
    
      - **Filesystem Mountpoints**:
        - id: esp_mount
        - type: mount
        - device: esp_filesystem
        - path: /boot/efi
    
        - id: boot_mount
        - type: mount
        - device: boot_filesystem
        - path: /boot
    
        - id: root_mount
        - type: mount
        - device: root_filesystem
        - path: /
    
    - **Late Commands**:
      - 'echo "admin ALL=(ALL) NOPASSWD:ALL" > /target/etc/sudoers.d/ubuntu-nopw'
      - chmod 440 /target/etc/sudoers.d/ubuntu-nopw
      - curtin in-target --target=/target -- sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT=""/GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"/' /etc/default/grub
      - curtin in-target --target=/target -- apt-get install -y ubuntu-desktop plymouth-theme-ubuntu-logo grub-gfxpayload-lists grub-efi-amd64-signed shim-signed
      - curtin in-target --target=/target -- cp /etc/apt/trusted.gpg /etc/apt/trusted.gpg.d/
      - curtin in-target --target=/target -- apt update && apt upgrade -y
      - curtin in-target --target=/target -- apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
      - curtin in-target --target=/target -- apt install -y google-chrome-stable docker-ce docker-ce-cli containerd.io
      - curtin in-target --target=/target -- rm /etc/apt/sources.list.d/chrome.list
      - curtin in-target --target=/target -- wget -O /etc/openvpn/client/ca.crt http://10.65.2.108/sources/vpn/ca.crt
      - curtin in-target --target=/target -- wget -O /etc/openvpn/client/client092029.crt http://10.65.2.108/sources/vpn/client092029.crt
      - curtin in-target --target=/target -- wget -O /etc/openvpn/client/client092029.key http://10.65.2.108/sources/vpn/client092029.key
      - curtin in-target --target=/target -- wget -O /etc/openvpn/client/vpn.ovpn http://10.65.2.108/sources/vpn/vpn.ovpn
      - curtin in-target --target=/target -- wget -O /etc/systemd/system/custom-startup.service http://10.65.2.108/preseed/ubuntu/custom-startup.service
      - curtin in-target --target=/target -- wget -O /root/custom-startup.sh http://10.65.2.108/preseed/ubuntu/custom-startup.sh
      - curtin in-target --target=/target -- chmod +x /root/custom-startup.sh
      - curtin in-target --target=/target -- systemctl daemon-reload
      - curtin in-target --target=/target -- systemctl enable custom-startup
      - curtin in-target --target=/target -- rm /etc/netplan/*.yaml
      - curtin in-target --target=/target -- wget -O /etc/netplan/01-network-manager-all.yaml http://10.65.2.108/ubuntu/preseed/01-network-manager-all.yaml
      - curtin in-target --target=/target -- netplan generate
      - curtin in-target --target=/target -- netplan apply
      - curtin in-target --target=/target -- apt update && apt upgrade -y
      - reboot
    
    ## Version: 1


### 2. PXE Menu Entry

    :ubuntu-2204 
    kernel ${boot-url}/ubuntu/vmlinuz
    initrd ${boot-url}/ubuntu/initrd
    imgargs vmlinuz initrd=initrd ip=dhcp cloud-config-url=/dev/null url=${boot-url}/iso/ubuntu-22.04-live-server-amd64.iso boot

### 3. Custom-startup.sh
    This script is to install some applications/stuff not deployable during the master

    #!/bin/bash
    
    ## VPN profil
    nmcli connection import type openvpn file /etc/openvpn/client/ManoMano.ovpn
    
    # Wallpaper
    wget -O /tmp/manomano.png http://10.65.2.108/sources/wallpaper/manomano.png
    sudo mv /usr/share/backgrounds/warty-final-ubuntu.png /usr/share/backgrounds/warty-final-ubuntu-old.png
    sudo cp /tmp/manomano.png /usr/share/backgrounds/warty-final-ubuntu.png
    
    # Install Desktop Central Agent
    wget -O DesktopCentral_LinuxAgent.bin http://10.65.2.108/sources/dcagent/DesktopCentral_LinuxAgent.bin
    wget -O serverinfo.json http://10.65.2.108/sources/dcagent/serverinfo.json
    sudo chmod +x DesktopCentral_LinuxAgent.bin
    sudo ./DesktopCentral_LinuxAgent.bin
    
    # Install Slack
    wget https://downloads.slack-edge.com/linux_releases/slack-desktop-4.13.0-amd64.deb
    sudo apt install ./slack-desktop-*.deb
    
    # Zoom-client
    wget https://zoom.us/client/latest/zoom_amd64.deb
    sudo apt install -y ./zoom_amd64.deb
    
    # Installation Keeper
    sudo wget -O keeperpasswordmanager.deb http://10.65.2.108/sources/keeper/keeperpasswordmanager_16.7.1_amd64.deb
    sudo chmod +x keeperpasswordmanager.deb
    sudo dpkg -i keeperpasswordmanager.deb
    
    # Install Display Link
    wget -O /tmp/displaylink.run http://10.65.2.108/sources/displaylink/displaylink-driver-5.6.0-59.176.run
    sudo chmod +x /tmp/displaylink.run
    sudo sh /tmp/displaylink.run

# Rename Computer
    OLD_HOSTNAME=$(hostname)
    MY_HOSTNAME="ML-$(dmidecode -s system-serial-number | cut -d'-' -f2- | tr -d ' ')"
    sudo sed -i "s/$OLD_HOSTNAME/$MY_HOSTNAME/g" /etc/hosts
    sudo hostnamectl set-hostname $MY_HOSTNAME
    echo $MY_HOSTNAME

## Remove custom startup
    sudo rm zoom_amd64.deb
    sudo rm slack-desktop-*.deb
    rm keeperpasswordmanager.deb
    rm DesktopCentral_LinuxAgent.bin
    rm serverinfo.json
    systemctl disable custom-startup
    rm /etc/systemd/system/custom-startup.service
    rm /root/custom-startup.sh
### 4. Custom-startup.service
    [Unit]
    Description=Custom Startup
    
    [Service]
    ExecStartPre=/bin/sleep 15
    ExecStart=/root/custom-startup.sh
    
    [Install]
    WantedBy=multi-user.target

