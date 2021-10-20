# Variables:
## Ansible defines several variables that are always available in a playbook

|Parameter |Description   |
|---|---|
|hostvars|A dictionary whose keys are Ansible host names and values are dictionaries that map variable names to values|
|inventory_hostname|Name of the current host as known by Ansible|
|group_names|A list of all groups that the current host is a member of|
|groups|A dictionary whose keys are Ansible group names and values are a list of hostnames that are members of the group. Includes all and ungrouped groups: {"all": […], "web": […], "ungrouped": […]}|
|play_hosts|A list of inventory hostnames that are active in the current play|
|ansible_version|A dictionary with Ansible version info: {'string': '2.9.13', 'full': '2.9.13', 'major': 2, 'minor': 9, 'revision': 13}|


## Define Variables :
### There are two types of variables which you can define in an inventory i.e. host variables and group variables:

## Host variables
We will use our default inventory from /etc/ansible/hosts which has below entries

```
server1
server2 http_port=8080 pkg=httpd
server3
```

```yml
---
 - name: Verifying host variable
   hosts: server2
   tasks:
   - name: Testing first variable (http_port)
     debug:
       msg: "The HTTP PORT is {{ http_port }}"
   - name: Testing second variable (pkg)
     debug:
       msg: "The package name is {{ pkg }}"
```

## Group Variables

```
[app]
server1
server3

[db]
server2

[db:vars]
http_port=8080
pkg=httpd
```

```yml
---
 - name: Verifying group variable
   hosts: db
   tasks:
   - name: Testing first variable (http_port)
     debug:
       msg: "The HTTP PORT is {{ http_port }}"
   - name: Testing second variable (pkg)
     debug:
       msg: "The package name is {{ pkg }}"
```



## Defining variables in playbook:

```yml
---
 - hosts: server2
   vars:
     port_no: 80
     pkg_name: httpd

   gather_facts: no
   tasks:
   - debug:
       msg:
        - "The value of port is {{ port_no }}"
        - "The value of pkg is {{ pkg_name }}"
        - "The value of hostname is {{ inventory_hostname }}"
        - "Ansible version is {{ ansible_version }}"
```

## Defining variables using command line:

```(Highest) ansible-playbook -e var=value```

