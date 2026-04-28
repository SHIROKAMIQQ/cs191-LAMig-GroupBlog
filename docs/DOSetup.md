# Digital Ocean VM Setup

This page would be for setting up domains, installing dependencies, and setting up the Digital Ocean droplet. 

This assumes the use of:

- Digital Ocean Droplet (Ubuntu 24)
- Netim (this is used for getting a domain)
- Mailgun (this is used for sending emails)

You should also get these from the developers: 

- `saln-server/.env`
- private `saln_ssh` and public `saln_ssh.pub` SSH key pair from the developers. Save these in `~/.ssh`.

**NOTE:** Instructions for setting up the production repository will be in [Updating Production](ProdUpdate.md)

## Digital Ocean Droplet Access

There are two ways of accessing the Digital Ocean droplet.

1. You use the console on the Digital Ocean dashboard.

2. On your terminal, you run the command `ssh -i ~/.ssh/saln_ssh root@178.128.117.229`. To simplify this, you could run `sudo nano ~/.ssh/config` and then put:
```bash
Host saln-do
	Hostname 178.128.117.229
	User root
	IdentityFile ~/.ssh/saln_ssh
```
Then run `ssh -A saln-do`.

## Netim Domain

From Netim, the development team got the domain `lamig-saln.online`. <span style="color: red;">This domain will be valid up until February 9, 2027</span>, but could be renewed.

We will also assign subdomain `mg.lamig-saln.online` for Mailgun's mailing.

## Mailgun Mailing Service

For mailgun, we used the sending domain `mg.lamig-saln.online`. Then, we copied its DNS Records given by Mailgun into the DNS records of `lamig-saln.online` over on Netim. The sending domain is now setup.

Since SMTP is blocked on Digital Ocean, we will be using the Mailgun API. The API key was generated on the Mailgun website and is currently being used in the `saln-server/.env` file as `MAILGUN_SECRET`. The option to generate an new API key could be seen on the Mailgun dashboard. 

In `saln-server/.env`, we now use these fields for mailing:
```bash
MAIL_MAILER=mailgun
MAIL_FROM_ADDRESS=saln_app@mg.lamig-saln.online
MAIL_FROM_NAME="${APP_NAME}"
MAILGUN_DOMAIN=mg.lamig-saln.online
MAILGUN_SECRET=<API-KEY>
MAILGUN_ENDPOINT=api.mailgun.net
```

## Firewall Setup

There are two layers to the Firewall.

On the *Networking* tab of the Digital Ocean Droplet's dashboard, we setup limited the open inbound ports:

| Type  | Protocol | Port Range | Sources |
|:-----:|:--------:|:----------:|:-------:|
| SSH   | TCP      | 22         | All IPv4, All IPv6 |
| HTTP  | TCP      | 80         | All IPv4, All IPv6 |
| HTTPS | TCP      | 443        | All IPv4, All IPv6 |

We also setup the outbound ports:

| Type  | Protocol | Port Range | Sources |
|:-----:|:--------:|:----------:|:-------:|
| ICMP | ICMP      |          | All IPv4, All IPv6 |
| All TCP | TCP      | All ports         | All IPv4, All IPv6 |
| All UDP | UDP      | All ports        | All IPv4, All IPv6 |

Then, on the terminal, we also run this code block to setup the firewall on the virtual machine:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

With this setup, our virtual machine should only take in SSH, HTTP, and HTTPS messages on ports 22, 80, and 443, respectively.

## Installing Git
Install git and then clone the code repository. 
```bash
sudo apt install git -y
```

## Installing npm

To install npm, the package manager of Node.js, get it here: [https://nodejs.org/en/download](https://nodejs.org/en/download)
```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 24

# Verify the Node.js version:
node -v # Should print "v24.13.0".

# Verify npm version:
npm -v # Should print "11.6.2".
```

## Process Manager

For this setup, we have the MarkoJS client served by a Node server. This is usuall done by `npm run build`, then `node dist/index.mjs`.
But, this disconnects once the terminal gets terminated.
Process Manager 2 (`pm2`) allows us to keep this process alive.
```bash
npm install -g pm2
pm2 -v
```

Also, we would want to configure the `swapfile` as so:
```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

## Installing PHP 8.4

### PHP 8.4
To install PHP 8.4:
```bash
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
sudo apt install php8.4 php8.4-cli php8.4-common php8.4-mbstring php8.4-xml php8.4-curl php8.4-mysql php8.4-zip php8.4-gd php8.4-intl php8.4-fpm php8.4-bcmath unzip -y
```

### Composer
We would also need to install Composer, PHP's package manager:
```bash
sudo apt install composer
```

### Laravel
Laravel is the PHP Framework to be used for the backend server side of the web app.
```bash
composer global require laravel/installer
echo 'export PATH="$PATH:$HOME/.config/composer/vendor/bin"' >> ~/.bashrc
source ~/.bashrc
```

## Database Setup

### Install MySQL
To install MySQL Server:
```bash
sudo apt update
sudo apt install mysql-server -y
```

You might want to keep note of these commands to do some sanity checks for the MySQL server:

- To start MySQL Server: `sudo service mysql start` 
- To check status: `sudo service mysql status` 
- To login locally: `sudo mysql -u root -p` 
- To stop MySQL Server: `sudo service mysql stop`

From here, you could do MySQL queries directly to the database. Do note though that the source code uses the `Crypt` library of Laravel to encrypt stored database entries.

### Database Initialization
Create `saln_app_DB` in MySQL and a user for Laravel migrations to use.
```bash
sudo mysql -u root -p
```
```sql
CREATE DATABASE saln_app_DB;
CREATE USER 'saln_user'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON saln_app_DB.* TO 'saln_user'@'localhost';
FLUSH PRIVILEGES;
```

Note that `<password>` must be kept secret. The current `<password>` is found in `saln-server/.env` as the `DB_PASSWORD` field for production. 

### Automatic Backups

Compressed backup files will live in `/var/backups/mysql`.
```bash
mkdir /var/backups/mysql
```

Then, we will make a bash script that creates a compressed backup of the database.
```bash
sudo nano /usr/local/bin/saln_app_db_backup.sh
```
```bash
#!/bin/bash

# CONFIGURATION
DB_USER="<USER>"
DB_PASSWORD="<PASSWORD>"
DB_NAME="saln_app_DB"
BACKUP_DIR="/var/backups/mysql"
DATE=$(date +%m_%d_%Y)
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

# RUN MYSQL DUMP + COMPRESS
mysqldump --no-tablespaces -u "$DB_USER" -p"$DB_PASSWORD" "$DB_NAME" | gzip > "$BACKUP_FILE"

# REMOVE BACKUPS OLDER THAN 30 DAYS
find "$BACKUP_DIR" -type f -name "*.sql.gz" -mtime +30 -delete
```
Then, make the script executable
```bash
sudo chmod +x /usr/local/bin/saln_app_db_backup.sh 
```

## Crontab
This part sets up the cron job for scheduled tasks:

- 5-day old SALN Form cleanup
- Daily database backups

```bash
crontab -e
```

Then put these lines on the bottom of the file
```
* * * * * cd /var/www/SALN-App/saln-server && php artisan schedule:run >> /dev/null 2>&1
0 2 * * * /usr/local/bin/mysql_backup.sh >> /var/log/mysql_backup.log 2>&1
```

## NGINX Setup

### Installing NGINX

Install NGINX
```bash
sudo apt install nginx -y
```

Then, we want nginx to work on port 80, but apache2 might already be in there. So, we disable apache2 since we have no use for it in this app.
```bash
sudo ss -tulpn | grep :80
sudo service apache2 stop
sudo systemctl disable apache2
sudo ss -tulpn | grep :80
sudo service nginx start
sudo service nginx status
```

### NGINX Config

For the client side routing, we do two things. On the browser, we listen on port `80`. 
But, since our MarkoJS client is being served on port `3000` by a Node server, we proxy it over to port `3000`. 
If we see `/api/`, we proxy it over to port `82`, where the Laravel server-side logic is. 
Additionally, this is declares the SSL certificates for port `80`, which allows HTTPS protocol.
```bash
sudo nano /etc/nginx/sites-available/saln-client
```
```nginx
server {
    listen 80;
    server_name lamig-saln.online www.lamig-saln.online;

    # Redirect all HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name lamig-saln.online www.lamig-saln.online;

    ssl_certificate /etc/letsencrypt/live/lamig-saln.online/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lamig-saln.online/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # --------- MARKOJS FRONTEND ---------
    location / {
        proxy_pass http://127.0.0.1:3000/;  # Node MarkoJS server
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # --------- LARAVEL BACKEND ---------
    location /api/ {
        proxy_pass http://127.0.0.1:82;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Optional: serve Laravel static assets correctly
    location /storage/ {
        root /var/www/SALN-App/saln-server/public;
    }

    # Protect hidden files
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

For the server side routing:
```bash
sudo nano /etc/nginx/sites-available/saln-server
```
```nginx
server {
    listen 82;
    server_name _;

    root /var/www/SALN-App/saln-server/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Then, add syslinks to `sites-enabled`:
```bash 
sudo ln -s /etc/nginx/sites-available/saln-client /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/saln-server /etc/nginx/sites-enabled
```

For checking nginx logs: open `/var/log/nginx/error.log` \
To clear nginx logs: `sudo trunctate -s 0 /var/log/nginx/error.log`

**ALWAYS** run `sudo systemctl reload nginx` when making changes to the nginx config files. 

## SSL Certificate
Do notice that the [NGINX Config](#nginx-config) for client has SSL certificates. For the sake of documentation, this is how we got them.

This assumes you have `snap` installed. Try `snap --version`.

```bash
snap install --classic certbot
certbot --nginx
sudo systemctl reload nginx
```