# -*- mode: ruby -*-
# vi: set ft=ruby :

hosts = [ "minio1", "minio2", "minio3", "minio4", "nginx" ]

Vagrant.configure(2) do |config|
  hosts.each_with_index do |name, num|
    config.vm.define name do |machine|
      machine.vm.box = "parallels/centos-7.3"
      machine.vm.provider "parallels" do |v|
        v.name = name
        v.memory = "512"
        v.linked_clone = true
        v.update_guest_tools = false
        v.check_guest_tools = false
        v.customize ["set", :id, "--sh-app-guest-to-host", "off"]
        v.customize ["set", :id, "--shared-cloud", "off"]
        v.customize ["set", :id, "--shared-profile", "off"]
        v.customize ["set", :id, "--time-sync", "on"]
      end
      machine.vm.hostname = name
      #machine.vm.network :forwarded_port, guest: 9000, host: 9000+num
      if name == hosts.last
        machine.vm.provision "ansible" do |ansible|
          ansible.config_file = "tests/ansible.cfg"
          ansible.verbose = "v"
          ansible.limit = "all"
          ansible.groups = {
            "minio" => hosts.grep(/^minio/),
            "nginx" => hosts.grep(/^nginx/),
            "localhost" => ["localhost"]
          }
          ansible.playbook = "tests/test_install.yml"
        end
      end
    end
  end
end
