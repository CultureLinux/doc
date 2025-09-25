# Grub
## Kernel en cours 
    uname -r 
## Kernels installés
```
    ls -l /boot/vmlinuz-*
```
### Rocky 9

```
/boot/loader/entries/858382f092494811bf89e090de079ab1-0-rescue.conf:title Rocky Linux (0-rescue-858382f092494811bf89e090de079ab1) 9.5 (Blue Onyx)
/boot/loader/entries/858382f092494811bf89e090de079ab1-5.14.0-503.14.1.el9_5.x86_64.conf:title Rocky Linux (5.14.0-503.14.1.el9_5.x86_64) 9.5 (Blue Onyx)
/boot/loader/entries/a343998b88a34ce78a8a06339e65eeab-5.14.0-503.33.1.el9_5.x86_64.conf:title Rocky Linux (5.14.0-503.33.1.el9_5.x86_64) 9.5 (Blue Onyx)
/boot/loader/entries/a343998b88a34ce78a8a06339e65eeab-0-rescue.conf:title Rocky Linux (0-rescue-a343998b88a34ce78a8a06339e65eeab) 9.5 (Blue Onyx)
```

### Debian 12 (proxmxo)
```
grep -P "submenu|menuentry" /boot/grub/grub.cfg | cut -d "'" -f2
```

## Changer le kernel
### Rocky9
#### Prochain kernel au boot
    grub2-editenv list
#### Forcer le prochain kernel au boot
    grub2-reboot "Rocky Linux (5.14.0-503.14.1.el9_5.x86_64) 9.5 (Blue Onyx)"
#### Changer definitivement le kernel au boot
    grub2-set-default "Rocky Linux (5.14.0-503.33.1.el9_5.x86_64) 9.5 (Blue Onyx)"

### Proxmox 8
#### Prochain kernel au boot
```
grep GRUB_DEFAULT /etc/default/grub 
```
```
0
```
* C'est le 1er koernel qui va booter
```
grep -P "submenu|menuentry" /boot/grub/grub.cfg
```
```
if [ x"${feature_menuentry_id}" = xy ]; then
  menuentry_id_option="--id"
  menuentry_id_option=""
export menuentry_id_option
menuentry 'Proxmox VE GNU/Linux' --class proxmox --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-f768d6b7-5898-4c6b-827d-aab79581f769' {
submenu 'Advanced options for Proxmox VE GNU/Linux' $menuentry_id_option 'gnulinux-advanced-f768d6b7-5898-4c6b-827d-aab79581f769' {
        menuentry 'Proxmox VE GNU/Linux, with Linux 6.8.12-9-pve' --class proxmox --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-6.8.12-9-pve-advanced-f768d6b7-5898-4c6b-827d-aab79581f769' {
        menuentry 'Proxmox VE GNU/Linux, with Linux 6.8.12-9-pve (recovery mode)' --class proxmox --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-6.8.12-9-pve-recovery-f768d6b7-5898-4c6b-827d-aab79581f769' {
        menuentry 'Proxmox VE GNU/Linux, with Linux 6.8.12-8-pve' --class proxmox --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-6.8.12-8-pve-advanced-f768d6b7-5898-4c6b-827d-aab79581f769' {
        menuentry 'Proxmox VE GNU/Linux, with Linux 6.8.12-8-pve (recovery mode)' --class proxmox --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-6.8.12-8-pve-recovery-f768d6b7-5898-4c6b-827d-aab79581f769' {
```
* Ici ca sera `Proxmox VE GNU/Linux, with Linux 6.8.12-9-pve`
#### Forcer le prochain kernel au boot
    grub2-reboot "Rocky Linux (5.14.0-503.14.1.el9_5.x86_64) 9.5 (Blue Onyx)"
#### Changer definitivement le kernel au boot
    grub2-set-default "Rocky Linux (5.14.0-503.33.1.el9_5.x86_64) 9.5 (Blue Onyx)"