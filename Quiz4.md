1. What is the preferred method for declaring inventory variables?
In the host_vars and group_vars directories

2. How do you make modules return values which would be stored as Ansible facts?
The JSON returned by the module should have the 'ansible_facts' key, and then all the key/values would be stored as facts

3. Which module would you use to add a fact for hosts during runtime?
set_fact

4. Which strategy can significantly speed up the initial task run for some plays?
Disable gather_facts

5. Inventory variable paths can be used with which of the following?
include_vars

6. Which of the following lists variable precedence (from highest to lowest)?
['extra vars', 'role_vars/included_vars/play_vars/etc', 'gathered facts', 'inventory file connection vars', 'other inventory variables', 'role defaults']

7. Which would evaluate to the OS family of a server, defined in inventory as "someserver," while Ansible connected to a different server?
{{ hostvars['someserver']['ansible_os_family'] }}

8. In which directory should local facts be placed?
/etc/ansible/facts.d/

9. What must you add to the task in order to store the output of an Ansible module in a variable called 'results'?
register: results

10. Which of the following is not a valid variable name?
50_Shades_of_Ansible

11. Which of that he following can be used to import variables from a YAML file?
include_vars

12. Which of the following is a valid way of defining variables via the command line?
-e 'hosts=vipers user=starbuck'

