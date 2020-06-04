# 2. Configure Static inventory to add host groups

1. Append the host group to the end of the */home/devops/ansible_implementation/hosts* static inventory file:
`cat << EOF >> /home/devops/ansible_implementation/hosts \`
`[lb] \`
`frontend1.${GUID}.internal \`

`[webservers] \`
`app1.${GUID}.internal \`
`app2.${GUID}.internal \`

`[db] \`
`appdb1.${GUID}.internal \`

`EOF`

# 3. Write a Playbook to verify connectivity
1. As the devops user, write a playbook to check connectivity to webservers:
`cat << EOF > check_webservers.yml \`
`- hosts: webservers \`
`  tasks: \`
`  - name: Check connectivity \`
`    ping: \`
`EOF `

2. Use *--syntax-check*  to verify the syntax of your playbook:
`ansible-playbook --syntax-check check_webservers.yml -u devops --private-key=~/.ssh/id_rsa`

playbook: check_webservers.yml

3. Use --check to perform a dry run of the playbook:
`ansible-playbook --check check_webservers.yml -u devops --private-key=~/.ssh/id_rsa`
~~~bash
PLAY [webservers] ******************************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

TASK [Check connectivity] **********************************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

PLAY RECAP *************************************************************************************************
app1.cf3d.internal         : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
app2.cf3d.internal         : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

4. Run the playbook
`ansible-playbook check_webservers.yml -u devops --private-key=~/.ssh/id_rsa`
~~~bash
PLAY [webservers] ******************************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [app2.cf3d.internal]
ok: [app1.cf3d.internal]

TASK [Check connectivity] **********************************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

PLAY RECAP *************************************************************************************************
app1.cf3d.internal         : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
app2.cf3d.internal         : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

# Set up static inventory to use inventory variables
Configure the hosts inventory file to use an inventory variable so that you do not have to include -u and --private-key options to specify the remote user and private key.

1. Append the inventory to the hosts file:
`cat << EOF >> /home/devops/ansible_implementation/hosts \`
`[webservers:vars] \`
`ansible_user = devops \`
`ansible_ssh_private_key_file = /home/devops/.ssh/id_rsa \`
`EOF`

2. Run the check_webservers.yml playbook again without specifying any options:
`ansible-playbook check_webservers.yml`
~~~bash
PLAY [webservers] ******************************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [app2.cf3d.internal]
ok: [app1.cf3d.internal]

TASK [Check connectivity] **********************************************************************************
ok: [app1.cf3d.internal]
ok: [app2.cf3d.internal]

PLAY RECAP *************************************************************************************************
app1.cf3d.internal         : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
app2.cf3d.internal         : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

# 5. Write, deploy, and test playbook
Write a playbook to deploy an Apache web server on the *webservers* host group using the *yum*, *service*, and *copy* modules.

1. Write a playbook to deploy the Apache (HTTPD) web server—but this time, rather than specifying the *--become* or *-b* option for privileged escalation, use *become: yes* in your playbook:
`cat << EOF > /home/devops/ansible_implementation/deploy_apache.yml \`
~~~yaml
- hosts: webservers
  become: yes
  tasks:
  - name: Install httpd package
    yum:
      name: httpd
      state: latest
  - name: Enable and start httpd service
    service:
       name: httpd
       state: started
       enabled: yes
  - name: Create index.html file for hosting static content
    copy:
      content: "Hoorraaayyy!!! My first playbook ran successfully"
      dest: /var/www/html/index.html
~~~
`EOF`

2. Run the playbook:
`ansible-playbook deploy_apache.yml`
~~~bash
PLAY [webservers] ******************************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [app2.cf3d.internal]
ok: [app1.cf3d.internal]

TASK [Install httpd package] *******************************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

TASK [Enable and start httpd service] **********************************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

TASK [Create index.html file for hosting static content] ***************************************************
changed: [app1.cf3d.internal]
changed: [app2.cf3d.internal]

PLAY RECAP *************************************************************************************************
app1.cf3d.internal         : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
app2.cf3d.internal         : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
~~~

3. Verify that you are able to access the web page on the app1 host:
`curl http://app1.${GUID}.internal`
Hoorraaayyy!!! My first playbook ran successfully

