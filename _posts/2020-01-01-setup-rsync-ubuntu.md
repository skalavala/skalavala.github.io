## Setting up `rsync` server on Ubuntu

Setup your Ubuntu to accept rsync connections with a separate user/authentication list to your computer's local user accounts. This is more like samba or FTP where your server accepts file transfers with login details that aren't local users.

### Setup

Create the rsyncd.conf file (chances are that it doesn't already exist).
```
sudo editor /etc/rsyncd.conf
```

Put the following content in, obviously swapping out the relevant configuration variables like uid or gid.
```
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid

[code]
    path = /home/folder/code
    comment = User's code folder
    uid = username
    gid = username
    read only = yes
    list = yes
    auth users = username
    secrets file = /etc/rsyncd.secrets
    hosts allow = 192.168.1.0/255.255.255.0
```

Remember that uid and gid are the user and group of the local machine user that will be used when accessing/creating files, and do not have to be the same username as you use to connect to the rsync server as.

### Create Authentication File
You will have seen that in the configuration file, we had a secrets file option. We now need to create that file. This file will contain the usernames and passwords of the accounts you wish to allow to connect to the rsync service. Remember that the username and passwords can be completely separate from the local user accounts, and when your client connects using rsync, they will need to specify the user/password here, not a local user.
```
sudo vim /etc/rsyncd.secrets
```

Fill in the usernames and passwords like so:

```
username:secretpassword
username2:secretPassword2
```

Give the file the following permissons:

```
sudo chmod 600 /etc/rsyncd.secrets
```

Now that we have configured our rsync server, we need to start it so that our server listens for incoming rsync requests,

Managing The Rsync Daemon With Systemctl
The easiest way to manage the rsync daemon in Ubuntu 16.04 is to use:

```
sudo systemctl start rsync
```
If you want the rsync daemon to start up on boot, then use this command:

```
sudo systemctl enable rsync
```

### Manually Managing the Rsync Daemon

If you wish to manually manage the rsync daemon, then you can launch it with the following command:

```
sudo rsync --daemon
```
... and then you would kill it with:
```
sudo kill `cat /var/run/rsyncd.pid`
```

