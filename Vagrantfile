# -*- mode: ruby -*-
# vi: set ft=ruby :
# See: https://docs.vagrantup.com/v2/vagrantfile/tips.html



VAGRANTFILE_API_VERSION = "2"

VIRTUAL_MACHINES = {
  :left => {
    :ip             => '192.168.9.41',
  },
  :right => {
    :ip             => '192.168.9.42',
  }
}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
#  config.hostmanager.enabled = true
  config.vm.box = "geerlingguy/ubuntu1604"  
  config.ssh.insert_key = false

  VIRTUAL_MACHINES.each do |name,cfg|

    config.vm.define name do |vm_config|
      vm_config.vm.hostname = name
      vm_config.vm.network :private_network, ip: VIRTUAL_MACHINES[name][:ip]

      config.vm.provider :virtualbox do |vb|
        vb.memory = 1024
        vb.cpus = 2
        # see https://www.vagrantup.com/docs/virtualbox/configuration.html
        # for linked_clones
        vb.linked_clone = true

        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--ioapic", "on"]

        unless File.exist? ".vagrant/#{name}.vdi"
          vb.customize ["createhd", "disk", "--filename", ".vagrant/#{name}.vdi",
              "--size", "2048"]
        end

        # connect the disk to SATA port 4
        vb.customize ["storagectl", :id, '--add', 'sata', 
                      '--name', 'Vagrant']
          
        vb.customize ["storageattach", :id, 
                      '--storagectl', 'Vagrant',
                      '--type', 'hdd',
                      '--port', '4',
                      '--medium', ".vagrant/#{name}.vdi"]

         # Set serial and model number to the disk
        vb.customize ["setextradata", :id,
              "VBoxInternal/Devices/ahci/0/Config/Port4/SerialNumber",
              "data"]
        vb.customize ["setextradata", :id,
              "VBoxInternal/Devices/ahci/0/Config/Port4/ModelNumber",
              "VIRTUAL"]
      end # provider

      config.vm.provision :ansible do |ansible|
            ansible.playbook = "site.yml"
            ansible.host_key_checking = false
            ansible.sudo_user = 'root'
            ansible.sudo = true
            ansible.verbose = "v"

            #ansible.tags = 'setup'
            #ansible.tags = 'bricks'
            #ansible.tags = 'mount'

            ansible.groups = {
              'postgres' => [:left, :right],
              'ubuntu'  => VIRTUAL_MACHINES.keys,
              'all:vars' => {
                    'master_server' => :left,
              }
            }
            # if you want to fire ansible on all machines at parallel, use this!
            #ansible.limit = 'all'
      end
    end
  end
end

