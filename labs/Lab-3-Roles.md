# Lab 3 Ansible Roles
In this lab you will create your first role. You will do this by refactoring the playbook you created in Lab 2.

## Step 1: Create the Role Scaffold
- Make a directory for this lab, and a subdirectory for the roles:

```bash
mkdir lab-3
cd lab-3
mkdir roles
cd roles
```

- Then use ansible-galaxy to create the role scaffold

```bash
ansible-galaxy role init nodejs-express-sample
```
## Step 2: Create the tasks for your role
- Edit the ``main.yml`` file under ``roles\nodejs-express-sample\tasks``

- Copy the tasks from your lab-2 playbook and paste them into the ``main.yml`` file. 
    - You will need to remove two indent steps per line to format this correctly.
    - In VSCode you can do this by selecting all the tasks, and typing ``SHIFT+TAB`` twice. 
    - ensure you don't change the relative indenting or the file will not be in valid YAML format.

## Step3: Copy Variables into place
- In our lab-2 playbook we had just a single variable, ``node_apps_location``. Copy this variable and its value into the ``vars/main.yml`` file.

## Step 4: Copy The task files into place
- The playbook required a directory ``app``, with the javascript server file (``app.js``), and the ``package.json`` configuration file. 
- Copy the whole ``app`` directory into the ``files`` directory of the role.

## Step 5: Create a playbook to call this role
- Create a new file in your ``lab-3`` directory caled ``nodejs-role-playbook.yml``.
- State the ``hosts`` and ``become`` attributes as you would previously

```yml
---
- hosts: web
  become: yes
```
- Then you just need to reference the role!

```yml
---
- hosts: web
  become: yes

  roles:
    - nodejs-express-sample
```

## Step 6: Run the Playbook
- The only way you can run a role is in a playbook (which provides information about the hosts)
- You can copy your ``inventory`` file and ``ansible.cfg`` files into the lab-3 directory and run the playbook without a ``-i`` option, or you can just reference the inventory as follows:

``bash
ansible-playbook nodejs-role-playbook.yml -i ../inventory
```

- If all goes to plan, you should see something like this printed out:

```bash
$ ansible-playbook nodejs-role-playbook.yml -i ../inventory 

PLAY [web] *******************************************************************************

TASK [Gathering Facts] *******************************************************************
ok: [13.42.14.38]

TASK [nodejs-express-sample : Setup the yum repository for Nodejs] ***********************
changed: [13.42.14.38]

TASK [nodejs-express-sample : run the setup file] ****************************************
changed: [13.42.14.38]

TASK [nodejs-express-sample : Install Nodejs] ********************************************
changed: [13.42.14.38]

TASK [nodejs-express-sample : Install Forever (to run our Node.js app).] *****************
changed: [13.42.14.38]

TASK [nodejs-express-sample : Ensure Node.js app folder exists.] *************************
changed: [13.42.14.38]

TASK [nodejs-express-sample : Copy example Node.js app to server.] ***********************
changed: [13.42.14.38]

TASK [nodejs-express-sample : Install app dependencies defined in package.json.] *********
changed: [13.42.14.38]

TASK [nodejs-express-sample : Check list of running Node.js apps.] ***********************
ok: [13.42.14.38]

TASK [nodejs-express-sample : Start example Node.js app.] ********************************
changed: [13.42.14.38]

PLAY RECAP *******************************************************************************
13.42.14.38                : ok=10   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- That's it! A NodeJS Express website setup with only 4 lines of code!

## Step 7: Verify your install
- In any web browser type in ``http://<WEB-SERVER-IP>`` and you should once again be met with the heart-warming text ``Hello World!``!