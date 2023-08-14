# -*- mode: ruby -*-
# vim: set ft=ruby :

GLOBAL_VARS ={local_repo: '10.0.2.2'}

MACHINES = {

  :master => {
    :box_name => "almalinux/9",
    :net => [
      {ip: '192.168.0.1', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "kube-net"},
    ],
    :vars => {}
  },
  :node1 => {
    :box_name => "almalinux/9",
    :net => [
      {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "kube-net"},
    ],
    :vars => {}
  },
  :node2 => {
    :box_name => "almalinux/9",
    :net => [
      {ip: '192.168.0.3', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "kube-net"},
    ],
    :vars => {}
  },
}

Vagrant.configure("2") do |config|
  config.vm.box_check_update = false
  config.vm.provider :virtualbox
  config.vm.provider "virtualbox" do |vb|
    vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
  end
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      boxconfig[:net].each do |ipconf|
        box.vm.network "private_network", **ipconf
      end
      box.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = "2.0"
        ansible.playbook = "ansible/playbook.yaml"
        ansible.extra_vars = GLOBAL_VARS.merge(boxconfig[:vars]).merge({host_name: boxname.to_s})
        ansible.host_key_checking = "false"
        ansible.become = "true"
        ansible.limit = "all"
      end
    end
  end
end
