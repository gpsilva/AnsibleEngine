# Topics:

* Roles organize Ansible tasks so they can be reused and shared.

* Define role variables in defaults/main.yml if they are going to be used as parameters. If not, define them in vars/main.yml.

* A role’s dependencies can be defined in the dependencies section of the role’s meta/main.yml file.

* Tasks can be applied before and after roles and are included by using pre_tasks and post_tasks in a playbook.

* Ansible Playbooks define roles in the roles section.

* Roles defined in playbooks can override default role variables.

* Ansible Galaxy is a public library of Ansible roles written by Ansible users.

* The ansible-galaxy command can search for, display information about, install, list, remove, and initialize roles.

* The ansible-galaxy init --offline command creates the directory structure for a new role.

# Roles overview

* Datacenters include variety of host types:

  * Web servers

  * Database servers

  * Hosts containing software development tools

* Playbooks require tasks and handlers to manage these

  * Result: Large, complex playbooks

* Roles can split playbooks into smaller playbooks and files

## Role Uses

* Enable Ansible to load components from external files:

  * Tasks

  * Handlers

  * Variables

* Associate and reference:

  * Static files

  * Templates

* Files defining roles:

  * Given specific names

  * Organized in directory structure

* Roles written as general purpose can be reused

## Benefits

* Roles promote easy sharing of content

* Roles can define essential elements of system type:

  * Web server

  * Database server

  * Git repository

  * Other purposes

* Roles make larger projects more manageable

* Administrators can work on different project roles in parallel

## Structure

* Role functionality defined by directory structure

  * Top-level directory: Defines role name

  * Some subdirectories: Contain main.yml file

  * files and templates subdirectories: Contain objects referenced by main.yml files

### Structure Example

* tree command displays user.example directory structure:

`tree user.example`

~~~bash
user.example/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
└── main.yml
~~~

#### The role structure subdirectories.

* The main.yml file in the defaults directory contains the default values of role variables that can be overwritten when the role is used.

* The files directory contains static files that are referenced by role tasks.

* The main.yml file in the handlers directory contains the role’s handler definitions.

* The main.yml file in the meta directory defines information about the role, including author, license, platforms, and optional dependencies.

* The main.yml file in the tasks directory contains the role’s task definitions.

* The templates directory contains Jinja2 templates that are referenced by role tasks.

* The tests directory can contain an inventory and test.yml playbook that can be used to test the role.

* The main.yml file in the vars directory defines the role’s variable values.

## Variables and Defaults

* To define role variables, create vars/main.yml with name/value pairs in hierarchy

  * YAML uses role variables like any other variable: {{ VAR_NAME }}

  * High priority

  * Cannot be overridden by inventory variables

* Use default variables to set default values for included or dependent role variables

  * To define default variables, create defaults/main.yml with name/value pairs in hierarchy

  * Lowest priority of any variables

  * Overridden by any other variable
   
* Best practice: Define variable in vars/main.yml or defaults/main.yml

* Use default variable when role needs value to be overridden

## Roles in playbooks

* Simple to use roles in playbooks

* Example:

~~~yaml
---
- hosts: remote.example.com
  roles:
    - role1
    - role2
~~~

### Included Components

* For each role, include the following in playbook in this order:

  * Tasks

  * Handlers

  * Variables

  * Dependencies

* Role tasks (copy, script, template, include) reference files, templates, tasks

* Ansible searches for items in these locations:
  
  * Files: files
  
  * Templates: templates
  
  * Tasks: tasks

* Eliminates need for absolute or relative path names

### Alternative syntax

* role1 same as previous example

* If role2 used, default variable values overridden:
  
~~~yaml
---
- hosts: remote.example.com
  roles:
    - { role: role1 }
    - { role: role2, var1: val1, var2: val2 }
~~~

### Dependencies

* To include roles in playbook based on inclusion of other roles, use dependencies
  
* Example: Role defining documentation server depends on role that installs and configures web server
  
* Define roles in meta/main.yml in directory hierarchy:
  
~~~yaml
---
dependencies:
  - { role: apache, port: 8080 }
  - { role: postgres, dbname: serverlist, admin_user: felix }
~~~

#### Dependencies behavior

* Default: Role added as dependency to playbook once
  
* If role is listed as dependency again, it does not run
  
* To override default, set allow_duplicates to yes in meta/main.yml

### Order of Execution

* Default: Role tasks execute before tasks of playbooks in which they appear

* To override default, use pre_tasks and post_tasks

  * pre_tasks: Tasks performed before any roles applied

  * post_tasks: Tasks performed after all roles completed

* Order of Execution Example

~~~yaml
---
- hosts: remote.example.com
  pre_tasks:
    - shell: echo 'hello'
  roles:
    - role1
    - role2
  tasks:
    - shell: echo 'still busy'
  post_tasks:
    - shell: echo 'goodbye'
~~~

### Role Creation

* Simple to create roles in Linux

  * No special development tools required

* Three-step process:

  * Create role directory structure

  * Define role content

  * Use role in playbook

### Directory Structure

* Ansible looks for roles in:

  * roles subdirectory

  * Directories referenced by roles_path

    * Located in Ansible configuration file

    * Contains list of directories to search

* Each role has directory with specially named subdirectories

* Directory Structure Example
  * Define motd role:

~~~bash
tree roles/
roles/
└── motd
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    ├── tasks
    │   └── main.yml
    └── templates
        └── motd.j2
~~~

### Subdirectories

* files and templates

  * Contain fixed-content files and templates

  * Can be deployed by role when it is used

* Other subdirectories

  * Contain main.yml files

  * Define default variable values, handlers, tasks, role metadata, variables

* Empty subdirectory is ignored

* Subdirectory not used by role can be omitted

### Content

* After creating structure, define role content

* Use ROLENAME/tasks/main.yml

  * Defines modules to call on managed hosts where role is applied

#### Content Example

* tasks/main.yml file manages /etc/motd on managed hosts

  * Uses template to copy motd.j2 to managed host

  * Retrieves motd.j2 from role’s templates subdirectory:

~~~bash
cat roles/motd/tasks/main.yml
~~~
~~~yaml
---
# tasks file for motd

- name: deliver motd file
  template:
    src: templates/motd.j2
    dest: /etc/motd
    owner: root
    group: root
    mode: 0444
~~~

### Content Output

* Display contents of templates/motd.j2 template of motd role

  * References Ansible facts and system_owner variable:

~~~bash
`cat roles/motd/templates/motd.j2`
This is the system {{ ansible_hostname }}.

Today's date is: {{ ansible_date_time.date }}.

Only use this system with permission.
You can ask {{ system_owner }} for access.
~~~

### Use in Playbook

* To access role, reference it in roles: playbook section

* Example: Playbook referencing motd role

  * No variables specified

  * Role applied with default variable values:

~~~bash
cat use-motd-role.yml
~~~
~~~yaml
---
- name: use motd role playbook
  hosts: remote.example.com
  user: devops
  become: true

  roles:
    - motd
~~~

#### Playbook Output

* In playbook output, tasks executed due to role identified by role name preceding task

* Example: deliver motd file task prefaced by motd role name

* Indicates task triggered by role:

~~~bash
ansible-playbook -i inventory use-motd-role.yml

PLAY [use motd role playbook] **************************************************

TASK [setup] *******************************************************************
ok: [remote.example.com]

TASK [motd : deliver motd file] ************************************************
changed: [remote.example.com]

PLAY RECAP *********************************************************************
remote.example.com : ok=2 changed=1 unreachable=0 failed=0
~~~

#### Variables

* Use variables with roles to override default values

* When referencing roles with variables, must specify variable/value pairs

##### Variable Example

* Use motd with different value for system_owner

* someone@host.example.com replaces variable reference when role is applied to managed host:

~~~bash
cat use-motd-role.yml
~~~
~~~yaml
---
- name: use motd role playbook
  hosts: remote.example.com
  user: devops
  become: true

  roles:
    - { role: motd, system_owner: someone@host.example.com }
~~~

