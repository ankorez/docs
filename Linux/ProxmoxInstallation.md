# Proxmox - New installation
## Table of contents
##### Installation

##### Enable ZFS data pool storage

##### Copy SSH Keys

##### Configuration de apparmor

##### Backup the VM to NFS Share NAS01

##### IPMI Interface

##### Issue with NFS Backup

##### Config Reverse Proxy Nginx

##### VM Windows 10 on Proxmox

##### Remove Proxmox Node from web interface

##### Add Proxmox ISO to PXE

We have 4 supermicro servers in cluster pxmxadm01 pxmxadm02 pxmxadm03 and pxmxadm04 in clusteradm Each Supermicro have 2 SSD 32GB for system and 2 SSD 2TO for data (VM)

By IPMI or physically boot on USB with Proxmox OS Installer or use pxe for boot on proxmox

##### **Installation**

 - Select Install Proxmox VE
 - Agree
  - Clic on Options then RAID1

Set IP and others informations

After installation go on https://ipofproxmox:8006 then create ZFS RAID1 on 2TO SSD (if we don’t see any disk we need to run cfdisk in ssh and remove previously partitions created on them.

Connect to proxmox in SSH and add this line to /etc/apt/sources.list (for Debian 10)

    # PVE pve-no-subscription repository provided by proxmox.com, # NOT recommended for production use
    
    deb http://download.proxmox.com/debian/pve buster pve-no-subscription
Remove

    rm /etc/apt/sources.list/pve-entreprise.list
Create bridge vmbr1 on each node
Then create and reboot for apply

Edit each node /etc/hosts with hostname and fqdn

    10.65.1.100 pxmx01.mondomaine.com pxmx01
    10.65.1.200 pxmx02.mondomaine.com pxmx02  
    10.65.1.150 pxmx03.mondomaine.com pxmx03
    10.65.2.100 pxmx04.mondomaine.com pxmx04
    
    #Cluster
    
    172.52.0.1 pxmx01
    172.52.0.2 pxmx02
    172.52.0.3 pxmx03
    172.52.0.4 pxmx04

After restart return on web interface and Create Cluster
and create
Copy past Join Information
Then on each other node Join Cluster
And That’s It

##### Enable ZFS data pool storage
After created cluster “data” pool is not visible on new node
go on Datacenter then clic on data

Add new node in the list for exemple pxmx02
after that we can see the data pool on the new node
##### Copy SSH Keys

copy ssh keys from another nodes to the new node

copy ssh keys from new node to another nodes

copy ssh keys from all sysadmins

##### Configuration de apparmor
Add in the file /etc/apparmor.d/lxc/lxc-default-cgns the two lines :

    mount fstype=rpc_pipefs,
    
    mount fstype=nfs,
After the line

    mount fstype=cgroup -> /sys/fs/cgroup/**,
Here is the complete file:

    # Do not load this file. Rather, load /etc/apparmor.d/lxc-containers, which
    
    # will source all profiles under /etc/apparmor.d/lxc
    
    profile lxc-container-default-cgns flags=(attach_disconnected, mediate_deleted) {
    
    #include <abstractions/lxc/container-base>
    
    # the container may never be allowed to mount devpts. If it does, it # will remount the host's devpts. We could allow it to do it with # the newinstance option (but, right now, we don't).
    
    deny mount fstype=devpts,
    
    mount fstype=cgroup -> /sys/fs/cgroup/**,
    
    mount fstype=rpc_pipefs,
    
    mount fstype=nfs,
    }
Redémarrer le service apparmor

    systemctl restart apparmor
Usage
##### Backup the VM to NFS Share NAS01

![](https://lh3.googleusercontent.com/vtI7Nz42Db9_p9SAPeZBVncijJfPvEVlqkbKM0uswWeVdmhqo6XGCYq7Rkdyw7jyBQruVloUvomDMG2w3hZXnJdRBqjMnYdciM3eipm5MKyp2Yt5a5FpyS1Y9WGMQLbyu6tmvCl63F2fG1LMPW_SeZg)Using NFS for backup is not secure enough. Please refer to this documentation: Proxmox - Backup & Restore

Datacenter > Storage > Add > NFS

ID : backup-clusteradm

Server : 10.65.0.11

Export : /volume1/it/backupvm/backup-cluster-adm/

Content : VZDump backup file

Max Backups : 7

Datacenter > Backup > Add

Storage :backup-cluster-adm/

Day of week : Everyday

Start time : 00:00

Selection mode : All

leaves the rest and click on Create.

##### IPMI Interface

IPMI allows access to hypervisors via a management interface.

The 4 proxmoxs installed above are connected to our management switch

    192.168.52.1 pour pxmx01
    
    192.168.52.2 pour pxmx02
    
    192.168.52.3 pour pxmx03
    
    192.168.52.4 pour pxmx04
    
Windows : https://www.supermicro.com/SwDownload/SwSelect_Free.aspx?cat=IPMI

Linux : apt install ipmitool

https://linux.die.net/man/1/ipmitool

##### Issue with NFS Backup

Backup VM with unpriviliged container checked on proxmox system local storage

Restore VM and uncheck unpriviliged container before run restore on ZFS volume

Go on Features and check "NFS"

##### Config Reverse Proxy Nginx
    
       server {
    
    listen 80;
    
    listen 443 ssl;
    
    server_name toto.mondomaine.com;
    
    if ($scheme != 'https') { rewrite ^ https://$host$request_uri? permanent; }
    
    ssl on;
    
    ssl_certificate /mnt/certif/mondomaine.com/fullchain.pem; ssl_certificate_key /mnt/certif/mondomaine.com/privkey.pem;
    
    location / {
    
    proxy_set_header Host $host;
    
    proxy_set_header X-Real-IP $remote_addr;
    
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header X-Forwarded-Proto https;
    
    proxy_pass https://10.67.1.100:8006;
    
    proxy_read_timeout 90;
    
    # Enable proxy websockets for the noVNC console to work
    
    proxy_http_version 1.1;
    
    proxy_set_header Upgrade $http_upgrade;
    
    proxy_set_header Connection "upgrade";
    
    }
    
    }
##### VM Windows 10 on Proxmox

Preparation

To get a good level of performance we need to install the VitrIO drivers during the Windows installation available https://fedorapeople.org/groups /virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso

Upload iso on Proxmox

Create a new VM and select "Microsoft Windows 10/2016 continue and mount our Windows 10 ISO in the Virtual CDROM For our virtual disk we need to select "VirtIO" as bus and "Write back" as cache for better performance (the default no Cache is ok but slower). Let's configure the desired RAM settings and set the network device to "VirtIO (paravirtualized)".

Once the VM is created, we need to edit it and add a second virtual CDrom in the Hardware tab and mount the Vitrio Drivers iso Edit the hardware of the VM and switch the Keyboard to French

Now we just have to turn on our VM and launch the installation of Windows 10

Installation Windows 10

After booting our VM you have to open the console

Continue the installation steps by selecting "custom" until you choose on which disk to install Windows

Click on "load a driver" and browse the VirtIO CD

for storage: viostor\w10\amd64\

for network : NetKVM\w10\amd64\

for memory : Balloon\w10\amd64\

once we have loaded all the drivers we can select the disk on which we want to install Windows

Install VirtIO drivers on an already installed system

Browse the file explorer, select the virtual cd drive where the VirtIO iso is mounted

Ethernet adapter : NetKVM\w8.1\amd64 here right click on the installation information file "netkvm" and select install. Virtual memory balloon driver: Balloon\w8.1\amd64 here right click on the installation information file "balloon" and select install.

##### Remove Proxmox Node from web interface

Step 1 : Migrate all VMs to another active node

Migrate all VMs to another active node. You can use the live migration feature if you have a shared storage or offline migration if you only have local storage.

Step 2 : Display all active nodes

Display all active nodes in order identify the name of the node you want to remove

    root@proxmox-node2:~# pvecm nodes
    
    Membership information
    
    ----------------------
    
    Nodeid Votes Name
    
    1 1 proxmox-node1 (local)
    
    2 1 proxmox-node2
    
    3 1 proxmox-node3
    
    4 1 proxmox-node4
Step 3: Shutdown (permanently) the node that you want to remove

Please be carefull, it a permanently remove !!!

Never restart the removed node

Don’t assign the local ip address of the removed node to a new node

Never assign the name of the removed node to a new node

Step 4 : Remove the node from the proxmox cluster

Connect to an active node, for example proxmox-node2.

    root@proxmox:~# pvecm delnode NodeName
For Example :

    root@proxmox-node2:~# pvecm delnode proxmox-node3
Step 5 : Remove the removed node from the proxmox GUI

Log in to an active node, for example proxmox-node2.

    root@proxmox-node2:~# ls -l /etc/pve/nodes/
    
    proxmox-node1 proxmox-node2 proxmox-node3 proxmox-node4
All nodes have is own directory (VM’s inventory, for example), the directory /etc/pve/nodes/ is synced between all cluster nodes. The removed node is still visible in GUI until the node directory exists in the directory /etc/pve/nodes/.

If you want to remove from Proxmox GUI the node previously deleted , you just need to delete the directory /etc/pve/nodes/NodeName.

    root@proxmox-node2:~# mv /etc/pve/nodes/NodeName /root/NodeName
Src: https://sysadmin-community.com/remove-node-from-cluster-proxmox/

##### Add Proxmox ISO to PXE

https://github.com/morph027/pve-iso-2-pxe
