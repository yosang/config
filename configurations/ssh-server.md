- [docker image](#docker-image)
  - [openssh-server](#openssh-server)
  - [ssh commands](#ssh-commands)
  - [ssh files](#ssh-files)
  - [ssh permissions](#ssh-permissions)
  - [ssh config](#ssh-config)
  - [ssh restart](#ssh-restart)
- [client](#client)
- [security](#security)
  - [disable root login with ssh](#disable-root-login-with-ssh)

# docker image
- docker run -d -p 2222:22 ubuntu - creates a ubuntu container and maps host port 2222 to container port 22
  - 22 is the default port of openssh-server running the daemon 

- docker exec -it container_id bash - makes it interactive, attaches a tty, and executes bash

## openssh-server
- apt update
- apt upgrade
- apt install -y openssh-server

## ssh commands
- service ssh start - starts the daemon
- service ssh restart - restarts it

## ssh files
- authorized_keys - here is where we paste the .pub key

## ssh permissions
- mkdir /root/.ssh - required for ssh daemon
- touch /root/.ssh/authorized_keys - required for the .pub key
- chmod 700 /root/.ssh - full permissions to the .ssh for the daemon
- chmod 600 /root/.ssh/authorized_keys - read+write owner, 0 group and 0 others

## ssh config
- vi /etc/ssh/sshd_config
    - PubKeyauthentication yes - allows ssh auth
    - PasswordAuthentication no - removes password requirement

## ssh restart
- service ssh restart - restarts the daemon

# client
- ssh-keygen - creates a new ssh key pair, located in ~/.ssh
- cat docker.pub and paste it in docker image /root/ssh/authorized_keys
- login wiht ssh -i ~/.ssh/docker root@localhost -p 2222

# security
should never use root, instead lets create a new user:

on the server we are ssh'ing into, log in as root and:
- adduser yosang - create a new user
- usermod -aG sudo yosang - append the new user to sudo group 
- su yosang - swiwtch to the created user

this user now needs the public key so we can log in with it
- create a `~/.ssh` folder and create a `authorized_keys` file and paste the `.pub` key inside.
- like earlier, the deamon needs to be able to read this file, add permissions with `chmod 600 authorized_keys`
- we should now be able to `ssh yosang@localhost` using the private key

## disable root login with ssh
- ssh into the server with the new user and sudo `vi /etc/ssh/sshd_config`
- find PermitRootLogin and set it to no
- restart ssh with `sudo service ssh restart`
