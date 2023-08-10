# ansible-lgsm-l4d-l4d2

> :warning: You need to have some experience with Ansible and left4dead games before using this repo !

## Preparing ssh connection on ansible nodes :

As root user :

```bash
apt-get install sudo openssh-server python3 python3-apt
nodeuser=example
echo "${nodeuser} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/"${nodeuser}"
ip a
```

Copy ip of ansible nodes, it will necessary to ssh copy command for ansible controller.

##  Preparing ssh connection on the controller :

As root user install sudo for ansible user :

```bash
adduser ansible
apt-get install sudo openssh-server openssh-client whois python3 python3-apt python3-venv python3-full
echo ansible ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/ansible
```

As ansible user copy ssh key each ansible nodes :

```bash
ssh-keygen -o -a 256 -t ed25519 -C "$(hostname)-$(date +'%d-%m-%Y')"
nodeip=x.x.x.x
nodeuser=example
ssh-copy-id ${nodeuser}@${nodeip}
```

##  Installing packages on the ansible controller :

As ansible user to create a virtual environment and install ansible packages :

```bash
cd
python3 -m venv venv
echo "source $HOME/venv/bin/activate" | tee -a $HOME/.profile
. $HOME/.profile
pip install pip --upgrade
pip install ansible ansible-core ansible-lint
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

Get repository

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

(optional) If you need to configure firewall nftables or iptables with a harden ssh :

Run iptables or nftables playbook

```bash
ansible-playbook --limit production configure_ssh_nftables.yml
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

Playbooks have been tested with packages of ansible (8.2.0), ansible-core (2.15.2) and ansible-lint (6.17.2).

Tested on Debian 11 and 12

Documentation of lgsm :

https://docs.linuxgsm.com/

https://linuxgsm.com/servers/l4dserver/

https://linuxgsm.com/servers/l4d2server/
