# 2. Create task file
1. Create a *tasks* directory under *~/ansible_implementation*.

2. In the *tasks* directory, create the *environment.yml* task file.

3. In the playbook, define the two tasks that install and start the web server.

4. Use the *package* variable for the package name, *service* for the service name, and *svc_state* for the service state.

`mkdir tasks`
`cd tasks`
`cat << EOF > environment.yml`
~~~yaml
  - name: Install the {{ package }} package
    yum:
      name: "{{ package }}"
      state: latest
  - name: Start the {{ service }} service
    service:
      name: "{{ service }}"
      state: "{{ svc_state }}"
EOF
~~~

`cd ..`

# 3. Create variable file
1. Create and change to the *vars* directory:
`mkdir vars`
`cd vars`

2. Create the *variables.yml* variable file with the following content:
~~~bash
`cat << EOF > variables.yml`
firewall_pkg: firewalld
EOF
~~~

* The file defines the firewall_pkg variable in YAML format.

# 4. Create main playbook

* Create and edit the main playbook, main_playbook.yml, which imports the tasks and variables, and installs and configures the firewalld service.

1. Create the playbook *main_playbook.yml*.

2. Add the *webservers* host group and define a *rule* variable with a value of *http*.

3. Define the first task with the *include_vars* module and the *variables.yml* variable file.
   * The *include_vars* module imports extra variables that are used by other tasks in the playbook.

4. Define a task that uses the *import_tasks* module to include the base *environment.yml* playbook:

    a. Because the three defined variables are used in the base playbook, but are not defined, include a *vars* block.
    b. Set three variables in the *vars* section:
       * *package: httpd*
       * *service: httpd*
       * *svc_state: started*

5. Create a task that installs the *firewalld* package using the *firewall_pkg variable*.

6. Create a task that starts the *firewalld* service.

7. Create a task that adds a firewall rule for the HTTP service using the *rule* variable.

8. Add a task that creates the *index.html* file for the web server using the *copy* module:
   a. Create the file with the Ansible *ansible_fqdn* fact, which returns the fully qualified domain name.
   b. Include a time stamp in the file using an Ansible fact.

`cat << EOF > main_playbook.yml`
~~~yaml
- hosts: webservers
  become: yes
  vars:
    rule: http
  tasks:
    - name: Include the variables from the YAML file
      include_vars: vars/variables.yml

    - name: Include the environment file and set the variables
      import_tasks: tasks/environment.yml
      vars:
        package: httpd
        service: httpd
        svc_state: started

    - name: Install the firewall
      yum:
        name: "{{ firewall_pkg }}"
        state: latest

    - name: Start the firewall
      service:
        name: firewalld
        state: started
        enabled: true

    - name: Open the port for {{ rule }}
      firewalld:
        service: "{{ rule }}"
        immediate: true
        permanent: true
        state: enabled

    - name: Create index.html
      copy:
        content: "{{ ansible_fqdn }} has been customized using Ansible on the {{ ansible_date_time.date }}\n"
        dest: /var/www/html/index.html
~~~
`EOF`

9. Verify the syntax of the main_playbook.yml playbook:
`ansible-playbook --syntax-check main_playbook.yml `
playbook: main_playbook.yml

# 5. Run playbook
`ansible-playbook main_playbook.yml`
~~~bash
PLAY [webservers] ******************************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [app2.cf3d.internal]
ok: [app1.cf3d.internal]

TASK [Include the variables from the YAML file] ************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

TASK [Install the httpd package] ***************************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

TASK [Start the httpd service] *****************************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

TASK [Install the firewall] ********************************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

TASK [Start the firewall] **********************************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

TASK [Open the port for http] ******************************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

TASK [Create index.html] ***********************************************************************************
changed: [app2.cf3d.internal]
changed: [app1.cf3d.internal]

PLAY RECAP *************************************************************************************************
app1.cf3d.internal         : ok=8    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
app2.cf3d.internal         : ok=8    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

* Ansible starts by including the environment.yml playbook and running its tasks, then continues to execute the tasks defined in the main playbook.

2. Confirm that the *app1* web server is reachable from *bastion*:
`curl http://app1.${GUID}.internal`
ip-192-168-0-117.ec2.internal has been customized using Ansible on the 2020-06-04

