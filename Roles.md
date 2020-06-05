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

