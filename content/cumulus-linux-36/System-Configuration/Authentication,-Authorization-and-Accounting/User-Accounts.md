---
title: User Accounts
author: Cumulus Networks
weight: 281
pageID: 8362048
---
By default, Cumulus Linux has two user accounts: *cumulus* and *root*.

The *cumulus* account:

  - Uses the default password *CumulusLinux\!*
  - Is a user account in the *sudo* group with sudo privileges.
  - Can log in to the system through all the usual channels, such as
    console and
    [SSH](/cumulus-linux-36/System-Configuration/Authentication-Authorization-and-Accounting/SSH-for-Remote-Access).
  - Along with the cumulus group, has both show and edit rights for
    [NCLU](/cumulus-linux-36/System-Configuration/Network-Command-Line-Utility-NCLU/).

The *root* account:

  - Has the default password disabled by default.
  - Has the standard Linux root user access to everything on the switch.
  - Disabled password prohibits login to the switch by SSH, telnet, FTP,
    and so on.

For optimal security, change the default password with the `passwd`
command before you configure Cumulus Linux on the switch.

You can add additional user accounts as needed. Like the *cumulus*
account, these accounts must use `sudo` to 
{{<link text="Using sudo to Delegate Privileges" title="Using sudo to Delegate Privileges" >}};
be sure to include them in the *sudo* group.

To access the switch without a password, you need to 
{{<link text="boot into a single shell/user mode" url="Single-User-Mode-Boot-Recovery" >}}.

You can add and configure user accounts in Cumulus Linux with read-only
or edit permissions for NCLU. For more information, see 
{{<link text="Configuring User Accounts" url="Network-Command-Line-Utility-NCLU/" >}}.

## Enabling Remote Access for the root User

The root user does not have a password and cannot log into a switch
using SSH. This default account behavior is consistent with Debian. To
connect to a switch using the root account, you can do one of the
following:

  - Generate an SSH key
  - Set a password

### Generating an SSH Key for the root Account

1.  In a terminal on your host system (not the switch), check to see if
    a key already exists:
    
        root@host:~# ls -al ~/.ssh/
    
    The key is named something like `id_dsa.pub`, `id_rsa.pub` or
    `id_ecdsa.pub`.

2.  If a key does not exist, generate a new one by first creating the
    RSA key pair:
    
        root@host:~# ssh-keygen -t rsa

3.  You are prompted to enter a file in which to save the key
    (/root/.ssh/id\_rsa)*.* Press Enter to use the home directory of the
    root user or provide a different destination.

4.  You are prompted to enter a passphrase (empty for no passphrase).
    This is optional but it does provide an extra layer of security.

5.  The public key is now located in `/root/.ssh/id_rsa.pub`. The
    private key (identification) is now located in `/root/.ssh/id_rsa`.

6.  Copy the public key to the switch. SSH to the switch as the cumulus
    user, then run:
    
        cumulus@switch:~$ sudo mkdir -p /root/.ssh
        cumulus@switch:~$ echo <SSH public key string> | sudo tee -a /root/.ssh/authorized_keys

### Setting the root User Password

1.  Run the following command:
    
        cumulus@switch:~$ sudo passwd root

2.  Change the `PermitRootLogin` setting in the `/etc/ssh/sshd_config`
    file from *without-password* to *yes*.
    
    ``` 
    cumulus@switch:~$ sudo nano /etc/ssh/sshd_config
     
    ... 
          
    # Authentication:
    LoginGraceTime 120
    PermitRootLogin yes
    StrictModes yes
          
    ...  
    ```

3.  Restart the `ssh` service:
    
        cumulus@switch:~$ sudo systemctl reload ssh.service
