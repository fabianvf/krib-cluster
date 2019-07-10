# krib-cluster

A vagrant-libvirt environment for testing out Digital Rebar Provision + KRIB.

## Dependencies
- libvirt
- vagrant
- vagrant-libvirt
- vagrant-hostmanager
- ansible

## Usage
```
$ vagrant up
```

The first machine (`krib.example.net`) is provisioned with Ansible using the [`drp.yml`](drp.yml) playbook,
and all additional machines `node{1..n}.example.net` are PXE booted into the discovery workflow.

Once the nodes have booted into discovery, apply the `krib-cluster` profile to them and apply either the
`krib-live-cluster` or `krib-install-cluster` workflows. This will install Kubernetes with krib and will
install/configure the rook-ceph operator and traefik ingress controller.

### Options
There are a few options you can provide to `vagrant` via environment variables:

| name | default | description |
| ---- | ------- | ----------- |
| NUM_NODES | 3 | The number of Kubernetes nodes to spin up |
| NUM_MASTERS | 1 if NUM_NODES 3, otherwise 3 | The number of Kubernetes masters to provision |
| VERBOSITY | "" | The ansible verbosity level as you would pass it to the `ansible-playbook` command (specified by one or more `v`s) |
| SSH_PUB_KEY | ~/.ssh/id_rsa.pub | Path to the ssh public key file to be added to the authorized_hosts for all provisioned machines |

Additional options can be provided via the `config.yml` file, which is imported by the `drp.yml` playbook. This file can be used to
override any of the options exposed by the `digitalrebar-provision` role.
