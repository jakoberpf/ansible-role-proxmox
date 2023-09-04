Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64" # https://stackoverflow.com/questions/16708917/how-do-i-include-variables-in-my-vagrantfile

  config.vm.provider :libvirt do |libvirt|
    libvirt.memory = 8192
    libvirt.cpus = 2
    libvirt.storage :file, :size => '128M'
    libvirt.storage :file, :size => '128M'
  end

  N = 3
  (1..N).each do |machine_id|
    config.vm.define "pve-#{machine_id}" do |machine|
      machine.vm.hostname = "pve-#{machine_id}"
      machine.vm.network "private_network", ip: "172.30.1.#{machine_id}"

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
        end
      end
    end
  end
end
