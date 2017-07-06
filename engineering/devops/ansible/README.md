

# Ansible 
[Source](https://github.com/ansible/ansible#ansible "Permalink to Ansible Readme on github")


Ansible is a radically simple IT automation system.  It handles configuration-management, application deployment, cloud provisioning, ad-hoc task-execution, and multinode orchestration - including trivializing things like zero downtime rolling updates with load balancers.

Read the documentation and more at https://ansible.com/

Many users run straight from the development branch (it's generally fine to do so), but you might also wish to consume a release.

You can find instructions [here](https://docs.ansible.com/ansible/intro_installation.html) for a variety of platforms.

Design Principles 
=================
[Source](https://github.com/ansible/ansible#ansible "Permalink to Ansible Readme on github")

   * Have a dead simple setup process and a minimal learning curve
   * Manage machines very quickly and in parallel
   * Avoid custom-agents and additional open ports, be agentless by leveraging the existing SSH daemon
   * Describe infrastructure in a language that is both machine and human friendly
   * Focus on security and easy auditability/review/rewriting of content
   * Manage new remote machines instantly, without bootstrapping any software
   * Allow module development in any dynamic language, not just Python
   * Be usable as non-root
   * Be the easiest IT automation system to use, ever.


Ansible Introduction
====================
[Source](https://en.wikipedia.org/wiki/Ansible_(software) "Permalink to Ansible Wikipedia")

Ansible is an open-source automation engine that automates software provisioning, configuration management, and application deployment. As with most configuration management software, Ansible has two types of servers: controlling machines and nodes. First, there is a single controlling machine which is where orchestration begins. Nodes are managed by a controlling machine over SSH. The controlling machine describes the location of nodes through its inventory.The Inventory is a description of the nodes that can be accessed by Ansible. By default, the Inventory is described by a configuration file, in INI format, whose default location is in /etc/ansible/hosts.

Control machines have to be a Linux/Unix host (for example, Red Hat Enterprise Linux, Debian, CentOS, OS X, BSD, Ubuntu), and Python 2.6 or 2.7 is required.


Ansible Usage
=============

Push Mode
---------

Ansible can be used in multiple ways, below are 3 described ones

1 Commandline directly specifying which shell command to use, on hosts(or group of hosts) specified over ssh. (ssh access needs to be configured for all push modes) [link](https://github.com/team-avesta/wiki/blob/master/engineering/devops/ansible/push1.md)

2 Commandline using ansible predefined modules to execute module based function, on specified hosts or group of hosts. If same module is run again, and effective new changes are possible, it wont run it again. If one tries to install an application which is already installed for the same corresponding parameters, it will not reinstall it. [link](https://github.com/team-avesta/wiki/blob/master/engineering/devops/ansible/push2.md)

3 Using a predefined yaml playbook, this method is very useful for executing a longer list of modules, generating better log files, and repeat use of same playbooks. It is merely a collection of multiple commands \ module functions grouped together to be executed. It also allows finer control for conditional\sequential execution of all tasks in the playbook(tasks = commands\ module function list) [link](https://github.com/team-avesta/wiki/blob/master/engineering/devops/ansible/pushplaybook.md)
 


Pull Mode 
---------
[Source](https://www.stavros.io/posts/automated-large-scale-deployments-ansibles-pull-mo/ "Permalink to https://www.stavros.io/posts/automated-large-scale-deployments-ansibles-pull-mo/")

“Pull mode” is a single, very simple script. You can invoke it with ansible-pull, and all it does is 

It pulls the specified git repository into a specified directory.
If the repository has changed, it runs a file called hostname.yml or local.yml in the repository’s root directory.

That is literally everything ansible-pull does. Therefore, it should be obvious by now how you use it:

* Add it to your crontab.
* Add the hostname.yml or local.yml file to your repository.
 

[Example](https://github.com/team-avesta/wiki/blob/master/engineering/devops/ansible/pushnpull.md)





Modules 
-------
[Source](https://en.wikipedia.org/wiki/Ansible_(software) "Permalink to Ansible Wikipedia")

Modules are considered to be the units of work in Ansible. Each module is mostly standalone and can be written in a standard scripting language (such as Python, Perl, Ruby, Bash, etc.). One of the guiding properties of modules is idempotency, which means that even if an operation is repeated multiple times (e.g., upon recovery from an outage), it will always place the system into the same state.



Playbooks 
---------
[Source](https://en.wikipedia.org/wiki/Ansible_(software) "Permalink to Ansible Wikipedia")

Playbooks express configurations, deployment, and orchestration in Ansible. The Playbook format is YAML. Each Playbook maps a group of hosts to a set of roles. Each role is represented by calls to Ansible tasks.

[Source](http://docs.ansible.com/ansible/playbooks_intro.html "Permalink to Ansible Documentation")

Ansible uses playbooks written in YAML format. It's basically a human-readable structured data format.

All YAML files (regardless of their association with Ansible or not) can optionally begin with "---" and end with "...". This is part of the YAML format and indicates the start and end of a document.
All members of a list are lines beginning at the same indentation level starting with a "- " (a dash and a space)
A dictionary is represented in a simple key: value form (the colon must be followed by a space)
More complicated data structures are possible, such as lists of dictionaries, dictionaries whose values are lists or a mix of both.

Ansible doesn’t really use these too much, but you can also specify a boolean value (true/false) in several forms:

```
create_key: yes
needs_agent: no
knows_oop: True
likes_emacs: TRUE
uses_cvs: false
```


Example playbook running one play:

```
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: name=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running (and enable it at boot)
    service: name=httpd state=started enabled=yes
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```









#### Modules we use:


* [Apt][] -- Manages apt packages (such as for Debian/Ubuntu).
* [Docker][] -- This is the original Ansible module for managing the Docker container life cycle.
* [Docker_container(future)][] -- Manage the life cycle of docker containers.
* [Docker_images (future)][] -- Build, load or pull an image, making the image available for creating containers.
* [Fetch][] -- It is used for fetching files from remote machines and storing them locally in a file tree, organized by hostname.
* [Copy][] -- The copy module copies a file on the local box to remote locations. 
* [SetUp][] -- This module is automatically called by playbooks to gather useful variables about remote hosts that can be used in playbooks.
* [Cron][] -- Use this module to manage crontab and environment variables entries. 
* [Command][] -- The command module takes the command name followed by a list of space-delimited arguments.

[Apt]:http://docs.ansible.com/ansible/apt_module
[Docker]:http://docs.ansible.com/ansible/docker_module
[Docker_container(future)]:http://docs.ansible.com/ansible/docker_container_module
[Docker_images (future)]:http://docs.ansible.com/ansible/docker_image_module
[Fetch]:http://docs.ansible.com/ansible/fetch_module
[Copy]:http://docs.ansible.com/ansible/copy_module
[SetUp]:http://docs.ansible.com/ansible/setup_module
[Cron]:http://docs.ansible.com/ansible/cron_module
[Command]:http://docs.ansible.com/ansible/command_module






Information links
-----------------


How to use ansible pull mode

https://www.stavros.io/posts/automated-large-scale-deployments-ansibles-pull-mo/

Ansible Man page for ansible-pull command 

https://linux.die.net/man/1/ansible-pull

Ansible Documentation

http://docs.ansible.com/ansible/intro_patterns.html

http://docs.ansible.com/ansible/playbooks_intro.html



