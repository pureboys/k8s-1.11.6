# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"

boxes = [
    {
        :name => "k8s-master",
        :eth1 => "11.11.11.111",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-node1",
        :eth1 => "11.11.11.112",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-node2",
        :eth1 => "11.11.11.113",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-store",
        :eth1 => "11.11.11.114",
        :mem => "2048",
        :cpu => "2"
    }
]

Vagrant.configure(2) do |config|

  config.vm.box = "bento/centos-7.3"
  boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.hostname = opts[:name]
      config.vm.provider "vmware_fusion" do |v|
        v.vmx["memsize"] = opts[:mem]
        v.vmx["numvcpus"] = opts[:cpu]
      end
      config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--memory", opts[:mem]]
        v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
      end
        config.vm.network :private_network, ip: opts[:eth1]
     
    end
  end
  config.vm.synced_folder "./labs", "/home/vagrant/labs"
end
