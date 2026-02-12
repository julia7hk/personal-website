---
title: "SASE Sniping Discord Bot - Day 4"
date: "2025-11-08"
description: "Setting up the server infrastructure and configuring pyenv on Ubuntu for the Discord bot"
dayNumber: 4
---

server day !

start with checking the server hosts i have saved?

```bash
cat /etc/hosts
```

used to have old free oracle cloud servers that werent being used, reset/remade servers

fix server wiring and `/etc/hosts` file to update it

---

for convenience when ssh-ing into the server,

in `.ssh` directory, see contents of `config` file

```bash
cat config
```

```bash
IdentityFile=~/.ssh/julia7hk

UseKeychain yes
AddKeysToAgent yes

Host oc03
    User ubuntu

Host data
    User kang616

Host borg01
    Hostname borg01.cs.purdue.edu
    User kang616
```

---

log in to server

```bash
ssh ubuntu@oc03
```

things to have `bot.py` run:

1. screen
2. clone github repo
3. install pyenv, etc etc

copy my ssh key into server so that i can git clone my repo onto server

```bash
scp julia7hk ubuntu@oc03:/home/ubuntu/.ssh
```

### install pyenv on the server (ubuntu)

Installing Pyenv on Ubuntu involves a few steps, primarily using the `pyenv-installer` script for simplicity.

**1. Update System and Install Dependencies:**

Before installing Pyenv, update your system's package list and install necessary build dependencies for Python:

Code

```bash
sudo apt update
```

```bash
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \xz-utils tk-dev libffi-dev liblzma-dev git
```

**2. Install Pyenv using pyenv-installer:**

The `pyenv-installer` script simplifies the installation process:

```bash
curl https://pyenv.run | bash
```

**3. Configure your Shell Profile:**

The installer will likely provide instructions to add Pyenv to your shell's environment. You typically need to add lines similar to these to your `~/.bashrc` (or `~/.zshrc` if you use Zsh):

Code

`echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrcecho 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrcecho -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc`

### now double check `cat ~/.bashrc`

you should see 

```jsx
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi
```

appended to the end of your `.bashrc` file 😁

(since thats exactly what the command above is doing)

### now exit and ssh back into server

### pyenv should be setup on server

### now install python 3.11 on the server

```jsx
pyenv install 3.11
```
