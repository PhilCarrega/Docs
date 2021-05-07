# Zpool

## Création d'un pool

## Ajout d'un disque
* https://docs.oracle.com/cd/E53394_01/html/E54801/gayrd.html

## Etendre un disque
* https://memo-linux.com/proxmox-etendre-lespace-disque-avec-lvm/

# Acces wmfs

* Selon la version de vmWare :
```bash
apt install vmfs-tools
```

```bash
apt install vmfs6-tools
```
Attention : Le montage wmfs6 n'est pas possible sur le pve ou sur un container, obligé de passer par une VM et de monter le disk en raw.

* Création rep + mountage

```bash
mkdir /mnt/wmfs
```
* Recherche la partition à monter 

  ```bash
  fdisk -l
  ```

Repérer la partition de données et la monter :

```bash
vmfs-fuse /dev/sdb3 /mnt/vmfs
ou
vmfs6-fuse /dev/sdb3 /mnt/vmfs
```
* Source : <http://woshub.com/how-to-access-vmfs-datastore-from-linux-windows/>

# Ajout disk en RAW
```bash
ls /dev/disk/by-id/
```
  
Répérer le bon disque et :
```bash
qm set VM-ID -virtio2 /dev/disk/by-id/DISK-ID
```
  
* Source : <https://johnkeen.tech/proxmox-physical-disk-to-vm-only-2-commands/>
