# pve-automator

## Préparation de l'iso
### Installation de l'assistant

```
apt install proxmox-auto-install-assistant
```

### Modification de l'iso (certificat valide)

```
proxmox-auto-install-assistant prepare-iso proxmox-ve_9.1-1.iso \
    --output proxmox-ve-auto_9.1-1.iso \
    --fetch-from http --url "https://pve-automator.local.clinux.fr/answer"
```

### Modification de l'iso (certificat autosigné sha-256)

```
proxmox-auto-install-assistant prepare-iso \
    proxmox-ve_9.1-1.iso \
    --output proxmox-ve-auto-self_9.1-1.iso \
    --fetch-from http \
    --url "https://pve-automator.local.clinux.fr:8000/answer" \
    --cert-fingerprint "BE:40:80:2F:42:6E:AC:A7:97:DF:8B:56:40:15:17:39:42:02:E4:54:06:CD:C0:CA:6D:FE:96:08:C5:93:12:E7"
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
