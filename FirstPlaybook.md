# Inventory

* Host inventory defines hosts managed by Ansible

* Hosts may belong to groups
  * Typically used to identify host’s role in datacenter
  * Host can be member of more than one group

* Two ways to define host inventories:
  * Static host inventory defined by text file
  * Dynamic host inventory generated from outside providers

## Static Host Inventory

* Static host inventory defined in INI-like text file

* Each section in file defines a host group
  * Starts with host group name enclosed in brackets: [hostgroupname]
  * Lists host entries
    * One line for each managed host
    * Entries are host names or IP addresses

* Host entries define how Ansible communicates with managed host
  * Include transport and user account information

* Default location for host inventory file: /etc/ansible/hosts

* ansible* commands use different host inventory file when used with --inventory PATHNAME
  * -i PATHNAME for short

### Groups of Groups
* Host inventories can include groups of host groups

* To do this, use :children suffix

* Example: Create new group nwcapitols that includes all hosts from olympia and salem groups

~~~bash
[olympia]
washington1.example.com
washington2.example.com
[salem]
oregon01.example.com
oregon02.example.com
[nwcapitols:children]
olympia
salem
~~~

### Ranges
* To simplify host inventories, specify ranges in host names or IP addresses
  * Supports numeric and alphabetic ranges

* Range syntax: [START:END]

* Ranges match all values between START and END

* Examples:
  * 192.168.[1:5].[0:255]: All IP addresses in 192.168.1.0/24 through 192.168.5.0/24 networks
  * server[01:20].example.com: All hosts named server01.example.com through server20.example.com

### Playbook Variables
* Specify values for variables used by playbooks in host inventory files
  * To specify variable values for individual host, append them at end of host line in inventory
  * To specify values for group of hosts, declare them in stanza with :vars suffix

* Example: Defines two group-level variable values, http_port and maxRequestsPerChild, for webservers group
  * Value of http_port is set to 8080 for web2.example.com:
~~~bash
[webservers]
web1.example.com
web2.example.com:1234 http_port=8080
[webservers:vars]
http_port=80 maxRequestsPerChild=500
[db-servers]
web1.example.com
db1.example.com
~~~

## Dynamic Host Inventory

* Host inventory information can be dynamically generated
* Sources for dynamic inventory information include:
  * Public and private cloud providers
  * Cobbler system information
  * LDAP database
  * Configuration management database (CMDB)

* Ansible includes scripts that handle dynamic host, group, and variable information from common providers
  * Examples: Amazon EC2, Cobbler, Rackspace Cloud, OpenStack®

* For cloud providers, authentication and access information must be defined in files that scripts can access

* Default: Ansible provides text-based inventory format
  * Defines hosts to be managed

* For large infrastructures, system information usually maintained in multiple inventory system solution
  * Examples: Cobbler, Zabbix

* Ansible uses scripts to support external inventories
  * Retrieve hosts information via method used by external inventory
  * Example: RESTful APIs

* Also uses scripts to retrieve inventories from cloud and virtualization solutions
  * Provide dynamic way to create and delete hosts
  * Example: Red Hat OpenStack Platform

### Supported Platforms

* Ansible maintains a GitHub site that contains scripts for retrieving hosts information for a large number of platforms, including the following:
  * Private cloud platforms such as Red Hat OpenStack Platform.
  * Public cloud platforms such as Rackspace Cloud, AWS, or Google Compute Engine.
  * Virtualization platforms such as oVirt. This is upstream of Red Hat Virtualization.
  * Platform-as-a-Service solutions such as OpenShift.
  * Life cycle management tools such as Spacewalk. This is upstream of Red Hat Satellite.
  * Hosting providers such as Digital Ocean or Linode.

* Each platform script has a different set of requirements—for example, cloud credentials for Red Hat OpenStack Platform. See the section on developing dynamic inventory sources in the Ansible documentation for the specific requirements.

## YAML 

* Ansible Playbooks written in YAML language
  * Need understanding of YAML syntax basics

* YAML designed to represent data structures in easy-to-write, human-readable format
  * Examples: Lists, associative arrays

* Abandons enclosure syntax used to structure data hierarchy in other languages
  * Examples: Brackets, braces, open/close tags

* Uses outline indentation to maintain data hierarchy structures
  
### YAML File Syntax

* YAML files can begin with a three-dash "start of document" marker and terminate with a three-period "end of file" marker. Both markers are optional.

* Between the beginning and ending document markers, data structures are represented in an outline format that uses space characters for indentation. There are no requirements about the exact number of spaces to use for indentation. The only rules are that elements at the same level in the data hierarchy must have the same indentation, and that child data elements must be indented further than their parents to indicate the nested relationships.

* To improve readability, you have the option to insert blank lines in the file.

### YAML in Playbooks

#### Dictionaries

* A key/value data pair used in YAML is also referred to as a dictionary, hash, or associative array. In key/value pairs, keys are separated from values using a delimiter string of a colon and a space, as shown in the following example
~~~yaml
keys: value
~~~

* Dictionaries are commonly expressed in indented block format, as shown in this example:
~~~yaml
---
  name: Automation using Ansible
  code: DO407
~~~

* Dictionaries can optionally be expressed in inline block format. In this format, multiple key/value pairs are enclosed between curly brackets and separated by a delimiter string of a comma and a space, as shown in the third example:
~~~yaml
---
  {name: Automation using Ansible, code: DO407}
~~~

#### Lists
* Lists in YAML = arrays in other languages
* To represent list, precede each item with - followed by space:
~~~yaml
---
  - red
  - green
  - blue
~~~

* Optional: Express lists in inline format
  * Enclose list items between square brackets
  * To separate items, use , followed by space:
~~~yaml
---
fruits:
  [red, green, blue]
~~~

#### Syntax Verification

* Syntax errors in playbook cause execution to fail

* Execution output may or may not help pinpoint source of syntax error

* Best practice: Verify YAML syntax in playbook prior to execution

* Several ways to do this

#### Python

* To read playbook YAML file using Python, use command similar to:

`python -c 'import yaml, sys; print yaml.load(sys.stdin)' < myyaml.yml`

##### Python: Valid Syntax

* If no syntax error, Python prints file contents to stdout in JSON format

* Example: Use of Python on file with valid syntax:

`cat myyaml.yml`
~~~yaml
---
- first item
- second item
- third item
...
~~~

`python -c 'import yaml, sys; print yaml.load(sys.stdin)' < myyaml.yml`
['first item', 'second item', 'third item']

#### YAML Lint

* Online YAML syntax verification tools available
  * Useful for administrators not familiar with Python

* Example: YAML Lint website: http://yamllint.com/
  * Copy and paste playbook’s YAML contents into form on home page
  * Submit form
    * Result:
      * Reports syntax verification results
      * Displays cleaned-up version of content

#### --syntax-check

* Ansible offers native feature for validating playbook YAML syntax

* Use --syntax-check option with ansible-playbook command to check for syntax errors

* --syntax-check method:
  * Conducts more rigorous review
  * Ensures data elements specific to playbooks are not missing
  * Recommended for verifying playbook YAML syntax

## Playbooks and Ad Hoc Commands
### Playbooks

* Playbooks: Files that describe configurations or steps to implement on managed hosts

* More flexible and powerful configuration management and deployment than ad hoc commands

* Turn complex operations into mundane routines with predictable and successful outcomes

* Written in simple, human-readable YAML format

* Syntax easier to write and understand than other configuration management tool languages

### Plays, Tasks, and Hosts

* Sports playbook analogy:
  * Contains collection of plays
  * Each has different patterns for execution

* Ansible Playbooks also contain plays
  * Each defines set of operations to perform on set of managed hosts
  * Operations called tasks
  * Managed hosts referred to as hosts

* To perform tasks, invoke Ansible modules and pass arguments to accomplish operation

* Order of contents within playbook matters
  * Plays and tasks executed in order presented

#### Multiple Plays

* Playbook can contain one or more plays
* Play applies set of tasks to set of hosts
* Multiple plays required when:
  * One set of tasks performed on one set of hosts
  * Different set of tasks performed on different set of hosts

* While multiple plays can be defined in a playbook, only a single playbook can be defined within a given YAML file. If multiple playbooks are desired, they must be created as separate YAML files.

### Playbook Creation

#### Plays and Attributes
* Within playbook file, plays expressed in list
  * Each defined using dictionary data structure
  * Each starts with - followed by space

* Within play, can define various attributes
  * Attribute definitions consist of name followed by :, space, value:
~~~yaml
attribute: value
~~~

#### Syntax
* Each attribute in play indented with spaces
  * Indicate place in data structure hierarchy

* Attributes at same indentation level are at same level in hierarchy

* Example: Two attributes indented to indicate relationship to parent play
  * Hyphen in first entry marks beginning of play
  * Second entry indented to same level as first
  * Both attributes refer to same parent play
~~~yaml
---
  - attribute1: value1
    attribute2: value2
~~~

#### Name Attribute
* Gives descriptive label to play
  * Optional but recommended
  * Especially useful in playbook with multiple plays
~~~yaml
name: my first play
~~~

#### Host Attribute
* Defines set of managed hosts on which to perform tasks
  * Integral component to play
  * Must be defined in every play

* To define, use host pattern to reference hosts in inventory:
~~~yaml
host: managedhost.example.com
~~~

#### User Attribute
* Playbook tasks executed through network connection to managed hosts

* User account used for execution depends on parameters in */etc/ansible/ansible.cfg*

* To define user to execute task, use remote_user
  * If privilege escalation enabled, other parameters such as become_user can have impact

* To overwrite user from configuration file and define different user, use user
~~~yaml
user: remoteuser
~~~

#### Privilege Escalation Attribute
* Attributes define privilege escalation parameters within playbook

* To enable/disable privilege escalation, use become boolean
  * Takes effect regardless of how escalation is defined in configuration file
~~~yaml
become: yes/no
~~~

* To define privilege escalation method for specific play, use become_method
  * Example: Use sudo for privilege escalation:
~~~yaml
become_method: sudo
~~~

* To define user account to use for privilege escalation within specific play, use become_user:
~~~yaml
become_user: privileged_user
~~~

#### Tasks Attribute
* Use to define operations to execute on managed hosts
  * Defined as list of dictionaries
  * Each task composed of set of key/value pairs

* Example: tasks list consisting of single task
  * First item entry defines task name
  * Second invokes service module and supplies arguments as values:
~~~yaml
tasks:
    - name: first task
      service: name=httpd enabled=true
~~~

* Hyphen in first entry marks beginning of list of task attributes

* Second entry indented to same level as first
  * Conveys that both attributes refer to same parent task

#### Multiple Tasks
* For multiple tasks, repeat same syntax for each:
~~~yaml
tasks:
    - name: first task
      service: name=httpd enabled=true
    - name: second task
      service: name=sshd enabled=true
...output omitted...
    - name: last task
      service: name=sshd enabled=true
~~~ 

#### Comments
* Recommended: Add comments to playbooks

* Use to enhance readability and documentation

* Precede by #:
~~~yaml
# This is a comment
~~~

#### Example: Writing a Playbook
~~~yaml
---
  # This is a simple playbook with a single play
  - name: a simple play
    host: managedhost.example.com
    user: remoteuser
    become: yes
    become_method: sudo
    become_user: root
    tasks:
    - name: first task
      service: name=httpd enabled=true
    - name: second task
      service: name=sshd enabled=true
...
~~~

### Playbook Formatting
* Several options for formatting playbooks
  * Help improve readability and organization of contents

* Playbook’s tasks dictionary contains key/value pair entry
  * Key: Name of module being invoked
  * Value: List of arguments to be passed to module

* Arguments list may be extremely long
  * Two formatting options: Multiline formatting and dictionary formatting

#### Multiline Formatting
* Break up long module arguments using multiline format
* Break line on space
* Must indent continuation lines with spaces
  * Indent enough spaces to exceed indentation level of first line

##### Example: Multiline Formatting
* Single-line format:
~~~yaml
tasks:
    - name: first task
      service: name=httpd enabled=true state=started
~~~

* Multiline format:
~~~yaml
tasks:
    - name: first task
      service: name=httpd
               enabled=true
               state=started
~~~

#### Dictionary Formatting
* Format long module arguments using YAML’s dictionary structure
* Option/value pairs passed as arguments converted to dictionary entries
* Indent entries beyond indentation level of module entry
  * Indicates child relationship

##### Example: Dictionary Formatting
* Single-line format:
~~~yaml
tasks:
    - name: first task
      service: name=httpd enabled=true state=started
~~~

* Dictionary format:
~~~yaml
tasks:
    - name: first task
      service:
        name: httpd
        enabled: true
        state: started
~~~

#### Blocks
* Complex playbooks may contain long list of tasks
  * Some tasks related in function

* Blocks: Alternative method of task organization
* Use to group related tasks
  * Improves readability
  * Allows performance of task parameters at block level

##### Examples: Block Formatting
~~~yaml
tasks:
    - name: first task
      yum: name=httpd state=latest
    - name: second task
      yum: name=sshd enabled=openssh-server state=latest
    - name: third task
      service: name=httpd enabled=true state=started
    - name: fourth task
      service: name=sshd enabled=true state=started
tasks:
    - block:
      - name: first package task
        yum: name=httpd state=latest
      - name: second package task
        yum: name=sshd enabled=openssh-server state=latest
    - service:
      - name: first service task
        service: name=httpd enabled=true state=started
      - name: second service task
        service: name=sshd enabled=true state=started
~~~

### Multiple Plays
* Playbook can contain one or more plays
* Plays map managed hosts to tasks
  * If different tasks performed on different hosts, must use different plays

* Supports multiple plays in same playbook file
* Plays expressed in list
* Start of each play indicated by - followed by space

#### Example: Playbook with Multiple Plays
~~~yaml
---
  # This is a simple playbook with two plays

  - name: first play
    host: web.example.com
    tasks:
    - name: first task
      service: name=httpd enabled=true

  - name: second play
    host: database.example.com
    tasks:
    - name: first task
      service: name=mariadb enabled=true

...
~~~

### Playbook Execution

#### Syntax Verification
* Recommended: Perform syntax verification prior to executing playbook
  * Ensures correct playbook contents syntax

* ansible-playbook offers --syntax-check option
  * Use to verify playbook file syntax

* Example: Successful playbook syntax verification:
`ansible-playbook --syntax-check webserver.yml`
~~~bash
  playbook: webserver.yml
~~~

##### Syntax Error
* Error reported if syntax verification fails
* Output includes approximate location of issue in playbook
* Example: Failed playbook syntax verification
  * Space separator missing after name:

`ansible-playbook --syntax-check webserver.yml`
~~~bash
  ERROR! Syntax Error while loading YAML.

  The error appears to have been in '/home/student/webserver.yml': line 3, column 8, but may
  be elsewhere in the file depending on the exact syntax problem.

  The offending line appears to be:

  - name:play to setup web server
    hosts: servera.lab.example.com
         ^ here
~~~

#### Executing Dry Run
* ansible-playbook also offers -C option

* When run with -C:
  * Ansible does not make changes on managed hosts
  * Instead reports what changes would occur if playbook is executed

##### Example: Executing Dry Run
* Dry run of playbook containing single task
  * Ensure latest version of httpd installed on managed host
  * Dry run reports that task would make change on managed host:

`ansible-playbook -C webserver.yml`
~~~bash

  PLAY [play to setup web server] ************************************************

  TASK [setup] *******************************************************************
  ok: [servera.lab.example.com]

  TASK [latest httpd version installed] ******************************************
  changed: [servera.lab.example.com]

  PLAY RECAP *********************************************************************
  servera.lab.example.com : ok=2 changed=1 unreachable=0 failed=0
  ~~~

#### Step-by-Step Execution
* For new playbooks, helpful to execute interactively
  
* ansible-playbook offers --step option

* When run with --step, Ansible steps through each playbook task

* Before executing task, Ansible prompts for input

* Options:
  * y: Execute task
  * n: Skip task
  * c: Exit step-by-step and execute remaining tasks without further interaction

##### Example: Step-by-Step Execution
* Task: Ensure latest version of httpd installed on managed host

`ansible-playbook webserver.yml`
~~~bash

  PLAY [play to setup web server] ************************************************
  Perform task: TASK: setup (y/n/c): y

  Perform task: TASK: setup (y/n/c): *********************************************

  TASK [setup] *******************************************************************
  ok: [servera.lab.example.com]
  Perform task: TASK: latest httpd version installed (y/n/c): y

  Perform task: TASK: latest httpd version installed (y/n/c): ********************

  TASK [latest httpd version installed] ******************************************
  ok: [servera.lab.example.com]

  PLAY RECAP *********************************************************************
  servera.lab.example.com : ok=2 changed=0 unreachable=0 failed=0
~~~

### Facts

* Ansible facts are variables that are automatically discovered by Ansible from a managed host. Facts are pulled by the setup module and contain useful information stored into variables that you can reuse. Ansible facts can be part of playbooks as conditionals, loops, or any other dynamic statement that depends on a value for a managed host.

* Examples of facts include the following:
  * A server can be restarted depending on the current kernel version.
  * The MySQL configuration file can be customized depending on the available memory.
  * Users can be created depending on the host name.

#### Using Facts
* Facts can be part of blocks, loops, conditionals, etc.
  * Virtually no limits on their use

* Provide convenient way to:
  * Retrieve managed host state
  * Decide action to take based on state

* Useful to create dynamic groups of hosts that match criteria

##### Information Provided
* Facts provide information about:
  * Host name
  * Kernel version
  * Network interfaces
  * IP addresses
  * Operating system version
  * Environment variables
  * Number of CPUs
  * Available/free memory
  * Available disk space

##### Use in Playbooks
* Fact output returned in JSON

* Values stored in Python dictionary
  * To retrieve value, browse dictionaries

* Use of facts from managed host in playbook:

*Fact*                      *Variable*
Hostname                    {{ ansible_hostname }}

Main IPv4 address           {{ ansible_default_ipv4.address }}
(based on routing)

Main disk first partition   {{ ansible_devices.vda.
partition size (based          partitions.vda1.size }}
on disk name: vda, vdb
etc.)

DNS server                  {{ ansible_dns.nameservers }}

kernel version              {{ ansible_kernel }}

##### Variable Values
* When using fact, Ansible dynamically swaps in value for variable name:
~~~yaml
---
- hosts: all
  tasks:
  - name: Prints various Ansible facts
    debug:
      msg: The default IPv4 address of {{ ansible_fqdn }} is {{ ansible_default_ipv4.address }}
~~~

##### Filters
* To have only some facts returned, use filters
* Examples: Only retrieve information about:
  * Network cards
  * Disks
  * Users

* Limit results when gathering managed host facts
* Facts returned in JSON-nested structure
  * Therefore, can filter various levels

* To use filters, pass expression as argument: -a 'filter=EXPRESSION'

##### Disabling Facts
* Disable facts for managed hosts if managing large number of servers
* To disable facts, set gather_facts to no in playbook:
~~~yaml
---
- hosts: large_farm
  gather_facts: no
~~~

#### Custom Facts
* Create and push custom facts to managed hosts
  * Integrated and read by setup

* Use to define:
  * System value based on custom script
  * Value based on program execution

* Save facts file in /etc/ansible/facts.d
  * Required for Ansible to find facts

* File must end in .fact
  * Plain-text INI or JSON file
  * Both return same result
  * Result put in ansible_local

##### INI File
* Contains:
  * Top level defined by section
  * Key/value pairs for facts to define:
  
~~~ini
[packages]
web_package = httpd
db_package = mariadb-server

[users]
user1 = joe
user2 = jane
~~~

##### JSON File
* Required syntax:
~~~json
{
  "packages": {
    "web_package": "httpd",
    "db_package": "mariadb-server"
  },
  "users": {
    "user1": "joe",
    "user2": "jane"
  }
}
~~~

