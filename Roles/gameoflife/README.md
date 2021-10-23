Role Name
=========

This role helps in creating a tomcat environment and deploying game of life.

NOTE: if you want to use manager app on tomcat gui use these credentials username: "ansible" , password: "passwd"

Requirements
------------

some basic reqirments are that you should have a ubuntu machine that can connect to internet and make sure that python was installed on the node

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    ---
    - hosts: appserver(in my casse)
      become: yes
      roles:
         - gameoflife

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
