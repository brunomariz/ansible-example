# ansible-example

## Prerequisites

For this example, a Proxmox server with 4 Ubuntu Server 22.04 VMs were setup, one for the controller, and 3 for the managed hosts. However, only one host is referenced in the playbook, so 2 machines will suffice for this example.

Before running the ansible commands, `/etc/hosts` file was edited to contain ansible-host1's IP address, and an ssh key pair was generated on the controller and the public key was copied to the host with ssh-copy-id ansible-host1. Of course, both machines were also configured with static IPs.

Controller node's `/etc/hosts`:

```
127.0.0.1 localhost
127.0.1.1 ubutnu-server-base

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

192.168.0.2 ansible-host1
192.168.0.3 ansible-host2
192.168.0.4 ansible-host3
```

Ansible was installed on the controller node with `apt install ansible`. 

## Playbook example

Clone this repo into the controller node

```
git clone https://github.com/brunomariz/ansible-example.git
cd ansible-example
```

View ansible version and active config file

```
ansible --version
ansible [core 2.16.5]
  config file = /home/maintenance/ansible-example/ansible.cfg
  configured module search path = ['/home/maintenance/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/maintenance/ansible-example/collections
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.0.3
  libyaml = True
```

View inventory information

```
ansible-inventory --graph
@all:
  |--@ungrouped:
  |--@myhosts:
  |  |--ansible-host1
  |  |--ansible-host2
  |  |--ansible-host3
```

The playbook makes sure a user is present on ansible-host1. View before state on ansible-host1:

```
getent passwd tuser
```

Run the playbook on the controller

> `-K` will prompt you for the managed host's sudo password. If the generated ssh key in the prerequisites has a password, use `-kK` to enter the ssh key's password aswell.

```
ansible-playbook playbook.yaml -K
BECOME password:

PLAY [Configure test user] *****************************************************************************

TASK [Gathering Facts] *********************************************************************************
Thursday 28 March 2024  14:04:16 +0000 (0:00:00.015)       0:00:00.016 ********
Thursday 28 March 2024  14:04:16 +0000 (0:00:00.014)       0:00:00.014 ********
ok: [ansible-host1]

TASK [Test user exists] ********************************************************************************
Thursday 28 March 2024  14:04:18 +0000 (0:00:01.955)       0:00:01.971 ********
Thursday 28 March 2024  14:04:18 +0000 (0:00:01.955)       0:00:01.970 ********
changed: [ansible-host1]

PLAY RECAP *********************************************************************************************
ansible-host1              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Playbook run took 0 days, 0 hours, 0 minutes, 3 seconds
Thursday 28 March 2024  14:04:19 +0000 (0:00:01.502)       0:00:03.474 ********
===============================================================================
Gathering Facts --------------------------------------------------------------------------------- 1.96s
Test user exists -------------------------------------------------------------------------------- 1.50s
Thursday 28 March 2024  14:04:19 +0000 (0:00:01.503)       0:00:03.473 ********
===============================================================================
gather_facts ------------------------------------------------------------ 1.96s
ansible.builtin.user ---------------------------------------------------- 1.50s
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
total ------------------------------------------------------------------- 3.46s
```

View after state on ansible-host1:

```
getent passwd tuser
tuser:x:9999:9999::/home/tuser:/bin/sh
```

