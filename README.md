# libvirt_provision

Create simple reproducable VMs using [libvirt](https://www.libvirt.org/) `virsh` and [`community.libvirt`](https://docs.ansible.com/projects/ansible/latest/collections/community/libvirt/index.html). Created for testing my own Ansible recipes in virtual machine. This replaces Hashicorp Vagrant. The community has rightfully stopped supporting it due to licensing issues.

Predefined distros:

- trixie (tested)
- noble  (tested)
- rocky9
- rocky10

## Versions

- `1.2.0` --- avoid race condition with temp files
- `1.1.0` --- added ability to use custom temp path
- `1.0.0` --- initial version

## Requirements

Required Python packages on all machines the role runs on:

- `libvirt-python`
- `lxml`

Install on Debian:

```bash
sudo apt install python3-libvirt python3-lxml
```

### OS compatibility

Only tested on Debian Trixie, but no limitations on which OS you can run role on.

Tested with `ansible-core = "~=2.20.0"`.

### Known caveats

- It is difficult to install `libvirt-python` and `lxml` in an virtual environment. Set `PIPENV_SITE_PACKAGES=1` to allow for using system packages in addition to your virtual environment. Can be set in `.env` when using `pipenv`.
- Temporary cloud images is not deleted on `state=absent` to reduce downloads.
- All images used must be [`qcow2`]() images that support `cloud-init`.

## Role Variables

- `libvirt_provision_connection` --- A valid connection URL, default `qemu:///system`.
- `libvirt_provision_distros` --- Dictionary of cloud images. Each list element as below:
  - `arbitrary-name` -- Name of distro profile.
    - `url` --- Url to cloud image.
    - `os_variant` ---  Valid `virt-install --os-variant list` OS short-name.
    - `boot` --- Boot mode, most common `uefi`.
    - `sha` --- Optional checksum of image.
- `libvirt_provision_storage_pool` --- Libvirt storage pool for VM disks, default `default`.
- `libvirt_provision_root_ssh_key` --- SSH key to inject for root, default not set.
- `libvirt_provision_root_password` --- Root user password, default not set.
- `libvirt_provision_temp_path` --- Temp folder to store images and temporary files, defaults `/tmp`.
- `libvirt_provision_virtual_machine_config` --- Default VM configuration, see [main.yml](defaults/main.yml).
- `libvirt_provision_virtual_machine_net_config` --- Default cloud image network, see [main.yml](defaults/main.yml).
- `libvirt_provision_virtual_machines` --- List of virtual machines to create. Defaults, see below. For more advanced setup, see [test.yml](tests/test.yml).
    ```yaml
    - name: my-trixie-vm
      distro: trixie
      networks:
    ```
- `state` --- Create or destroy VMs and disks with `present`and `absent`, default`present`.

## Dependencies

Requires installation of [community.libvirt](https://docs.ansible.com/projects/ansible/latest/collections/community/libvirt/index.html) collection.

```yaml
---
collections:
  - name: community.libvirt
    version: ">=2.0.0,<=2.1.0"
```

## Example Playbook

```yaml
---
- hosts: all

  pre_tasks:
    - name: Set variables that need to be set on localhost
      ansible.builtin.set_fact:
        libvirt_provision_root_ssh_key: "{{ lookup('file', lookup('env', 'HOME') + '/.ssh/id_ed25519.pub') }}"
      delegate_to: localhost

  roles:
    - role: ../..
      vars:
        # Don't define password if you're working with remote VMs.
        # Then use only the SSH key. Keep safe.
        libvirt_provision_root_password: root
        libvirt_provision_virtual_machines:
          - name: trixie01
            memory_mb: 2000
            disk_gb: 20
            vcpus: 2
            distro: trixie
            net_config:
              ethernets:
                enp1s0:
                  addresses:
                    - 192.168.122.11/24
          - name: trixie02
            memory_mb: 1500
            distro: trixie
            net_config:
              ethernets:
                enp1s0:
                  addresses:
                    - 192.168.122.12/24
          - name: ubuntu01
            distro: noble
            # Undefine networks, run with libvirt defaults
            networks:
```

## Testing

Enter the role and install requirements.

```bash
mkdir .venv
pipenv install
pipenv shell
ansible-galaxy install -r tests/collections.yml
cd tests
```

Running the `test.yml` playbook expects you have `.ssh/ed25519.pub` SSH key available. Change the playbook if you use an RSA key.

### Create VMs

Download cloud images and start VMs. `state=present` is implicit.

```bash
ansible-playbook -i inventory test.yml
```

### Remove VMs

Removes VMs and disks.

```bash
ansible-playbook -i inventory test.yml -e state=absent
```

## License

GPL-2.0-or-later

## Author Information

- Name: https://github.com/aheimsbakk
