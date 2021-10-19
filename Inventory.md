# Inventory in Ansible

Generally when we work with deployments we would have number of servers to deal and servers are grouped according to the applications they run for example webservers, db servers, ubuntu server etc..
In the inventory file we can create groups

```
[webserver]
172.31.17.152

[dbserver]
172.31.29.133

[ubuntu]
172.31.17.152
localhost

[centos]
172.31.29.133
```

## Ansible has two kinds of inventory
* Static Inventory
* Dynamic Inventory


## Inventory basics: formats, hosts, and groups

    mail.example.com

    [webservers]
    foo.example.com
    bar.example.com

    [dbservers]
    one.example.com
    two.example.com
    three.example.com

## Adding ranges of hosts
```
  webservers:
    hosts:
      www[01:50].example.com:
```

## Inventory aliases

hosts:
    jumper:
      ansible_port: 5555
      ansible_host: 192.0.2.50

## Assigning a variable to many machines: group variables:
    [atlanta]
    host1
    host2

    [atlanta:vars]
    ntp_server=ntp.atlanta.example.com
    proxy=proxy.atlanta.example.com

```yml
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.com
```
## Inheriting variable values: group variables for groups of groups:

```yml
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

# Organizing host and group variables:
## Using multiple inventory sources ```ansible-playbook get_logs.yml -i staging -i production```

## Connecting to hosts: behavioral inventory parameters
* ansible_connection
Connection type to the host. This can be the name of any of ansible’s connection plugins. SSH protocol types are smart, ssh or paramiko. The default is smart. Non-SSH based types are described in the next section.

### General for all connections:

### ansible_host
The name of the host to connect to, if different from the alias you wish to give to it.

### ansible_port
The connection port number, if not the default (22 for ssh)

### ansible_user
The user name to use when connecting to the host

### ansible_password
The password to use to authenticate to the host (never store this variable in plain text; always use a vault. See Keep vaulted variables safely visible)

## Specific to the SSH connection:

### ansible_ssh_private_key_file
Private key file used by ssh. Useful if using multiple keys and you don’t want to use SSH agent.

### ansible_ssh_common_args
This setting is always appended to the default command line for sftp, scp, and ssh. Useful to configure a ProxyCommand for a certain host (or group).

### ansible_sftp_extra_args
This setting is always appended to the default sftp command line.

### ansible_scp_extra_args
This setting is always appended to the default scp command line.

### ansible_ssh_extra_args
This setting is always appended to the default ssh command line.

### ansible_ssh_pipelining
Determines whether or not to use SSH pipelining. This can override the pipelining setting in ansible.cfg.

### ansible_ssh_executable (added in version 2.2)
This setting overrides the default behavior to use the system ssh. This can override the ssh_executable setting in ansible.cfg.

## Privilege escalation (see Ansible Privilege Escalation for further details):

### ansible_become
Equivalent to ansible_sudo or ansible_su, allows to force privilege escalation

### ansible_become_method
Allows to set privilege escalation method

### ansible_become_user
Equivalent to ansible_sudo_user or ansible_su_user, allows to set the user you become through privilege escalation

### ansible_become_password
Equivalent to ansible_sudo_password or ansible_su_password, allows you to set the privilege escalation password (never store this variable in plain text; always use a vault. See Keep vaulted variables safely visible)

### ansible_become_exe
Equivalent to ansible_sudo_exe or ansible_su_exe, allows you to set the executable for the escalation method selected

### ansible_become_flags
Equivalent to ansible_sudo_flags or ansible_su_flags, allows you to set the flags passed to the selected escalation method. This can be also set globally in ansible.cfg in the sudo_flags option

### Remote host environment parameters:

### ansible_shell_type
The shell type of the target system. You should not use this setting unless you have set the ansible_shell_executable to a non-Bourne (sh) compatible shell. By default commands are formatted using sh-style syntax. Setting this to csh or fish will cause commands executed on target systems to follow those shell’s syntax instead.

### ansible_python_interpreter
The target host python path. This is useful for systems with more than one Python or not located at /usr/bin/python such as *BSD, or where /usr/bin/python is not a 2.X series Python. We do not use the /usr/bin/env mechanism as that requires the remote user’s path to be set right and also assumes the python executable is named python, where the executable might be named something like python2.6.

### ansible_*_interpreter
Works for anything such as ruby or perl and works just like ansible_python_interpreter. This replaces shebang of modules which will run on that host.

## New in version 2.1.

### ansible_shell_executable
This sets the shell the ansible controller will use on the target machine, overrides executable in ansible.cfg which defaults to /bin/sh. You should really only change it if is not possible to use /bin/sh (in other words, if /bin/sh is not installed on the target machine or cannot be run from sudo.)

## Non-SSH connection types
### local
This connector can be used to deploy the playbook to the control machine itself.

### docker

### This connector deploys the playbook directly into Docker containers using the local Docker client. The following parameters are processed by this connector:

### ansible_host
The name of the Docker container to connect to.

### ansible_user
The user name to operate within the container. The user must exist inside the container.

### ansible_become
If set to true the become_user will be used to operate within the container.

### ansible_docker_extra_args
Could be a string with any additional arguments understood by Docker, which are not command specific. This parameter is mainly used to configure a remote Docker daemon to use.


## local
This connector can be used to deploy the playbook to the control machine itself.

## docker
This connector deploys the playbook directly into Docker containers using the local Docker client. The following parameters are processed by this connector:

## ansible_host
The name of the Docker container to connect to.

## ansible_user
The user name to operate within the container. The user must exist inside the container.

## ansible_become
If set to true the become_user will be used to operate within the container.

## ansible_docker_extra_args
Could be a string with any additional arguments understood by Docker, which are not command specific. This parameter is mainly used to configure a remote Docker daemon to use.

# Dynamic Inventory:
For the business requirements, we need to create, remove and modify the hosts in our infrastructure environment very frequently and to manage those nodes via Ansible, one need to update the inventory file very frequently. So, static inventory file is not suitable. In such case, we need an inventory file which updates itself.

This type of inventory file is called Dynamic inventory file, which have dynamic contents. In our environment, the sources of host nodes can be AWS, OpenStack, Cobbler, LDAP systems. These systems have their list of nodes and Ansible supports to integrates such external dynamic inventory.


## In Ansible we have two ways to connect to external dynamic inventories:
* Via Scripts: This is the older way and ensures backward compatibility. But don’t use it with latest versions.
* Using Plugins: This is recommended over scripts for dynamic inventory, as plugins are updated with Ansible core code to take advantage of it.
## Using below command you can get the list of available inventory plugins:
### Code:
```
ansible-doc -t inventory -l
```
* From that list you can select the plugin which is reasonable for your source environment.

* Then you can get details or documentation about that plugin using command like below:

### Code:
```
ansible-doc -t inventory <plugin name>
```
## Examples of Ansible Dynamic Inventory
* Given below are the examples of ansible dynamic inventory:
* As this is an advance feature in Ansible, we will use a very useful and common inventory plugin named aws_ec2.

### Example #1
* For this we create a source file named demo_aws_ec2.yaml. which have below contents.
```yml
plugin: aws_ec2 regions:
ap-south-1 filters:
tag:tagtype: testing
```
* Also, we need to set and export at least below env variables for making connection with AWS environment.
* The values give to these are Access Key ID and Secret Access Key of your account.
```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```
## Once above is completed, you must have below requirements on your Ansible controller node:
* Boto3
* botocore

## Now, in our environment, we have EC2 instances tagged differently.
* We try to get list of same using tags like below:

### Code:
```
ansible-inventory -i demo_aws_ec2.yaml –graph
```
