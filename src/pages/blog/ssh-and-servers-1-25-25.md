---
layout: ../../layouts/BlogPost.astro
title: "SSH and Servers"
date: "2025-01-25"
description: "Notes on SSH configuration and server setup"
---

- if you want to make an ssh key, run the ssh-gen command (just google it)

- **you can find ssh keys in the .ssh/ dir**
    - config file

        - Host names
        - User usernames
        - Keychain
            - contains every password, ssh key, wifi password, etc. that ive ever used
                
    - ssh key
    - ssh key pub (paste this into wherever to ssh into there (ex. data.cs.purdue.edu)
    - known hosts (list of servers you have ssh-ed into before)
        - every time you ssh into a server for the first time you are prompted to
- oracle oc06 ssh
    - add ip address with ‘sudo nano ~/etc/home’
- [data.cs.purdue.edu](http://data.cs.purdue.edu) ssh
    - pinging [data.cs.purdue.edu](http://data.cs.purdue.edu) to find the ip address
    - add ip address with sudo ~/etc/homes
    - `ssh data` works :D
- OR borg01 ssh method
    - in .ssh/config
    - add Hostname borg01.cs.purdue.edu
- visual studio ssh into data
    - F1 → Remote-SSH: Connect to Host
    - (therefore can cursor ssh into data)

2/7/2025 note:

- added borg01 to .ssh config
- ssh [kang616@borg01.cs.purdue.edu](mailto:kang616@borg01.cs.purdue.edu) works ‼️
- but ssh borg01 doesnt work 😔
    - gotta set up etc/hosts
    - sudo nano /etc/homes

2/9/2025 note: check borg01 method or sudo NANO!!! etc hosts