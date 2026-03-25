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

2. Content of `$HOME/.ssh/id_ed25519.pub` is either:
    * sent to admin for addition to user `.ssh/authorized_keys`  
    OR  
    * the key is copied with `ssh-copy-id` command:   
    ``` bash
    ssh-copy-id -i ~/.ssh/id_ed25519 -p 60000 <username>@issaf.spacescience.ro
    ```


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


## SSH through host connect

SSH ProxyJump option is a way to hop between hosts.  
The general way to use is:  
```
ssh -J ssh_id_of_jump_host ssh_id_of_destination_host
```

in particular, for `issaf` case (with `issaf` alias defined above):  
```
ssh -J issaf ssh_id_of_destination_host

```

or, defining an alias that contain this information in `.ssh/config` file:  
```
Host issaf
HostName issaf.spacescience.ro
User my_issaf_username
Port 60000

Host my_host_with_private_ip
HostName private_ip_of_my_host
User my_user_name_on_this_host
Port my_port_on_this_host
ProxyJump issaf
```

The requirements for this to work is to establish passwordless authentication  
between all pairs of hosts: `client->issaf`, `client->destination_host`, `issaf->destination_host`.  
This would allow to use `my_host_with_private_ip` directly and transparently(in all tools using ssh).  

!!! info "Example of usage"
    ```
    scp my_file issaf:~/  
    rsync my_file issaf:~/  
    ```



## SSH PortForward

Secure Shell (SSH) port forwarding, often called SSH tunneling, is a technique that redirects network traffic
through an encrypted SSH connection. This method creates a secure tunnel between two computers,
allowing you to safely transmit data even across unsecured networks like public Wi-Fi or the internet.  


### Local port forwarding

Forwards traffic from your local machine to a remote server. Useful for accessing remote services as if they were running locally.
General structure of usage(for local usage):  
```
ssh -L PORT_LOCAL:TARGET_HOST:TARGET_PORT ssh_remote_server_alias
```

e.g. if there is a remote service listening on 9090 (but firewalled and/or no direct access to it)  
and remote server have an ssh alias named `my_remote_server` (ssh alias that could include the ProxyJump setup), one would use:  
```
ssh -L some_local_port_bigger_than_1024:localhost:9090 my_remote_server
```
In this way on `localhost:some_local_port_bigger_than_1024` one can connect to `my_remote_server:9090`

Recommended way to use ssh for local port forwarding:
```
ssh -fqNTC -o ServerAliveInterval=5 -o ServerAliveCountMax=6 -o ExitOnForwardFailure=no -L ${PORT_LOCAL}:${TARGET_HOST}:${TARGET_PORT} ssh_remote_server_alias

```


In the `.ssh/config` file a port forward can be defined as:
```
Host           my_alias_for_port_forward
HostName       remote_host_name
User           remote_user_name
Port           remote_port_number
ServerAliveInterval 5
ServerAliveCountMax 8
LocalForward *:local_port localhost:remote_port
```

Thus, doing:
```
ssh -CfqNT my_alias_for_port_forward
```
would start a local port forward as specified in configuration


Arguments details:  
```
-f  Requests ssh to go to background just before command execution.  This is useful if ssh is going to ask for passwords or passphrases, but the user wants it in the background. This implies -n.
-n  Redirects stdin from /dev/null (actually, prevents reading from stdin).  This must be used when ssh is run in the background.
-N  Do not execute a remote command.  This is useful for just forwarding ports.
-T  Disable pseudo-terminal allocation.
-C  Requests compression of all data (including stdin, stdout, stderr, and data for forwarded X11, TCP and UNIX-domain connections).

-L [bind_address:]port:host:hostport
Specifies that connections to the given TCP port or Unix socket on the local (client) host are to be forwarded to the given host and port, or Unix socket, on the remote
side. This works by allocating a socket to listen to either a TCP port on the local side, optionally bound to the specified bind_address, or to a Unix socket.
Whenever a connection is made to the local port or socket, the connection is forwarded over the secure channel, and a connection is made to either host port hostport,
or the Unix socket remote_socket, from the remote machine.
Port forwardings can also be specified in the configuration file.  Only the superuser can forward privileged ports.
IPv6 addresses can be specified by enclosing the address in square brackets.
By default, the local port is bound in accordance with the GatewayPorts setting. However, an explicit bind_address may be used to bind the connection to a specific
address. The bind_address of .localhost. indicates that the listening port be bound for local use only,
while an empty address or .*. indicates that the port should be available from all interfaces.
```


### Dynamic port forwarding

Dynamic port forwarding creates a SOCKS proxy that routes network traffic through a secure SSH tunnel.
This powerful feature creates a local SOCKS proxy server on your machine that forwards all traffic through the SSH connection to the remote server,
which then makes the actual requests to the internet.  
This is useful to access resources that are available only from institutional IP range.  
General structure of the command:
```
ssh -D local_port ssh_remote_server_alias
```

e.g.:
```
ssh -D local_port issaf
```

Recommended way to use ssh for creating a socks proxy:
```
ssh -o "IPQoS=throughput" -fNTC -D local_port_for_proxy issaf
```

For easy usage in browsers see [FoxyProxy Extension](https://getfoxyproxy.org/help/browsers/)  
Detailed help for FoxyProxy usage:

* [How to Use Your Proxy Service with Chrome and the FoxyProxy Extension](https://help.getfoxyproxy.org/index.php/knowledge-base/how-to-use-your-proxy-service-with-chrome-and-foxyproxy-extension/)  
* [How to Use Your Proxy Service with Firefox and the FoxyProxy Extension](https://help.getfoxyproxy.org/index.php/knowledge-base/how-to-use-your-proxy-service-with-firefox-and-foxyproxy-extension/)  




## Resources

* [DigitalOcean:: SSH Essentials: Working with SSH Servers, Clients, and Keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)
* [DigitalOcean:: How to Create an SSH Key in Linux: Easy Step-by-Step Guide](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
* [DigitalOcean:: Create SSH Keys with OpenSSH on macOS, Linux, or Windows](https://www.digitalocean.com/community/tutorials/how-to-create-ssh-keys-with-openssh-on-macos-or-linux)
* [DigitalOcean:: How to Set Up SSH Keys on Ubuntu: A Comprehensive Guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-22-04)
* [DigitalOcean:: How To Use SSHFS to Mount Remote File Systems Over SSH](https://www.digitalocean.com/community/tutorials/how-to-use-sshfs-to-mount-remote-file-systems-over-ssh)
* [DigitalOcean:: SSH Port Forwarding](https://www.digitalocean.com/community/tutorials/ssh-port-forwarding)
* [DigitalOcean:: How To Create SSH Keys With PuTTY to Connect to a VPS](https://www.digitalocean.com/community/tutorials/how-to-create-ssh-keys-with-putty-to-connect-to-a-vps)
* [WayneStateUni:: How to Generate and Use SSH Keys with PuTTY](https://services.wayne.edu/TDClient/277/Portal/KB/ArticleDet?ID=20243)
* [ssh.com:: Using PuTTYgen on Windows to generate SSH key pairs](https://www.ssh.com/academy/ssh/putty/windows/puttygen)


