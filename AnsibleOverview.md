# Ansible overview

* Ansible is an agentless configuration management tool built on Python.

* Ansible is installed on the control node and keeps its configuration files there.

* Ansible copies modules from the control node to the managed hosts, where it executes them in the order specified in the playbook.

* Ansible is ideal for deploying applications in parallel on Red Hat Enterprise Linux, Red Hat JBoss Middleware, and Red Hat OpenShift Container Platform, and it can assist with Red Hat Satellite system management.

* Ansible can orchestrate zero-downtime rollover application upgrades.

* Native SSH is Ansible’s default connection plug-in, but the Paramiko plug-in provides efficient SSH communications with Red Hat Enterprise Linux 5 and 6 systems.

* Ansible can be configured through a variety of methods.

* Ansible ships with a module library--a collection of modules which can be executed directly on remote hosts or through playbooks.

* Ad hoc commands in Ansible allow you to execute simple tasks at the command line against one or all of your hosts.


* Ansible: Open source configuration management and orchestration utility

* Automates and standardizes configuration of remote hosts and virtual machines

* Coordinates launch and shutdown of multitiered applications

* Performs rolling updates of multiple systems with zero downtime

* Originally written by Michael DeHaan

* Creator of Cobbler provisioning application

* System administrators find it simple to use

* Developers can learn it easily

* Built on Python

* Supported by DevOps tools such as Vagrant and Jenkins

## Ansible Limitations

* There are several things Ansible cannot do:
  
  * Ansible cannot audit changes made locally by other users on a system—for example, to determine who made a change to a file.

  * Ansible can add packages to an installation, but it does not perform the initial minimal installation of the system. A system can start with a minimal installation, either via Kickstart or a base cloud starter image, then use Ansible for further configuration.

  * Although Ansible can remediate configuration drift, it does not monitor for it.

  * Ansible does not track the changes made to files on the system, nor does it track the users or processes that made those changes. These types of changes are best tracked with a version control system or the Linux Auditing System.

## Architecture

* There are two types of machines in the Ansible architecture: the control node and the managed hosts. Ansible software is installed on the control node, and all of its components are maintained on it. The managed hosts are listed in a host inventory, a text file on the control node that includes a list of managed host names or IP addresses.

* System administrators log in to the control mode and launch Ansible, providing it with a playbook and a target host to manage. You can specify a single system, a group of hosts, or a wild card to process. Ansible uses SSH as a network transport to communicate with the managed hosts. The modules referenced in the playbook are copied to the managed hosts and then executed, in order, with the arguments specified in the playbook. Ansible users can write their own custom modules, but the core modules that come with Ansible can perform most system administration tasks.

### Control Node Components

* Ansible components are maintained on the control node.

* Ansible has configuration settings that define how it behaves. These settings include such things as the remote user to use to execute commands, and the passwords to provide when executing remote commands with sudo. Default configuration values can be overridden by environment variables or values defined in configuration files.

* Ansible’s host inventory defines the configuration groups to which hosts belong. The inventory can define how Ansible communicates with a managed host. It also defines host and group variable values.

* Modules are the programs that are copied to the managed hosts to perform the work for Ansible. Ansible comes with over 400 core modules built in.

* Users can extend Ansible’s functionality by writing their own modules and adding them to the Ansible library. Modules are typically written in Python but can be written in any interpreted programming language, such as shell or Ruby.

* Ansible Playbooks are files written in YAML syntax that define the modules, with arguments, to apply to managed hosts. They declare the tasks that need to be performed.

* Connection plug-ins enable communication with remote hosts and the cloud. These include native SSH (the default), Paramiko SSH, and local. Paramiko is a Python implementation of OpenSSH for Red Hat Enterprise Linux 5 and 6 that provides the ControlPersist performance setting that Ansible requires.

* Additional plug-ins extend Ansible’s functionality. Examples include email notifications and logging.

#### Control Node Role 

* System administrators log in and initiate Ansible operations from control node

* Ansible software installed and configuration files maintained on control node

* Other names for control node: Ansible host and control machine

#### Managed Host Role

* Ansible does the following on managed host systems:
  * Logs in
  * Installs modules
  * Executes remote commands for configuration

* Other names for managed host: managed node and remote node

## Use Cases

* Configuration management

  * Deploy and manipulate remote host’s configuration files
  * Use static files or create files on fly using templates

* Multi-node deployment tool

  * Use playbooks to define applications installed and configured on remote machines

  * Apply playbook to multiple machines, building them in consistent manner

  * Orchestrate multi-node applications with Ansible rules

* Remote task execution

  * Example: Specify ad hoc commands on command line
    * Causes Ansible to execute commands on remote hosts

## Deployments

* Ansible strength: Simplifies software configuration of servers

* When Ansible accesses managed hosts, it can discover version of Red Hat Enterprise Linux running on remote server

* Ansible determines if host is properly entitled by comparing installed applications and applied software subscriptions

* Ansible Playbooks can consistently build development, test, and production servers

  * Kickstart can get bare-metal servers running

  * Ansible builds them further

  * Provision servers to corporate baseline standard or specific role within datacenter

### Red Hat JBoss Middleware

* Ansible can discover Red Hat JBoss Middleware versions and reconcile subscriptions

* Ansible supports managed hosts running Windows

  * Red Hat JBoss Middleware products can be deployed consistently, regardless of target machine operating systems

* Ansible can also deploy and manage Red Hat JBoss Middleware applications

  * All Red Hat JBoss Middleware configurations are centrally stored on Ansible control node

### Red Hat OpenShift

* Ansible can manage software development life cycle for applications deployed into Red Hat OpenShift Container Platform

  * OpenShift Container Platform 3.1 provides:

    * Ansible software for Red Hat Enterprise Linux

    * Playbooks for provisioning and managing applications

### Red Hat Satellite

* Ansible can supplement functionality provided by Red Hat Satellite

  * Deploy Satellite agents to existing servers in datacenter

  * Discover and manage software subscriptions on Red Hat Satellite clients

  * Perform post-install configuration of hosts provisioned by Red Hat Satellite

## Orchestration Methods

* Ansible commonly used to finish provisioning application servers

* Example: Write playbook to perform these steps on newly installed base system:

  * Configure software repositories

  * Install application

  * Tune configuration files

  * (Optional) Download content from version control system

  * Open required service ports in firewall

  * Start relevant services

  * Test application and confirm it is functioning

## Connection Plug-ins

* Connection plug-ins: Allow Ansible to communicate with managed hosts and cloud providers

* Preferred connection plug-in for newer versions of Ansible is native SSH plug-in, ssh

  * Default connection method used by Ansible

    * If OpenSSH on control node supports ControlPersist option

* Ansible supports passwords for SSH authentication

  * Most common practice: Use SSH user keys to access managed hosts

### local

* local: Another connection plug-in for Linux applications

* Use to manage Ansible control node locally, without SSH

* Common uses:

  * When writing playbooks that interface with cloud services or other API

  * When Ansible is invoked locally by cron job

### paramiko and ControlPersist

* paramiko: Connection plug-in used on Red Hat Enterprise Linux 5 and 6 machines

  * Paramiko SSH is Python-based OpenSSH implementation that implements persistent SSH connections

  * Connection solution for older systems using versions of OpenSSH that do not implement ControlPersist

* ControlPersist allows for persistent SSH connections

  * Improves Ansible performance

  * Eliminates SSH connection overhead when multiple SSH commands execute in succession

### winrm and docker

* winrm: Allows Microsoft Windows machines to be managed hosts

  * pywinrm Python module must be installed on Linux control node to support winrm

* docker: Allows Ansible to treat Docker containers as managed hosts without using SSH

  * Introduced in Ansible 2

### Ansible Configuration

#### Configuration file
  * Settings in Ansible adjustable via configuration file (ansible.cfg)

  * Default configuration file, /etc/ansible/ansible.cfg, sufficient for most users

    * There may be reasons to change it

#### Environment Variables
  * Ansible also allows configuration of settings using environment variables
    * If set, they override settings loaded from configuration file

#### Command Line Options
  * Not all configuration options available via command line, just those deemed most useful or common

  * Settings in command line override those passed through configuration file and environment

#### Ansible Configuration Settings
* ansible-config utility allows users to see all available configuration settings, their defaults, how to set them, and where current values come from

* Changes can be made in configuration file

* Ansible searches for file to use in this order:

  * ANSIBLE_CONFIG (environment variable, if set)
  * ansible.cfg (in current directory)
  * ~/.ansible.cfg (in home directory)
  * /etc/ansible/ansible.cfg

* First file found is used, all others ignored

## Modules

* Use modules to perform operations on managed hosts

  * Ready-to-use tools for specific tasks
  * Run from command line or use in playbooks
  * Copied to and run from managed host

* Over 200 prepackaged modules
  * Let you perform wide range of tasks
  * Examples: Cloud, user, package, service management

### Module Types

* Core modules, which are maintained by the Ansible Engineering Team. These modules are integral to the basic foundations of the Ansible distribution.

* Network modules, which are maintained by the Ansible Network Team. Please note there are additional networking modules that are categorized as Certified or Community and not maintained by Ansible.

* Certified modules, which are part of a future planned program currently in development.

* Community modules, which are submitted and maintained by the Ansible community. These modules are not maintained by Ansible, and are included as a convenience.

* The modules are hosted on GitHub in a subdirectory of the Ansible repository.

#### Use of categories: Documentation and Organization

* Module documentation indexed by category on Ansible documentation website
  * Helps in searching for module for specific task

* Module storage on Ansible control node organized by categories
  * Modules installed under /usr/lib/python2.7/site-packages/ansible/modules
  * Core and extra modules housed under separate directories
  * Modules within directories organized into subdirectories by category

#### Module Categories

* For better organization and management, Ansible modules are grouped into the following functional categories:
  * Cloud               * Clustering
  * Commands            * Database
  * Files               * Inventory
  * Messaging           * Monitoring
  * Network             * Notification
  * Packaging           * Source Control
  * System              * Utilities
  * Web Infrastructure  * Windows

#### Module Documentation

* Prepackaged modules provide tools for common tasks
* To learn about modules, see the Ansible documentation website
* Use Module Index to search for modules for specific functions
  * Modules for user and service management found under System modules
  * Modules for database administration found under Database modules
* For each module, website provides:
  * Summary of functions
  * Instructions on using options to invoke each function

* Documentation also provides examples of each module and its options

* Accessing Documentation Locally
  * Documentation available locally on control node
  * To see modules available on control node, run ansible-doc with -l option
* Outputs:
  * List of module names
  * Synopsis of module functions

* To display detailed documentation on a specific module, pass the module name to ansible-doc. Like the Ansible documentation website, this command provides a synopsis of the module’s function and details regarding its various options. Examples of the module’s uses and options are also included.

* Detailed Documentation
  * To display documentation on specific module, pass module name to ansible-doc

  * Output similar to documentation website

  * Provides:
    * Synopsis of module function
    * Details regarding options
    * Examples of each

##### Modules

* -s Option

The ansible-doc command also offers a --snippet, or -s option, which produces output for using the module in a playbook. This output can serve as a starter template that can be included in a playbook to implement the module for task execution. Comments are included in the output to remind you how to use each option.

#### Methods to invoke Modules

* To call modules as part of ad hoc command, use ansible
  * -m specifies which module to use
  * Example: Use ping to test connectivity to all managed hosts:
    `ansible -m ping all`

* Can call modules in playbooks as part of task
  * Example: Invoke yum module
  * Arguments: Package name and desired state:
  
~~~yaml
tasks:
  - name: Installs a package
  yum:
    name: Postfix
    state: latest
~~~

* To call modules from Python scripts, use Ansible Python API
  * Not supported in case of failures

### Ad Hoc Commands

* Ansible lets you run on-demand tasks on managed hosts
* Ad hoc commands: Most basic operations you can perform
* To perform ad hoc commands, run ansible on control node
  * As part of command, specify operation to perform

* Each command can perform only one operation
  * Multiple operations require series of commands

* *Benefits*

* Easy way for administrators to get started using Ansible
  
* Introduce advanced Ansible features: modules, tasks, plays, playbooks
  
* Quickly make configuration changes to large number of managed hosts
  
* Perform noninvasive tasks

#### Syntax

* For ad hoc commands, run ansible as follows:

`ansible host pattern -m module [-a module arguments] [-i inventory]`

* Host pattern defines list of managed hosts on which Ansible performs command

* List of managed hosts determined by applying host pattern against default inventory file
  * Located at /etc/ansible/hosts

* To specify alternative inventory file location, use -i

* Control node can include itself as managed host
  * To define control node as managed host, add control node name, its IP address, localhost name, or IP address 127.0.0.1 to inventory

#### Modules and Arguments

* -m indicates module to use to perform remote operation
  * Module: Tool designed to accomplish specific task

* Arguments passed to module using -a
  * Some modules cannot accept arguments
  * Others accept multiple arguments

* If no argument needed, omit -a

* If multiple arguments needed, enter as single-quoted, space-separated list:
`ansible host pattern -m module -a 'argument1 argument2' [-i inventory]`

#### Default Module

* To define default module, use module_name setting under defaults section of /etc/ansible/ansible.cfg:
~~~bash
# default module name for /usr/bin/ansible
# module_name = command
~~~

* Some Ansible configuration settings are predefined internally and have values set
  * Applies even if settings commented out in configuration file

* module_name predefined with command module as default

#### Using Predefined Module

* When -m omitted, Ansible:
  * Consults configuration file
  * Uses module defined there

* If no modules defined, predefined command module used
  * Result: Following commands are technically equivalent:

`ansible host pattern -m command -a module arguments`
`ansible host pattern -a module arguments`

#### command Module

* Lets you run command on managed hosts

* Command specified by arguments following -a

* Example: Run hostname on managed hosts referenced by mymanagedhosts host pattern:

`ansible mymanagedhosts -m command -a /usr/bin/hostname`
~~~bash
host1.lab.example.com | SUCCESS | rc=0 >>
host1.lab.example.com
host2.lab.example.com | SUCCESS | rc=0 >>
host2.lab.example.com
~~~

*-o Option*

* Previous example returned two lines of output for each managed host
  * First line is status report showing:
    * Managed host on which operation was performed
    * Outcome of operation

  * Second line is output of remotely run command

* -o option generates just one line of output for each operation performed:

`ansible mymanagedhosts -m command -a /usr/bin/hostname -o`
~~~bash
host1.lab.example.com | SUCCESS | rc=0 >> (stdout) host1.lab.example.com
host2.lab.example.com | SUCCESS | rc=0 >> (stdout) host2.lab.example.com
~~~

* Offers better readability and parsing of command output

#### shell Module

* command module lets you quickly run remote commands on managed hosts
  * Not processed by shell on managed hosts
  * Cannot access shell environment variables
  * Cannot perform shell operations

* To run commands that require shell processing, use shell module
  * Pass commands to run as arguments to module
  * Ansible runs command remotely

* shell commands processed through shell
* Can use shell environment variables
* Can perform shell operations

#### Ad Hoc Command Configuration

* When running ad hoc command, things occur in background

* Configuration file (/etc/ansible/ansible.cfg) consulted for parameters
  * Example: module_name parameter

* Other parameters determine how to connect to managed hosts

#### Connection Settings

* After reading parameters, Ansible makes connections to managed host

* Default: Connections initiated with SSH
  * Requires connection established using account on managed host
  * Account referred to as remote user
  * Defined by remote_user setting under [defaults] section in /etc/ansible/ansible.cfg:
~~~bash
# default user to use for playbooks if user is not specified
# (/usr/bin/ansible will use current user as default)
#remote_user = root
~~~

#### Remote Operations

* Default: remote_user parameter commented out in /etc/ansible/ansible.cfg

* With parameter undefined, commands connect to managed hosts using same remote user account as one on control node running command

* After making SSH connection to managed host, specified module performs operation

* After operation completed, output displayed on control node

* Operation restricted by limits on permissions of remote user who initiated it

#### Privilege Escalation

* After connecting to managed host, Ansible can switch to another user before executing operation

* Example: Using sudo, can run ad hoc command with root privilege

* Only if connection to managed host authenticated by nonprivileged remote user

* Settings to enable privilege escalation located under [privilege_escalation] section of ansible.cfg:
~~~bash
#become=True
#become_method=sudo
#become_user=root
#become_ask_pass=False
~~~

#### Enabling Privilege Escalation

* Privilege escalation not enabled by default

* To enable, uncomment become parameter and define as True:
~~~bash
become=True
~~~ 

* Enabling privilege escalation makes become_method, become_user, and become_ask_pass parameters available
  * Applies even if commented out in /etc/ansible/ansible.cfg
  * Predefined internally within Ansible
  * Predefined values:
~~~bash
become_method=sudo
become_user=root
become_ask_pass=False
~~~

#### Command Line Options

* Configure settings for remote host connections and privilege escalation in /etc/ansible/ansible.cfg

* Alternative: Define using options in ad hoc commands
  * Command line options take precedence over configuration file settings

*Setting*                   *Command Line Option*
inventory                   -i
remote user                 -u
become                      --become, -b
become_method               --become-method
become_user                 --become-user
become_ask_pass             --ask-become-pass, -k

#### Obtaining Current Setting Values

* Before configuring privilege escalation settings using options, can determine currently defined values

* To do so, consult output of ansible --help:
`ansible --help`
~~~bash
-b, --become		run operations with become (nopasswd implied)
  --become-method=BECOME_METHOD
			privilege escalation method to use (default=sudo),
			valid choices: [ sudo | su | pbrun | pfexec | runas |
			doas ]
~~~

### Playbookds and Ad Hoc Commands

#### Ad Hoc Commands

* Use modules to perform operations on managed hosts with ad hoc commands
  * Useful for simple operations
  * Not suited for complex configuration or orchestration scenarios

* Ad hoc commands invoke one module and one set of arguments at a time
  * Multiple operations must be executed over multiple commands

