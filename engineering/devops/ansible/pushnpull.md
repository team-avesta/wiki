# Using push and pull mode together

Requirements -- we  need ansible installed on this hosts, as well as the python libraries for whichever module we use should also be installed on the hosts machine which run pull based playbooks and use the modules. eg if i want to pull a docker image after polling a repository for a new playbook, i would require docker-py to be installed on the host which pulls the docker image for running the docker module, also docker engine is assumed to be installed. 


# Use Case

A playbook installing a new cron tab entry for ansible-pull which runs on given specific interval, in our case we kept it at every reboot, so whenever the host machine is reboot it will poll the github repository for changes made and if true, (it compares md5 hash for a cloned copy of the repository vs the original repository) it pull the local.yml playbook from root directory and runs it, which in turn will pull docker images, install packages and add new configuration changes as required.


## Step 1 Push the crontab entry

Remember for using docker module on host, it should be pre installed on host, or add a task in playbook to install it along with required python add on libraries for ansible. Also the git repository mentioned for ansible pull should be network reachable by the hosts, and be ready to pull from.

Run pushcron.yml on server to all connected hosts. The playbook will install crontab entry which will run ansible-pull on every reboot, the first reboot will run the playbook from git repository by default. After first run it will check for change and run only on change.

example

```
--- # first push of ansible to all hosts
- hosts: hostgroup
  user: userhost
  sudo: yes
  connection: ssh
  gather_facts: no
  tasks:
  - name: installation of htop on all hosts connected
    apt: name=htop state=present
  - name: adding new crontab entry for ansible pull
    cron: name=reboot special_time=reboot job='"ansible-pull -o -C master -d /home/pi/newrepo -U http://thegitrepoaddressgoeshere.git"'

```


After the first push is run and the client machine is reboot, it will pull the ansible playbook described below and run it, our playbook removes a package installed, adds a docker image and fetches a new configuration file for some package.


```
--- # this file will run on first reboot and change the host system
- hosts: localhost
  user: userhost
  sudo: yes
  connection: ssh
  gather_facts: no
  tasks:
  - name: install
    apt: name=htop state=absent
  - name: add image
    docker: name=busybox image=busybox state=absent pull=always 
  
- hosts: master
  user: masterhost
  sudo: yes
  connection: ssh
  gather_facts: no
  tasks:
  - name: fetch
    fetch:
      src: master/dest/location/file.txt
      dest: /home/client/where/i/want/copied/file.txt
      flat: yes

```



# Step 2 Update Git repo

We edit Git repository, in our case we make a minimal change of a word in the local.yml file. We change state of htop from absent to present. This will change md5 hash of original git repository and it will be different from host git repository clone. Hence it on reboot, the host will find run the ansible-pull command and find a change in repository, it will then pull the new updated playbook from the guit repo and run it. 

example

```
--- # New local playbook which will now run on host reboot
- hosts: localhost
  user: userhost
  sudo: yes
  connection: ssh
  gather_facts: no
  tasks:
  - name: install
    apt: name=htop state=present #this line is altered from original
  - name: add image
    docker: name=busybox image=busybox state=absent pull=always 
  
- hosts: master
  user: masterhost
  sudo: yes
  connection: ssh
  gather_facts: no
  tasks:
  - name: fetch
    fetch:
      src: master/dest/location/file.txt
      dest: /home/client/where/i/want/copied/file.txt
      flat: yes

```


