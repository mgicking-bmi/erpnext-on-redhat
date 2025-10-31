# Running ERPNext on RedHat

### You can follow the steps below, or just use the [startup script](./startup.sh), which will get you to the point of setting up the DB connection, and you can resume manual setup from there.

### Install Dependencies
##### Base Dependencies
`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y`
`sudo yum install fail2ban -y`
`sudo dnf install pkgconfig python3.12-devel gcc mariadb-devel -y`

##### Check Python installed
`python3 --version`

##### Install Pip
`sudo dnf install python3.12-pip -y`

##### Install Redis 
`sudo dnf install redis -y`
`sudo systemctl start redis`
`sudo systemctl enable redis`

##### Check Redis installed
`redis-server -v`

##### Install Git
`sudo dnf install git -y`

##### Install MariaDB (Should only need the client, but installing both just to prove it works)
`sudo dnf install mariadb.x86_64 -y`
`sudo yum install mariadb-server -y`

##### Install Node
`sudo dnf module enable nodejs:20 -y`
`sudo dnf install nodejs --setopt=install_weak_deps=False -y`
###### NOTE: It should also be noted that the Node.js installation comes with some weak dependencies, such as docs. To skip installing these optional packages, you can use this command

##### Install Yarn 
`sudo npm install -g yarn`

#### Install Superagent
`sudo npm install superagent`
###### NOTE: Don't install this globally, but inside the Frappe Bench folder

##### Install Supervisor
`python3.12 -m pip install supervisor`

##### Install wkhtmltopdf
`wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox-0.12.6-1.centos8.x86_64.rpm`
`sudo dnf install wkhtmltox-0.12.6-1.centos8.x86_64.rpm -y`

### Install Frappe Framework

##### Install Bench
`pip3 install frappe-bench`

##### Create a bench for working
`cd ~`
`bench init erpnext`
_NOTE: Replace `erpnext` with whatever you want to call your app_

### [Configure Database](./DBSetup.md)

### Create your app
`bench new-site [APP_NAME]`

### Add ERPNext and Payments
`bench switch-to-branch version-15`
`bench get-app payments`
`bench get-app --branch version-15 erpnext`
`bench --site [APP_NAME] install-app erpnext`

### Add your custom app
`bench get-app [APP_URL]`
`bench --site [APP_NAME] install-app [CUSTOM_APP_NAME]`

### Target your app
`bench use [APP_NAME]`

### Start your app
`bench start`


## RESTARTING
You may find yourself running into situations where you need to restart Redis/Bench, such as when installing a new app to your site, and `bench restart` fails. To do this, you can use the following steps to restart bench and redis:
1. Get the PIDs for bench and redis
```
ps aux | grep bench
ps aux | grep redis
```
2. Use `kill` with the PID to kill running processes
```
sudo kill -9 [PID]
```
3. Restart your site
```
bench start
```