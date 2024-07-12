## Step-by-Step Guide to Creating and Deploying a Django Project on Ubuntu Using uWSGI and Nginx
### 1. Setting Up Your Environment
#### 1.1 Update and Upgrade Your System
```bash
sudo apt update
sudo apt upgrade
```
#### 1.2 Install Python and Virtual Environment
```bash
sudo apt install python3-pip python3-dev libpq-dev python3-venv
```
#### 1.3 Install Nginx
```bash
sudo apt install nginx
```
### 2. Creating Your Django Project
#### 2.1 Create a Directory for Your Project
```bash
mkdir ~/myproject
cd ~/myproject
```
#### 2.2 Set Up a Virtual Environment
```bash
python3 -m venv myenv
source myenv/bin/activate
```
### 3. Configuring Your Django Project
#### 3.1 Edit `settings.py`
Open `myproject/settings.py` and update the following:
* __Allowed Hosts:__
```bash
ALLOWED_HOSTS = ['your_server_ip_or_domain']
```
* __Static Files:__
```bash
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
```
#### 3.2 Collect Static Files
```bash
python manage.py collectstatic
```
### 4. Setting Up uWSGI
#### 4.1 Install uWSGI
```bash
pip install uwsgi
```
#### 4.2 Create a uWSGI Configuration File
Create a file named `myproject.ini` in your project directory:
```bash
[uwsgi]
chdir = /home/yourusername/myproject
module = myproject.wsgi:application

master = true
processes = 5

socket = /home/yourusername/myproject/myproject.sock
chmod-socket = 660
vacuum = true

die-on-term = true
```
### 5. Setting Up Nginx
#### 5.1 Create a Nginx Configuration File
Create a new file in /etc/nginx/sites-available/ named `myproject`:
```bash
server {
    listen 80;
    server_name your_server_ip_or_domain;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/yourusername/myproject/myproject.sock;
    }

    location /static/ {
        alias /home/yourusername/myproject/static/;
    }
}
```
#### 5.2 Enable the Nginx Configuration
```bash
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```
### 6. Creating a Systemd Service for uWSGI
#### 6.1 Create a uWSGI Service File
Create a new file in `/etc/systemd/system/` named `myproject.service`:
```bash
[Unit]
Description=uWSGI instance to serve myproject
After=network.target

[Service]
User=yourusername
Group=www-data
WorkingDirectory=/home/yourusername/myproject
Environment="PATH=/home/yourusername/myproject/myenv/bin"
ExecStart=/home/yourusername/myproject/myenv/bin/uwsgi --ini myproject.ini

[Install]
WantedBy=multi-user.target
```
#### 6.2 Start and Enable the uWSGI Service
```bash
sudo systemctl start myproject
sudo systemctl enable myproject
```
### 7. Adjust Firewall Settings (optional)
#### 7.1 Allow Nginx Full Access
```bash
sudo ufw allow 'Nginx Full'
```
### 8. Check Everything
#### 8.1 Check Nginx Status
```bash
sudo systemctl status nginx
```
#### 8.2 Check uWSGI Service Status
```bash
sudo systemctl status myproject
```
### 9. Enjoy Your Deployed Django App
Your Django app should now be accessible via your server's IP address or domain name. Ensure everything is running smoothly by visiting your app in the browser.

Made with 🧡 by Ashomidi.
