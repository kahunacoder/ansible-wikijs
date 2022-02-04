# README

A Wiki.js server provisioned using Ansible

## Installing Wiki.js using Ansible

Using Ansible we will install packages needed to provision wiki.js

## Preparing the Control Node

The `Control Node` is the machine you will be using to manage and configure the Wiki.js server

The machine will need the following software.

- [Install Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html) for your Mac or Linux machine.

The following steps are required.

- [Git Clone of this repository](https://github.com/kahunacoder/anisble-wikijs)
- SSH key provided for this installation.
- Configure Ansible for this installation by pasting this in your terminal
  - `sudo ansible-galaxy install geerlingguy.pip`
  - `sudo ansible-galaxy install geerlingguy.docker`
  - `sudo ansible-galaxy install geerlingguy.apache`

## Edit the configuration

Edit the `group_vars/all/server.yml` file to customize this installation. The default values work fine for a Virtual Box installation. (See below). You may need to edit the file for a live server. There are some advanced configuration settings not shown below.

```yaml
hostname: "Wikijs" # your hostname if your using vagrant
tz: "America/New_York" # the timezone if your using vagrant

# The developer user is a privileged user with shell and sudo access.
developer: "dev" # user for the optional developer role

### Project specific settings ###
PROJECT_ACCOUNT: "wiki" # The account name of the project
PROJECT_DOMAIN: "wiki.example" # The domain name of the project
```

---

You'll also need to edit the `/hosts` file:

```ini
[servers]
wiki
```

Now create or edit a file in `/host_vars` with the same name as your servers in the above section. Here's what the default `/host_vars/wiki.yml` file looks like;

```yaml
# For machines without a domain name
# you can use it's public IP address
ansible_host: wiki.mydomain.com
# This is the default user on an ubuntu install.
ansible_ssh_user: wiki
# The path on the control node where the
# ssh private key is stored.
# The default value is common
ansible_ssh_private_key_file: "~/.ssh/id_rsa"
hostname: "mydomain.com" # your hostname
tz: "America/New_York" # the timezone of your server
PROJECT_DOMAIN: "wiki.mydomain.com" # The domain name of the project
```

## Software we will be installing

The following software is going to be installed on the server as required by wiki.js

- [Docker](https://www.docker.com/use-cases) is an open-source project for automating the deployment of applications as portable, self-sufficient containers that can run on the cloud or on-premises.

## Containers we will be installing

These containers will be installed.

- PostgreSQL the database for wiki.js
  - [PostgreSQL](https://www.postgresql.org/) The World's Most Advanced Open Source Relational Database
- Wiki.js is the wiki software
  - [Wiki.js](https://wiki.js.org/) The most powerful and extensible open source Wiki software
- Companion Update Agent for Wiki.js
  - [wiki-update-companion](https://github.com/Requarks/wiki-update-companion) Automated upgrade for Wiki.js in a Docker environment
- Web interface to manage docker containers on the server.
  - [portainer](https://www.portainer.io/) A centralized service delivery platform for containerized apps
- Web interface to manage the wiki.js database similar to phpMyAdmin.
  - [adminer](https://www.adminer.org/) Database management in a single php file.

Wiki.js requires a proxy server to run on a web server that host multiple sites. The server I'm using is running apache so I've included apache specific settings need to support this.

The ansible configuration will add the following configurations to an existing apache server configuration. No other configurations will be modified.

```yaml
apache_vhosts:
  - servername: "{{PROJECT_DOMAIN}}"
    extra_parameters: |
      ProxyPreserveHost Off
      ProxyPass / http://0.0.0.0:3000/
      ProxyPassReverse / http://0.0.0.0:3000/
      Header always set X-Robots-Tag "noindex, nofollow"

apache_mods_enabled:
  - proxy_http.load
  - proxy.load
  - headers.load
```

## Developer Tools (Optional)

The following shell enhancements and developer tools are going to be installed on the server based on the values set in the `/roles/developer/tasks/main.yml` file.

- [TMUX](https://www.nuno-sarmento.com/how-to-install-and-use-tmux-on-ubuntu/) allows users to switch between multiple programs in one terminal. Users can detach and reattach the multiple programs to different terminal sections. TMUX sessions are persistent. The running programs continue to run even if you are disconnected.
- [Mosh](https://linuxhandbook.com/mosh/) stands for MObile SHell and is a replacement of SSH for remote connections to Unix/Linux systems that brings a few noticeable advantages over well known SSH connections. In brief, it’s faster and more responsive, especially on long delay and/or unreliable links. So we can say that Mosh is an alternative for SSH.
- [ZSH](https://www.tecmint.com/install-zsh-in-ubuntu/) stands for Z Shell which is a shell program for Unix-like operating systems. ZSH is an extended version of Bourne Shell which incorporates some features of BASH, KSH, TSH.
- [powerline](https://ubunlog.com/en/powerline-customize-command-line/) provides status lines and prompts and offers useful information on the terminal prompt.
  - fonts-powerline
- [Powerlevel10k](https://dev.to/abdfnx/oh-my-zsh-powerlevel10k-cool-terminal-1no0) is a theme for Zsh. It emphasizes speed, flexibility and out-of-the-box experience.
- [zsh-syntax-highlighting](https://linuxhint.com/enable-syntax-highlighting-zsh/) automatically highlights your commands as you type them, which can help you catch syntax errors and fix them before running the command.

We will also be installing a set of of `.dot` files and fonts for the developer.

- .zshrc
- .aliases
- .tmux.conf
- .p10k.zsh
- Fira Code Nerd Mono
- Fira Code Nerd Regular

#### Can I test it first

You can test this server setup with the Vagrantfile included.

- Install Virtualbox
- Install Vagrant
- Install a vagrant plugin
  - `vagrant plugin install vagrant-auto_network`
- Start the virtual machine with
  - `vagrant up`
- Vagrant will start the VM and automatically provision it from Ansible.
- Note the `ipaddress` of the virtual machine when running `vagrant up` otherwise ssh into the vm using `vagrant ssh`.
- While in the VM, copy the db password in `/etc/wiki/.db-secret`
- View the database gui at [http://ipaddress:3001](http://ipaddress:3001) and enter the following credentials; `system:postgreSQL`, `server:db`, `username:wiki`, `password:paste from /etc/wiki/.db-secret`, `database:wiki`
- View the docker web interface at [http://ipaddress:9000](http://{protainer.HOSTNAME})
- To test apache, edit your `/etc/hosts` file and add the following to your `/etc/hosts` file:

```ini
ipaddress wiki.example
```

- open your browser to the following urls.
  - [http://wiki.example](http://wiki.example)
  - [http://wiki.example:9000](http://wiki.example:9000) Docker GUI
  - [http://wiki.example:3001](http://wiki.example:3001) Adminer Database GUI

## OK Lets Do It

- Run provision command
- `ansible-playbook playbook.yml -i hosts`
- open your browser to the following urls.
  - `http://PROJECT_DOMAIN`
  - `http://PROJECT_DOMAIN:9000` Docker GUI
  - `http://PROJECT_DOMAIN` Adminer Database GUI

### Who do I talk to

- Gary Smith bk@kc.gs
