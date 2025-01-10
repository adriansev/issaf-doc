# How to use SSH to interact with ISS-AF cluster

Strong preference for SSH key-based authentication.

Command line: `ssh -p 60000 <username>@issaf.spacescience.ro`

## SSH key-based authentication

On client device (and for _each_ client device):

1. Generate a local SSH key
``` bash

cd $HOME/.ssh
ssh-keygen -t ed25519 -N ''

```

2. Content of `$HOME/.ssh/id_ed25519.pub` is either sent to admin for
addition to user `.ssh/authorized_keys`

    OR

    the key is copied with `ssh-copy-id` command:   
    `ssh-copy-id -i ~/.ssh/id_ed25519 -p 60000 <username>@issaf.spacescience.ro`


3. Now test the passwordless authentication

## SSH alias definition

For convenience (especially for usage within IDEs and other graphical tools)   
is useful to define an SSH alias within `$HOME/.ssh/config` file   
(if absent, create with `touch $HOME/.ssh/config`)

```
Host issaf
HostName issaf.spacescience.ro
User <username>
Port 60000
```

Now, the alias `issaf` can be used in any and all programs that use libssh (rsync, sftp, scp, and various other graphical tools)

## SSH default configuration

It is useful to have some good defaults set up.   
Add (and keep) as a last stanza in `$HOME/.ssh/config` the following:   
```
Host *
ControlMaster auto
ControlPath ~/.ssh/%C.sshsock
ControlPersist 600
VerifyHostKeyDNS no
CheckHostIP no
ServerAliveInterval 5
ServerAliveCountMax 10
TCPKeepAlive no
ConnectionAttempts 5
IPQoS af21 throughput
ForwardAgent yes
ForwardX11 no
ForwardX11Trusted no
ForwardX11Timeout 0
ExitOnForwardFailure yes
PreferredAuthentications publickey,password,keyboard-interactive
```


## SSHFS - mount a remote filesystem using SFTP

For detailed information see [sshfs page](https://github.com/libfuse/sshfs)

### Installation

* For RedHat(EL) based distros : `dnf install fuse-sshfs.x86_64`
* For Debian based distros : `apt install sshfs`

### Usage

#### Mounting

With `issaf` alias defined previously:   
`sshfs issaf:</directory_on_issaf> <local_mountpoint>`

Additional arguments can be set, see `sshfs --help`   
Useful to use :   
```
-o compression=yes,sshfs_sync,noatime,reconnect,follow_symlinks,transform_symlinks,auto_unmount
```

#### Unmounting

`fusermount -u <local_mountpoint>`



