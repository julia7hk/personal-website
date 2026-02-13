---
title: "pyenv - Fixing Python Installation"
date: "2026-02-06"
description: "Fixing pyenv and certifi issues after Python got messed up"
---

[01/24/2026](#january-24-2026):
ok my python got messed up when i was trying to make some research lab code work and i feel like i need to fix it cuz its creating a lot of problems even in places where i didnt used to have problems like tdm

[02/06/2026](#february-6-2026):
cs471 assignment 1 required me to use python version 3.10 (but my computer system uses 3.11). the assignment told me to use conda to get the new version but ill use pyenv!

---

# **January 24, 2026**


1. show which python versions you have with pyenv
```
pyenv versions
```


2. use pyenv to install python 3.11
```
pyenv install 3.11
```

^ installed python 3.11.13 (i used to have 3.11.11)


3. shows which python version youre using
```
which python
```


4. written record of which version you want to use on your system
```
nano .pyenv/version
```

^ update to 3.11.13 (it used to say 3.11.11)


5. 

i tried messing with this in my `.zshrc` file but its still cooked


before:

```jsx
(in .zshrc)
## python cert
export SSL_CERT_FILE=$(python -m certifi)
```

```jsx
(in terminal)
Last login: Fri Jan 23 23:08:55 on ttys001
/Users/julia7hk/.zshrc:124: command not found: python
➜  ~
➜  ~
```

after: (changed python to python3)

```jsx
(in .zshrc)
## python cert
export SSL_CERT_FILE=$(python3 -m certifi)
```

```jsx
(in terminal)
Last login: Fri Jan 23 23:07:19 on ttys000
/opt/homebrew/opt/python@3.14/bin/python3.14: No module named certifi
➜  ~
➜  ~
```


fixes the error in the section above:
```
/Users/julia7hk/.pyenv/versions/3.11.13/bin/python -m pip install certifi
```


---

final 

```jsx
➜  ~ pyenv versions
  system
* 3.11.13 (set by /Users/julia7hk/.pyenv/version)
```

⏺ Found it. In your ~/.zshrc, this line runs python -m certifi to set SSL_CERT_FILE:

```jsx
export SSL_CERT_FILE=$(python -m certifi)
```

Since your current pyenv python resolves to 3.11.13 (which doesn't exist yet), it produces a broken path.

Fix: temporarily unset it and install:p.

SSL_CERT_FILE="" pyenv install 3.11

This will use the system default certificates for the download. After that, you may want to update the line in ~/.zshrc to be more resilient — e.g., by using Homebrew's
cert bundle instead:

export SSL_CERT_FILE=$(brew --prefix)/etc/openssl@3/cert.pem

This avoids the circular dependency where the cert path depends on the very Python version you might be installing.





# **February 6, 2026**

installing a different python version for cs471 (3.10 when i use 3.11)

use pyenv since i already have it installed and set up on my computer

```jsx
➜  ~ pyenv versions
  system
* 3.11.13 (set by /Users/julia7hk/.pyenv/version)
```

```jsx
in .zshrc:

## pyenv
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

install a new python version 3.10

```jsx
pyenv install 3.10
```

```jsx
➜  ~ pyenv versions
  system
  3.10.18
* 3.11.13 (set by /Users/julia7hk/.pyenv/version)
```

make python 3.10 the default python version in a specific folder

```jsx
➜  cs471 pyenv versions
  system
  3.10.18
* 3.11.13 (set by /Users/julia7hk/.pyenv/version)
➜  cs471
➜  cs471
**➜  cs471 pyenv local 3.10.18**
➜  cs471
➜  cs471
➜  cs471 pyenv versions
  system
* 3.10.18 (set by /Users/julia7hk/_proj/00_purdue/cs471/.python-version)
  3.11.13
➜  cs471
```

initialize shell

```jsx
python -m venv venv
```

run shell

```jsx
source venv/bin/activate
```