## Login through ssh
1. Please make sure you have the ICRR account.
2. Login to icrlogin1.icrr.u-tokyo.ac.jp:
```bash
[jerry@asus ~]$ ssh cchou@icrlogin1.icrr.u-tokyo.ac.jp
```
3. Login to icrhome05:
```bash
[cchou@icrlogin1 ~]$ ssh icrhome05
```
4. Login to KAMALAN-R750:
```bash
[cchou@icrhome05 ~]$ ssh cchou@172.16.32.46
```

---
## Create ssh key pairs
Create a ssh-key without passphrase in `~/.ssh`:
```bash
[jerry@asus ~]$ cd .ssh
[jerry@asus ~/.ssh]$ ssh-keygen -t ed25519
```
You can change the name of the generated ssh-key. When you are prompted to entering a passphrase, you can directly press enter to generate a ssh-key pair without a passphrase. A pair of ssh-keys will be generated, one private key `LOCAL_SSH_KEY (This will be the name you enter. The default name is id_ed25519)` and one public key `LOCAL_SSH_KEY.pub`.
Change the access mode of `YOUR_SSH_KEY`
```bash
[jerry@asus ~/.ssh]$ chmod 700 LOCAL_SSH_KEY
```

---
## Add public ssh key to the "authorized_keys" file
We can  use the generated ssh-key pairs to connect to the remote servers by setting the authorized public key in the remote server. There are 3 remote servers: `icrlogin1`, `icrhome05`, `kamalan-R750`. We login to ICRR's network through the gateway `icrlogin1` and then login to KAGRA's node `icrhome05` and from there we login to our GPU server `kamalan-r750`. We need to set the authorized publish key on these 3 remote servers.
1. Get the public ssh key:
```bash
[jerry@asus ~]$ cd ~/.ssh
[jerry@asus ~/.ssh] cat ./LOCAL_SSH_KEY.pub
ssh-ed25519 ... jerry@asus
```
The `...` part is the public key. Copy the public key from `ssh-ed25519` all the way to `jerry@asus` (the last part will be your local user name at local computer).
2. Login to `icrlogin1` and go to `~/.ssh` folder:
```bash
[jerry@asus ~]$ ssh cchou@icrlogin1.icrr.u-tokyo.ac.jp
[cchou@icrlogin1 ~]$ cd ~/.ssh
```
3. Create a file called `authorized_keys` under `~/.ssh` and paste the local public ssh key in this file:
```bash
[cchou@icrlogin1 ~/.ssh]$ vim authorized_keys
[cchou@icrlogin1 ~/.ssh]$ cat authorized_keys
ssh-ed25519 ... jerry@asus
```
Here `...` should be the public ssh key `LOCAL_SSH_KEY.pub` in your local computer.
4. Login to `icrhome05` and add the public ssh key to `~/.ssh/authorized_keys:
```bash
[cchou@icrlogin1 ~]$ ssh icrhome05
[cchou@icrhome05 ~]$ cd ~/.ssh
[cchou@icrhome05 ~/.ssh]$ vim authorized_keys
[cchou@icrhome05 ~/.ssh]$ cat authorized_keys
ssh-ed25519 ... jerry@asus
```
5. Login to `kamalan-R750` and add the public ssh key to `~/.ssh/authorized_keys`:
```bash
[cchou@icrhome05 ~]$ ssh cchou@172.16.32.46
cchou@kamalan-R750:~$ cd .ssh
cchou@kamalan-R750:~/.ssh$ vim authhorized_keys
cchou@kamalan-R750:~/.ssh$ cat authhorized_keys
ssh-ed25519 ... jerry@asus
```

---
## Set ssh config
After adding the public ssh key on all the remote servers, we can edit the file `~/.ssh/config` on the local computer to connect to the remote servers using `LOCAL_SSH_KEY`.
Put the following In the file `~/.ssh/config`:
```
Host icrr-login-1
	HostName icrlogin1.icrr.u-tokyo.ac.jp
	User cchou
	IdentityFile %d/.ssh/LOCAL_SSH_KEY
	ForwardAgent yes

Host icrhome05
	ProxyCommand ssh -W %h:%p icrr-login-1
	User cchou
	IdentityFile %d/.ssh/LOCAL_SSH_KEY
	ForwardAgent yes

Host kamalan-r750
	HostName 172.16.32.46
	ProxyCommand ssh -W %h:%p icrhome05
	User cchou
	IdentityFile %d/.ssh/LOCAL_SSH_KEY
```
Please change "cchou" to your user name and "YOUR_SSH_KEY" to the name of the ssh-key.

---
