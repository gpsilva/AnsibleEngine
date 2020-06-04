# 2. Configure Ansible Controller

* 2.1 Install Ansible
`sudo -i`
`yum install ansible`

* 2.2 Create a user on bastion host and generate SSH key pair
`useradd devops`
`echo 'r3dh4t1!' | passwd --stdin devops`
`su - devops`
`ssh-keygen -N '' -f ~/.ssh/id_rsa`
`ls -l ~/.ssh/`

# 3.2 Create user and setup SSH keys for remote host

1. Create the *devops* user on the remote hosts using Ansible *user* module:
`ansible all -m user -a "name=devops"`

2. Display the SSH public key for the *devops* user:
`cat /home/devops/.ssh/id_rsa.pub`

3. Add the SSH key to the authorized keys for the devops user:
`ansible all -m authorized_key -a "user=devops state=present key='ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDamxJkLdmbU5qx5FR/R2gCtn4Z30r57UV96Is0uvCksmT/AbG6jLlHZpvbhWfRnfyjfFeezYF4/dDMk/t8ye9HIhYX09EaluhOAxaPqEfw0u9ngi+9a6HVzXUWbZi1WtypkHwrC/DN7HQM4+6mPT3YRjd3VEJcetu0CwCqC1PbmcBMgN2pLlHuZNytcl8+2Ze3uPo1S+Lh9sOUZd1MABnuwF3AVT8AFAlE+zOFOsouXgrNVP8M2jVvGb50xa1F0iqYS9Lw+rEgRxOE2bpZB2m1kgXEnynCu/0nCAaDu4nmoxvK0+Oy0Hl4uIzmcn6I+7h7GMwcEZXYu/hFjAsnLXgD devops@bastion.cf3d.example.opentlc.com'"`

4. Configure *sudo* on the remote host for privileged escalation for the *devops* user:
`ansible all -m lineinfile -a "dest=/etc/sudoers state=present line='devops ALL=(ALL) NOPASSWD: ALL'"`

5. Verify the connection to the remot hosts from *bastion* (controller) as the *devops* user from the app1 server:
`su - devops`
`ssh app1`
`sudo -i`

# 3. Explore Ad Hoc commands
Configure ansible.cfg and the static inventory needed to complete this lab. You use -u to specify the devops user and --private-key to specify the private key.

1. Verify connectivity to the remote hosts:
`ansible frontends -m ping`

* The command is expected to fail because it is using the default ansible.cfg and the SSH keys defined in the default /etc/ansible/hosts static inventory.

2. Create a directory called ansible_implementation as your working directory for all future labs and an ansible.cfg file with a [defaults] section for specifying user-specific settings:
`mkdir ansible_implementation`
`cd ansible_implementation`
`cat << EOF > ansible.cfg \`
`> [defaults] \`
`> inventory = /home/devops/ansible_implementation/hosts \`
`> host_key_checking = False \`
`> EOF`

3. Create /home/devops/ansible_implementation/hosts as the static inventory, which contains the hostnames of all of the remote hosts:
`export GUID='hostname | awk -F"." '{print $2}'`
`cat << EOF > /home/devops/ansible_implementation/hosts \`
`frontend1.${GUID}.internal \ `
`appdb1.${GUID}.internal \`
`app1.${GUID}.internal \`
`support1.${GUID}.internal \`
`app2.${GUID}.internal \`
`EOF`

4. Test connectivity again:
`ansible frontend1.$GUID.internal -m ping -u devops --private-key=~/.ssh/id_rsa`

5. Test connectivity to all of the hosts:
`ansible all -m ping -u devops --private-key=~/.ssh/id_rsa`

6. Execute an ad hoc command on localhost to identify the user account used by Ansible to perform operations on managed hosts:
`ansible localhost -m command -a 'id'`
localhost | CHANGED | rc=0 >>
uid=1001(devops) gid=1001(devops) groups=1001(devops) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

7. Execute an ad hoc command to display the contents of the /etc/motd file on app1.${GUID}.internal as the devops user:
`ansible app1.${GUID}.internal -m command -a 'cat /etc/motd' -u devops --private-key=~/.ssh/id_rsa`
app1.cf3d.internal | CHANGED | rc=0 >>

8. Execute an ad hoc command using the copy module and the devops account to change the contents of the /etc/motd file to include the message "Managed by Ansible" on all of the remote hosts:
`ansible all -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd' -u devops --private-key=~/.ssh/id_rsa`

* Expect the ad hoc command to fail due to insufficient permissions.

9. Create the /etc/motd file on all of the hosts, but this time, escalate the root user’s privileges using -b or --become:
`ansible all -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd' -u devops --private-key=~/.ssh/id_rsa --become`

