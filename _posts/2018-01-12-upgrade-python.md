---
layout: post
title:  "Upgrading to Python 3.6 in venv"
author: skalavala
categories: [ raspberrypi, python ]
image: assets/images/9.jpg
---
Do you have a Raspberry Pi/Ubuntu that is running an older version of python and you want to upgrade python version?

## Upgrading to Python 3.6  

Starting 0.65, Home Assistant requires minimum Python version 3.5.3. For those who are running on older versions of Python **in virtual environment**, you can use the following instructions to upgrade quickly.

Before you continue, it is always a good practice to update your operating system. Run the following commands to update:

```
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade
```

Install Python3 components:
```
sudo apt-get install python3 python3-venv python3-pip
```

Make sure the `homeassistant` user is created.
```
sudo useradd -rm homeassistant
```

Prepare the working directory where the virtual environment will reside. Note that it will rename the existing `homeassistant` virtual environment to `homeassistant_old` before it creates a new virtual environment with the same name.
If your old virtual environment is named differently, feel free to change the commands.

```
cd /srv
sudo mv homeassistant homeassistant_old
sudo mkdir homeassistant
sudo chown homeassistant:homeassistant homeassistant
```

### Installing Python 3.6 on Ubuntu/Debian Systems:

```
sudo add-apt-repository ppa:jonathonf/python-3.6
sudo apt-get update
sudo apt-get install python3.6
sudo apt-get install python3.6-venv
sudo apt-get install python3.6-dev
```

### Installing Python 3.6 on Raspberry Pi:

```
sudo apt-get update
sudo apt-get install build-essential tk-dev libncurses5-dev libncursesw5-dev libreadline6-dev libdb5.3-dev libgdbm-dev libsqlite3-dev libssl-dev libbz2-dev libexpat1-dev liblzma-dev zlib1g-dev

wget https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tar.xz
tar xf Python-3.6.0.tar.xz
cd Python-3.6.0
./configure
make
sudo make altinstall

sudo rm -r Python-3.6.0
rm Python-3.6.0.tar.xz
```

Now that you installed Python 3.6, run thefollowing commands to setup virtual environment:

```
sudo su -s /bin/bash homeassistant
cd /srv/homeassistant
python3.6 -m venv .
source /srv/homeassistant/bin/activate
```

Finally, Install/Upgrade and run Home Assistant:

```
(homeassistant) homeassistant@<hostname>:/srv/homeassistant python3 -m pip install wheel
(homeassistant) homeassistant@<hostname>:/srv/homeassistant pip3 install homeassistant
(homeassistant) hass
```

If everything is running well, you may want to delete the old virtual environment by running the following command:

```
sudo rm -r /srv/homeassistant_old
```

You are all set!


To install 3.5.3 version of python, the procedure is exactly the same, except download the file from the path:
```
wget https://www.python.org/ftp/python/3.5.3/Python-3.5.3.tar.xz
```
