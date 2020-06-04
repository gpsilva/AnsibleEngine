# 2. Create playbook to set up web services
1. Create the *variable_test.yml*  playbook and define the following variables in the vars section:

* *web_pkg* defines the name of the package to install for the web server
* *firewall_pkg* defines the name of the firewall package
* *web_service* defines the name of the web service to manage
* *firewall_service* defines the name of the firewall service to manage
* *python_pkg* defines a package to be installed for the uri module
* *rule* defines the service to open

2. Create the *tasks* block and add a first task, which uses the *yum* module to install the required packages.

3. Add two more tasks to start and enable the *httpd* and *firewalld* services.

4. Add a task that creates content in */var/www/html/index.html*.

5. Add a task that uses the *firewalld* module to add a rule for the web service.

`cat << EOF > variable_test.yml`
~~~yaml
- name: Install Apache and start the service
  hosts: webservers
  become: yes
  vars:
    web_pkg: httpd
    firewall_pkg: firewalld
    web_service: httpd
    firewall_service: firewalld
    python_pkg: python-httplib2
    rule: http
  tasks:
    - name: Install the required packages
      yum:
        name:
          - "{{ web_pkg  }}"
          - "{{ firewall_pkg }}"
          - "{{ python_pkg }}"
        state: latest
    - name: Start and enable the {{ firewall_service }} service
      service:
        name: "{{ firewall_service }}"
        enabled: true
        state: started

    - name: Start and enable the {{ web_service }} service
      service:
        name: "{{ web_service }}"
        enabled: true
        state: started
    - name: Create web content to be served
      copy:
        content: "Example web content"
        dest: /var/www/html/index.html
    - name: Open the port for {{ rule }}
      firewalld:
        service: "{{ rule }}"
        permanent: true
        immediate: true
        state: enabled

EOF
~~~
6. Check the syntax of the variable_test.yml playbook:
`ansible-playbook --syntax-check variable_test.yml`

# 3. Create playbook for smoke test

1. Name your playbook *webserver_smoketest.yml*.
2. Configure it to run only on *localhost*.
3. Create a task that uses the *uri* module to check a URL.
4. In this task, check for a status code of *200* to confirm that the server is running and configured properly.

`cat << EOF > webserver_smoketest.yml`
~~~yaml
- name: Verify the Apache service
  hosts: localhost
  tasks:
    - name: Ensure the webserver is reachable
      uri:
        url: http://app1.${GUID}.internal
        status_code: 200
~~~
`EOF`

# 4. Run playbooks
1. Run the *variable_test.yml* playbook:
`ansible-playbook variable_test.yml`

~~~bash
PLAY [Install Apache and start the service] ****************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

TASK [Install the required packages] ***********************************************************************
changed: [app2.cf3d.internal]
changed: [app1.cf3d.internal]

TASK [Start and enable the firewalld service] **************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

TASK [Start and enable the httpd service] ******************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

TASK [Create web content to be served] *********************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

TASK [Open the port for http] ******************************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

PLAY RECAP *************************************************************************************************
app1.cf3d.internal         : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
app2.cf3d.internal         : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

* Note that Ansible starts by installing the packages, and then starts and enables the services.

2. Run the *webserver_smoketest.yml* playbook to make sure that the web server is reachable:
`ansible-playbook webserver_smoketest.yml`
~~~bash
PLAY [Verify the Apache service] ***************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [localhost]

TASK [Ensure the webserver is reachable] *******************************************************************
ok: [localhost]

PLAY RECAP *************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

