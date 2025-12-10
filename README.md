# DevPod Machine Provider configuration for Proxmox

This provider will:
* Clone your template VM for each workspace
* Configure it with cloud-init
* Inject SSH keys automatically
* Detect the VM's IP address via the QEMU guest agent
* Manage the full lifecycle

The provider uses standard Proxmox API calls and should work with any Proxmox VE 6.x or 7.x installation.

## Prerequisites:
Before using this provider, you need:
* A cloud-init enabled VM template in Proxmox (typically ID 9000)

The template should have:

* Cloud-init drive configured
* QEMU guest agent installed
* SSH server enabled

Run these commands from the Proxmox server shell to set up the VM template:

````
# Go to home folder
cd /root

# download the image
wget https://dl-cdn.alpinelinux.org/alpine/v3.22/releases/cloud/generic_alpine-3.22.2-x86_64-bios-cloudinit-r0.qcow2

# create a new VM with VirtIO SCSI controller
qm create 9000 --memory 2048 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci

# import the downloaded disk to the local-lvm storage, attaching it as a SCSI drive
qm set 9000 --scsi0 local-lvm:0,import-from=/root/generic_alpine-3.22.2-x86_64-bios-cloudinit-r0.qcow2

#configure a CD-ROM drive, which will be used to pass the Cloud-Init data to the VM.
qm set 9000 --ide2 local-lvm:cloudinit

#set boot parameter to order=scsi0 to boot from this disk only to speed up booting
qm set 9000 --boot order=scsi0

#configure a serial console and use it as a display
qm set 9000 --serial0 socket --vga serial0

#convert the VM into a template
qm template 9000
````

# Usage:
Save provider.yaml and add as provider with DevPod.
