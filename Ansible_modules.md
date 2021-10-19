
## Ansible Modules :

```yml
  - name: update packages and install java
      apt:
        name: openjdk-8-jdk
        update_cache: yes
        state: present
      when: ansible_facts['distribution'] == "Ubuntu"
    - name: install java for centos instances
      yum:
        name: java-11-openjdk-devel
        state: present
      when: ansible_facts['distribution'] == "CentOS" 

```
## copy Modules in Ansible :
```yml

- name: Copy file with owner and permissions
  ansible.builtin.copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: '0644'

- name: Copy file with owner and permission, using symbolic representation
  ansible.builtin.copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: u=rw,g=r,o=r

- name: Another symbolic mode example, adding some permissions and removing others
  ansible.builtin.copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: u+rw,g-wx,o-rwx

- name: Copy a new "ntp.conf" file into place, backing up the original if it differs from the copied version
  ansible.builtin.copy:
    src: /mine/ntp.conf
    dest: /etc/ntp.conf
    owner: root
    group: root
    mode: '0644'
    backup: yes

- name: Copy a new "sudoers" file into place, after passing validation with visudo
  ansible.builtin.copy:
    src: /mine/sudoers
    dest: /etc/sudoers
    validate: /usr/sbin/visudo -csf %s

- name: Copy a "sudoers" file on the remote machine for editing
  ansible.builtin.copy:
    src: /etc/sudoers
    dest: /etc/sudoers.edit
    remote_src: yes
    validate: /usr/sbin/visudo -csf %s

- name: Copy using inline content
  ansible.builtin.copy:
    content: '# This file was moved to /etc/other.conf'
    dest: /etc/mine.conf

- name: If follow=yes, /path/to/file will be overwritten by contents of foo.conf
  ansible.builtin.copy:
    src: /etc/foo.conf
    dest: /path/to/link  # link to /path/to/file
    follow: yes

- name: If follow=no, /path/to/link will become a file and be overwritten by contents of foo.conf
  ansible.builtin.copy:
    src: /etc/foo.conf
    dest: /path/to/link  # link to /path/to/file
    follow: no

```

## untar files in Ansible Modules
```yml
- name: Extract foo.tgz into /var/lib/foo
  ansible.builtin.unarchive:
    src: foo.tgz
    dest: /var/lib/foo

- name: Unarchive a file that is already on the remote machine
  ansible.builtin.unarchive:
    src: /tmp/foo.zip
    dest: /usr/local/bin
    remote_src: yes

- name: Unarchive a file that needs to be downloaded (added in 2.0)
  ansible.builtin.unarchive:
    src: https://example.com/example.zip
    dest: /usr/local/bin
    remote_src: yes

- name: Unarchive a file with extra options
  ansible.builtin.unarchive:
    src: /tmp/foo.zip
    dest: /usr/local/bin
    extra_opts:
    - --transform
    - s/^xxx/yyy/
```
## Url Modules

```yml 
- name: Check that you can connect (GET) to a page and it returns a status 200
  uri:
    url: http://www.example.com

- name: Check that a page returns a status 200 and fail if the word AWESOME is not in the page contents
  uri:
    url: http://www.example.com
    return_content: yes
  register: this
  failed_when: "'AWESOME' not in this.content"

- name: Create a JIRA issue
  uri:
    url: https://your.jira.example.com/rest/api/2/issue/
    user: your_username
    password: your_pass
    method: POST
    body: "{{ lookup('file','issue.json') }}"
    force_basic_auth: yes
    status_code: 201
    body_format: json

- name: Login to a form based webpage, then use the returned cookie to access the app in later tasks
  uri:
    url: https://your.form.based.auth.example.com/index.php
    method: POST
    body_format: form-urlencoded
    body:
      name: your_username
      password: your_password
      enter: Sign in
    status_code: 302
  register: login

- name: Login to a form based webpage using a list of tuples
  uri:
    url: https://your.form.based.auth.example.com/index.php
    method: POST
    body_format: form-urlencoded
    body:
    - [ name, your_username ]
    - [ password, your_password ]
    - [ enter, Sign in ]
    status_code: 302
  register: login

- name: Upload a file via multipart/form-multipart
  uri:
    url: https://httpbin.org/post
    method: POST
    body_format: form-multipart
    body:
      file1:
        filename: /bin/true
        mime_type: application/octet-stream
      file2:
        content: text based file content
        filename: fake.txt
        mime_type: text/plain
      text_form_field: value

- name: Connect to website using a previously stored cookie
  uri:
    url: https://your.form.based.auth.example.com/dashboard.php
    method: GET
    return_content: yes
    headers:
      Cookie: "{{ login.cookies_string }}"

- name: Queue build of a project in Jenkins
  uri:
    url: http://{{ jenkins.host }}/job/{{ jenkins.job }}/build?token={{ jenkins.token }}
    user: "{{ jenkins.user }}"
    password: "{{ jenkins.password }}"
    method: GET
    force_basic_auth: yes
    status_code: 201

- name: POST from contents of local file
  uri:
    url: https://httpbin.org/post
    method: POST
    src: file.json

- name: POST from contents of remote file
  uri:
    url: https://httpbin.org/post
    method: POST
    src: /path/to/my/file.json
    remote_src: yes

- name: Create workspaces in Log analytics Azure
  uri:
    url: https://www.mms.microsoft.com/Embedded/Api/ConfigDataSources/LogManagementData/Save
    method: POST
    body_format: json
    status_code: [200, 202]
    return_content: true
    headers:
      Content-Type: application/json
      x-ms-client-workspace-path: /subscriptions/{{ sub_id }}/resourcegroups/{{ res_group }}/providers/microsoft.operationalinsights/workspaces/{{ w_spaces }}
      x-ms-client-platform: ibiza
      x-ms-client-auth-token: "{{ token_az }}"
    body:

- name: Pause play until a URL is reachable from this host
  uri:
    url: "http://192.0.2.1/some/test"
    follow_redirects: none
    method: GET
  register: _result
  until: _result.status == 200
  retries: 720 # 720 * 5 seconds = 1hour (60*60/5)
  delay: 5 # Every 5 seconds

# There are issues in a supporting Python library that is discussed in
# https://github.com/ansible/ansible/issues/52705 where a proxy is defined
# but you want to bypass proxy use on CIDR masks by using no_proxy
- name: Work around a python issue that doesn't support no_proxy envvar
  uri:
    follow_redirects: none
    validate_certs: false
    timeout: 5
    url: "http://{{ ip_address }}:{{ port | default(80) }}"
  register: uri_data
  failed_when: false
  changed_when: false
  vars:
    ip_address: 192.0.2.1
  environment: |
      {
        {% for no_proxy in (lookup('env', 'no_proxy') | regex_replace('\s*,\s*', ' ') ).split() %}
          {% if no_proxy | regex_search('\/') and
*     no_proxy | ipaddr('net') != '' and
*     no_proxy | ipaddr('net') != false and
*     ip_address | ipaddr(no_proxy) is not none and
*     ip_address | ipaddr(no_proxy) != false %}
* 'no_proxy': '{{ ip_address }}'
          {% elif no_proxy | regex_search(':') != '' and
*       no_proxy | regex_search(':') != false and
*       no_proxy == ip_address + ':' + (port | default(80)) %}
* 'no_proxy': '{{ ip_address }}:{{ port | default(80) }}'
          {% elif no_proxy | ipaddr('host') != '' and
*       no_proxy | ipaddr('host') != false and
*       no_proxy == ip_address %}
* 'no_proxy': '{{ ip_address }}'
          {% elif no_proxy | regex_search('^(\*|)\.') != '' and
*       no_proxy | regex_search('^(\*|)\.') != false and
*       no_proxy | regex_replace('\*', '') in ip_address %}
* 'no_proxy': '{{ ip_address }}'
          {% endif %}
        {% endfor %}
      }
  
```
# Conditional Modules

## Basic conditionals with when

```yml
tasks:
  - name: Configure SELinux to start mysql on any port
    ansible.posix.seboolean:
      name: mysql_connect_any
      state: true
      persistent: yes
    when: ansible_selinux.status == "enabled"
    # all variables can be used directly in conditionals without double curly braces
```

## Conditionals based on ansible_facts
```yml
- name: Show facts available on the system
  ansible.builtin.debug:
    var: ansible_facts
```

## If you have multiple conditions, you can group them with parentheses:
```yml
tasks:
  - name: Shut down CentOS 6 and Debian 7 systems
    ansible.builtin.command: /sbin/shutdown -t now
    when: (ansible_facts['distribution'] == "CentOS" and ansible_facts['distribution_major_version'] == "6") or
          (ansible_facts['distribution'] == "Debian" and ansible_facts['distribution_major_version'] == "7")
```
## You can use logical operators to combine conditions. When you have multiple conditions that all need to be true (that is, a logical and), you can specify them as a list:
```yml
tasks:
  - name: Shut down CentOS 6 systems
    ansible.builtin.command: /sbin/shutdown -t now
    when:
      - ansible_facts['distribution'] == "CentOS"
      - ansible_facts['distribution_major_version'] == "6"

#  If a fact or variable is a string, and you need to run a mathematical comparison on it, use a filter to ensure that Ansible reads the value as an integer:

tasks:
  - ansible.builtin.shell: echo "only on Red Hat 6, derivatives, and later"
    when: ansible_facts['os_family'] == "RedHat" and ansible_facts['lsb']['major_release'] | int >= 6

```
## Conditions based on registered variables
```yml
- name: Test play
  hosts: all

  tasks:

      - name: Register a variable
        ansible.builtin.shell: cat /etc/motd
        register: motd_contents

      - name: Use the variable in conditional statement
        ansible.builtin.shell: echo "motd contains the word hi"
        when: motd_contents.stdout.find('hi') != -1
- name: Registered variable usage as a loop list
  hosts: all
  tasks:

    - name: Retrieve the list of home directories
      ansible.builtin.command: ls /home
      register: home_dirs

    - name: Add home dirs to the backup spooler
      ansible.builtin.file:
        path: /mnt/bkspool/{{ item }}
        src: /home/{{ item }}
        state: link
      loop: "{{ home_dirs.stdout_lines }}"
      # same as loop: "{{ home_dirs.stdout.split() }}"

- name: check registered variable for emptiness
  hosts: all

  tasks:

      - name: List contents of directory
        ansible.builtin.command: ls mydir
        register: contents

      - name: Check contents for emptiness
        ansible.builtin.debug:
          msg: "Directory is empty"
        when: contents.stdout == ""

tasks:
  - name: Register a variable, ignore errors and continue
    ansible.builtin.command: /bin/false
    register: result
    ignore_errors: true

  - name: Run only if the task that registered the "result" variable fails
    ansible.builtin.command: /bin/something
    when: result is failed

  - name: Run only if the task that registered the "result" variable succeeds
    ansible.builtin.command: /bin/something_else
    when: result is succeeded

  - name: Run only if the task that registered the "result" variable is skipped
    ansible.builtin.command: /bin/still/something_else
    when: result is skipped
```
## Using conditionals in loops
```yml
tasks:
    - name: Run with items greater than 5
      ansible.builtin.command: echo {{ item }}
      loop: [ 0, 2, 4, 6, 8, 10 ]
      when: item > 5

- name: Skip the whole task when a loop variable is undefined
  ansible.builtin.command: echo {{ item }}
  loop: "{{ mylist|default([]) }}"
  when: item > 5

- name: The same as above using a dict
  ansible.builtin.command: echo {{ item.key }}
  loop: "{{ query('dict', mydict|default({})) }}"
  when: item.value > 5

```

## Loading custom facts

```yml 
tasks:
    - name: Gather site specific fact data
      action: site_facts

    - name: Use a custom fact
      ansible.builtin.command: /usr/bin/thingy
      when: my_custom_fact_just_retrieved_from_the_remote_system == '1234'

```

## Conditionals with re-use
``` yml 
# Conditionals with imports
# When you add a conditional to an import statement, Ansible applies the condition to all tasks within the imported file. This behavior is the equivalent of Tag inheritance: adding tags to multiple tasks. Ansible applies the condition to every task, and evaluates each task separately. For example, you might have a playbook called main.yml and a tasks file called other_tasks.yml:

# all tasks within an imported file inherit the condition from the import statement
# main.yml
- import_tasks: other_tasks.yml # note "import"
  when: x is not defined

# other_tasks.yml
- name: Set a variable
  ansible.builtin.set_fact:
    x: foo

- name: Print a variable
  ansible.builtin.debug:
    var: x
```


## Ansible expands this at execution time to the equivalent of:
```yml

- name: Set a variable if not defined
  ansible.builtin.set_fact:
    x: foo
  when: x is not defined
  # this task sets a value for x

- name: Do the task if "x" is not defined
  ansible.builtin.debug:
    var: x
  when: x is not defined
  # Ansible skips this task, because x is now defined

```

## Conditionals with includes:
```yml 
# Includes let you re-use a file to define a variable when it is not already defined

# main.yml
- include_tasks: other_tasks.yml
  when: x is not defined

# other_tasks.yml
- name: Set a variable
  ansible.builtin.set_fact:
    x: foo

- name: Print a variable
  ansible.builtin.debug:
    var: x

```

```yml
# main.yml
- include_tasks: other_tasks.yml
  when: x is not defined
  # if condition is met, Ansible includes other_tasks.yml

# other_tasks.yml
- name: Set a variable
  ansible.builtin.set_fact:
    x: foo
  # no condition applied to this task, Ansible sets the value of x to foo

- name: Print a variable
  ansible.builtin.debug:
    var: x
  # no condition applied to this task, Ansible prints the debug statement
```

```yml
For example, you can template out a configuration file that is very different between, say, CentOS and Debian:

- name: Template a file
  ansible.builtin.template:
    src: "{{ item }}"
    dest: /etc/myapp/foo.conf
  loop: "{{ query('first_found', { 'files': myfiles, 'paths': mypaths}) }}"
  vars:
    myfiles:
      - "{{ ansible_facts['distribution'] }}.conf"
      -  default.conf
    mypaths: ['search_location_one/somedir/', '/opt/other_location/somedir/']
```

## Systemd:
```yml
- name: Make sure a service unit is running
  ansible.builtin.systemd:
    state: started
    name: httpd

- name: Stop service cron on debian, if running
  ansible.builtin.systemd:
    name: cron
    state: stopped

- name: Restart service cron on centos, in all cases, also issue daemon-reload to pick up config changes
  ansible.builtin.systemd:
    state: restarted
    daemon_reload: yes
    name: crond

- name: Reload service httpd, in all cases
  ansible.builtin.systemd:
    name: httpd.service
    state: reloaded

- name: Enable service httpd and ensure it is not masked
  ansible.builtin.systemd:
    name: httpd
    enabled: yes
    masked: no

- name: Enable a timer unit for dnf-automatic
  ansible.builtin.systemd:
    name: dnf-automatic.timer
    state: started
    enabled: yes

- name: Just force systemd to reread configs (2.4 and above)
  ansible.builtin.systemd:
    daemon_reload: yes

- name: Just force systemd to re-execute itself (2.8 and above)
  ansible.builtin.systemd:
    daemon_reexec: yes

- name: Run a user service when XDG_RUNTIME_DIR is not set on remote login
  ansible.builtin.systemd:
    name: myservice
    state: started
    scope: user
  environment:
    XDG_RUNTIME_DIR: "/run/user/{{ myuid }}"

state: 
  - reloaded
    restarted
    started
    stopped
```

## File Module 
```yml
- name: Change file ownership, group and permissions
  ansible.builtin.file:
    path: /etc/foo.conf
    owner: foo
    group: foo
    mode: '0644'

- name: Give insecure permissions to an existing file
  ansible.builtin.file:
    path: /work
    owner: root
    group: root
    mode: '1777'

- name: Create a symbolic link
  ansible.builtin.file:
    src: /file/to/link/to
    dest: /path/to/symlink
    owner: foo
    group: foo
    state: link

- name: Create two hard links
  ansible.builtin.file:
    src: '/tmp/{{ item.src }}'
    dest: '{{ item.dest }}'
    state: hard
  loop:
    - { src: x, dest: y }
    - { src: z, dest: k }

- name: Touch a file, using symbolic modes to set the permissions (equivalent to 0644)
  ansible.builtin.file:
    path: /etc/foo.conf
    state: touch
    mode: u=rw,g=r,o=r

- name: Touch the same file, but add/remove some permissions
  ansible.builtin.file:
    path: /etc/foo.conf
    state: touch
    mode: u+rw,g-wx,o-rwx

- name: Touch again the same file, but do not change times this makes the task idempotent
  ansible.builtin.file:
    path: /etc/foo.conf
    state: touch
    mode: u+rw,g-wx,o-rwx
    modification_time: preserve
    access_time: preserve

- name: Create a directory if it does not exist
  ansible.builtin.file:
    path: /etc/some_directory
    state: directory
    mode: '0755'

- name: Update modification and access time of given file
  ansible.builtin.file:
    path: /etc/some_file
    state: file
    modification_time: now
    access_time: now

- name: Set access time based on seconds from epoch value
  ansible.builtin.file:
    path: /etc/another_file
    state: file
    access_time: '{{ "%Y%m%d%H%M.%S" | strftime(stat_var.stat.atime) }}'

- name: Recursively change ownership of a directory
  ansible.builtin.file:
    path: /etc/foo
    state: directory
    recurse: yes
    owner: foo
    group: foo

- name: Remove file (delete file)
  ansible.builtin.file:
    path: /etc/foo.txt
    state: absent

- name: Recursively remove directory
  ansible.builtin.file:
    path: /etc/foo
    state: absent
```
