# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'yes'

Vagrant.configure("2") do |config|

  nodes = (1..(ENV["NUM_NODES"]||3).to_i).map {|i| "node#{i}.example.org"}
  verbosity = ENV["VERBOSITY"]||""


  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
    config.cache.synced_folder_opts = {
      type: 'rsync',
    }
  end


  config.vm.define 'krib.example.org', :primary => true do |krib|
    krib.vm.box = "centos/7"
    krib.vm.hostname = 'krib.example.org'
    krib.hostmanager.enabled = true
    krib.hostmanager.manage_host = true
    krib.hostmanager.manage_guest = false
    krib.vm.network :private_network,
      :mac => "52:11:22:33:44:41",
      :ip => '192.168.17.11',
      :libvirt__network_name => "home-cluster",
      :libvirt__dhcp_enabled => true,
      :libvirt__netmask => "255.255.255.0",
      :libvirt__dhcp_bootp_file => "lpxelinux.0",
      :libvirt__dhcp_bootp_server => "192.168.17.11"

    krib.vm.synced_folder '.', '/vagrant', disabled: true

    krib.vm.provision :file, :source => "~/.ssh/id_rsa.pub", :destination => "/tmp/key"
    krib.vm.provision :shell, :inline => <<-EOF
      cat /tmp/key >> /home/vagrant/.ssh/authorized_keys
      mkdir -p /root/.ssh
      cp /home/vagrant/.ssh/authorized_keys /root/.ssh/authorized_keys
      sed -i 's/127.0.0.1.*krib.example.org/192.168.17.11 krib.example.org/' /etc/hosts
    EOF
    krib.vm.provision :ansible do |ansible|
      ansible.groups = {
        "krib" => ["krib.example.org"],
      }
      ansible.verbose = verbosity
      ansible.playbook = 'drp.yml'
      ansible.become = true
      ansible.force_remote_user = false
      ansible.limit = 'all,localhost'
      ansible.raw_ssh_args = ["-o IdentityFile=~/.ssh/id_rsa"]
      ansible.host_vars = {
        "krib.example.org" => {
          "boot_disk": "vda",
          "subnet": {
            "Name": "local_subnet",
            "Subnet": "192.168.17.0/24",
            "ActiveStart": "192.168.17.50",
            "ActiveEnd": "192.168.17.200",
            "ActiveLeaseTime": 60,
            "Enabled": true,
            "Proxy": true,
            "ReservedLeaseTime": 7200,
            "Strategy": "MAC",
            "Options": [{
                "Code": 3,
                "Value": "192.168.17.1",
                "Description": "Default Gateway"
              }, {
                "Code": 6,
                "Value": "8.8.8.8",
                "Description": "DNS Servers"
              }, {
                "Code": 15,
                "Value": "example.org",
                "Description": "Domain Name"
              }]
      }}}
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
        domain.management_network_name = 'home-cluster'
        domain.management_network_address = "192.168.17.0/24"
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
