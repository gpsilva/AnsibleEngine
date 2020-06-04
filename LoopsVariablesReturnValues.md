# Variables Overview

* Reusing content in playbooks helps reduce errors
  * Need to verify original material only

* To reuse content, use variables
  * Variable: String or number that gets assigned value

* When variable is invoked, Ansible replaces with value

* Lets you reuse information across:
* Playbooks
* Inventory files
* Tasks and roles
* Jinja2 template files

## Objects to Define with Variables

* Variables help organize workflow
* Consistent way to manage dynamic values for environment
* Use variables to define:
  * Users
  * Packages
  * Services
  * Files
  * Archives

## Multiple Variables
* You can assign multiple variables to piece of configuration related to same element
  * Examples: List of packages, services, users

* Alternative: Use arrays
  * Lets you build array that can be browsed

* Example: Variables assigned as multiple variables:
  * user1_first_name: Bob
  * user1_last_name: Jones
  * user1_home_dir: /users/bjones
  * user2_first_name: Anne
  * user2_last_name: Cook
  * user3_home_dir: /users/acook*

#### Array Example
* Example: users array
~~~yaml
users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook
~~~

#### Accessing Variables from an Array
* To access users from array:

* Returns 'Bob'
users.bjones.first_name

* Returns '/users/acook'
users.acook.home_dir

* Variable is defined as Python dictionary
  * Alternative method for accessing users:

* Returns 'Bob'
users['bjones']['first_name']

* Returns '/users/acook'
users['acook']['home_dir']

#### Considerations for Accessing Variables
* Both access methods valid
* Best practice: Use same access variables across files
  * Reduces errors
  * Facilitates troubleshooting

* Dot notation can cause problems if keys are same as Python methods/attributes
  * Examples: discard, copy, add
  * Python reserves names to perform operations against element

* Brackets notation helps reduce errors

* To avoid reserved keywords, use custom notation/explicit keywords
  * Example: users.my_first_user

#### Ad Hoc Commands
* To pass variables as arguments for ad hoc commands, use -a
  * Allows module arguments

* Commands use variables passed as arguments to run module:

~~~bash
ansible all -i hosts -m debug -a "msg='This line will appear as a message'"
demo.lab.example.com | SUCCESS => {
    "msg": "This line will appear as a message"
}
~~~

#### Playbooks
* When writing playbooks, you can use your own variables and call them in a task. For example, you can define a variable web_package with a value of httpd to be called by the yum module to install the httpd package. In playbooks, you can define variables in a vars block at the beginning of a playbook. Alternatively, you can pass variables as arguments by including a file in a playbook.

##### Playbook Examples
* In vars block:
~~~yaml
- hosts: all
  vars:
    user: joe
    home: /home/joe
~~~

* Passed as arguments:
~~~yaml
include: extra_args.yml
  name: joe
  group: wheel
~~~

#### Variables in Tasks
* Call variables in tasks

* When task runs, variable name replaced with value:
~~~yaml
vars:
  user: joe

tasks:
  # This line will read: Creates the user joe
  - name: Creates the user {{ user }}
    user:
      # This line will create the user named Joe
      name: "{{ user }}"
~~~

#### Using quotation marks " "
* Important note: When use a variable as the first element of a value, double quotation marks are mandatory. This prevents Ansible from misinterpreting the variable as the start of a YAML dictionary.

#### Definition reuse
* Defined variable in playbook can be used to define other variables
* Useful for:
  * Working with long file names
  * Defining base structure

* Example: User home directory on NFS server with path /srv/users/host1/mgmt/sys/users
  * To build path, use multiple variables
  * Easier than using full path every time

##### Example: Definition Reuse
~~~yaml
- name: Installs Apache and starts the service
  hosts: webserver
  vars:
    base_path: /srv/users/host1/mgmt/sys/users
    users:
      joe:
        name: Joe Foo
        home: "{{ base_path }}/joe"
      bob:
        name: Bob Bar
        home: "{{ base_path }}/bob"

  tasks:
  - name: Print user details records
    debug: msg="User {{ item.key }} full name is {{ item.value.name }}. Home is {{ item.value.home }}"
    with_dict: "{{users}}"
~~~

#### Variable Precedence
* Variables can be defined in multiple locations
* If Ansible finds variables with same name, uses chain of precedence
* Ansible 2: Variables evaluated in 16 categories of precedence order
  * Lower number = lower precedence

##### Precedence Categories 1-2
1. Role default variables
  * Set in roles vars directory:
~~~yaml
[configuration]
users:
  - joe
  - jane
  - bob
~~~

1. Inventory variables:
~~~yaml
[host_group]
demo.exmample.com ansible_user: joe
~~~

##### Precedence Categories 3-4
3. Inventory group_vars variables:
~~~yaml
[hostgroup:children]
host_group1
host_group2

[host_group:vars]
user: joe
~~~ 

4. Inventory host_vars variables:
~~~yaml
[hostgroup:vars]
user: joe
~~~

##### Precedence Categories 5-6

5. group_vars variables defined in group_vars directory:
~~~yaml
--
user: joe
~~~

6. host_vars variables defined in host_vars directory:
~~~yaml
---
user: joe
~~~

##### Precedence Category 7
7. Host facts

   * Facts discovered by Ansible:
~~~yaml
"ansible_facts": {
        "ansible_all_ipv4_addresses": [
        "172.25.250.11"
        ],
        "ansible_all_ipv6_addresses": [
        "fe80::5054:ff:fe00:fa0b"
        ],
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "01/01/2011",
        "ansible_bios_version": "0.5.1",
...
~~~

##### Precedence Category 8
8. Registered variables
   * Registered with register keyword:
~~~yaml
---
- hosts: all
  tasks:
    - name: Checking if Sources are Available
      shell: echo "This is a test"
      register: output
~~~

##### Precedence Categories 9-10
9. Variables defined via set_fact:
~~~yaml
- set_fact:
  user: joe
~~~

10. Variables defined with -a or --args:
~~~yaml
ansible-playbook main.yml -a "user=joe"
ansible-playbook main.yml --args "user=joe"
~~~

##### Precedence Categories 11-12
11. vars_prompt variables:
~~~yaml
vars:
from: "user"
  vars_prompt:
    - name: "user"
      prompt: "User to create"
~~~

12. Variables included using vars_files:
~~~yaml
vars_files:
  - /vars/environment.yml
~~~

##### Precedence Category 13
13. role and include variables:
~~~yaml
---
- hosts: all
  roles:
    - { role: user, name: 'joe' }
  tasks:
    - name: Includes the environment file and sets the variables
      include: tasks/environment.yml
      vars:
        package: httpd
        state: started
~~~

##### Precedence Category 14
14. Block variables
    * For tasks defined in block statement:
~~~yaml
tasks:
  - block:
    - yum: name={{ item }} state=installed
      with_items:
        - httpd
        - memcached
~~~

##### Precedence Categories 15-16
15. Task variables
    * Only for task itself:
~~~yaml
- user: name=joe
extra variables
~~~

16. Precedence over all other variables
`ansible-playbook users.yml -e "user=joe"`

### Variable Scope

* When declaring variables in multiple locations, the location in which you declare a particular variable determines its scope; that is, it determines from where the variable is accessible.

* Ansible recognizes three scope levels:
  * Global scope: This scope is set by the configuration, the environment variables, and the command line.
  
  * Play scope: This scope is set by the playbook and the play itself. The vars, include, and include_vars statements define this scope.
  
  * Host scope: This scope is set at the level of the host. For example, the ansible_user variable defines the name of the user to connect with on the managed host.

#### Variable Placement
* Scopes let you determine best variable placement

* To define variable only for playbook, use vars block
  * Variable evaluated when playbook is played:
~~~yaml
vars:
  user: joe
~~~

* To make variable available for host independent of play, define as group variable in inventory file:
~~~yaml
[servers]
demo.example.com

[servers:vars]
user: joe
~~~

* To enable variable to override playbook, declare as extra:
`ansible-playbook users.yml -e 'user=joe'`

### Variables management
* Declare playbook variables in various locations:
* In inventory file as host or group variables
* In vars statement as playbook variables
* In register statement
* Passed as arguments using -a
* Passed as extra arguments using -e

#### Location and scope
* The location where variables are declared depends on the required scope for the variables. For example, if you require a common behavior for your hosts, declaring the variables as part of host variables is recommended. If the variables only need a scope limited to the playbook, using the vars statement is recommended. If the variables need to ensure a base behavior but still be able to override variables when required, using the -e flag when executing the playbook will override all other variables.

#### Default variables
* Create default variables by defining variables for hosts
  * Avoids needing to define variables for every playbook
  * Prevents errors
  * Ensures consistent variable value across playbooks
  
#### Definition Levels in Inventory File
* Host variables
~~~yaml
[servers]
demo.example.com ansible_user=joe
~~~

* Group variables
~~~yaml
[servers]
demo1.example.com
demo2.example.com

[servers:vars]
user=joe
~~~

* Host Group Variables
~~~yaml
[servers1]
demo1.example.com
demo2.example.com

[servers2]
demo3.example.com
demo4.example.com

[servers:children]
servers1
servers2

[servers:vars]
user=joe
~~~

#### Levels and Scope
* Define scope for variables at each level
  * Avoids need to define same variables multiple times
  * Keeps set host hierarchy

* To define variables for all hosts in group, use host variables
* To define variables for multiple groups, use group variables

* As explained earlier, variables defined for hosts (inventory host_vars) take precedence over variables defined for host groups (inventory group_vars).

##### Example 1: General Value for All Servers
* Sample scenario: Managing two datacenters
* Goal: Define general value for all servers in both datacenters
* Recommended: Use group variables
~~~yaml
[datacenter1]
demo1.example.com
demo2.example.com

[datacenter2]
demo3.example.com
demo4.example.com

[all:vars]
package=httpd
~~~ 

##### Example 2: Value Varying by Datacenter
* Recommended: Use group variables
~~~yaml
[datacenter1]
demo1.example.com
demo2.example.com

[datacenter2]
demo3.example.com
demo4.example.com

[datacenter1:vars]
package=httpd

[datacenter2:vars]
package=apache
~~~

##### Example 3: Value Varying by Host
* Recommended: Use host variables
~~~yaml
[datacenter1]
demo1.example.com package=httpd
demo2.example.com package=apache

[datacenter2]
demo3.example.com package=mariadb-server
demo4.example.com package=mysql-server

[datacenters:children]
datacenter1
datacenter2
~~~

##### Example 4: Default Value That Host Overrides
* Recommended: Host group variable with manual override
~~~yaml
[datacenter1]
demo1.example.com
demo2.example.com

[datacenter2]
demo3.example.com
demo4.example.com

[datacenters:children]
datacenter1
datacenter2

[datacenters:vars]
package=httpd
~~~

`ansible-playbook demo2.exampe.com main.yml -e "package=apache"`

#### Registered variables
* To capture command’s output, use register statement
* Output saved into variable
* Use for debugging purposes or other tasks
  * Example: Configuration based on command output

### Inclusion

#### Tasks and Variables

* Tasks included with include executed same as if defined in playbook

* Variables included with include_vars parsed same as if declared in playbook

* Using multiple external files for tasks and variables:
  * Lets you build main playbook in modular way
  * Facilitates reuse of elements across playbooks

#### External Variables
* Use external variables:
  * Same as global or host variables
  * Generic variables not dependent on current infrastructure

* Examples: Use to indicate:
  * Name of package consistent across systems
  * Generic user in cloud environments (e.g., cloud-user)

* Other use cases for declaring variables in external files:
  * There are hundreds of variables to manage; dedicated file keeps variables in one location
  * Custom script or program can output values in JSON; can separate code and program output
  * Users need to view/modify variables without accessing playbooks

#### Variables in Tasks
* Variables can be set in task
  * Used by Ansible if no variables set on inclusion

* Variables overwritten during task inclusion take precedence over variables in task file

* Tasks included in a playbook are evaluated depending on their order in the playbook. For example, if the playbook starts a service for a package that is installed by an external task, the task must be included before the service statement.

### Loops
* Ansible supports use of loops to manage tasks
  * Eliminates need to rewrite tasks that are repeated—DRY (Don’t Repeat Yourself) for modules that support list
  * Example: If installing multiple packages, loop avoids using yum module multiple times

* Loops require use of arrays
  * You define array and task that iterates over array
  * Loop iterates over values defined in array

* To pass loop as argument, use item keyword
  * Enables Ansible to parse array

* Ansible supports number of loop types:
  * Simple loops
  * Lists of hashes
  * Nested loops

#### Simple Loops
* List of items Ansible reads and iterates over
  * To define simple loop, provide list of items to with_items
  * Example of play without loop:
    * yum used twice to install two packages:
~~~yaml
- yum:
    pkg: postfix
    state: latest

- yum:
    pkg: dovecot
    state: latest
~~~

##### Simple Loop Example
* Rewrite similar tasks with simple loop
* Only one task needed to install both packages:
~~~yaml
- yum:
    name: "{{ item }}"
    state: latest
  loop:
    - postfix
    - dovecot
~~~

##### Simple Loop Array
* Another option: Put packages inside array called loop
* Example: Pass loop array as argument
  * Module loops over it to retrieve values:
~~~yaml
vars:
  mail_services:
    - postfix
    - dovecot

tasks:
  - yum:
      name: "{{ item }}"
      state: latest
    loop: "{{ mail_services }}"
~~~

##### List of Hashes
* Arrays passed as arguments can be list of hashes

* Example: Pass multidimensional array (array with key/pair values) to user module
  * Customizes name and group:
~~~yaml
- user:
    name: {{ item.name }}
    state: present
    groups: {{ item.groups }}
  loop:
    - { name: 'jane', groups: 'wheel' }
    - { name: 'joe', groups: 'root' }
~~~

##### register Keyword
* To capture output of module that uses array, use register with array
* Ansible saves output in variable
  * Useful for reviewing module execution result

* Example: Register array content after iteration:
~~~yaml
- shell: echo "{{ item }}"
  loop:
    - one
    - two
  register: echo
~~~

## Ansible Return Values
* When running tasks in a playbook, modules return a value that states the status of the task. The status depends on the module used. If the task fails, the module returns failed. If a condition is not met, then may be task is skipped.

* Ansible modules normally return a data structure that can be registered into a variable or seen directly when output by the Ansible program.

* Optionally, each module can document its own unique return values. These values, which can be viewed using the ansible-doc command, include the following:
  * changed: This is a boolean indicating if the task had to make changes.

  * failed: This is a boolean that indicates if the task failed.

  * msg: This is a string with a generic message relayed to the user.

  * results: If this key exists, it indicates that a loop was present for the task and that it contains a list of the normal module result for each item.

  * stderr: Some modules execute command-line utilities that are used to run commands directly—for example, raw, shell, command, and so on. This field contains the error output of these utilities.

  * ansible_facts: This key contains a dictionary that is appended to the facts assigned to the host. These are directly accessible and do not require using a registered variable.

  * warnings: This key contains a list of strings that is presented to the user.

  * deprecations: This key contains a list of dictionaries that is presented to the user. The dictionary keys are msg and version, the values are a string, and the value for the version key can be an empty string.

### Review the return values in the output shown here.

The TASK [Check connectivity if OS is "MacOSX"] output shows ok, as the play runs successfully.

TASK [Check connectivity if OS is windows] shows that the task is skipped, because the OS does not match.

TASK [Install Packages] shows FAILED, and PLAY RECAP shows failed=1, because, on MacOS X, the yum module is not used to install packages.

~~~bash
`cat return_value.yml`

- hosts: localhost
  gather_facts: true
  connection: local
  tasks:
  - name: Check connectivity if OS is "MacOSX"
    ping:
    when: ansible_distribution == "MacOSX"
  - name: Check connectivity if OS is windows
    win_ping:
    when: ansible_distribution == "Windows"
  - name: Install package
    yum:
     name: httpd
     state: latest
# ansible-playbook return_value.yml

PLAY [localhost] ***********************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
ok: [localhost]

TASK [Check connectivity if OS is "MacOSX"] ********************************************************************************************
ok: [localhost]

TASK [Check connectivity if OS is windows] *********************************************************************************************
skipping: [localhost]

TASK [Install Packages] ****************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "The Python 2 bindings for rpm are needed for this module. If you require Python 3 support use the `dnf` Ansible module instead.. The Python 2 yum module is needed for this module. If you require Python 3 support use the `dnf` Ansible module instead."}
	to retry, use: --limit @/Users/psrivast/test1.retry

PLAY RECAP *****************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=1
~~~

# Summary:
* Variables let you reuse values in Ansible Playbooks, inventory files, tasks, and roles, as well as Jinja2 templates.

* Variables can be written as Python dictionaries that can be browsed or searched.

* When a variable is the first element to start a value, double quotation marks are mandatory.

* The register keyword can be used to capture the output of a command in a variable.

* Ansible facts are variables that are automatically discovered by Ansible from a managed host.

* You can use include_vars modules to include variable files in JSON format in playbooks.

* You can use loops to iterate with a specified number of items.

* Certain return values are common to all modules.