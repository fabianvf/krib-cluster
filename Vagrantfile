# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'yes'

Vagrant.configure("2") do |config|

  nodes = (1..(ENV["NUM_NODES"]||3).to_i).map {|i| "node#{i}.example.net"}
  verbosity = ENV["VERBOSITY"]||""
  ssh_key_file = ENV["SSH_PUB_KEY"]||"~/.ssh/id_rsa.pub"


  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
    config.cache.synced_folder_opts = {
      type: 'rsync',
    }
  end

  config.vm.define 'krib.example.net', :primary => true do |krib|
    krib.vm.box = "centos/7"
    krib.vm.hostname = 'krib.example.net'
    krib.hostmanager.enabled = true
    krib.hostmanager.manage_host = true
    krib.hostmanager.manage_guest = true
    krib.vm.network :private_network,
      :mac => "52:11:22:33:44:41",
      :ip => '192.168.87.11',
      :libvirt__network_name => "krib-cluster",
      :libvirt__dhcp_enabled => true,
      :libvirt__netmask => "255.255.255.0",
      :libvirt__dhcp_bootp_file => "lpxelinux.0",
      :libvirt__dhcp_bootp_server => "192.168.87.11",
      :libvirt__domain_name => "example.net"

    krib.vm.synced_folder '.', '/vagrant', disabled: true

    krib.vm.provision :file, :source => ssh_key_file, :destination => "/tmp/key"
    krib.vm.provision :shell, :inline => <<-EOF
      cat /tmp/key >> /home/vagrant/.ssh/authorized_keys
      mkdir -p /root/.ssh
      cp /home/vagrant/.ssh/authorized_keys /root/.ssh/authorized_keys
    EOF
    krib.vm.provision :ansible do |ansible|
      ansible.groups = {
        "krib" => ["krib.example.net"],
      }
      ansible.verbose = verbosity
      ansible.playbook = 'drp.yml'
      ansible.become = true
      ansible.force_remote_user = false
      ansible.limit = 'all,localhost'
      ansible.raw_ssh_args = ["-o IdentityFile=~/.ssh/id_rsa"]
      ansible.extra_vars = {
        "master_count": ENV["NUM_MASTERS"]||(nodes.length >= 3 ? 3 : 1),
        "admin_ssh_pub_key_file": ssh_key_file,
      }
    end
  end

  nodes.each_with_index do |name, idx|
    config.vm.define name, :autostart => true do |node|
      node.hostmanager.manage_guest = false
      node.hostmanager.manage_host = false
      node.vm.hostname = name

      node.vm.provider :libvirt do |domain|
        domain.storage :file, :size => '40G', :type => 'raw'
        domain.storage :file, :size => '40G', :type => 'raw'
        domain.mgmt_attach = 'false'
        domain.management_network_name = 'krib-cluster'
        domain.management_network_address = "192.168.87.0/24"
        domain.management_network_mode = "nat"
        domain.boot 'network'
        domain.boot 'hd'
      end
    end
  end


  config.vm.provider :libvirt do |libvirt|
    libvirt.driver = "kvm"
    libvirt.memory = 8000
    libvirt.cpus = `grep -c ^processor /proc/cpuinfo`.to_i
  end
end
