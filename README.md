# Tasks:

Implement deployment of 3 tier application, which would run on Ubuntu server 18.04 LTS, would use Nginx, Postgres and Python code in consitent and repeatable way. You would need to automate deployment of:
1. Setup use of 10.0.0.2/18 static IP address, Netmask 255.255.0.0, gateway 10.0.0.1/182.    
2. Install Nginx, configure it to serve static pages and dynamic pages via FCGI (python application)
3. Install PostgreSQL DBMS and create DB, user for DB, set users password.
4. Install simple Python application which would serve "Hello World!" via FCGI.
5. Make sure all your changes are persistent after reboot.

## Network configuration on the target machine:

```
rry@test:~$ cat /etc/netplan/50-cloud-init.yaml 
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        ens160:
            addresses:
            - 10.0.0.2/18
            gateway4: 10.0.0.1
            nameservers:
                addresses:
                - 10.0.0.1
    version: 2
rry@test:~$ sudo netplan apply
```

## Configuration of the taget machine using Ansible playbook:

1. Clone git repository:
```
git clone https://github.com/romryzh/test.git
```
2. Generate SSH key-pair:
```
ssh-keygen -f ~/.ssh/ansible
```
3. Replace ansible_user parameter in hosts inventory file with name of the user on the target machine.
4. Run playbook:
```
rry@ansible:~/test/ansible$ ansible-playbook test_deploy.yaml --ask-sudo-pass -i hosts
SUDO password: 

PLAY ***************************************************************************

TASK [Check for Python] ********************************************************
ok: [host1]

TASK [Install Python] **********************************************************
skipping: [host1]

TASK [Ensure apt cache is up to date] ******************************************
ok: [host1]

TASK [Ensure packages are installed] *******************************************
ok: [host1] => (item=[u'postgresql', u'nginx', u'spawn-fcgi', u'python-flup', u'python-pip', u'python-psycopg2'])

TASK [Install web.py] **********************************************************
ok: [host1]

PLAY ***************************************************************************

TASK [Ensure database is created] **********************************************
ok: [host1]

TASK [Ensure user has access to database] **************************************
ok: [host1]

TASK [Ensure user does not have unnecessary privilege] *************************
ok: [host1]

TASK [Ensure no other user can access the database] ****************************
ok: [host1]

PLAY ***************************************************************************

TASK [Copy the nginx config file] **********************************************
ok: [host1]

TASK [Create symlink] **********************************************************
ok: [host1]

TASK [Copy default html page] **************************************************
ok: [host1]

TASK [Create /var/www/hello-app dir] *******************************************
ok: [host1]

TASK [Clone git repo] **********************************************************
changed: [host1]

TASK [Add exec permmisions to index.py] ****************************************
changed: [host1]

TASK [Copy and add exec permissions to fcgi init-script] ***********************
ok: [host1]

TASK [Add fcgi to autoboot] ****************************************************
ok: [host1]

TASK [Restart fcgi] ************************************************************
changed: [host1]

TASK [Restart nginx] ***********************************************************
changed: [host1]

PLAY RECAP *********************************************************************
host1                      : ok=18   changed=4    unreachable=0    failed=0   
```
5. Open page in browser:
![1](https://raw.githubusercontent.com/romryzh/test/pictures/pictures/img1.png)
![2](https://raw.githubusercontent.com/romryzh/test/pictures/pictures/img2.png)
![3](https://raw.githubusercontent.com/romryzh/test/pictures/pictures/img3.png)
