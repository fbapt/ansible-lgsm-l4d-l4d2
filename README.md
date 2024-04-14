# ansible-lgsm-l4d-l4d2

> :warning: You need to have some experience with Ansible and left4dead games before using this repo !

# Exemple of a basic ansible configuration

## 1/ On the ansible controller

As root user :

Create a user ansible :

```bash
controlleruser=ansible
adduser "${controlleruser}"
```

Install requirements packages :

```bash
apt-get install sudo openssh-server openssh-client whois python3 python3-apt python3-venv python3-full
```

Add NOPASSWD sudo :

```bash
echo "${controlleruser} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/"${controlleruser}"
```

Install ansible package with python3 environnement as ansible user :

```bash
su - "${controlleruser}"
python3 -m venv venv
echo "source ~/venv/bin/activate" | tee -a ~/.profile
source ~/venv/bin/activate
pip install pip --upgrade
pip install ansible ansible-core ansible-lint
```

Generate a ssh key as ansible user :

```bash
ssh-keygen -o -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')"
```

## 2/ On each ansible nodes

As root user :

Create a user ansible :

```bash
nodeuser=ansible
adduser "${nodeuser}
```

Install requirements package :

```bash
apt-get install sudo openssh-server python3 python3-apt
```

Add NOPASSWD sudo :

```bash
echo "${controlleruser} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/"${controlleruser}"
```

Save ip of each ansible nodes :

```bash
ip a
```

## 3/ On the ansible controller

As ansible user, copy ssh key with user and ip of each ansible nodes :

```bash
nodeip=x.x.x.x
nodeuser=ansible
ssh-copy-id ${nodeuser}@${nodeip}
```

# Configuration and installation

## 1/ Configure files of left4dead1 and/or left4dead2

Download repository

```bash
git clone https://github.com/fbapt/ansible-lgsm-l4d-l4d2.git
```

If you are on vscode add this to your c:/Users/USER/AppData/Roaming/Code/User/settings.json

```bash
    ..........,
	"terminal.integrated.shellArgs.linux": ["-l"],
	"terminal.integrated.defaultProfile.linux": "bash",
	"terminal.integrated.profiles.linux": {
	  "bash": {
		"path": "/bin/bash",
		"icon": "terminal-bash",
		"args": [ "-l" ]
	  },
	}
```

Edit files :

- inventory/lgsmhosts.yml
- host_vars/production/production.yml
- host_vars/production/vault.yml
- .vault_pass
- Add l4d1/2 configuration files in each roles if variables in host_vars/production/production.yml are on true, examples :

	## In role/lgsminstallation/files/l4d[2]server/lgsm_cfg
	
	put file like l4dserver.cfg
	  
	## In role/lgsminstallation/files/l4d[2]server/server
	
  	put file like host.txt or mymotd.txt
	  
	## In role/lgsminstallation/files/l4d[2]server/server_cfg
	
  	put file like server.cfg, l4dserver.cfg
	  
	## In role/maps/files/l4d[2]server/maps

	put maps not on the steam workshops

	## In role/maps/files/l4d2server/maps/workshops

  	put workshops maps in workshops folder for l4d2 only
	  
	## In role/metamod/files/l4d[2]server/metamod_plugins
	
  	put files in metamod folder
	  
	## In role/sourcemod/files/l4d[2]server/sourcemod_plugins
	
  	put files in addons and cfg/sourcemod folders
	  
	## In role/strippersource/files/l4d[2]server/stripper_cfg
	
  	put cfg maps in dumps and maps folders

## 2/ Installation of left4dead1 and/or left4dead2 dedicated servers

On the ansible controller, as ansible user run playbooks on a Debian 11 or 12:

Install left4dead1 and/or left4dead2 dedicated servers :

```bash
ansible-playbook --limit production system_update.yml
ansible-playbook --limit production lgsm.yml
```

(optional) configure firewall with a harden ssh :
> :warning: If you use 'configure_ssh_authenticationmethods: publickey' --> You need to create each user ssh key on your linux server and download on your computer before executing this playbook ! use a sofware such as pageant (putty)...

> Accepted hostkey rsa and ed25519, ssh is restricted to IPV4

```bash
ansible-playbook --limit production configure_ssh_iptables.yml
```

(optional) improve performance of the server :

```bash
ansible-playbook --limit production performance.yml
```

## About

Playbooks have been tested with packages of ansible (8.4.0), ansible-core (2.15.4) and ansible-lint (6.20.0).

Tested on Debian 11 and 12

Documentation of lgsm :

https://docs.linuxgsm.com/

https://linuxgsm.com/servers/l4dserver/

https://linuxgsm.com/servers/l4d2server/

Ansible :

[https://www.ansible.com/](https://docs.ansible.com/ansible/latest/getting_started/index.html)
