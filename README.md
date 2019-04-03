# Setup kubenetes cluster with Virtual machine
## Target:
Init kubenetes cluster with:
* 1 master node
* 2 worker nodes

## Host machine info:
* Ubuntu 18.04
* 48 cores
* Vagrant 2.1.2

## Work Start
### Setup virtual machine with vagrant
Cript to setup 3 virtual machine is below:
[Vagranfile](./Vagrantfile)

#### Step-1: Define box for virtual machine
```
worker1.vm.box = "ubuntu/xenial64"
```
In this example I choose **ubuntu/xenial64**

#### Step-2: Setup vm machine network
```
worker1.vm.network :private_network, ip: "192.168.205.11"
worker1.vm.network :forwarded_port, guest: 22, host: 32221, id: "ssh"
```

Note: I config forwarded_port ssh port(22) to host machine: 32221. So that you ssh to vm machine by *ssh* command. With this setup you have 2 way to ssh to vm machine:
1. > $ vagrant ssh ${machine_name}
2. > $ ssh -p 32221 {username}@127.0.0.1

But right now you can't use **2** way because default ssh server is not allow *PasswordAuthentication*

#### Step-3: Setup memory and CPUs
```
worker1.vm.provider "virtualbox" do |worker1|
    worker1.memory = "2404"
    worker1.cpus = "2"
end
```

#### Step-4: Config ssh

```
sed -i 's/PermitRootLogin.*$/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication.*$/PasswordAuthentication yes/g' /etc/ssh/sshd_config
service ssh restart
```

We change sshd_config at **/etc/ssh/sshd_config**
1. PermitRootLogin is yes
2. PasswordAuthentication yes

then restart ssh service to apply our changes

#### Step-5: Change password of root user:
```
echo -e 'nctam123\nnctam123' | passwd root
```
#### Step-6: Start virtual machine
Now you can run below command to start virtual machine:
> $ vagrant up

Check ssh-config
```
$ vagrant ssh-config
Host master
  HostName 127.0.0.1
  User vagrant
  Port 32222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/chitam/workspace/invest/kube_cluster/vagrant/.vagrant/machines/master/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL

Host worker1
  HostName 127.0.0.1
  User vagrant
  Port 32221
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/chitam/workspace/invest/kube_cluster/vagrant/.vagrant/machines/worker1/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL

Host worker2
  HostName 127.0.0.1
  User vagrant
  Port 32223
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/chitam/workspace/invest/kube_cluster/vagrant/.vagrant/machines/worker2/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

#### Step-7: Copy public ssh key to vm machine
``` bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub -p 32222 root@127.0.0.1
$ ssh-copy-id -i ~/.ssh/id_rsa.pub -p 32221 root@127.0.0.1
$ ssh-copy-id -i ~/.ssh/id_rsa.pub -p 32223 root@127.0.0.1
```

### Config ansible-play book
#### Step-1: Creat hosts file
[hosts](hosts)

You can check this link for more info about [Ansible's Inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#)

#### Step-2: Creat initial.yml file
```yaml
- hosts: all
  become: yes
  tasks:
    - name: create the 'ubuntu' user
      user: name=ubuntu append=yes state=present createhome=yes shell=/bin/bash

    - name: allow 'ubuntu' to have passwordless sudo
      lineinfile:
        dest: /etc/sudoers
        line: 'ubuntu ALL=(ALL) NOPASSWD: ALL'
        validate: 'visudo -cf %s'

    - name: set up authorized keys for the ubuntu user
      authorized_key: user=ubuntu key="{{item}}"
      with_file:
        - ~/.ssh/id_rsa.pub
```

Above is a playbook file with yaml format. Check this link for mor info [Working With Playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)

When you run command:
> $ ansible-playbook -i hosts initial.yml

Ansible with create ssh session to hosts defined in file **hosts** to do tasks:
1. create the 'ubuntu' user
2. allow 'ubuntu' to have passwordless sudo
3. set up authorized keys for the ubuntu user

Very easy to understand.

#### Step-2: Install kubenetes dependencies
All task is define in file [kube-dependencies.yml](kube-dependencies.yml)

> $ ansible-playbook -i hosts kube-dependencies.yml

#### Step-3: Start kubenetes cluster at master machine
> $ ansible-playbook -i hosts master.yml

#### Step-4: Join cluster at worker machine

> $ ansible-playbook -i hosts worker.yml

#### Step-5: Check status of cluster

```bash
$ ssh -p 32222 ubuntu@127.0.0.1
$ kubectl get nodes
```

Status of cluster should be:
```
NAME            STATUS   ROLES    AGE   VERSION
master          Ready    master   89s   v1.14.0
worker1         Ready    <none>   11s   v1.14.0
worker2         Ready    <none>   11s   v1.14.0
```

## Issuse:
Incase you want to setup cluster by hand. You may facebook some issue like me. Here some issues I faced and some advisions to fix it.

### Worker node cant join cluster
#### Describe: Command
> $ ansible-playbook -i hosts worker.yml

Stuck at task: **join cluster**
#### Troubleshoot
1. Check worker node can communicate with master host. Run command below on host machine
> $ curl {master_id}:6443

Output:
> Client sent an HTTP request to an HTTPS server.

Check network config in file [Vagrant](Vagrant)
### Worker joined but master did not see
#### Describe
Every thing sames to be ok but when you check at master node:
```bash
$ kubectl get nodes
NAME            STATUS   ROLES    AGE   VERSION
master          Ready    master   89s   v1.14.0
```

Only master node in cluster

And 
```
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
```
in file  /root/node_joined.txt

#### Troubleshoot
1. Check your cluster network config of cluster
* 2 option when you start __kubeadm__:*apiserver-advertise-address* and *apiserver-cert-extra-sans*  should be addess that workers can see
* Option: _node-ip_ of kubelet should be addess that workers can see

2. Hostnames of worker nodes and master nodes should not them same. [Github Issues](https://github.com/kubernetes/kubernetes/issues/61224)


## Conlusion
This tutorial intro the way to setup kubenetes cluster in virtual machine. I learned a lot of things:
1. Setup virtual machine with vagrant
2. Learn new automation tool **ansible**
3. How to config you kubenetes cluster.

Some good links:  
[Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)  
[single master cluster with kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)  
[Multi Node Kubernetes Cluster with Vagrant](https://medium.com/@wso2tech/multi-node-kubernetes-cluster-with-vagrant-virtualbox-and-kubeadm-9d3eaac28b98)
