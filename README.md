# Proxmox Cloud-Init Template Ansible Role

This Ansible role automates the creation and configuration of a cloud-init template for use in Proxmox Virtual Environment (Proxmox VE). It handles tasks such as downloading the cloud image, resizing the disk, creating the VM template, configuring network settings, adding user credentials, and more. The role sets up the Proxmox VM to be used as a cloud-init enabled template.

---

## Role Overview

The role performs the following tasks:

- Downloads the cloud image template and stores it on the Proxmox server.
- Resizes the cloud image disk to the specified size.
- Creates a Proxmox virtual machine (VM) and configures it for cloud-init.
- Configures network settings, SSH keys, user credentials, and vendor data for cloud-init.
- Converts the VM into a template that can be used to create new VMs.

This role ensures that the VM template is configured to work seamlessly with cloud-init for automating cloud-based setups.

---

## Role Variables

The role uses several variables to customize the configuration of the Proxmox VM template. Below are the available variables with their defaults (if applicable):

| Variable                        | Description                                                                 | Default Value                                                                 |
|----------------------------------|-----------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| `proxmox_template_url`           | URL of the cloud image template to download.                                 | `https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img` |
| `proxmox_template_name`          | Name of the cloud image template.                                            | `noble-server-cloudimg-amd64`                                                |
| `proxmox_template_disk_size`     | Size to resize the cloud image disk (e.g., `+20G`).                          | `8G`                                                                          |
| `proxmox_template_vm_id`         | VM ID for the Proxmox VM template.                                           | `9000`                                                                        |
| `proxmox_template_vendor_template`| Path to the cloud-init vendor template.                                      | `templates/vendor.yaml.j2`                                                   |
| `proxmox_template_ciuser`        | Cloud-init user to create.                                                   | `root`                                                                        |
| `proxmox_template_cipassword`    | Cloud-init password to set for the user.                                     | `root`                                                                        |
| `proxmox_template_os_type`       | OS type for the Proxmox VM (e.g., `linux` or `ubuntu`).                      | `l26`                                                                         |
| `proxmox_template_bios`          | BIOS type for the Proxmox VM (`seabios` or `ovmf`).                        | `ovmf`                                                                       |
| `proxmox_template_machine`       | Machine type for the Proxmox VM (e.g., `pc-i440fx-2.9`).                    | `q35`                                                                         |
| `proxmox_template_efidisk0`      | EFI disk for UEFI-based VMs (optional).                                     | `local-zfs:0,pre-enrolled-keys=0`                                            |
| `proxmox_template_net0`          | Network configuration for the VM (e.g., `virtio,bridge=vmbr0`).              | `virtio,bridge=vmbr0`                                                       |
| `proxmox_template_agent`         | Whether to enable the Proxmox agent (`1` or `0`).                           | `true`                                                                       |
| `proxmox_template_disk_location` | Location where the disk image will be stored on Proxmox (e.g., `local-zfs`). | `local-zfs`                                                                  |

---

## Example Playbook

Here’s an example Ansible playbook demonstrating how to use this role:

```yaml
---
- hosts: proxmox_servers
  roles:
    - role: proxmox_cloudinit_template
      proxmox_template_url: "https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img"
      proxmox_template_name: "noble-server-cloudimg-amd64"
      proxmox_template_vm_id: 9000
      proxmox_template_disk_size: "8G"
      proxmox_template_os_type: "l26"
      proxmox_template_bios: "ovmf"
      proxmox_template_machine: "q35"
      proxmox_template_efidisk0: "local-zfs:0,pre-enrolled-keys=0"
      proxmox_template_net0: "virtio,bridge=vmbr0"
      proxmox_template_agent: true
      proxmox_template_disk_location: "local-zfs"
      proxmox_template_vendor_template: "templates/vendor.yaml.j2"
      proxmox_template_ciuser: "root"
      proxmox_template_cipassword: "root"
```

## Usage Notes

- **Custom Configuration**: 
  - Create a `vendor.yaml.j2` file to customize the cloud-init configuration for the VM.
  
  ```yaml
  # templates/vendor.yaml.j2
  #cloud-config
  package_update: true
  package_upgrade: true

  # Install additional packages
  packages:
    - htop
    - git
    - git
    - qemu-guest-agent

  # Make sure qemu-guest-agent service is enabled
  runcmd:
    - systemctl enable qemu-guest-agent
    - systemctl start qemu-guest-agent
  ```

  - Modify the `proxmox_template_vendor_template` variable to point to your custom vendor data template.

---

## Installation Steps (Performed by the Role)

1. **Download Cloud Image**: Fetches the specified cloud image template from the provided URL and stores it on the Proxmox server.
2. **Resize Cloud Image**: Resizes the downloaded cloud image to the specified disk size.
3. **Create VM Template**: Creates a Proxmox VM using the `qm create` command, with the necessary configuration (e.g., OS type, memory, CPU, network).
4. **Import Cloud Image**: Imports the downloaded cloud image into Proxmox using the `qm importdisk` command.
5. **Set Boot Disk**: Configures the boot disk for the VM using `qm set`.
6. **Set Boot Order**: Configures the boot order to ensure the VM boots from the correct device.
7. **Configure Cloud-Init**:
   - Generates and copies the vendor data (cloud-init) file to the appropriate directory on Proxmox.
   - Configures cloud-init by linking the vendor data file (`vendor.yaml`) to the Proxmox VM.
8. **Set SSH Keys and Network**: Configures SSH keys and network settings for the cloud-init-enabled VM.
9. **Convert to Template**: Converts the configured VM to a Proxmox template using the `qm template` command.

---

## Troubleshooting

- **Cloud-Init Errors**: If cloud-init is not working as expected, check the logs on the new VM for more details (`/var/log/cloud-init.log`).
- **VM Creation Issues**: If the VM creation fails, check the Proxmox logs (`journalctl -u pve-cluster`) for error details.
- **Network Configuration**: Ensure the network configuration is correct and that the Proxmox server can communicate with any external resources required by cloud-init.
