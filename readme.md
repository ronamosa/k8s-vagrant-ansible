# K8s: Vagrant, Ansible Setup

## Overview

### pre-requisites

* Install VirtualBox

```sh
# add /etc/apt.sources.list
deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian <mydist> contrib

# add keys && verify (after)
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -

# update && install
sudo apt-get update
sudo apt-get install virtualbox-6.1
```

* Install [Vagrant](https://www.vagrantup.com/docs/installation)
* Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu)

> install `libarchive-tools` to get tool `bsdtar`, which is required but not supported in Ubuntu 19+: `sudo apt install -y libarchive-tools`.

### Deploy

In the rep folder, run: `vagrant up`

This will do the following:

1. create the `k8s-control` ubuntu VM Node.
2. install docker, kubernetes, kubeadm and init the cluster in `k8s-control`
3. create each `worker-N` ubuntu VM Node.
4. install docker, kubernetes, kubeadm and join each worker to `k8s-control` Node.

### Check

Once it's all up, check for errors, then check with commands: `vagrant status` and you should see:

```sh
Current machine states:

k8s-control                running (virtualbox)
worker-1                    running (virtualbox)
worker-2                    running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

Jump into a node with: `vagrant ssh k8s-control` or `vagrant ssh node-1`

Check your cluster:

```bash
vagrant@k8s-control:~$ kubectl get nodes
NAME          STATUS   ROLES                  AGE   VERSION
k8s-control   Ready    control-plane,master   87m   v1.23.5
worker-1      Ready    <none>                 83m   v1.23.5
worker-2      Ready    <none>                 80m   v1.23.5
```

### KUBECONFIG

Access your cluster via `kubectl` from the `k8s-control` Node.

(TODO) - access the cluster from your host computer using the `~/.kube/config` file from `k8s-control`.

### troubleshooting

if you see this error when you run: `journalctl -xeu kubelet`

```bash
64] "Docker Info" dockerInfo=&{ID:JECY:YHJK:J7PW:2LCZ:B64F:M6PQ:BBXC:ORXL:HNRW:UXKT:X4CO:LM4D Containers:0 ContainersRunning:0 ContainersPaused:0 ContainersStopped:0 Images:7 Driver:ov
led to run kubelet" err="failed to run Kubelet: misconfiguration: kubelet cgroup driver: \"systemd\" is different from docker cgroup driver: \"cgroupfs\""
tatus=1/FAILURE
```

it means you need `/etc/docker/daemon.json` with this config in it:

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

then restart docker: `systemctl restart docker`

## References

* [k8s blog](https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/)
* [k8s kubeadm docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
* [docker docs](https://docs.docker.com/engine/install/ubuntu/)
* [ansible blockinfile](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/blockinfile_module.html)
* [vagrant ubuntu/xenial64](https://app.vagrantup.com/ubuntu/boxes/xenial64/versions/20211001.0.0)
