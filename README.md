# Celestis 1.12.1 Server Overview

**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [Celestis 1.12.1 Server Overview](#)
    - [Introduction](#)
            - [What is CMaNGOS](#)
            - [What is Celestis](#)
    - [Hypothetical CMaNGOS Installation](#)
        - [Specific Guide](#)
        - [We're done!](#)
    - [WoW Client Installation Windows, OS X, GNU/Linux](#)
            - [Windows](#)
            - [OS X](#)
            - [GNU/Linux](#)
    - [WoW Addons](#)
            - [Downloading addons](#)
            - [Installing Addons](#)
            - [Useful Addons](#)
    - [WoW Useful Commands](#)
            - [Change password](#)
            - [GM Commands](#)

## Introduction

#### What is CMaNGOS

[Wiki](https://github.com/cmangos/issues/wiki)

#### What is Celestis

Celestis is a fork of the CMaNGOS server emulation software with a guide to installation. Celestis is not changed much, but rather emphasis is put on hypothetical installation first which is what this guide is for. Celestis takes a look into hypothetical installation of a classic-server.

## Hypothetical CMaNGOS Installation

### Specific Guide

This guide is mostly complete with a few changes listed below: [CMaNGOS Guide](https://github.com/cmangos/issues/wiki/Installation-Instructions)

1) We'll be using an AWS t2.micro [EC2 instance](https://aws.amazon.com/ec2/instance-types/) which should allow probably 5-10 players with no issues

2) We'll be using AWS Ubuntu 14.04.4 LTS build
```
sudo apt-get install build-essential gcc g++ automake git-core \
autoconf make patch libmysql++-dev mysql-server libtool \
libssl-dev grep binutils zlibc libc6 libbz2-dev cmake subversion vim libboost-all-dev
```
3) First thing we should do is setup our directory structure

* Create user mangos and change login to them
```
useradd -m -d /home/mangos -c "MANGoS" -g mangos mangos
passwd mangos
su - mangos
cd /home/mangos
```

* Clone repositories

```
git clone git://github.com/cmangos/mangos-classic.git mangos
git clone git://github.com/ACID-Scripts/Classic.git acid
git clone git://github.com/classicdb/database.git classicdb
mkdir run build
```

4) Now we can compile the build

* make
```
cd build
cmake ../mangos -DCMAKE_INSTALL_PREFIX=/home/mangos/run -DDEBUG=0
make
make install
```

* config files
```
cp /home/mangos/mangos/src/mangosd/mangosd.conf.dist.in /home/mangos/run/etc/mangosd.conf
cp /home/mangos/mangos/src/realmd/realmd.conf.dist.in /home/mangos/run/etc/realmd.conf
cp /home/mangos/mangos/src/game/AuctionHouseBot/ahbot.conf.dist.in /home/mangos/run/etc/ahbot.conf
```

* edit mangosd.conf and change this line to:
```
DataDir = "/home/mangos/run"
```

5) Extracting the map files

* This part is listed in [this section](https://github.com/cmangos/issues/wiki/Installation-Instructions#extract-files-from-the-client) of the guide. I would highly recomend just installing your WoW client on windows, and running those scripts to get your needed `vmaps maps dbc` folders. Note that the scripts seem not to include .exe extensions to files, which should be added. Also note you need to extract from a vanilla 1.12.1 client.

```
mv vmaps /home/mangos/run/
mv maps /home/mangos/run/
mv dbc /home/mangos/run/
```

6) Importing Databases

```
mysql -u root -p < /home/mangos/mangos/sql/create/db_create_mysql.sql
mysql -u root -p mangos < /home/mangos/mangos/sql/base/mangos.sql
mysql -uroot -p characters < /home/mangos/mangos/sql/base/characters.sql
mysql -uroot -p realmd < /home/mangos/mangos/sql/base/realmd.sql
```

7) Initializing the world

* Initialize Installer config
```
cd /home/mangos/classicdb
./InstallFullDB.sh
vim InstallFullDB.config
```

* At this point add to the following lines:

```
CORE_PATH="/home/mangos/mangos"
ACID_PATH="/home/mangos/acid"
```

* Install
```
./InstallFullDB.sh
cd ..
```

8) More database stuff

```
mysql -u root -p mangos < /home/mangos/mangos/sql/scriptdev2/scriptdev2.sql
mysql -u root -p mangos < /home/mangos/acid/*.sql
```

9) Setting up your public ip address, and opening ports

```
mysql -u root
USE realmd;
SELECT * FROM realmlist;
UPDATE realmlist set name="my server name", address="my public EC2 address";
exit;
```

* From within your EC2 instance Security Group add 2 inbound rules

```
Custom TCP Rule     TCP     3724        Anywhere        0.0.0.0/0
Custom TCP Rule     TCP     8085        Anywhere        0.0.0.0/0
```

10) [Configuring your 1.12.1 WoW client](https://github.com/cmangos/issues/wiki/Installation-Instructions#configuring-your-wow-client)

11) Running the server

* This section gets a bit complicated due to the implementation I chose. When running the server you get access to a shell to do commands onto the server. For this I wanted to have a running detatchable SCREEN instance as to allow easy ssh access to the shell without disrupting the server.

* First we need to make 2 files to run our server

* /home/mangos/realmd.sh
```
#!/usr/bin/env sh

/home/mangos/run/bin/realmd -c /home/mangos/run/etc/realmd.conf
```

* /home/mangos/mangosd.sh
```
#!/usr/bin/env sh

/home/mangos/run/bin/mangosd -c /home/mangos/run/etc/mangosd.conf -a /home/mangos/run/etc/ahbot.conf
```

* Initial run of the server

```
su - mangos
script /dev/null
screen
./realmd.sh &
./mangosd.sh
CTRL+A CTRL+D
```

* From within the SCREEN session you can talk to the mangosd shell and do commands such as "account create"

* How to get back into our SCREEN session

```
su - mangos
script /dev/null
screen -r
CTRL+A CTRL+D
```

[Creating first account](https://github.com/cmangos/issues/wiki/Installation-Instructions#creating-first-account)

[First Login](https://github.com/cmangos/issues/wiki/Installation-Instructions#first-login)

### We're done!

## WoW Client Installation Windows, OS X, GNU/Linux

* Download your vanilla 1.12.1 WoW client such as [here](https://redd.it/2wc63i)

#### Windows

Edit your realmlist.wtf to your EC2 server address and click wow.exe to play

#### OS X

Edit your realmlist.wtf to your EC2 server address

```
brew tap caskroom/cask
brew cask install java xquartz
brew install wine
wine wow.exe
```

#### GNU/Linux

Edit your realmlist.wtf to your EC2 server address

```
sudo add-apt-repository ppa:ubuntu-wine/ppa -y && sudo apt-get update && sudo apt-get install wine
wine wow.exe
```

## WoW Addons

#### Downloading addons

* [Here](http://www.vanilla-addons.com/dls/) is a decent website with many vanilla addons, it's a tad slow, but they work.

#### Installing Addons

* Very simple, extract your addon, and drag it into your WoW/Interface/AddOns directory
* From the character screen bottom left you can edit and add your new addons

#### Useful Addons

* A really nice breakdown of useful addons can be found at this [forum](forum.twinstar.cz/showthread.php/91251-Useful-User-Experience-Addons-for-1-12)

My personal choices are:

* Atlas v1.8.1 (Map)
* Bartender2 (Action Bars)
* Questie (Quest helper - this is a necessity)
* eCastingBar (Casting Bar - similar to quartz)

Still looking for a Sexymap 1.12.1 but I'll keep looking.


## WoW Useful Commands

#### Change password

From within game, in the chat box type:

```
.account password <old-password> <new-password> <new-password>
```

#### GM Commands

* List of commands
```
.commands
```

* your player info
```
.pinfo
```

* Teleport
```
.tele <location>
```

* Go to player
```
.goname <character-name>
```

* Player to self
```
.namego <character-name>
```

* Modify speed
```
.modify aspeed <rate>
```

* Find all accounts and characters on the server (change empty string to search string to narrow down results)
```
.lookup player account ""
```

* Server commands
```
.server
```

* Banning players et al
```
.ban
.kick
.baninfo
.banlist
```
