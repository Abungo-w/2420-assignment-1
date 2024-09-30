# 2420-assignment-1
## Creating an Arch Linux Droplet using `doctl`
### Introduction
This is a tutorial that will teach you step-by-step how to:
- [Create SSH keys on your local machine.](#SSH)
- [Install and configure `doctl`.](#doctl)
- [Configure files with cloud-init](#cloud)
- [Set Up an Arch Linux Droplet using `doctl` command-line tool.](#droplet)
- [Connect to your droplet from your local machine using SSH](#connect)

<a name="SSH"></a>

### Creating the SSH keys on your local machine
SSH is a way to securely connect to another computer over the internet. It lets you control the other computer, run commands, and share files between the two machines[[1]](#refernce).

Creating an SSH key on your local machine will allow you to link your computer to a Droplet.

#### Step 1: Creating the SSH key
1. Open up terminal
2. Use the following command:
```
ssh-keygen -t ed25519 -f ~/.ssh/doctl-key -C "Your name here"
```
**Note:** "Your name here" can be any name you want for the SSH key.
- `ssh-keygen` is the command to create SSH keys[[2]](#refernce).
- `-t` selects the type of algorithm to create the SSH key[[2]](#refernce).
- `ed25519` is the algorithm used to create the SSH key[[3]](#refernce).
- `-f` specifies name of the file in which to store the create SSH key[[2]](#refernce).
- `~` represents the current home directory[[4]](#refernce).
- `-C` provide a comment for the key[[2]](#refernce).

Its going to ask you if you want to enter a passphrase. This is optional, its extra protection for your SSH key, press Enter to skip it.

If you did the steps correctly, it should look like this:
![SSH key](/assets/SSH_key.png)

#### Step 2: Checking the files for SSH key
You can check if the SSH key is created by typing this command in your terminal:
```
ls ~/.ssh
```
- `ls` list directory contents.

If you see the file `doctl-key` and `doctl-key.pub` then you have successfully created a SSH key
![Files in the directory](/assets/SSH_key_check.png)

**Caution:** `doctl-key` is your private key. Do not share it. `doctl-key.pub` is the public key that you can share.

<a name="doctl"></a>

### Install and configure `doctl`
`doctl` allows you to work with the DigitalOcean API using commands in your computer's terminal[[5]](#refernce).

#### Step 1: Installing `doctl`
We will be installing `doctl` in an exsiting Arch Linux droplet with `pacman`.
1. Use the following command:
```
sudo pacman -S doctl
```
`sudo` allows you to execute a command as the superuser[[6]](#refernce).
`pacman` is a package manager[[7]](#refernce).
`-S` to install a package from the remote repository[[7]](#refernce).

Now that you have installed `doctl`, we 

#### Step 2: Creating a new Personal Access Token
1. Login to your DigitalOcean account.
2. Select **API** in the menu on the left side.

![API in the menu](/assets/API.png)

3. Click **Generate New Token**

![generate new token](/assets/new_token.png)

4. Type in a Token Name
5. Select **Full Access**

It should look something like this:
![creating token](/assets/creating_token.png)

6. Click **Generate Token**

#### Step 3: Connecting your Personal Access Token to `doctl`
1. Use the following command:
```
doctl auth init --context <name>
```
**note:** <name> is any name you want for `--context`
- `doctl auth init` allows you to initialize doctl with a token[[9]](#refernce).
- `--context` allows you to add authentication for multiple accounts[[9]](#refernce).
2. Copy and paste your Personal Access Token from DigitalOcean when it ask you to "Enter your access token".

![connecting the token](/assets/doctl_auth.png)

3. Switch to your account by using the following command:
```
doctl auth switch --context <name>
```
**note:** For this command, <name> here is the name you used from the previous command `doctl auth init --context <name>`.
- `switch` allows you to switch between authentication contexts you’ve already created[[9]](#refernce).

![connecting the token](/assets/doctl_switch.png)

You can check if you have done it correctly by using the following command:
```
doctl account get
```
If you see your account then you have succesfully conneted `doctl` to your DigitalOcean account!

#### Step 4: Connecting SSH key to DigitalOcean
Make sure you have created an SSH key already
1.Look for you SSH key in your files by using the following command:
```
cat ~/.ssh/doctl-key.pub
```
- `cat` views content of a file[[11]](#refernce).

It should return your public key.

2. Then use the following command to connect your SSH key to DigitalOcean:
```
doctl compute ssh-key create "<key name>" --public-key "<Your public key here>"
```
**Note:** `<key name>` is a name you want to give your key, while `<Your public key here>` is the public key you got from the previous step.
- `doctl compute ssh-key create` add a new SSH key to your DigitalOcean account[[11]](#refernce).
- `--public-key` is key contents[[11]](#refernce).

If you follow the steps correctly, it should show you this:
![Example](/assets/connecting_ssh_key.png)

You have now sucessfullt connected your SSH key to DigitalOcean!

<a name="cloud"></a>

### Configuring files with cloud-init
Cloud-Init is an open-source tool used for configuring cloud instances during their initialization process[12]. 

We will create a cloud-init file and use it to create a user account, install some packages, add a public ssh key to the authorized key file, and disable root access via ssh.

#### Step 1: Creating the cloud-init file
**Note:** We will be using neovim to create the files. If you don't have neovim install, use the following command:
```
sudo pacman -S neovim
```
1. create the cloud-init file by using the following command:
```
nvim cloud-init.yaml
```
2. Press **i** on your keyboard to edit the file.
3. Copy and paste the following into the file:
```
#cloud-config
users:
  - name: <username>
    primary_group: <group name>
    groups: wheel
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    shell: /bin/bash
    ssh-authorized-keys:
      - <your public key here>
packages:
  - neovim
  - tmux
  - htop
  - wget
  - bash-completion

disable_root: true
```
**Note:** You need to copy and paste your SSH public key where it says `<your public key here>` inside of `ssh-authorized-keys:`
![ssh_authorized_key](/assets/ssh_auth_key.png)

- `#cloud-config` tells the file that it is a cloud-init configuration[[13]](#refernce).
- `users:` adds user to the system[[13]](#refernce).
- `name:` user's login name[[13]](#refernce).
- `primary group:` defines the primary group[[13]](#refernce).
- `Groups:` additional groups to add the user to(optional)[[13]](#refernce).
- `sudo: ['ALL=(ALL) NOPASSWD:ALL']` allow a user unrestricted sudo access[[13]](#refernce).
- `shell: /bin/bash` path of the shell[[13]](#refernce).
- `ssh-authorized-keys:` adds key to user's authorized keys file[[13]](#refernce).
- - `<your public key> ` is the public key you have created.
- `packages:` installs the following packages[[13]](#refernce).
-- `neovim` is a text editor.
-- `tmux` is a terminal multiplexer[[14]](#refernce).
-- `htop` is an interactive process viewer.
-- `wget` retrieves content from web servers.
-- `bash-completion` is a command completion for the Bash shell[[15]](#refernce).
- `disable_root: true` disables root access via ssh[[13]](#refernce).

The file should look like this:
![cloud-init file](/assets/cloud_init_file.png)

4. Press **Esc** on your keyboard then type in `:wq` to save and exit the file.

You have now created a `.yaml` cloud-init configuration file!

<a name="droplet"></a>

### Setting Up an Arch Linux Droplet using `doctl` command-line tool.
Now that we have finished setting up `doctl` and cloud-init file, it's time to create a Droplet using `doctl`.

#### Step 1: Looking for project, Arch Linux image, and SSH key ID
We need to find a id for project, Arch Linux image, and SSH key so we can use it in the next step.
1. Use the following command to look for a list of project:
```
doctl projects list
```
![project list](/assets/project_list.png)

- `doctl projects list` retrieves a list of projects on your DigitalOcean account[[5]](#refernce).

There should be a project listed with its id.

2. Use the following command to look for your Arch Linux image:
```
doctl compute image list-user
```
![Arch Linux image](/assets/linux_image.png)

- `doctl compute image list-user` list images you have uploaded to your account[[5]](#refernce).

There should be a Arch Linux image listed with its id.

3. Use the following command to look for your SSH key id:
```
doctl compute ssh-key list
```
- `doctl compute ssh-key list` list your ssh key id connected to DigitalOcean account.
![ssh key id](/assets/ssh_key_id.png)

#### Step 2: Creating the droplet
1. Use the following commmand to create the droplet:
```
doctl compute droplet create <droplet name> --project-id <project id> --image <arch linux image id> --ssh-keys <ssh key id> --size s-1vcpu-1gb-amd --region sfo3 --user-data-file ~/cloud-init.yaml
```
**Note**: Use any name you want for `<droplet name>`. Use the project, image, SSH key id you have found in step 1  for`<project id>`, `<arch linux image id>`, and `<ssh key id>`:

- `doctl compute droplet create` creates a new Droplet on your account[[5]](#refernce).
- `--project-id` id of the project to assign the Droplet to[[5]](#refernce).
- `--image` an id specifying the image to use to create the Droplet[[5]](#refernce).
- `--ssh-keys` list of SSH key id/fingerprint to embed in the Droplet’s root account[[5]](#refernce).
- `--size` is a slug indicating the Droplet’s number of vCPUs, RAM, and disk size(required)[[5]](#refernce).
- `--region` is a slug specifying the region to create the Droplet in[[5]](#refernce).
- `--user-data-file` specify the path to Cloud-init YAML file to run on the Droplet’s first boot[[5]](#refernce).

It should look like this if you use command correctly:
![create droplet](/assets/create_droplet.png)

You can check if you have created the droplet with the following command:
```
doctl compute droplet list --format Name,PublicIPv4
```
if it shows the droplet name you just created then you have completed all the steps correcly.
![droplet name](/assets/droplet_name.png)

2. Use the following command to initialize your droplet
```
ssh -i .ssh/doctl-key <droplet username>@<droplet Public IPv4 Address>
```
It will ask you "Are you sure you want to continue connecting (yes/no/[fingerprint])?" if you use command correctly.
3. Type yes.

This is what it looks like:
![initial login](/assets/init_login.png)

4. Type `exit` to log out

You have now succesfully created a Arch Linux droplet using `doctl`!

<a name="connect"></a>

### Connecting to your droplet from your local machine using SSH
1. Use the following command to create a `config` file on your local machine:
```
nvim ~/.ssh/config
```
2. Press **i** on keyboard to edit file
3. Copy and paste the following into the `nvim config` file:
```
Host <name>
    HostName <droplet Public IPv4 Address>
    User <username in your cloud-init file>
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/doctl-key
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
```
**Note:** You can put any name for `<name>`

Here's an example:
![ssh config file](/assets/ssh_to_droplet.png)
4. after your done editing, press **Esc** on keyboard, the type `:wq` to save and exit.
5. Log in to your Arch linux droplet on your local machine
6. Use the following command in terminal:
```
ssh <name>
```
**Note:** <name> is the name you put for `Host` in the config file

If you are able to login then you have succesfully connected your droplet to your local machine.

Congrats! You made it to the end of the tutorial!

<a name="refernce"></a>
### Refernce
[1]“SSH Essentials: Working with SSH Servers, Clients, and Keys,” DigitalOcean. https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys

[2]“Ssh-keygen is a tool for creating new authentication key pairs for SSH. This is a tutorial on its use, and covers several special use cases.,” www.ssh.com. https://www.ssh.com/academy/ssh/keygen

[3]“Ed25519 signing — Cryptography 38.0.0.dev1 documentation,” cryptography.io. https://cryptography.io/en/latest/hazmat/primitives/asymmetric/ed25519/

[4]“Tilde Expansion (The GNU C Library),” Gnu.org, 2024. https://www.gnu.org/software/libc/manual/html_node/Tilde-Expansion.html#:~:text=It (accessed Sep. 29, 2024).

[5]https://docs.digitalocean.com/reference/doctl/.

[6]“What is the sudo (su ‘do’) command-line utility? – TechTarget Definition,” Security. https://www.techtarget.com/searchsecurity/definition/sudo-superuser-do#:~:text=Sudo%20is%20a%20command%2Dline

[7]“pacman - ArchWiki,” wiki.archlinux.org. https://wiki.archlinux.org/title/pacman

[8]https://docs.digitalocean.com/products/droplets/how-to/create/.

[9]https://docs.digitalocean.com/reference/doctl/reference/auth/init/.

[10]https://docs.digitalocean.com/reference/doctl/reference/.

[11]GeeksforGeeks, “Cat Command in Linux with Examples - GeeksforGeeks,” GeeksforGeeks, Sep. 14, 2017. https://www.geeksforgeeks.org/cat-command-in-linux-with-examples/

[12]“Introduction to cloud-init - cloud-init 24.2 documentation,” cloudinit.readthedocs.io. https://cloudinit.readthedocs.io/en/latest/explanation/introduction.html

[13]https://cloudinit.readthedocs.io/en/latest/reference/examples.html.

[14]“Home · tmux/tmux Wiki,” GitHub. https://github.com/tmux/tmux/wiki

[15]scop, “GitHub - scop/bash-completion: Programmable completion functions for bash,” GitHub, May 09, 2024. https://github.com/scop/bash-completion.