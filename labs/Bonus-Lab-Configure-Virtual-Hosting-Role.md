# Bonus Lab: Configure Virtual Hosting using an Ansible Role
In this lab we will create a vhost role that will configure an Apache Virtual Host on one of our inventory (we'll use the db server so as not to get muddled up with the NodeJS Express Server on web)

## Step 1: Create the vhost Role
- You can use the ``lab-3`` directory for this lab.
- ``cd`` to the ``lab-3\roles`` directory, then use ``ansible-galaxy`` to create the role skeleton

```bash
ansible-galaxy role init vhost
```

- You can see the role skeleton by typing ``tree`` in the ``vhost`` directory.

## Step 2: Create the Tasks
To create a virtual host we need to both install ``httpd``, then use a template which we will put in the templates directory to create the ``vhost.conf.j2`` file.

- Edit the ``tasks/main.yml`` file of the ``vhost`` role.

```yml
---
# tasks file for vhost
- name: install http
  yum:
    name: httpd
    state: latest
- name: start and enable httpd
  service:
    name: httpd
    state: started
    enabled: true
- name: install vhost config file
  template:
    src: vhost.conf.j2
    dest: /etc/httpd/conf.d/vhost.conf
    owner: root
    group: root
    mode: 0644
```

## Step 3: Create Handlers
In this role we need a handler to restart the Apache server whenever a configuration change is made. 

- In the ``handlers/main.yml`` file copy the handler instructions:

```yml
---
# handlers file for vhost

- name: restart_httpd
  service:
     name: httpd
     state: restarted
```

## Step 4: Create a template for the vhost configuration
Note: We have not looked at template files in this course, so this is a bonus example! 

For this role we need a Jinja2 template that Ansible will use to automatically fill in with parameters from the server on which it is installed. The variables ``ansible_fqdn`` and ``ansible_hostname`` are the two server facts that will be used:

- Create the file ``templates/vhost.conf.j2`` and fill it in as follows:

```html
# {{ ansible_managed }}

<VirtualHost *:80>
        ServerAdmin webmaster@{{ ansible_fqdn }}
        ServerName {{ ansible_fqdn }}
        ErrorLog logs/{{ ansible_hostname }}-error.log
        CustomLog logs/{{ansible_hostname }}-common.log common
        DocumentRoot /var/www/vhosts/{{ ansible_hostname }}/

        <Directory /var/www/vhosts/{{ ansible_hostname }}>        
                Options +Indexes +FollowSymlinks +Includes
                Order allow,deny
                Allow from all
        </Directory>
</VirtualHost> 
```
- That's it - the role is now complete!

## Step 6: Create an index.html for the server
As we did with the NodeJS example we can copy a file into place as part of our playbook. This will be done in a ``post_tasks`` section of our playbook. 

The HTML page can be whatever you like, but must be placed in the file ``files/html/index.html`` in your **top level directory** (not your role).

```bash
mkdir -p files/html
cat <<-EOF > files/html/index.html
<h1>This is my new Virtual Host!</h1>
<div>
<span>I have now written a playbook using roles, handlers and templates!</span>
</div>
EOF
```

## Step 7: Create a playbook to use this role
- In this case our playbook will run the role, then run the ``post-tasks`` necessary to copy the index page into place and notify the restart handler. 
- create a playbook file called ``apache-vhost.yml`` in the ``lab-3`` directory:

```yml
---
- name: create apache virtual host
  hosts: db ## Execute on these hosts only
  become: yes ## Perform tasks as root user

  roles:
     - vhost

  post_tasks:
     - name: install contents from local file
       copy:
         src: files/html/
         dest: "/var/www/vhosts/{{ ansible_hostname }}"
       changed_when: true  ## This explicitly tells Ansible that the task above was successful, so a change has occured.
       notify: restart_httpd  ## Execute the handler with the name restart_httpd
```
## Step 8: Verify that the WebServer has started
- In any web browser type in ``http://<WEB-SERVER-IP>`` and this time you should see whatever HTML you put in place.

# Step 9: See how the template was filled out
- Run the ad-hoc command:

```bash
ansible <WEB-SERVER-IP> -a 'cat /etc/httpd/conf.d/vhost.conf' -i ../inventory
```

- You should see a printout of the configuration file your template produced, like:

```html
# Ansible managed

<VirtualHost *:80>
        ServerAdmin webmaster@ip-10-1-0-154.eu-west-2.compute.internal
        ServerName ip-10-1-0-154.eu-west-2.compute.internal
        ErrorLog logs/ip-10-1-0-154-error.log
        CustomLog logs/ip-10-1-0-154-common.log common
        DocumentRoot /var/www/vhosts/ip-10-1-0-154/

        <Directory /var/www/vhosts/ip-10-1-0-154>        
                Options +Indexes +FollowSymlinks +Includes
                Order allow,deny
                Allow from all
        </Directory>
</VirtualHost> 
```

- Notice that the actual hostname allocated by AWS has been used.

