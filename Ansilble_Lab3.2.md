# 2. Gather Facts

1. Using the Ansible setup module, run an ad hoc command to retrieve the facts for all of the servers in the db group:
`ansible db -m setup`

* The output displays all of the facts gathered for appdb1 server in JSON format.

2. Review the variables displayed.

3. Filter the facts matching the ansible_user expression and append a wildcard to match all of the facts starting with ansible_user:
`ansible db -m setup -a 'filter=ansible_user*'`
~~~json
ppdb1.cf3d.internal | SUCCESS => {
    "ansible_facts": {
        "ansible_user_dir": "/home/devops", 
        "ansible_user_gecos": "", 
        "ansible_user_gid": 1001, 
        "ansible_user_id": "devops", 
        "ansible_user_shell": "/bin/bash", 
        "ansible_user_uid": 1001, 
        "ansible_userspace_architecture": "x86_64", 
        "ansible_userspace_bits": "64", 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}
~~~

# 3. Create Custom Facts

1. Create a fact file named *custom.fact*:
~~~bash
`cat << EOF > custom.fact
[general]
package = httpd
service = httpd
state = started
EOF`
~~~
* This defines the package to install and the service to start on app1 and app2.

2. Create a *setup_facts.yml* playbook to create the */etc/ansible/facts.d* remote directory and save the *custom.fact* file to it:
~~~bash
cat << EOF > setup_facts.yml
- name: Install remote facts
  hosts: webservers
  become: yes
  vars:
    remote_dir: /etc/ansible/facts.d
    facts_file: custom.fact
  tasks:
  - name: Create the remote directory
    file:
      state: directory
      recurse: yes
      path: "{{ remote_dir }}"
  - name: Install the new facts
    copy:
      src: "{{ facts_file }}"
      dest: "{{ remote_dir }}"
EOF
~~~

3. Run the playbook:   
~~~bash
`ansible-playbook setup_facts.yml`

PLAY [Install remote facts] ********************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

TASK [Create the remote directory] *************************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

TASK [Install the new facts] *******************************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

PLAY RECAP *************************************************************************************************
app1.cf3d.internal         : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
app2.cf3d.internal         : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

4. Using the setup module, run an ad hoc command to display only the ansible_local section, which contains user-defined facts:
`ansible webservers -m setup -a 'filter=ansible_local'`
~~~json
app2.cf3d.internal | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {
            "custom": {
                "general": {
                    "package": "httpd", 
                    "service": "httpd", 
                    "state": "started"
                }
            }
        }, 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}
app1.cf3d.internal | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {
            "custom": {
                "general": {
                    "package": "httpd", 
                    "service": "httpd", 
                    "state": "started"
                }
            }
        }, 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}
~~~

# 4. Use Facts to configure web servers

Write a playbook that uses both default and user-defined facts to configure the webservers host group, and make sure that all of the tasks are defined.

1. Create the first task, which installs the httpd package, using the user fact for the name of the package.

2. Create another task that uses the custom fact to start the httpd service:
~~~bash
cat << EOF > setup_facts_httpd.yml
- name: Install Apache and starts the service
  hosts: webservers
  become: yes
  tasks:
  - name: Install the required package
    yum:
      name: "{{ ansible_local.custom.general.package }}"
      state: latest

  - name: Start the service
    service:
      name: "{{ ansible_local.custom.general.service }}"
      state: "{{ ansible_local.custom.general.state }}"
EOF
~~~

3. Run the playbook
~~~bash
ansible-playbook setup_facts_httpd.yml 

PLAY [Install Apache and starts the service] ***************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [app2.cf3d.internal]
ok: [app1.cf3d.internal]

TASK [Install the required package] ************************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

TASK [Start the service] ***********************************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

PLAY RECAP *************************************************************************************************
app1.cf3d.internal         : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
app2.cf3d.internal         : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

4. Use an ad hoc command to determine whether the *httpd* service is running on *webservers*:
`ansible webservers -m command -a 'systemctl status httpd`
~~~bash
app1.cf3d.internal | CHANGED | rc=0 >>
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-06-04 11:34:12 UTC; 3min 35s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 2037 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─2037 /usr/sbin/httpd -DFOREGROUND
           ├─2038 /usr/sbin/httpd -DFOREGROUND
           ├─2039 /usr/sbin/httpd -DFOREGROUND
           ├─2040 /usr/sbin/httpd -DFOREGROUND
           ├─2041 /usr/sbin/httpd -DFOREGROUND
           └─2042 /usr/sbin/httpd -DFOREGROUND
app2.cf3d.internal | CHANGED | rc=0 >>
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-06-04 11:34:12 UTC; 3min 35s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 2054 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─2054 /usr/sbin/httpd -DFOREGROUND
           ├─2055 /usr/sbin/httpd -DFOREGROUND
           ├─2056 /usr/sbin/httpd -DFOREGROUND
           ├─2057 /usr/sbin/httpd -DFOREGROUND
           ├─2058 /usr/sbin/httpd -DFOREGROUND
           └─2059 /usr/sbin/httpd -DFOREGROUND
~~~
