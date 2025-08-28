# Installation
Main site

https://git-scm.com/

Docs

https://git-scm.com/docs

Installing git on Feodra is done through the console `sudo dnf install git`

# Configuration

Config file location: `~/.gitconfig`

Structure
```sh
[variable]
    key = value
```

Syntax for configuration
```bash
git config --global variable.key "value"
```

## Credentials
```sh
git config --global user.name "Yosmel Chiang"
git config --global user.email "email@email.com"
```

## Credential Store
The `Git Credential Manager` comes without a default store for Linux distributions, and since thats what I use, I have to set it up manually 

[GCM docs](https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/credstores.md)

[Credentialstore](https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/configuration.md#credentialcredentialstore)

```sh
git config --global credential.credentialStore secretservice
```

**OBS** - Using secretservice requires `KDEWallet` to be enabled

# Authentication through OAuth
[OAuth alternatives](https://git-scm.com/doc/credential-helpers)

Im using [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager) wich works just fine for windows and linux.

Installation is done from the latest release `tarball`

## 1. Download the tarball, at the time im writing this the version is `64.2.6.1`
```sh
wget https://github.com/git-ecosystem/git-credential-manager/releases/download/v2.6.1/gcm-linux_amd64.2.6.1.tar.gz
```

## 2. Extract it to `/usr/local/bin`
```sh
tar -xzf gcm-linux_amd64.2.6.1.tar.gz -C /usr/local/bin
```

## 3. Configure
```sh
git-credential-manager configure
```

This will write a configuration entries in `~/.gitconfig`

## 4. Git push
To log in first time and good to go.

# Authentication through SSH
## Generate a ssh key
use `ssh-keygen`

https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

## Add the key to git
https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
