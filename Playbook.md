## Playbook Instruction:

```yml
- name: < Name of the playbook >
  hosts: < where do you want to execute playbook > # Example: inventory [server ip Address ]
  become: yes # For installation do we need to become sudo user
  var :
    key: value # example Package Name: openjdk-11-jdk
  tasks: # run multiple tanks by using different Modules
  # apt-get install openjdk 
    - name: test
      apt :
        name: openjdk-11-jdk 
        update_cache: yes   # boolen expersions  ==> yes or No
        state: present

    - name: <name of the task>
      <module>: <desired state>
    
    - name: <name of the task>
      <module>: <desired state>
```

## Playbook for java installtion 

```yml
---
- name: java_install version 11
  hosts: all    
  become: yes 
  var : 
    centos_java_package: java-11-openjdk-devel
    ubuntu_java_package: openjdk-11-jdk  
  tasks:
     - name: fail on unsupported versions
      fail:
        msg: This playbook supports only centos and ubuntu distributions 
      when: 
        - ansible_facts['distribution'] != "Ubuntu"
        - ansible_facts['distribution'] != "CentOS" 
    - name: update packages and install java
      apt:
        name: "{{ ubuntu_java_package }}"
        update_cache: yes
        state: present
      when: ansible_facts['distribution'] == "Ubuntu"
    - name: install java for centos instances
      yum:
        name: "{{ centos_java_package }}"
        state: present
      when: ansible_facts['distribution'] == "CentOS" 

```

# ansible_facts[‘distribution’]
## Possible values (sample, not complete list):

* Alpine
* Altlinux
* Amazon
* Archlinux
* ClearLinux
* Coreos
* CentOS
* Debian
* Fedora
* Gentoo
* Mandriva
* NA
* OpenWrt
* OracleLinux
* RedHat
* Slackware
* SLES
* SMGL
* SUSE
* Ubuntu
* VMwareESX

