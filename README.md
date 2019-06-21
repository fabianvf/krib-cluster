# krib-cluster

A vagrant-libvirt environment for testing out Digital Rebar Provision + KRIB.

## Dependencies
- libvirt
- vagrant
- vagrant-libvirt
- vagrant-hostmanager
- ansible

## Usage
`vagrant up`

The first machine (`krib.example.net`) is provisioned with Ansible using the [`drp.yml`](drp.yml) playbook, 
and all additional machines `node{1..n}.example.net` are PXE booted into the discovery workflow. To provision those
nodes with KRIB, assign the `krib-cluster` profile and workflow to the machines.

By default 3 nodes will be created, you can change the number of nodes with the `NUM_NODES` environment variable.