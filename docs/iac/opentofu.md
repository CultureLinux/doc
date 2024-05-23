# OpenTofu
## Github
    wget https://github.com/opentofu/opentofu/releases/download/v1.6.2/tofu_1.6.2_linux_amd64.zip
    unzip tofu*
    cp tofu /usr/local/bin/

## Examples

* ✅ [GitHub repo](https://github.com/CultureLinux/opentofu)

## Providers

* ☠️ [Telmate](https://github.com/Telmate/terraform-provider-proxmox)
* ✅ [BPG](https://github.com/bpg/terraform-provider-proxmox)

### Add role
    pveum role add OpenTofu -privs "Datastore.Allocate Datastore.AllocateSpace Datastore.AllocateTemplate Datastore.Audit Pool.Allocate Sys.Audit Sys.Console Sys.Modify SDN.Use VM.Allocate VM.Audit VM.Clone VM.Config.CDROM VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Migrate VM.Monitor VM.PowerMgmt User.Modify"

    or 

    Use PVEAdmin
### Add user
    pveum user add opentofu@pve
### link role user
    pveum aclmod / -user opentofu@pve -role OpenTofu
### generate token
    pveum user token add opentofu@pve opentofu -expire 0 -privsep 0 -comment "OpenTofu token"
### test token
    curl -X GET 'https://$PROXMOX_URL:8006/api2/json/nodes' -H 'Authorization: PVEAPIToken=opentofu@pve!opentofu=$TOKEN'


## Exemple 
### Avec variables static
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

#### vars.tf
    variable  "pm_api_url" {
        default =  "https://$PROXMOX_URL:8006/api2/json"
    }
    variable  "pm_api_token" {
        default =  "opentofu@pve!opentofu=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
    }
    variable  "target_node" {
        default =  "proxmox2"
    }
    variable  "ssh_key" {
        default =  "ssh-ed25519 XXXXXXXX clinux@rocky9.local"
    }

### Avec variables d'environnement
#### envvars 
    export PROXMOX_VE_ENDPOINT='https://PROXMOX_URL:8006/'
    export PROXMOX_VE_API_TOKEN='USER@REMAM!TOKEN_KEY=TOKEN_VALUE'

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
        insecure = false  # Warning TLS cert of proxmox webui must be valid (Let's Encrypt)
    }


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
