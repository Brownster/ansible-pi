# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = "2"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbooks/site.yml"
    ansible.inventory_path = "inventory/vagrant.yml"
    ansible.extra_vars = {
      # Add any extra variables you need for testing here
      # For example, you might want to disable certain tasks
      # that are specific to the Raspberry Pi hardware.
    }
  end
end
