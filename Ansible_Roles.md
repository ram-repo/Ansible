# Ansible Roles:
* Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse them and share them with other users.

## Role directory structure
* An Ansible role has a defined directory structure with eight main standard directories. You must include at least one of these directories in each role. You can omit any directories the role does not use. For example:
```yml
# playbooks
site.yml
webservers.yml
fooservers.yml
roles/
    common/
        tasks/
        handlers/
        library/
        files/
        templates/
        vars/
        defaults/
        meta/
    webservers/
        tasks/
        defaults/
        meta/
```
## By default Ansible will look in each directory within a role for a main.yml file for relevant content (also main.yaml and main):

### ```tasks/main.yml ```- the main list of tasks that the role executes.

### ```handlers/main.yml ```- handlers, which may be used within or outside this role.

### ```library/my_module.py ```- modules, which may be used within this role (see Embedding modules and plugins in roles for more information).

### ```defaults/main.yml ```- default variables for the role (see Using Variables for more information). These variables have the lowest priority of any variables available, and can be easily overridden by any other variable, including inventory variables.

### ```vars/main.yml ```- other variables for the role (see Using Variables for more information).

### ```files/main.yml ```- files that the role deploys.

### ```templates/main.yml ```- templates that the role deploys.

### ```meta/main.yml ```- metadata for the role, including role dependencies.


```yml 
# roles/example/tasks/main.yml
- name: Install the correct web server for RHEL
  import_tasks: redhat.yml
  when: ansible_facts['os_family']|lower == 'redhat'

- name: Install the correct web server for Debian
  import_tasks: debian.yml
  when: ansible_facts['os_family']|lower == 'debian'

# roles/example/tasks/redhat.yml
- name: Install web server
  ansible.builtin.yum:
    name: "httpd"
    state: present

# roles/example/tasks/debian.yml
- name: Install web server
  ansible.builtin.apt:
    name: "apache2"
    state: present
```

## Storing and finding roles:
### By default, Ansible looks for roles in two locations:

* in a directory called roles/, relative to the playbook file

* in /etc/ansible/roles

* If you store your roles in a different location, set the roles_path configuration option so Ansible can find your roles. Checking shared roles into a single location makes them easier to use in multiple playbooks. See Configuring Ansible for details about managing settings in ansible.cfg.

### Alternatively, you can call a role with a fully qualified path:

```yml
---
- hosts: webservers
  roles:
    - role: '/path/to/my/roles/common'
Using roles
```


