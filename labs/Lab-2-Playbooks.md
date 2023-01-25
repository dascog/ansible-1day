# Lab 2: Ansible Playbooks
In this lab you will create a playbook to run a NodeJS Express server with a simple Hello World app on one of your inventory groups.

## Step 1: Get the app files onto the control node
- Login to your ansible account on the control node using ``ssh -A ansible<YOUR-NUMBER>@<CONTROL-NODE-IP>``
- create a directory for lab 2:

```bash
mkdir lab2
cd lab2
```
- copy the project files into place and unzip them:

```bash
curl -LJO https://github.com/dascog/ansible-1day/raw/master/labs/Lab-2-files.zip
unzip Lab-2-files.zip
```
- You should see an ``app`` subdirectory with two files: ``app.js`` and ``package.json``. We will not need to modify these files.
  - **``app.js``** is a very simple Express web server. 
  - **``package.json``** is the Node package dependencies for the ``npm install`` command.

## Step 2: Create your playbook file
-  In the same lab2 directory create a file called ``nodejs-playbook.yml`` using your favourite editor (you can use VSCode if you add your server to the SSH servers list)
   -  If you are not familiar with Linux editors your best bet is ``nano``)
-  We will work through the playbook step by step to build up the app

## Step 3: Set up the Play
Remember that a playbook consists of a series of plays at the top level. We do not need to name this play because there is only one in the playbook.

```yml
---
- hosts: web
  become: yes

  vars:
    node_apps_location: /usr/local/opt/node

  tasks:
```

- This playbook will work on the ``web`` group of your inventory (just one server at the moment).
- The play requires elevated privileges so we include the ``become: yes`` option.
- We have also introduced a ``vars:`` option. We will use this later on as a location where the Nodejs app will be stored. It will be referenced using ``{{node_apps_location}}``
- In the rest of the lab we will fill in the tasks list.

## Step 4: Install NodeJS
To run the web server we need to have NodeJS installed on our system, and we will use the [NPM Forever](https://www.npmjs.com/package/forever) package to ensure our webserver runs permanently.

It turns out installing NodeJS on Amazon Linux is harder than you would think!

- First of all download a setup script to configure the ``yum`` installer to work with npm.

```bash
    - name: Setup the yum repository for Nodejs
      get_url:
         url: https://rpm.nodesource.com/setup_16.x
         dest: ./yum-nodejs-setup.sh
         mode: 0755
```

- This uses the ``get_url`` module which is rather like ``curl``. Notice that we set the script to be executable using ``mode: 0755``
- Now run the setup script. There isn't a module to do this other than the default ``command``, which does mean this setup script will be run every time the playbook is run.

```bash
    - name: run the setup file
      command: ./yum-nodejs-setup.sh
 ```

 - Install NodeJS using the ``yum`` module. This should look fairly familiar already!

```bash
    - name: Install Nodejs
      yum:
        name: nodejs
        state: present
```

- Once NodeJS is installed, the ``npm`` module will function as expected. Use this to install the ``forever`` package:

```bash
    - name: Install Forever (to run our Node.js app).
      npm: name=forever global=yes state=present
```
- At this point the playbook should have set up the server and installed all the necessary software. Onto the app!

## Step 5: Copy the app files into place
Here we will use the ``copy`` module to copy files from the control node to the managed node. 

- First step is to make sure that the ``node_apps_location`` exists on the server, and to create it if it doesn't:
  
```bash
    - name: Ensure Node.js app folder exists.
      file: "path={{ node_apps_location }} state=directory"
```
- The ``state=directory`` argument will confirm that this is a directory, and will create both the directory and any other directories needed in the path.

- Now copy the app folder over to the server:

```bash
    - name: Copy example Node.js app to server.
      copy: "src=app dest={{ node_apps_location }}"
```

- At this point try to run your playbook:

```bash
$ ansible-playbook nodejs-playbook.yml --inventory=../inventory
```

- The ``--inventory=../inventory`` is necessary because we don't have an ``ansible.cfg`` file in this folder. You can set one up if you like. 
- If your playbook did not run successfully try to figure out what the problem is from the output. It is most likely that you have a spacing problem in your YAML, which is not at all tolerant to spacing variations. The full file should look like this:
```bash
---
- hosts: web
  become: yes

  vars:
    node_apps_location: /usr/local/opt/node

  tasks:
    - name: Setup the yum repository for Nodejs
      get_url:
         url: https://rpm.nodesource.com/setup_16.x
         dest: ./yum-nodejs-setup.sh
         mode: 0755

    - name: run the setup file
      command: ./yum-nodejs-setup.sh
    
    - name: Install Nodejs
      yum:
        name: nodejs
        state: present

    - name: Install Forever (to run our Node.js app).
      npm: name=forever global=yes state=present

    - name: Ensure Node.js app folder exists.
      file: "path={{ node_apps_location }} state=directory"

    - name: Copy example Node.js app to server.
      copy: "src=app dest={{ node_apps_location }}"
```
- Once your playbook is running you will see an output that concludes with something like (although the numbers will be different):
  
```bash
PLAY RECAP ************************************************
13.40.222.168              : ok=12   changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0 
```
- Login to your webserver using ``ssh -A ec2-user@<WEB-SERVER-IP>``
- You can get the ``<WEB-SERVER-IP>`` either from the playbook output, or from your inventory file.
- Once in the web server, run ``ls /usr/local/opt/node/app`` to see that your app files are now in place.
- Logout of the webserver using ``exit``

## Step 6: Start the Expresss Server
- Use the ``npm`` module to install the app dependencies:
```bash
    - name: Install app dependencies defined in package.json.
      npm: "path={{ node_apps_location }}/app"
```

The ``forever list`` command registers all the running node.js apps. We can use that list to check if our app is running, and only start it if it is not. 

```bash
    - name: Check list of running Node.js apps.
      command: forever list
      register: forever_list
      changed_when: false
```

- Now start Express server using ``forever``: 

```bash
    - name: Start example Node.js app.
      command: "forever start {{ node_apps_location }}/app/app.js"
      when: "forever_list.stdout.find(node_apps_location + '/app/app.js') == -1"
```

- In any web browser type in ``http://<WEB-SERVER-IP>`` and you should be met with the heart-warming text ``Hello World!``!



