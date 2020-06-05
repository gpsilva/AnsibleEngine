1. What executes before roles in a playbook?
pre_tasks

2. When do handlers execute?
Once, at the end of the play

3. What are handlers most often used for?
Restarting services

4. Which of the following describes how roles are used?
Roles are a way of automatically loading certain vars_files, tasks, and handlers

5. Under a role's tasks directory, main.yml should include which of the following?
A YAML list of tasks

6. Which of the following directories must be included in roles that are used by a playbook?
tasks

7. What is the default sequence for role and playbook task execution?
Playbook tasks execute as pre_tasks before applying role tasks

8. To configure a specific path for Ansible to look for roles, you can set the following in ansible.cfg:
roles_location

9. What can the meta directory set?
Dependencies on other roles

10. Variables that are declared in a task in the apache role can be used by the web role, if web is executed after apache.
TRUE

11. If roles/rolename/vars/main.yml exists, then the following is true:
Those variables overrule defaults/main.yml in that role

12. What is the defined process for creating and using a role?
Create the role directory structure; define the role content; use the role in a playbook

13. Using role defaults is least important in terms of variable precedence.
TRUE

14. Which best describes the dependency behavior of a role in a playbook?
A role can run multiple times as a dependency if allow_duplicates is set to "true" or "yes"

15. In a dependencies list, which entry would call the "apache" role with the variable "vhosts=true?"
- { role: apache, vhosts: true }

16. Under which conditions can Ansible Galaxy be used to install roles?
From the Ansible community to your playbook directories

17. Under which condition can you install multiple roles from Galaxy using a file with a list of role names?
Only if the file contains role names in the "username.rolename" format

18. What occurs when you execute the "ansible-galaxy init newrole1 -p /etc/ansible/roles" command?
It installs a role by the name of "newrole1" at /etc/ansible/roles

19. What is Ansible Galaxy?
A collection of community contributed roles

