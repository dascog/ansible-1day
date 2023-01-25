# Lab 1: Ansible First Steps
In this lab you will set up your Ansible control node, create an inventory list, and execute some ad-hoc commands.

## Step 1: Log in and set up your control node
- Your instructor will supply you with a username (like ``ansible3``), and an IP address for the control node server. 
- In an SSH shell on your local machine, log in using the command ``ssh <your-username>@<control-node-ip>``. 
- You will be asked for a password, which your instructor will supply.
- In your home directory there is a file ``setup-ssh.sh``. This is a shell script that will setup your account so ssh will work transparently. Run it by typing the following in your home directory:
```bash
./setup.ssh
```
- Confirm ansible is installed on your account:
```bash
ansible --version
```
- 


## Step 2: Create an inventory
- Your instructor will supply you with 3 IP addresses for your inventory
- You can either edit a file to conform to the heredoc below, or run the following command substituting your 3 IP addresses for IP1, IP2 and IP3

```
cat <<-EOF > inventory
[all:vars]
ansible_user=ec2-user

[web]
IP1

[app]
IP2

[db]
IP3
EOF
```
- The ``[all:vars]`` section sets variables for all the inventory. In this case all of our servers have the same user, ``ec2-user``, so we can set it there. We could have different users for each server, and we can also handle different login credentials for each if necessary. 
- Confirm your inventory using the ``ansible-inventory`` command:

```bash
ansible-inventory --graph -i inventory
```
- expected output:

```bash
@all:
  |--@app:
  |  |--IP2
  |--@db:
  |  |--IP3
  |--@ungrouped:
  |--@web:
  |  |--IP1
  ```
  - Try the ``--list`` option instead of ``--graph`` to get a more complete output.

## Step 3: Run your first command
The ``ansible`` installation includes a large number of modules you can use to process actions on your inventory. You can see these using the ``ansible-doc --list`` command.
- If you want to see all modules containing a string you can use ``grep`` to filter - for example ``ansible-doc --list | grep aws``
- if you want to see the documentation for a particular module (say ``ping``), then ``ansible-doc ping``.

Now you can check if your inventory is alive and well. For this we will use the ``ping`` module:

```bash
ansible all -m ping -i inventory
```
- You should see an output for each of your servers. Don't worry about the ``[WARNING]`` message. If your ping has worked each of the servers should return a line that begins with ``<server-IP> | SUCCESS => ``. 

## Step 4: Copy a file  
One of the common tasks for which Ansible can be used is to copy files to the managed nodes. For this we will use the ``copy`` module - remember you can check the documentation using ``ansible-doc copy``.

- First of all, create a file to copy. You can put anything you like in it, or nothing (using ``touch``). For example:

```bash
echo "The number of the count shall be three" > vitalstatistix
```

- Now suppose we just want to copy this to the web nodes, then the ad-hoc command would be:

```bash
ansible web -m copy -a "src=./vitalstatistix dest=/home/ec2-user/vitalstatistix" -i inventory
```

- You should get a response with ``CHANGED`` after the IP address, because this command has changed something on your server. 
- We can confirm that this has worked by using the ``command`` module to run a bash command on the CLI:

```bash
ansible web -m command -a "cat /home/ec2-user/vitalstatistix" -i inventory
```

- Now try to run your copy command again. What happens? Is the outcome the same as when you ran it the first time? Why not?

## Step 5: Set up a configuration file
Having to continually append ``-i inventory`` can be a little tedious. We can set up a configuration file in the same directory you are working that will resolve this:

- In the same directory as your inventory file run the following command

```bash
cat <<-EOF > ansible.cfg
[defaults]
inventory = inventory
interpreter_python = /usr/bin/python
EOF
```
- Now you no longer need to include the ``-i`` argument
- The second line removes the annoying warnings!
## Step 5: Install some software
Software installation requires root privileges on most systems. Although we log into our managed nodes using the user ``ec2-user``, this user is on the ``sudoers`` list, which means it can receive elevated privileges. 
On the command line you normally achieve elevated privileges using ``sudo``; in Ansible, this is done using the ``--become`` option of the ``yum`` module. 

- Install the vital ``cowsay`` application using the following command

```bash
ansible web -m yum -a "name=cowsay state=latest" --become
```
- Once again your should see a ``CHANGED`` message because you have changed your server.
- That message is enough to say it has successfully installed, but we can also confirm it by trying it out:

```bash
ansible web -m command -a "cowsay 'you are all individuals'" 
```

- If we decide we no longer need this software, then it can be removed by just changing the state:

```bash
ansible web -m yum -a "name=cowsay state=absent" --become
```

- The ``cowsay`` command above will now fail.



