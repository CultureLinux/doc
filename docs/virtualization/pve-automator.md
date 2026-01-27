# pve-automator

## Préparation de l'iso
### Installation de l'assistant

```
apt install proxmox-auto-install-assistant
```

### Modification de l'iso

```
proxmox-auto-install-assistant prepare-iso proxmox-ve_9.1-1.iso \
    --output proxmox-ve-auto_9.1-1.iso \
    --fetch-from http --url "https://pve-automator.local.clinux.fr/answer"
```

## Création de la clé usb
### Détection de la clé

```
lsblk
```

### Ecriture de l'iso

```
dd if=/root/proxmox-ve-auto_9.1-1.iso of=/dev/sdd bs=4M status=progress oflag=sync
sync
eject /dev/sdd
```
