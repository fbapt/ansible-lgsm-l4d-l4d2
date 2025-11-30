# ansible-lgsm-l4d-l4d2

> ⚠️ You need to have some experience with Ansible, Left 4 Dead games and linuxgsm
> before using this repo!

This repository use ansible to deployed and installed left4dead 1 or left4dead 2 dedicated servers with linuxgsm on Debian 13

# Example of a basic Ansible configuration

## 1. On the Ansible controller

As **root** user :

### Install requirement packages

``` bash
apt-get install sudo openssh-server openssh-client whois python3 python3-apt python3-venv python3-full git
```

### Create an `ansible` user with sudo

``` bash
controlleruser=ansible
adduser "${controlleruser}"
usermod -aG sudo "${controlleruser}"
```

### Add NOPASSWD sudo + harden sudoers

``` bash
echo "${controlleruser} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/"${controlleruser}"
chmod 750 /etc/sudoers.d
chmod 440 /etc/sudoers.d/"${controlleruser}"
```

### Install Ansible inside a python3 virtual environment (as ansible user)

``` bash
su - "${controlleruser}"
python3 -m venv --upgrade-deps ~/venv
echo "source ~/venv/bin/activate" | tee -a ~/.profile
source ~/venv/bin/activate
pip install ansible ansible-core ansible-lint passlib
```

### Generate SSH key (as ansible user)

**RSA 4096** or **ED25519**

``` bash
ssh-keygen -o -a 256 -t ed25519 -C "${USER}@${HOSTNAME}" -f ~/.ssh/id_ed25519_ansible -N ""
```

### Download repository

``` bash
git clone https://github.com/fbapt/ansible-lgsm-l4d-l4d2.git ~/ansible-lgsm-l4d-l4d2
```

## 2. On each Ansible node

As **root** user :

### Install requirement packages

``` bash
apt-get install sudo openssh-server python3 python3-apt cron
```

### Create ansible user

``` bash
nodeuser=ansible
adduser "${nodeuser}"
usermod -aG sudo "${nodeuser}"
```

### Add NOPASSWD sudo + harden sudoers

``` bash
echo "${nodeuser} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/"${nodeuser}"
chmod 750 /etc/sudoers.d
chmod 440 /etc/sudoers.d/"${nodeuser}"
```

### Save IP of each node

``` bash
ip a
```

## 3. On the Ansible controller

As **ansible** user, copy SSH key to each node :

``` bash
nodeip=x.x.x.x
nodeuser=ansible
sshtype=ed25519 or rsa
ssh-copy-id -i ~/.ssh/id_"${sshtype}"_ansible.pub "${nodeuser}@${nodeip}"
```

## 4. Configure Left4Dead1/2 configuration files

### Edit configuration files

#### inventory/lgsmhosts.yml

-   *Change parameters of your host.*

#### host_vars/production/production.yml

-   *Modify variables as you need.*

#### host_vars/production/vault.yml

-   Use a **strong password** for .vault_pass :

Comment in *ansible.cfg* :

    #vault_password_file=.vault_pass

Change password :

``` bash
ansible-vault rekey host_vars/debian13testing/vault.yml
```

Uncomment :

    vault_password_file=.vault_pass

Edit `.vault_pass` with new password.

``` bash
nano .vault_pass
```

### Add L4D1/2 configuration files into roles

#### role/lgsminstallation/files/l4d\[2\]server/lgsm_cfg

→ `l4dserver.cfg`

#### role/lgsminstallation/files/l4d\[2\]server/server

→ `host.txt`, `mymotd.txt`

#### role/lgsminstallation/files/l4d\[2\]server/server_cfg

→ `server.cfg`, `l4dserver.cfg`

#### role/maps/files/l4d\[2\]server/maps

→ Non-workshop maps

#### role/maps/files/l4d2server/maps/workshops

→ Workshop maps (L4D2 only)

#### role/metamod/files/l4d\[2\]server/metamod_plugins

→ Metamod plugin files

#### role/sourcemod/files/l4d\[2\]server/sourcemod_plugins

→ addons + cfg/sourcemod

#### role/strippersource/files/l4d\[2\]server/stripper_cfg

→ dumps + maps configs

## 5. Installation of L4D[2] dedicated servers

Run as **ansible** user on controller :

### Install servers

``` bash
ansible-playbook system_update.yml --limit production
ansible-playbook lgsm.yml --limit production
```

### (Optional) Harden firewall + SSH

> ⚠️ You must create each user SSH key (`l4d1`, `l4d2`) on your server
> and download it on your computer.

**On Windows :**

Use **PuTTYgen + Pageant**.

Enable :

-   "Use proven primes with even distribution (slowest)"
-   "Use 'strong' primes as RSA key factors"

![alt text](image.png)

Generate **RSA 4096** or **EdDSA** recommended.\
Add a **passphrase (min 16 characters)**.

Save private key.
Load into Pageant → *Add key*.

Copy/paste the public key into

    ssh-keys:
	  - type: ssh-ed25519
		key: <HERE>

**On Linux:**

Generate **RSA 4096** or **EdDSA** recommended.\
Add a **passphrase (min 16 characters)**.
``` bash
ssh-keygen -o -a 256 -t ed25519 -C "${USER}@${HOSTNAME}" -f ~/.ssh/id_ed25519_l4d[2]
cat id_ed25519_l4d[2].pub
```

Copy/paste the public key into

    ssh-keys:
	  - type: ssh-ed25519
		key: <HERE>

Recommended nftables
``` bash
ansible-playbook configure_ssh_nftables.yml --limit production
```

or iptables

``` bash
ansible-playbook configure_ssh_iptables.yml --limit production
```

### (Optional) Improve server performance

``` bash
ansible-playbook performance.yml --limit production
```

# About

Playbooks tested with packages :

-   ansible community
-   ansible-core
-   ansible-lint

Tested on **Debian 13**

**LGSM docs :**

-   https://docs.linuxgsm.com/
-   https://linuxgsm.com/servers/l4dserver/
-   https://linuxgsm.com/servers/l4d2server/

**Ansible docs :**

-   https://docs.ansible.com/ansible/latest/getting_started/index.html
