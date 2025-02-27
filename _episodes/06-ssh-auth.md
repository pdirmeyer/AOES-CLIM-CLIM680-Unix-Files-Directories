---
title: "SSH Authentication"
teaching: 10
exercises: 0
questions:
- "Why use SSH key authentication?"
- "How do I set up SSH key authentication?"
objectives:
- "Set up SSH key authentication between your laptop and the ORC cluster."
keypoints:
- "SSH key authentication enhances security."
- "SSH key authentication reduces the frustration of repeatedly typing passwords."
---

### What is SSH and what are SSH keys?

SSH, or secure shell, is an encrypted protocol used to communicate remotely with servers from another computer. 
When working on Unix or Linux servers like the ORC computing system, you will frequently be connecting via terminal sessions using SSH.

SSH keys provide an extremely secure way of logging in that does not require remembering and typing your password each time. 
It is highly recommended as a safe, secure, and more convenient way to work on remote computing systems from your personal computer.

### How does an SSH key work?

Surprisingly, password authentication is not the most secure way to use SSH. 
Although passwords are sent between client (your laptop) and server securely, they are usually not complex or long enough to resist hacking. 
Plus, passwords are vulnerable to being stolen visually by prying eyes or inattentive habits like keeping passwords written on a piece of paper. 

SSH key pairs are two cryptographically secure keys that can be used for authentication between an SSH client (e.g., your laptop) and an SSH server (e.g., HOPPER). 
Each key pair consists of a public key and a private key. The private key is kept by the client. 
Any compromise of the private key will allow an attacker to access servers that are configured with the associated public key, so it should be kept secure and private. 
As an additional precaution, the key can be encrypted with a passphrase which makes this approach twice as secure.

The associated public key can be shared freely without any negative consequences, as it is encrypted in a way that can only be opened in conjunction with the private key.
The public key is uploaded to the server  you want to log into with SSH. 
The key is added to a special file in your home directory on the server at `~/.ssh/authorized_keys`.

### On Windows computers...

On Windows, the MobusXterm software has menu settings to generate and use SSH keys for logging into remore servers. 
Follow the MobusXterm documentation for the version on your computer, as the method has varied with different software versions.

### On Macs and Linux laptops...

The first step to configure SSH key authentication is to generate an SSH key pair on your local computer.
To do this on a Mac or Linux system, use the `ssh-keygen` command. By default, this will create a 3072 bit RSA key pair.

On your personal computer, you can open a new terminal session and generate a SSH key pair by typing `ssh-keygen`. 
**But before doing so**, check to see if you have already done this previously. Evidence will be in the hidden `.ssh` directory under your home directory.
To check, type:

~~~
$ ls ~/.ssh
~~~
{: .language-bash}

If you get a response like:

~~~
-bash: cd: /home/username/.ssh: No such file or directory
~~~
{: .language-bash}

Then you can proceed with the next steps. Otherwise, you can skip ahead to 
<a href="#skip_rsa"><b>Copying an SSH public key to your server.</b></a>

To generate a SSH key pair, type:

~~~
$ ssh-keygen
~~~
{: .language-bash}

You will be offered a chance to select a location for the new keys. 
Usually, it is best to stick with the default location. 
The command will generate two files in your `~/.ssh` directory The private key will be called `id_rsa` and the public key will be `id_rsa.pub`.

If you had previously generated a SSH key pair, you will see a warning prompt that looks like this:

~~~
/home/username/.ssh/id_rsa already exists.
Overwrite (y/n)?
~~~
{: .language-bash}

If you choose to overwrite the key on disk, you will not be able to access any other previously authenticated servers automatically anymore. 
You will have to reestablish authentication with the newly generated key.
So, be very careful when selecting yes, as this is a destructive process that cannot be reversed.

Next you will be promted for an optional passphrase:

~~~
Created directory '/home/username/.ssh'.
Enter passphrase (empty for no passphrase):
~~~
{: .language-bash}

This can be used to encrypt the private key file on disk. 
This provides an extra layer of protection in case your laptop is lost or stolen, hacked or infected with certain malware, 
helping to prevent an attacker from further gaining access to your GMU computing (or any other SSH key authenticated) account.
Advantages include:

* The private SSH key is never exposed on the network. The passphrase is only used to decrypt the key on your computer locally. 
This means that network-based hacking will not be effective against the passphrase.
* The private key is kept within a restricted directory. The SSH client will not recognize private keys that are not kept in restricted directories. 
The key itself must also have restricted permissions (read and write only available for the owner, 
i.e. you must see `-rw-------` when you use `ls -l ~/.ssh/id_rsa` or authentication will fail). 
This means that if you have multiple userids on your computer, they cannot snoop the contents of your private key file.
* Any attacker hoping to crack the private SSH key passphrase must already have access to your computer. 
This means that they would already have access to your user account or the _root_ account. 
If you are in this position, a passphrase can prevent the attacker from immediately logging into other servers from your computer and doing more damage. 

Once you have completed creating your private and public key pair, you will get a response saying the keys have been created, and also:
* A key fingerprint, which will be a long string of random characters
* A randomart image

### Copying the public key to your server<a id="skip_rsa"> </a>

There are several ways to upload your public key to the ORC computing system. 
Here we describe three. 

#### Using ssh-copy-id
The easiest way, if the command is available on your computer, is to use the `ssh-copy-id` command.
The syntax is:

~~~
$ ssh-copy-id username@hopper.orc.gmu.edu
~~~
{: .language-bash}

Where `username` is your username. 

You should see a message like this:

~~~
The authenticity of host 'hopper.orc.gmu.edu (129.174.21.176)' cant be established.
ECDSA key fingerprint is fd:fd:d4:f9:77:fe:73:84:e1:55:00:ad:d6:6d:22:fe.
Are you sure you want to continue connecting (yes/no)?
~~~
{: .language-bash}

The hexcode in the "fingerprint" will be different for you, but this is the expected message the first time you connect to a new host in this way.
Type `yes` and press ENTER to continue. 

Next, you will be prompted for your password. type it in and press ENTER. You are done! 
You will not have to enter your password on HOPPER again until it expires or if you change your key (e.g., if you get a new laptop).

#### Using scp 
If `ssh-copy-id` isn't working, you can copy the public key to HOPPER and append it to your `authorized_keys` file.
The `scp` command (secure copy) uses ssh security to copy files and directories between computers. 
The syntax when securely copying a file from your laptop to HOPPER is:

~~~
$ scp local_filename username@hopper.orc.gmu.edu:path/to/destination
~~~
{: .language-bash}

Where `local_filename` would be the file on your laptop 
(including the path if you do not execute the `scp` command from the directory containing the file),
`username` is your username, and `path/to/destination` is the remote path if the destination is not your home directory.

In this specific case, first make sure that the ~/.ssh directory exists on HOPPER. If it does not, create it:

~~~
$ mkdir ~/.ssh
~~~
{: .language-bash}

From a terminal session on your personal computer:

~~~
$ cd ~/.ssh
$ scp id_rsa.pub username@hopper.orc.gmu.edu:~/.ssh/laptop.pub
~~~
{: .language-bash}

To execute the `scp` command, you will be asked for your password on HOPPER.

Once completed, from the terminal session on HOPPER:

~~~
$ cd ~/.ssh
$ cat laptop.pub >> authorized_keys
$ rm laptop.pub
~~~
{: .language-bash}

You can `cat` or edit the file `authorized_keys` and see that there is now a line
containing three elements in sequence:
* `ssh-rsa`
* A long string of random letters, numbers and symbols, starting with `AAAA`
* An identifier of your laptop, including your account on your laptop if it has been set up that way.


#### Manually copying your public key
If the approaches above do not work for you, you can do the above process manually. 
The content of your `id_rsa.pub` file can be added manually to the file ~/.ssh/authorized_keys on HOPPER.
To display the content of your id_rsa.pub key, type this into your local computer's terminal session:

~~~
$ cat ~/.ssh/id_rsa.pub
~~~
{: .language-bash}

You will see the key’s content, which will look something like this:

~~~
~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCqql6MzstZYh1TmWWv11q5O3pISj2ZFl9HgH1JLknLLx44+tXfJ7mIrKNxOOwxIxvcBF8PXSYvobFYEZjGIVCEAjrUzLiIxbyCoxVyle7Q+bqgZ8SeeM8wzytsY+dVGcBxF6N4JS+zVk5eMcV385gG3Y6ON3EG112n6d+SMXY0OEBIcO6x+PnUSGHrSgpBgX7Ks1r7xqFa7heJLLt2wWwkARptX7udSq05paBhcpB0pHtA1Rfz3K2B+ZVIpSDfki9UVKzT8JUmwW6NNzSgxUfQHGwnW7kj4jp4AT0VZk3ADw497M2G/12N0PPB5CnhHf7ovgy6nL1ikrygTKRFmNZISvAcywB9GVqNAVE+ZHDSCuURNsAInVzgYo9xgJDW8wUw2o8U77+xiFxgI5QSZX3Iq7YLMgeksaO4rBJEa54k8m5wEiEE1nUhLuJ0X/vh2xPff6SQ1BL/zkOhvJCACK6Vb15mDOeCSq54Cr7kvS46itMosi/uS66+PujOO+xt/2FWYepz6ZlN70bRly57Q06J+ZJoc9FfBCbCyYH7U/ASsmY095ywPsBo1XQ9PqhnN1/YOorJ068foQDNVpm146mUpILVxmq41Cj55YKHEazXGsdBIbXWhcrRf4G2fJLRcGUr9q8/lERo9oxRm5JFX6TCmj6kmiFqv+Ow9gI0x8GvaQ== username@hostname
~~~
{: .language-bash}
 
Go to the HOPPER terminal window (launch one if you do not have one open).
Make sure that the ~/.ssh directory exists. If it does not, create it:

~~~
$ mkdir ~/.ssh
~~~
{: .language-bash}

Now, you can create or modify the authorized_keys file within this directory. 
You can paste the contents of your id_rsa.pub file to the end of the authorized_keys file, creating it if necessary, using this:

~~~
$ echo "[public_key_string]" >> ~/.ssh/authorized_keys
~~~
{: .language-bash}

But replace the `[public_key_string]` with the output from the cat ~/.ssh/id_rsa.pub command that you executed on your local system. 
It should start with `ssh-rsa AAAA`... or something similar. 
You can use ctl-C (or on Mac, cmd-C) to copy and ctl-V (cmd-V) to past the text between terminal windows.

### Authenticating using SSH keys

Now you should be able to log into any of the ORC login nodes without typing a password every time!

The process is mostly the same as what you have already done. For example:

~~~
$ ssh -Y username@hopper.orc.gmu.edu
~~~
{: .language-bash}

If you used the `scp` or manual copy method above, 
you may see something like this the first time you try to login:

~~~
The authenticity of host 'hopper.orc.gmu.edu (129.174.21.176)' can't be established.
ECDSA key fingerprint is fd:fd:d4:f9:77:fe:73:84:e1:55:00:ad:d6:6d:22:fe.
Are you sure you want to continue connecting (yes/no)?
~~~
{: .language-bash}

The hexcode in the "fingerprint" will be different for you, but this is the expected message the first time you connect to a new host in this way.
Type `yes` and press ENTER to continue. 

If you did not supply a passphrase for your private key, you will be logged in immediately. 
If you supplied a passphrase for the private key when you created the key, you will be required to enter it now. 

Hereafter, you will be in a new `bash` shell session on the remote system.


###### This _episode_ is based on  the tutorial at: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server
