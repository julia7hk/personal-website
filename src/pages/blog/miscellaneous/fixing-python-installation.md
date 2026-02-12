---
title: "Fixing Python Installation"
date: "2026-01-24"
description: "Fixing pyenv and certifi issues after Python got messed up"
---

ok my python got messed up when i was trying to make some research lab code work and i feel like i need to fix it cuz its creating a lot of problems even in places where i didnt used to have problems like tdm

---

i tried messing with this in my `.zshrc` file but its still cooked

when i start my terminal i see

before:

```jsx
## python cert
export SSL_CERT_FILE=$(python -m certifi)
```

```jsx
Last login: Fri Jan 23 23:08:55 on ttys001
/Users/julia7hk/.zshrc:124: command not found: python
➜  ~
➜  ~
```

after:

```jsx
## python cert
export SSL_CERT_FILE=$(python3 -m certifi)
```

```jsx
Last login: Fri Jan 23 23:07:19 on ttys000
/opt/homebrew/opt/python@3.14/bin/python3.14: No module named certifi
➜  ~
➜  ~
```

---

```jsx
1. 
pyenv versions

2. 
pyenv install 3.11

^ installed python 3.11.13 (i used to have 3.11.11)

3. 
which python

4.
nano .pyenv/version

^ update to 3.11.13 (it used to say 3.11.11)

5. 
/Users/julia7hk/.pyenv/versions/3.11.13/bin/python -m pip install certifi

^ fixes the error in the section above
```

---

final 

```jsx
➜  ~ pyenv versions
  system
* 3.11.13 (set by /Users/julia7hk/.pyenv/version)
```