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

### Active Directory

I created a Windows2016 server named EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com on Amazon EC2.

* configure AD (create an example domain is JP-SBR-ANSIBLE.EXAMPLE.COM)
* create a user
* the user should be included in `Administrators`
* setup for Ansible with the PowerShell script provided at https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1
* setup DNS to resolve both `A` and `PTR`

### /etc/krb5.conf

```
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 dns_lookup_kdc = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = /etc/pki/tls/certs/ca-bundle.crt
# default_realm = EXAMPLE.COM
 default_realm = JP-SBR-ANSIBLE.EXAMPLE.COM
 default_ccache_name = KEYRING:persistent:%{uid}

[realms]
# EXAMPLE.COM = {
#  kdc = kerberos.example.com
#  admin_server = kerberos.example.com
# }
JP-SBR-ANSIBLE.EXAMPLE.COM = {
  kdc = EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com
  admin_server = EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com
}

[domain_realm]
# .example.com = EXAMPLE.COM
# example.com = EXAMPLE.COM
jp-sbr-ansible.example.com = JP-SBR-ANSIBLE.EXAMPLE.COM
```

### /etc/hosts or DNS

Name resolution is very important for Kerberos.

```
192.168.100.23 EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com
```

### Time synchronization

Time synchronization is also important for Kerberos

### Check with Kerberos suite

```
# yum install krb5-workstation
...
# kinit sugimura@JP-SBR-ANSIBLE.EXAMPLE.COM
Password for sugimura@JP-SBR-ANSIBLE.EXAMPLE.COM:
# klist
Ticket cache: KEYRING:persistent:0:0
Default principal: sugimura@JP-SBR-ANSIBLE.EXAMPLE.COM

Valid starting       Expires              Service principal
2018-11-23T06:14:19  2018-11-23T16:14:19  krbtgt/JP-SBR-ANSIBLE.EXAMPLE.COM@JP-SBR-ANSIBLE.EXAMPLE.COM
	renew until 2018-11-30T06:14:17
# kdestroy
```

### Check with `win_ping`

Create an inventory

```
[windows2016]
EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com

[windows2016:vars]
ansible_user=sugimura@JP-SBR-ANSIBLE.EXAMPLE.COM
ansible_password=XXXXXXXX
ansible_connection=winrm
ansible_winrm_transport=kerberos
ansible_winrm_server_cert_validation=ignore
ansible_port=5986
ansible_winrm_scheme=https
```

Invoke `win_ping`

```
# ansible -i inventory windows2016 -m win_ping  -vvvvv
ansible 2.7.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.5 (default, May 31 2018, 09:41:32) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
Using /etc/ansible/ansible.cfg as config file
setting up inventory plugins
Parsed /home/ec2-user/02251338/inventory inventory source with ini plugin
Loading callback plugin minimal of type stdout, v2.0 from /usr/lib/python2.7/site-packages/ansible/plugins/callback/minimal.pyc
META: ran handlers
Using module file /usr/lib/python2.7/site-packages/ansible/modules/windows/win_ping.ps1
<EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com> ESTABLISH WINRM CONNECTION FOR USER: sugimura@JP-SBR-ANSIBLE.EXAMPLE.COM on PORT 5986 TO EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com
checking if winrm_host EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com is an IPv6 address
creating Kerberos CC at /tmp/tmpQ5p7eg
calling kinit with subprocess for principal sugimura@JP-SBR-ANSIBLE.EXAMPLE.COM
kinit succeeded for principal sugimura@JP-SBR-ANSIBLE.EXAMPLE.COM
<EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com> WINRM CONNECT: transport=kerberos endpoint=https://EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com:5986/wsman
<EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com> WINRM OPEN SHELL: 89CCC0DC-E9F1-4184-9842-86B29B4B3B34
EXEC (via pipeline wrapper)
<EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com> WINRM EXEC 'PowerShell' ['-NoProfile', '-NonInteractive', '-ExecutionPolicy', 'Unrestricted', '-']
<EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com> WINRM RESULT u'<Response code 0, out "{"changed":false,"pi", err "">'
<EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com> WINRM CLOSE SHELL: 89CCC0DC-E9F1-4184-9842-86B29B4B3B34
EC2AMAZ-PGLO7IP.jp-sbr-ansible.example.com | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
META: ran handlers
META: ran handlers
```

### Troubleshoot and reference

```
PS C:\Users\Administrator> winrm get winrm/config
PS C:\Users\Administrator> winrm enumerate winrm/config/Listener
```

* https://docs.ansible.com/ansible/latest/user_guide/windows.html
* https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html

# AWS

# System Roles

## RHEL 7

## RHEL 8
