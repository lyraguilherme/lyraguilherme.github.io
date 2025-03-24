---
title: 'Routing Friends - Network Automation'
date: 2024-11-21T18:50:09-03:00
draft: false
ShowToC: false
TocOpen: true
cover:
    image: /static/posts/routing-friends-automation-handson/routing-friends.png
    alt: 'Routing Friends - Network Automation'
    caption: 'Routing Friends - Network Automation'
---

We are back with another exciting episode of Routing Friends! This time, we are going to talk about network automation and work in a practical laboratory with specific tools/frameworks to carry out operations with automation!

Link to YouTube (content in PT-BR):
[Desvendando a Automação de Redes: Lab Hands-On com Guilherme Lyra, CCIE #66666 | Episódio 167 ](https://www.youtube.com/watch?v=V8hF8toSAJ4)


## Laboratory Use Guide


### 1. Preparing your automation host
The first step is to organize your own automation host, which is nothing more than the computer where you will execute the scripts. The only point that should be noted is that this computer requires access to your laboratory's network equipment. 

I recommend using a Linux distribution for this host, but it can be used on Mac or Windows as well. I personally like to use a **Debian** VM for this.

Applications required on your automation host:

- Python
- Ansible


### 2. Access to my example scripts

I suggest you to clone the GitHub repository below. There you will find the topology and the scripts that were used during the podcast **Routing Friends - Lab Hands-on** carried out on November 21, 2024:

```shell
git clone https://github.com/lyraguilherme/LabHandsOn-Nov2024/
```


### 3. Preparing or selling dependencies

To use the scripts, you should preferably create a **venv**:
```shell
python -m venv venv
```

Activate the venv:
```shell
source venv/bin/activate
```

Install the dependencies:
```shell
pip install -r requirements.txt
```


### 4. Setting credentials as environment variables

After installing all the dependencies, define your network device's credentials as environment variables:

```shell
export LAB_USERNAME="my_username"
export LAB_PASSWORD="my_password"
```

From now on, you will be able to use the scripts that I shared.