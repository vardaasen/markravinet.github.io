# Some tips and tricks

As promised, I have written up a basic guide for making the cluster a little easier to use. I've also included a short guide for getting access to some of the scripts  we will be using this week via [github](https://github.com/markravinet/CEES_workshop_2017). 

## Logging on to clusters - the easy way

Accessing clusters multiple times in a day and having to *continually* type out the ``ssh`` command is really boring. 

Why type this and then have to enter a password:

```
ssh username@cod6.hpc.uio
```
When you could just type this and be on the nodes instantly?

```
ssh cod6
```

To do this, I use an ``ssh`` **alias**. It is very simple to set up.

Firstly, navigate to your home directory and look for a hidden directory called ``.ssh`` like so:

```
cd ~
ls .ssh
```

If you don't see anything, you don't have an ``.ssh`` directory, so you will need to make one and move inside it.

```
mkdir .ssh
cd .ssh
```

Now we need to make an ``ssh`` config file. This is very straightforward. Use ``nano`` or a similar command-line text editor to create a file called `config` in the `.ssh` directory. 

The file needs to contain the following:

```
Host cod6
 Hostname cod6.hpc.uio.no
 User username
```
Let's break this down:

* ``Host`` is the name you will assign for the alias. It is **cod6** in this case.
* ``Hostname`` is the actual login address.
* ``User`` is the username.

Once you have set this up with your username and go back to the directory. Then simply type ``ssh cod6`` and you should log in without having to type the full login command.

Note that other commands such as ``scp`` and ``rsync`` also make use of this ``config`` file. For example, you can transfer files like so:

```
scp myfile.txt cod6:/my/directory/path
```
Who said being lazy didn't pay off?

## Logging in without typing a password each time

OK so you now have an easy alias to log in with but what if you also don't want to type a password each time? This is straigthforward too. 

You need to create what is referred to as a **key pair**. You create this on your local machine and keep the private key stored locally. The public key is stored on the remote node you are logging in to.

When you log in, the private and public keys match and you are not prompted for a password. 

To achieve this, do the following:

```
cd ~
ssh-keygen
```

You will prompted to enter a password - you can add one to be secure but I leave mine blank. When complete, you will see message telling you that you have created a key. If you type ``ls .ssh`` you should see it stored as  ``id_rsa`` or something similar. 

Next you need to copy the **public key** to the node you want it on. You do this like so:

```
ssh-copy-id -i ~/.ssh/id_rsa.pub cod6
```

**Note: this assumes you have already set up the login alias in the previous set of steps**

You will have to enter your actual password now but if it worked correctly, you should be able to log in to the node without being prompted for a password. This is also true for ``scp`` and ``rysnc``. Handy.

## Aliases to prevent you from destroying your data

Earlier, I mentioned the tale of an acquaintance destroying months of work by carelessly using `rm`. The easiest way to overcome that problem is to set an **alias** for the rm command. 

To do this, we need to set up a `.bash_login` file on the cod nodes. You should already have one of these but if not, you can create one like so:

```
cd ~
nano .bash_login
```
Then you need to add the following lines:

```
alias rm='rm -i'
```
What this means is that whenever you type `rm`, it will ask you if you are sure you want to delete the files. This is useful, but we can also set up other very useful aliases. Here are two others I like to use:

```
alias ll='ls -lah'
alias grep='grep --colour'
```
The first of these means I can type `ll` to see the contents of a directory in list format with all files and with file size as a human-readable format.

The second means that whenever I run `grep`, I see search hits in colour. Useful!

To make sure these work, save the `.bash_login` file and type the following:

```
source .bash_login
```
There may be times when you need to override the aliases. For example, giving permission for every file when you delete say, 7000 files is going to get boring fast.

To do this, just type the command with a backslash in front, like so: `\rm`

## Getting the scripts we have used

So finally, I am going to show you how to get the scripts used during the week. I have stored the scripts in a [github repository](https://github.com/markravinet/CEES_workshop_2017).

Why did I do this? It makes it much easier for you to get scripts as and when I upload them and it also means I can updated scripts quickly and let you download them with a simple command. 

Let's give it a go. First of all, on the cod nodes go to the workshop directory you created.

```
cd /work/users/msravine/workshop
```
Obviously, you need to substitute your username for my own. Then type the following

```
git clone https://github.com/markravinet/CEES_workshop_2017
```

This will create a directory called `CEES_workshop_2017` full of scripts.

This is a clone of the [github repository](https://github.com/markravinet/CEES_workshop_2017) you see online. 

Every now and then, I will add or update scripts in this repository. All you need to do to sync these updates is to go into your newly created `CEES_workshop_2017` directory and type the following:

```
git pull
```
Easy!







