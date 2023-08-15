# Deployment Document

The documentation describes the way deploy your application to production environment.

## Prerequisites and Summaries

1. You should prepare the VPS (AWS, Auzre, Google, Hostinger,... ) before going on.
2. Using Nginx webserver

## Conecting to the VPS

To connect your VPS server, you can use your server IP, you can create a root password and enter the server with your IP address and password credentials. But the more secure way is using an SSH key.

### Generating SSH Key

Note: You can use the Terminal / Shell of your cloud provider to connect to the VPS server

1. Launch the Terminal app in your devices.
2. There are two commonly used algorithms for generating authentication keys:

Note: You can choose one of them to generate your ssh key.

Ed25519 – a more modern algorithm with a smaller standard key size of 256 bits. It is just as secure and efficient as an RSA key due to its strong cryptographic properties. The compatibility is lower, but newer operating systems support it.

```bash
ssh-keygen -t ed25519

```

RSA – an SSH RSA key is considered highly secure as it has typically larger key sizes, often 2048 or 4096 bits. It is also more compatible with older operating systems.

```bash
ssh-keygen -t rsa

```

3. Type your path to store the key or press `ENTER` to use default folder
4. Type a passphrase if you want to make security for your ssh or press `ENTER` to use default passphrase (empty string)
5. Confirm your passphrase to finish SSH Keygen.
6. View your ssh key with command (replace your location path):

Ed25519

```bash
cat ~/.ssh/id_ed25519.pub
```

RSA

```bash
cat ~/.ssh/id_rsa.pub
```

`After view your ssh key, you can select and copy the contents.`

or

`Using other command to copy your public SSH Key to your clipboard`

Ed25519

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

RSA

```bash
pbcopy < ~/.ssh/id_rsa.pub
```

7. Paste your SSH Key to hosting service provider dashboard

## Package Installer for Configuration on VPS

### NOTE:

1. If your VPS has been installed Apache Server by default. But you dont need it, so you can following steps to delete Apache Server

2. Using command if you dont want to use sudo previous your command: `sudo su`

3. Always check your current directory before executing the command

### Deleting APACHE Server

```
sudo systemctl stop apache2
```

```
sudo systemctl disable apache2
```

```
sudo apt remove apache2
```

to delete related dependencies:

```
sudo apt autoremove
```

### Cleaning and updating server

```
sudo apt clean all && sudo apt update && sudo apt dist-upgrade
```

```
rm -rf /var/www/html
```

### Installing and Configure Nginx

#### Install Nginx

```
sudo apt install nginx
```

Check Nginx Version

```
sudo nginx -v
```

#### Using and modifying default server configuration

```
vim /etc/nginx/sites-available/default
```

#### You can custom server configuration

##### Delete the default server configuration

```
 rm /etc/nginx/sites-available/default
```

```
 rm /etc/nginx/sites-enabled/default
```

##### First configuration

```
 nano /etc/nginx/sites-available/custom_app
```

```
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    include /etc/nginx/default.d/*.conf;

    # Set request size / default 1M;
    # client_max_body_size 100M;

    location / {
        alias /usr/share/nginx/html/;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
    }

}

```

###### Link server configuration to sites-enabled (important)

```
ln -s /etc/nginx/sites-available/custom_app /etc/nginx/sites-enabled/custom_app

```

###### Validate your nginx configuration

```
sudo nginx -t

```

###### Some command for nginx server

```
systemctl start nginx

```

```
systemctl start nginx

```

```
systemctl reload nginx

```

```
systemctl stop nginx

```

### NODEJS APP Deployment

Note: you can install nodejs and npm without using NVM following by:

#### Installing Nodejs and npm

```
sudo apt install nodejs
sudo apt install npm
```

#### Installing NVM

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

or

```
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

##### Load NVM into the current shell session

```
 source ~/.bashrc
```

```
 nvm -v
```

##### Installing Nodejs and npm by NVM

Change your node version that you want to install

```
 nvm install 18
```

#### Installing PM2

```
 npm i -g pm2
```

#### Installing API SERVER

```
 npm i
```

#### Start api server with PM2

Synctax

```
 pm2 start --name <app_name> <endpoint>
```

##### Example Commands

```
 pm2 start --name app_api app.js
```

```
 pm2 logs
```

```
 pm2 list
```

```
 pm2 restart app_api
```

```
 pm2 delete app_api
```

##### Startup with Ubuntu (optional)

```
 pm2 startup ubuntu
```

#### Let's make some nginx configuration for api

```
 nano /etc/nginx/sites-available/custom_app
```

```
    location /api {
        proxy_pass http://$host:<PORT>;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;
    }
```

Make sure reload nginx after change configuration

```
 systemctl reload nginx
```

### FRONTEND APP Deployment

```
 npm i
```

```
npm run build
```

Right now, we should move this build file into the main web file

```
cp -r build/* /usr/share/nginx/html
```

### Installing and configure Firewall

Note: Make sure allow ssh connect before running command `sudo ufw enable`

```
sudo apt install ufw
```

```
sudo ufw allow ssh
```

```
sudo ufw allow "Nginx Full"
```

```
sudo ufw enable
```

```
sudo ufw status verbose
```

### Adding Domain

1. Make sure that you created your A records on your domain provider website.
2. Change your server_name to your domain
3. Make sure your domain has only 1 A record or 1 AAA record(optional) on your domain provider

```
 nano /etc/nginx/sites-available/custom_app
```

`server_name example.com www.example.com;`

### SSL Certification

```
apt install certbot python3-certbot-nginx
```

Make sure that Nginx Full rule is available

```
ufw status
```

```
certbot --nginx -d example.com -d www.example.com
```

Let’s Encrypt’s certificates are only valid for ninety days. To set a timer to validate automatically:

```
systemctl status certbot.timer
```
