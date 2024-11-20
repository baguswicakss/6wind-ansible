# 6wind Ansible NETCONF Automation
Ansible supports configuring remote hosts using NETCONF (instead of the default SSH connection along with Linux shell commands). This guide explains how to leverage Ansible to configure multiple Virtual Service Router instances.

## Dependencies
This guide assumes that you have two (or more) Virtual Service Router instances that are booted and accessible on the network (NETCONF uses TCP port 830). Also, for clarity purposes, these machines should be reachable with their respective hostnames (thus, DNS or /etc/hosts must be configured accordingly).

To make sure it works, ansible version greater than 2.7.10 along with the ncclient and jxmlease python libraries are required. Here is how to install this in a python virtualenv:
```
python3 -m venv /tmp/ansible-netconf
. /tmp/ansible-netconf/bin/activate
which python
/tmp/ansible-netconf/bin/python
pip install -U pip setuptools wheel
...
Successfully installed pip-19.1.1 setuptools-41.0.1 wheel-0.33.4
pip install "ansible > 2.7.10" ncclient jxmlease
...
Successfully installed MarkupSafe-1.1.1 PyYAML-5.1 ansible-2.8.0
asn1crypto-0.24.0 bcrypt-3.1.6 cffi-1.12.3 cryptography-2.6.1 jinja2-2.10.1
jxmlease-1.0.1 lxml-4.3.3 ncclient-0.6.4 paramiko-2.4.2 pyasn1-0.4.5
pycparser-2.19 pynacl-1.3.0 six-1.12.0
```
## Configure
![image](https://github.com/user-attachments/assets/36a3f0b7-9191-4ba8-8f22-a5c8aa0e5e1d)

In this case using topology on picture above.

The 6wind1 has connection to 6wind-2 router using BGP via trunking vlan on seperated VRF, which are vrf Service-a for connection between 6wind-3 and 6wind-4 and vrf Service-b for connection between host1 and host2.
For push config vrouter 6wind-1 and 6wind-2 to initiate BGP connection each vrf, use file `playbook-core.yml`as a playbook, and specify the inventory `vrouters1-2.yml` file in directory `inventory/` using the `-i` flag :

```bash
(ansible-netconf) root@ubuntu-host:/tmp/ansible-netconf# ansible-playbook -i inventory/vrouters1-2.yml playbook-core.yml

PLAY [vrouters] **************************************************************************************************

TASK [Initialize] ************************************************************************************************
ok: [6wind-2]
ok: [6wind-1]

TASK [configure vrf service a] *******************************************************************************
ok: [6wind-1]
ok: [6wind-2]

TASK [configure vrf service b] *******************************************************************************
ok: [6wind-1]
ok: [6wind-2]

PLAY RECAP *******************************************************************************************************
6wind-1                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
6wind-2                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

For push config vrouter 6wind-3 and 6wind-4 to initiate OSPF to each 6wind-1 and 6wind-2 , use file `playbook-client.yml`as a playbook, and specify the inventory `vrouters3-4.yml` file in directory `inventory/` using the `-i` flag :

```bash
(ansible-netconf) root@ubuntu-host:/tmp/ansible-netconf# ansible-playbook -i inventory/vrouters3-4.yml playbook-client.yml

PLAY [vrouters] **************************************************************************************************

TASK [configure vrf main 6wind-3&4] ******************************************************************************
ok: [6wind-4]
ok: [6wind-3]

PLAY RECAP *******************************************************************************************************
6wind-3                    : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

6wind-4                    : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
