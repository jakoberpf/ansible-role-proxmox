Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64" 
  
  # https://stackoverflow.com/questions/16708917/how-do-i-include-variables-in-my-vagrantfile
  # https://stackoverflow.com/questions/14124234/how-to-pass-parameter-on-vagrant-up-and-have-it-in-the-scope-of-vagrantfile
  # https://stackoverflow.com/questions/16708917/how-do-i-include-variables-in-my-vagrantfile

  config.vm.provider :libvirt do |libvirt|
    libvirt.memory = 8192
    libvirt.cpus = 2
    libvirt.storage :file, :size => '128M'
    libvirt.storage :file, :size => '128M'
  end

  config.vm.provider :virtualbox do |virtualbox|
    virtualbox.memory = 4096
    virtualbox.cpus = 2
  end

  N = 3
  (1..N).each do |machine_id|
    config.vm.define "pve-#{machine_id}" do |machine|
      machine.vm.hostname = "pve-#{machine_id}"

      machine.vm.provider :virtualbox do |virtualbox|
        virtualbox.customize ['createmedium', 'disk', '--filename', "disks/pve-#{machine_id}_vdb.vdi", '--size', 128, '--format', 'VDI']
        virtualbox.customize ['createmedium', 'disk', '--filename', "disks/pve-#{machine_id}_vdc.vdi", '--size', 128, '--format', 'VDI']
        virtualbox.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--device', 0, '--port', 2, '--type', 'hdd', '--medium', "disks/pve-#{machine_id}_vdb.vdi"]
        virtualbox.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--device', 0, '--port', 3, '--type', 'hdd', '--medium', "disks/pve-#{machine_id}_vdc.vdi"]
        machine.vm.network "private_network", ip: "192.168.50.1#{machine_id}", virtualbox__intnet: true
      end

      if machine_id == N
        machine.vm.provision :ansible do |ansible|
          ansible.limit = "all,localhost"
          ansible.playbook = "tests/vagrant/package_role.yml"
          ansible.verbose = true
        end
        machine.vm.provision :ansible do |ansible|
          ansible.limit = "all"
          ansible.playbook = "tests/vagrant/provision.yml"
          ansible.verbose = true
          ansible.host_vars = {
            "pve-1" => {"vagrant_static_private_address" => "192.168.50.11/24" },
            "pve-2" => {"vagrant_static_private_address" => "192.168.50.12/24" },
            "pve-3" => {"vagrant_static_private_address" => "192.168.50.13/24" }
          }
        end
      end
    end
  end
end
