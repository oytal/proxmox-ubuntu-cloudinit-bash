# Proxmox-ubuntu-cloudinit-bash
Bash script that uses cloudinit to install a Ubuntu 20.04 vm on proxmox
Downloads newest daily build of Ubuntu Server 20.04
Some simple variables to set vm id, name, size etc.

If you prefer not downloading a new image every time just comment ut the wget and rm part.

## Usage

`wget https://raw.githubusercontent.com/oytal/proxmox-ubuntu-cloudinit-bash/main/script.sh`

Edit variables to match you needs

`bash script.sh`

---

## Script

```sh
#!/bin/bash

### Edit values below ###
id=999 #Unique proxmox ID 
name=ubuntu2004 #Machine name
ram=2048 #Amount of memory in megabytes
cpu=1 #Number of cpu cores
disk=18 #How much to expand the disk, default is 2G so 2+18=20GB total
dformat=qcow2 #Disk format, raw, qcow2 or vmdk
user=Ubuntu #Username
password=Ubuntu123 #Password
onboot=1 # 1 or 0 to enable/disable VM boot on proxmox startup
qemu_agent=1 # 1 or 0 for enable/disable of the qemu agent. Note: qemu-guest-agent needs to be installed in the VM afterwards
### Edit values above ###

idfree=`pvesh get /cluster/nextid -vmid $id`
if [ $idfree == $id ]; then
    echo -e "\e[32mVM ID free, running script. \e[0m]"
    echo -e "\e[33mDownloading Ubuntu 20.04 Cloud image. \e[0m"
    wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
    echo -e "\e[33mCreating VM. \e[0m"
    qm create $id --name $name --memory $ram --cores $cpu --net0 virtio,bridge=vmbr0
    qm importdisk $id focal-server-cloudimg-amd64.img local-lvm --format $dformat
    echo -e "\e[33mSetting parameters. \e[0m"
    qm set $id --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-$id-disk-0
    qm set $id --ide2 local-lvm:cloudinit
    qm set $id --boot c --bootdisk scsi0
    qm set $id --serial0 socket --vga serial0
    qm resize $id scsi0 +${disk}G
    qm set $id --ipconfig0 ip=dhcp
    qm set $id --sshkey ~/.ssh/id_rsa.pub
    qm set $id --cipassword $password
    qm set $id --onboot $onboot
    qm set $id --agent $qemu_agent
    echo -e "\e[33mCleaning up. \e[0m"
    rm focal-server-cloudimg-amd64.img
    echo -e "\e[33mStarting VM $name. \e[0m"
    qm start $id
    echo -e "\e[32mComplete. \e[0m"
else
    echo -e "\e[31mVM ID already exists, please choose an unused ID and run scrip again. \e[0m"
    echo -e "\e[31mAborted. \e[0m"
fi
```

---

My needs are pretty simple so i prefer this over template, I use Ansible to do everything else after VM is ready.

---

**Useful resources:**
- https://pve.proxmox.com/wiki/Cloud-Init_Support
- https://pve.proxmox.com/pve-docs/qm.1.html
- https://pve.proxmox.com/pve-docs/pvesh.1.html
- https://cloud-images.ubuntu.com/
- https://ubuntu.com/server/docs/cloud-images/introduction