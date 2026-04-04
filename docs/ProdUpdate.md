# Updating Production

This assumes you already have a fully set up Digital Ocean Droplet as per the [Digital Ocean Setup](DOSetup.md) page. 

You should also get these from the developers: 

- `saln-server/.env`
- private `saln_ssh` and public `saln_ssh.pub` SSH key pair from the developers. Save these in `~/.ssh`.

## Pulling the repository
Using git, we will clone the repository code. 

By convention, the `PRODUCTION` branch of the repository shall contain the current version to be used for production.
Also by convention, the source code shall live in the `/var/www` directory.
```bash
cd /var/www
git clone https://github.com/SHIROKAMIQQ/SALN-App.git
cd SALN-App
git checkout PRODUCTION
```
Now, you should have `/var/www/SALN-App` directory.

## Source Code Configuration for PRODUCTION

In `SALN-App/saln-client/src/api/config.js`, we want to edit the `API_BASE_URL` that the client side will use to communicate with the Laravel server-side.
```js
// FOR PROD:
export const API_BASE_URL = "https://lamig-saln.online/api";
```

In `SALN-App/saln-server/.env`, update `APP_URL`, `DB_PASSWORD`, and the `MAIL_` variables. There are comments in the `.env` file that show the correct values for these for the PRODUCTION server.

## Serving Client-Side Code

### Installing Node Dependencies
Since we already have `npm` installed on the virtual machine, we could simply install the client-side's Node dependencies:
```bash
cd /var/www/SALN-App/saln-client
npm install
```

### Building Client's Static Code
Normally in a development environment, we simply run `npm run dev`. This is just for the dev environment since browsers need to serve static JS files.

So instead, to get the static js files of the current MarkoJS code of `saln-client`, we run:
```bash
npm run build
```
The output will be in `saln-client/dist`.

### Process Manager
For this setup, we have the MarkoJS client served by a Node server. This is usually done by `npm run build` then `node dist/index.mjs`.
But, this disconnects once the terminal gets terminated. 
Process manager 2 (`pm2`) allows us to keep this process alive.
```bash
pm2 start dist/index.mjs --name saln-client --max-memory-restart 200M
pm2 save
pm2 startup
```

Additionally, you have these tools for checking logs and active processes from pm2: `pm2 list`, `pm2 logs saln-client`, `pm2 status`.

## Serving Server-Side Code

### Installing PHP dependencies
We should already have `composer` installed. So all that's left to do is install the PHP dependencies that the server-side code uses.
```bash
cd /var/www/SALN-App/saln-server
composer install
```

### In Case of Server-Side Code Changes (Optional)

In the event that the new PRODUCTION code has changes to the serverside code (like logic or configuration), you might want to run these:
```bash
php artisan config:clear
php artisan cache:clear
```

In the event that you have changes to the database schema (`database/migrations`) or just want to completely restart the database:
```bash
php artisan migrate:fresh
```

### MySQL Server
Though the entire MySQL should already be setup from the [Digital Ocean Setup](DOSetup.md) page, you might want to keep note of these commands to do some sanity checks for the MySQL server:

- To start MySQL Server: `sudo service mysql start`
- To check status: `sudo service mysql status` 
- To login locally: `sudo mysql -u root -p` 
- To stop MySQL Server: `sudo service mysql stop`

## NGINX Permissions

We need to give nginx (www-data group) permissions to read/traverse through SALN-App.
We also want to give Laravel write permissions for the saln-server's logs and cache.
```bash
sudo chown -R www-data:www-data /var/www/SALN-App

# Directories: readable + traversable
sudo find /var/www/SALN-App -type d -exec chmod 755 {} \;
# Files: readable
sudo find /var/www/SALN-App -type f -exec chmod 644 {} \;

# .env is readable only by www-data
sudo chmod 600 /var/www/SALN-App/saln-server/.env

# Give Laravel write access to storage (logs) and cache
sudo chmod -R 775 /var/www/SALN-App/saln-server/storage
sudo chmod -R 775 /var/www/SALN-App/saln-server/bootstrap/cache
```

**ALWAYS** run `sudo systemctl reload nginx` when making changes to the nginx config files. 