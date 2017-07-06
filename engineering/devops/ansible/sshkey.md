

# Ssh key add 

We need to generate a ssh key-pair and add the public key to the client machine for passwordless ssh. Generate a ssh key using "ssh-keygen" and select default options just press enter. Then use "ssh-copy-id hostname@hostip" to copy ssh keys to the hosts you need to connect, you will be prompted to enter password for the ssh connection for key transfer. After that it will be passwordless.

Example

```
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/user/.ssh/id_rsa.
Your public key has been saved in /home/user/.ssh/id_rsa.pub.
The key fingerprint is:
7a:24:f2:7e:24:7f:a8:52:55:93:c4:b4:70:59:cb:4a user@192.168.1.12
The key's randomart image is:
+--[ RSA 2048]----+
|        .+++.    |
|         o*o .   |
|         .E.o    |
|        .. .     |
|    . ..S .      |
|     oo+.        |
|     .o+..       |
|    .. .+ .      |
|     .oo .       |
+-----------------+

ssh-copy-id user@192.168.1.12
The authenticity of host '192.168.1.12 (192.168.1.12)' can't be established.
ECDSA key fingerprint is e9:c6:a0:ac:19:9e:d6:27:8a:82:7a:0f:7e:df:66:c7.
Are you sure you want to continue connecting (yes/no)? 
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
user@192.168.1.12's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'user@192.168.1.12'"
and check to make sure that only the key(s) you wanted were added.

```

## Edit Hosts file

Edit the hosts file and add all the hosts you want to control over ssh via ansible. Default host file location is /etc/ansible/hosts.
Default configuration file is /etc/ansible/ansible.cfg. Read this file for insight on ansible default parameters.
Adding hosts and hostgroup to /etc/ansible/hosts


```
sudo nano /etc/ansible/hosts

```

Add the following example, can have as many groups as needed, same host can exist in  multiple groups or none at all, see link for descrpition.
[groupname1]
host1
host2

[groupname2]
host6
host7
host1


example
```
[local]
localhost

[lan]
192.168.1.5
192.168.1.6
192.168.1.7
192.168.1.8
192.168.1.9

```

