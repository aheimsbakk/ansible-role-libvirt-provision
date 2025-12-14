# libvirt_vm_provision

Create simple reproducable VMs using [`virsh`]() and [`community.libvirt`](). Created for replacing Hashicorps' Vagrant, that the community has stopped supporting due to licencing issues.

## Versions


## Requirements

Required Python packages:

- `libvirt-python`
- `lxml`

Install on Debian with

```bash
sudo apt install python3-libvirt python3-lxml
```
### OS compatibility

### Known caveats

## Role Variables

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.


## Dependencies

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

## Testing

### Rerun role

## License

GPLv2

## Author Information

- Name: https://github.com/aheimsbakk
