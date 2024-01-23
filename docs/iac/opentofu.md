# OpenToFu
## Install
    wget https://github.com/opentofu/opentofu/releases/download/v1.6.0-alpha5/tofu_1.6.0-alpha5_linux_amd64.zip
    unzip tofu*
    cp tofu /usr/local/bin/

## Providers

* ☠️ https://github.com/Telmate/terraform-provider-proxmox
* ✅ https://github.com/bpg/terraform-provider-proxmox

## Auth (Proxmox >= 8.1)
### Add role
    pveum role add tofu -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit SDN.Use"
### Add user
    Interface Add PAM user
### link role user
    pveum aclmod / -user tofu@pam -role tofu
### generate token
    pveum user token add tofu@pam fortofu -expire 0 -privsep 0 -comment "Tofu token"
### test token
    curl -X GET 'https://$PROXMOX_URL:8006/api2/json/nodes' -H 'Authorization: PVEAPIToken=terraform@pve!terraform=$TOKEN'


### Files 
#### provider.tf
    terraform {
        required_providers {
            proxmox =  {
            source = "bpg/proxmox"
            version = ">= 0.38.1"
            }
        }
    }
    provider "proxmox" {
        endpoint = var.pm_api_url
        api_token = var.pm_api_token
        insecure = false  # Warning TLS cert of proxmox webui must be valid (Let's Encrypt)
        ssh {
            agent    = true
            username = "root"
        }
    }

### vars.tf
    variable  "pm_api_url" {
        default =  "https://$PROXMOX_URL:8006/api2/json"
    }
    variable  "pm_api_token" {
        default =  "tofu@pam!fortofu=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
    }
    variable  "target_node" {
        default =  "proxmox2"
    }
    variable "clone" {
        default =  "debian12-temp"
    }
    variable  "ssh_key" {
        default =  "ssh-ed25519 XXXXXXXX clinux@rocky9.local"


### main.tf
    resource "proxmox_virtual_environment_vm" "debian_vm" {
        name        = "bpg-debian12"
        description = "Managed by opentofu"
        tags        = ["opentofu", "debian"]
        node_name = var.target_node
        clone {
            vm_id = "1000"
        }

        cpu {
            cores = 2
            type = "host"
            numa = true
        }
        memory {
            dedicated = 2048
        }
        network_device {
            bridge = "vmbr0"
            model = "virtio"
        }

        efi_disk {
            datastore_id = "local-lvm"
            file_format = "raw"
            type    = "4m"
        }

        disk {
            datastore_id = "local-lvm"
            file_format = "raw"
            interface = "scsi0"
            size = "10"
        }

        operating_system {
            type = "l26"
        }
        machine = "q35"
        agent {
            enabled = false
        }

        initialization {
            ip_config {
                ipv4 {
                    address = "192.168.1.170/24"
                    gateway = "192.168.1.1"
                }
            }
            user_account {
                keys     = [var.ssh_key]
                password = "tofu"
                username = "debian"
            }
        }
    }
