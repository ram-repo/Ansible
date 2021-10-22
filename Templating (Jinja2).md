## Templating (Jinja2)
* Ansible uses Jinja2 templating to enable dynamic expressions and access to variables.
* First and foremost advantage of using Jinja2 is we can avoid static file copies.
* Lets create a new folder for templates and create a tomcat.service.j2 file
## Template 
```yml 
 -name: template
  ansibe.buildin.template:
    path: ./template/filename.j2
    desc: /etc/ansible/filename
```

```yml
---
- name: learning template expressions
  hosts: all
  tasks:
    - name: default value for undefined variable
      debug: 
        msg: "{{ username | default('admin')}}"
    - name: default value while looking up for environment variables
      debug:
        msg: "{{ lookup(env, 'MY_APP')| default('qthrms'), true}}" 
```


```yml
- hosts: localhost 
  connection: local 
  gather_facts: no 
  vars: 
    names: 
     - first: Paul 
       last: Thompson 
     - first: Rod 
       last: Johnson
  tasks:
    - debug: 
        msg={{ names | map(attribute='first') | list }} 
    - debug: 
        msg={{ names | map(attribute='last') | list }} 
    - debug: 
        msg={{ names | map('upper') | list }} 
    - debug: 
        msg={{ names | map(attribute='last') | map('upper') | list }}

```