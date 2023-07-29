## Preparing ssh connection on ansible nodes :

As root user :

```bash
apt-get install sudo openssh-client python3 python3-apt
nodeuser=example
echo "${nodeuser} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/"${nodeuser}"
ip a
```

Copy ip of ansible nodes, it will necessary to ssh copy command for ansible controller.

##  Preparing ssh connection on the controller :

As root user install sudo for ansible user :

```bash
adduser ansible
apt-get install sudo whois python3 python3-apt python3-venv python3-full
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
logout
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

##  Configure files

git clone https://github.com/fbapt/ansible-lgsm-l4d-l4d2.git

Edit 

- inventory/lgsmhosts.yml with you username ssh port to change and IP of the server

- host_vars/production.yml with your configuration :
  
  Server, user, ssh port
  
  Change users' ssh pub keys
  
  Change user passwords
  
  Change sourcemod admins
  
- group_vars/Debian_XX if you need to edit url of apt repository
- vault.yml change password of l4d1 and l4d2 user vault.yml
- .vault_pass change password of ansible vault
- Add l4d1/2 configuration files in each roles

  In role/lgsminstallation/files/l4d[2]server/lgsm_cfg ---> example l4dserver.cfg
  
  In role/lgsminstallation/files/l4d[2]server/server ---> example host.txt or mymotd.txt
  
  In role/lgsminstallation/files/l4d[2]server/server_cfg ---> example server.cfg, l4dserver.cfg
  
  In role/maps/files/l4d[2]server/maps ---> example workshops maps in workshops folder for l4d2 only
  
  In role/metamod/files/l4d[2]server/metamod_plugins ---> example files in metamod folder
  
  In role/sourcemod/files/l4d[2]server/sourcemod_plugins ---> example files in addons and cfg/sourcemod folders
  
  In role/strippersource/files/l4d[2]server/stripper_cfg --> example in dumps and maps put cfg maps

##  Installation of l4d1/2 server

As ansible user on the ansible controller :

Execute playbooks

```bash
ansible-playbook --limit production system_update.yml
ansible-playbook --limit production lgsm.yml
```

(optional) If you need to configure firewall nftables or iptables with a harden ssh

Execute iptables or nftables playbook

```bash
ansible-playbook --limit production configure_ssh_nftables.yml
ansible-playbook --limit production configure_ssh_iptables.yml
```

(optional) If you have a dedicated server (not a VM like virtualbox) and need a performance server

Execute performance playbook

```bash
ansible-playbook --limit production performance.yml
```

## About

Playbooks have been tested with ansible-lint

Tested on Debian 11 and 12

Documentation of lgsm :

https://docs.linuxgsm.com/

https://linuxgsm.com/servers/l4dserver/

https://linuxgsm.com/servers/l4d2server/
