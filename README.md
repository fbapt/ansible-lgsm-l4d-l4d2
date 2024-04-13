# ansible-lgsm-l4d-l4d2

> :warning: You need to have some experience with Ansible and left4dead games before using this repo !

## 1/ On the ansible controller
### Preparing requirements package / NOPASSWD sudo for ansible user / python env for ansible package

As root user :

```bash
controlleruser=ansible
adduser "${controlleruser}"
apt-get install sudo openssh-server openssh-client whois python3 python3-apt python3-venv python3-full
echo "${controlleruser} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/"${controlleruser}"
su - "${controlleruser}"
python3 -m venv venv
echo "source ~/venv/bin/activate" | tee -a ~/.profile
source ~/venv/bin/activate
pip install pip --upgrade
pip install ansible ansible-core ansible-lint
```

## 2/ On each ansible nodes
### Preparing requirements package / NOPASSWD sudo for a user / ip of each nodes

As root user :

```bash
nodeuser=ansible
adduser "${nodeuser}"
apt-get install sudo openssh-server python3 python3-apt
echo "${nodeuser} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/"${nodeuser}"
ip a
```

Save ip of each ansible nodes.

## 3/ On the ansible controller

As ansible user :

Generate a ssh key

```bash
ssh-keygen -o -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')"
```

Copy ssh key with user and ip each ansible nodes :

```bash
nodeip=x.x.x.x
nodeuser=example
ssh-copy-id ${nodeuser}@${nodeip}
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

##  Configure files of left4dead1 and/or left4dead2

Download repository

https://github.com/fbapt/ansible-lgsm-l4d-l4d2/releases

Edit files :

- inventory/lgsmhosts.yml
- host_vars/production/production.yml
- host_vars/production/vault.yml
- .vault_pass
- Add l4d1/2 configuration files in each roles if variables in host_vars/production/production.yml are on true, examples :

	#### In role/lgsminstallation/files/l4d[2]server/lgsm_cfg
	
	put file like l4dserver.cfg
	  
	#### In role/lgsminstallation/files/l4d[2]server/server
	
  	put file like host.txt or mymotd.txt
	  
	#### In role/lgsminstallation/files/l4d[2]server/server_cfg
	
  	put file like server.cfg, l4dserver.cfg
	  
	#### In role/maps/files/l4d[2]server/maps

	put maps not on the steam workshops

	#### In role/maps/files/l4d2server/maps/workshops

  	put workshops maps in workshops folder for l4d2 only
	  
	#### In role/metamod/files/l4d[2]server/metamod_plugins
	
  	put files in metamod folder
	  
	#### In role/sourcemod/files/l4d[2]server/sourcemod_plugins
	
  	put files in addons and cfg/sourcemod folders
	  
	#### In role/strippersource/files/l4d[2]server/stripper_cfg
	
  	put cfg maps in dumps and maps folders

##  Installation of left4dead1 and/or left4dead2 dedicated servers

As ansible user on the ansible controller :

Run the playbooks to install left4dead1 and/or left4dead2 dedicated servers on a Debian 11 or 12 :

```bash
ansible-playbook --limit production system_update.yml
ansible-playbook --limit production lgsm.yml
```

(optional) If you need to configure firewall with a harden ssh :

Run iptables playbook

```bash
ansible-playbook --limit production configure_ssh_iptables.yml
```

(optional) If you have a physical dedicated server (not a VM like virtualbox), it is recommended to :

Run performance playbook

```bash
ansible-playbook --limit production performance.yml
```

Reboot nodes
```bash
reboot now
```

## About

Playbooks have been tested with packages of ansible (8.4.0), ansible-core (2.15.4) and ansible-lint (6.20.0).

Tested on Debian 11 and 12

Documentation of lgsm :

https://docs.linuxgsm.com/

https://linuxgsm.com/servers/l4dserver/

https://linuxgsm.com/servers/l4d2server/
