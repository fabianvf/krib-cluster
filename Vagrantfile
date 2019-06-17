# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  nodes = (1..(ENV["NUM_NODES"]||3).to_i).map {|i| "node#{i}.example.org"}
  verbosity = ENV["VERBOSITY"]||""

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = false
  config.hostmanager.manage_guest = false
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = false

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
    config.cache.synced_folder_opts = {
      type: 'rsync',
    }
  end

  config.vm.box = "centos/7"
  config.vm.provision :file, :source => "~/.ssh/id_rsa.pub", :destination => "/tmp/key"
  config.vm.provision :shell, :inline => <<-EOF
    cat /tmp/key >> /home/vagrant/.ssh/authorized_keys
    mkdir -p /root/.ssh
    cp /home/vagrant/.ssh/authorized_keys /root/.ssh/authorized_keys
    sed -i 's/127.0.0.1.*krib.example.org/192.168.17.11 krib.example.org/' /etc/hosts
  EOF

  config.vm.define 'krib.example.org', :primary => true do |krib|
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
      :libvirt__dhcp_bootp_file => "undionly.kpxe",
      :libvirt__dhcp_bootp_server => "192.168.17.11"

    krib.vm.synced_folder '.', '/vagrant', disabled: true
    krib.vm.provision :shell, :inline => <<-EOF
      set -ex
      sudo yum install -y git
      curl -fsSL get.rebar.digital/stable | bash -s -- install
      sudo systemctl daemon-reload
      sudo systemctl enable dr-provision
      sudo systemctl start dr-provision
      /usr/local/bin/drpcli contents upload catalog:drp-community-content-stable
      while [[ "$?" -ne 0 ]]
      do
        /usr/local/bin/drpcli contents upload catalog:drp-community-content-stable
      done
      /usr/local/bin/drpcli bootenvs uploadiso sledgehammer
      /usr/local/bin/drpcli prefs set defaultWorkflow discover-base unknownBootEnv discovery
      /usr/local/bin/drpcli contents upload catalog:task-library-stable
      /usr/local/bin/drpcli bootenvs uploadiso centos-7-install
      /usr/local/bin/drpcli plugin_providers upload certs from catalog:certs-stable
      /usr/local/bin/drpcli contents upload catalog:krib-stable
      echo '{
        "Name": "local_subnet",
        "Subnet": "192.168.17.0/24",
        "ActiveStart": "192.168.17.50",
        "ActiveEnd": "192.168.17.200",
        "ActiveLeaseTime": 60,
        "Enabled": true,
        "ReservedLeaseTime": 7200,
        "Strategy": "MAC",
        "Options": [
          { "Code": 3, "Value": "192.168.17.1", "Description": "Default Gateway" },
          { "Code": 6, "Value": "8.8.8.8", "Description": "DNS Servers" },
          { "Code": 15, "Value": "example.org", "Description": "Domain Name" }
        ]
      }' > /tmp/local_subnet.json
      /usr/local/bin/drpcli subnets create - < /tmp/local_subnet.json

    EOF
  end

  nodes.each_with_index do |name, idx|
    config.vm.define name, :autostart => true do |node|
      node.hostmanager.manage_guest = false
      node.hostmanager.manage_host = false
      node.vm.hostname = name
      node_ip = "192.168.17.#{12 + idx}"

      node.vm.network :private_network,
        :ip => node_ip
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
