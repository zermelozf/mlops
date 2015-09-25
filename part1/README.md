# DevOps for machine learning, part I

## Provision the machines

```shell
zer@localhost:~$ vagrant up
```

## Add nodes to /etc/hosts

```shell
vagrant@node1:~$ sudo vim /etc/hosts
vagrant@node1:~$ cat /etc/hosts
127.0.0.1 localhost
 
# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
127.0.1.1 node1 node1
 
# Members of the cluster
192.168.33.10 node1
192.168.33.11 node2
192.168.33.12 node3
192.168.33.13 node4
```

## Configure ansible

```shell
vagrant@node1:~$ sudo apt-add-repository ppa:ansible/ansible
vagrant@node1:~$ sudo apt-get update
vagrant@node1:~$ sudo apt-get install ansible
vagrant@node1:~$ ssh-keyscan node1 node2 node3 node4 >> ~/.ssh/known_hosts
vagrant@node1:~$ mkdir ansible
vagrant@node1:~$ cd ansible/
vagrant@node1:~/ansible$ vim ansible.cfg
vagrant@node1:~/ansible$ cat ansible.cfg 
[defaults]
hostfile = inventory.ini
user = vagrant
ask_pass = True
vagrant@node1:~/ansible$ vim inventory.ini
vagrant@node1:~/ansible$ cat inventory.ini 
[master]
node1
 
[slave]
node2
node3
node4
```

## Run playbook

```YAML
 hosts: all
  remote_user: vagrant
  sudo: yes
  tasks:
      - name: update the hosts file
        template: src=~/ansible/hosts.j2 dest=/etc/hosts
```

```shell
vagrant@node1:~/ansible$ ansible-playbook playbook.yml 
SSH password: 

PLAY [all] ******************************************************************** 

GATHERING FACTS *************************************************************** 
ok: [node2]
ok: [node1]
ok: [node4]
ok: [node3]

TASK: [update the hosts file] ************************************************* 
ok: [node1]
changed: [node2]
changed: [node4]
changed: [node3]

PLAY RECAP ******************************************************************** 
node1                      : ok=2    changed=0    unreachable=0    failed=0   
node2                      : ok=2    changed=1    unreachable=0    failed=0   
node3                      : ok=2    changed=1    unreachable=0    failed=0   
node4                      : ok=2    changed=1    unreachable=0    failed=0
```