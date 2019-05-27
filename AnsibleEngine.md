# Engine Installation

## RHEL 7

```
# subscription-manager repos --enable rhel-7-server-ansible-2.8-rpms
# yum install ansible -y
...
# ansible --version
ansible 2.8.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.5 (default, Mar 26 2019, 22:13:06) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
```

## RHEL 8

```
# subscription-manager repos --enable ansible-2.8-for-rhel-8-x86_64-rpms
# yum install ansible -y
...
# ansible --version
ansible 2.8.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.6/site-packages/ansible
  executable location = /bin/ansible
  python version = 3.6.8 (default, Apr  3 2019, 17:26:03) [GCC 8.2.1 20180905 (Red Hat 8.2.1-3)]
```

# Windows

## Kerberos

# AWS

# System Roles

## RHEL 7

## RHEL 8
