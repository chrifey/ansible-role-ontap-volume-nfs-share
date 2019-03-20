ontap-volume-nfs-share
======================

This role is intended to manage a NFS export, consisting of:

- flexvol with junction path
- qtree
- export-policy on qtree

It should also give a example how groupvars can be handled in order to manage multiple volumes/systems.


Safemode
--------

During a few discussion we came to the point, that we would like to introduce some sort of "safemode" to prevent any unintended removal of volumes or qtree.

We basicly created two task files (no_safemode.yml and safemode.yml) that are chosen based on a environment variable that you can pass to the `ansible-playbook` command.

Example:

```bash
ansible-playbook -i inventory site.yml -e safemode=true
```

If you specify a volume or qtree to be `absent`, the safemode will ask for a confirmation for every volume/qtree that will be deleted. Without safemode variable set it will not ask for a confirmation and delete the volume/qtree.


Requirements
------------

- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) needs to be installed.
- [NetApp Lib](https://pypi.org/project/netapp-lib/)

**How to start:**

Install ansible with yum package manager (RHEL, CentOS):

```bash
yum install ansible python-pip
pip install netapp-lib
```
alternative (install python-virtualenv, e.g. with yum and use pip within this virtualenv):

```bash
yum install python-virtualenv
virtualenv ansible
source ansible/bin/activate
pip install ansible
pip install netapp-lib
```

Role Variables
--------------

We suggest to create groupvars to specify your environment specific parameters e.g. credentials, ...

First create an inventory file `inventory/site/hosts`:
```
[netapp]
localhost netapp_user=USERNAME netapp_password=SECRET
```

Since we store login credentials in this file, we strongly suggest to encrypt this file with [ansible-vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html)

You can use the following commands to encrypt the file:

```bash
ansible-vault encrypt --ask-vault-pass inventory/site/hosts
ansible-vault view --ask-vault-pass inventory/site/hosts
ansible-vault edit --ask-vault-pass inventory/site/hosts
```

Second create a vars file to configure your environment:

The following example shows `inventory/site/group_vars/all.yml` to be used with the `ontap-volume-nfs-share` role in this repo:

```yaml
# Environment specific variables
netapp_volumes:
  - { state: 'present', nacluster: 'cluster1.localdomain', vserver: 'vserver1', flexvol: 'flexvol1', space_guarantee: 'none', percent_snapshot_space: '5', aggregate: 'aggr1', size: '10', unit: 'gb', exportpolicy: 'default'  }

netapp_qtrees:
  - { state: 'present', nacluster: 'cluster1.localdomain', vserver: 'vserver1', qtree: 'qtree1', flexvol: 'flexvol1',
      exportpolicy: 'db-nodes', ro_rule: 'sys', rw_rule: 'sys', super_user_security: 'sys', client_match: '0.0.0.0/0' }

```

With that you can manage multiple volumes in one groupvars file (just add more lines in the netapp_volumes or netapp_qtrees list. You could also create different "sites" with different groupvars e.g. for specific applications that need a subset of volumes/qtrees.

Dependencies
------------

There are no dependent roles.

Example Playbook
----------------

The following example shows a simple playbook the utilize the `ontap-volume-nfs-share` role:

```yaml
---
- hosts: localhost
  connection: local
  gather_facts: no
  roles:
   - ontap-volume-nfs-share
```

You can then run the playbook like this (the options `--ask-vault-pass`prompts for the password of the previously encrypted file):
```bash
ansible-playbook --ask-vault-pass -i inventory/site/hosts playbooks/site.yml
```

License
-------

BSD

Author Information
------------------

Based on a hackathon with:

- Sven Mundschenk
- Stefan Gaertner
- Steffen Knoth
- Christian Fey
