# K8S on Ubuntu Xenial

IMAGE = "ubuntu/xenial64"
VERSION = "20211001.0.0"
N = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end
      
    config.vm.define "k8s-control" do |control|
        control.vm.box = IMAGE
        control.vm.box_version  = VERSION
        control.vm.network "private_network", ip: "192.168.50.10"
        control.vm.hostname = "k8s-control"
        control.vm.provision "ansible" do |ansible|
            ansible.playbook = "playbooks/control-playbook.yml"
            ansible.extra_vars = {
                node_ip: "192.168.50.10",
            }
        end
    end

    (1..N).each do |i|
        config.vm.define "worker-#{i}" do |worker|
            worker.vm.box = IMAGE
            worker.vm.box_version  = VERSION
            worker.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            worker.vm.hostname = "worker-#{i}"
            worker.vm.provision "ansible" do |ansible|
                ansible.playbook = "playbooks/worker-playbook.yml"
                ansible.extra_vars = {
                    node_ip: "192.168.50.#{i + 10}",
                }
            end
        end
    end
end
