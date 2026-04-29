# Updating Production

This assumes you already have a fully set up Digital Ocean Droplet and repository as per the [Digital Ocean Setup](DOSetup.md) page. 

You should also get these from the developers: 

- private `saln_ssh` and public `saln_ssh.pub` SSH key pair from the developers. Save these in `~/.ssh`.

## Pulling the repository
Using git, we can pull updates from the repository. 
```bash
cd /var/www/SALN-App
git pull
```


## Updating Client-Side
In the event that you have new node dependencies: 
```bash
cd /var/www/SALN-App/saln-client
npm install
```

Then always rebuild the client's static code and reinitiate the `pm2` process serving the client.
```bash
npm run build
pm2 del 0
pm2 start dist/index.mjs --name saln-client --max-memory-restart 200M
pm2 save
pm2 startup
```

## Updating Server-Side

### Installing PHP dependencies
In the event that you have new Composer dependencies:
```bash
cd /var/www/SALN-App/saln-server
composer install
```

Then run:
```bash
php artisan config:clear
php artisan cache:clear
```

Some useful commands:

- `php artisan migrate` - migrate database. Data is kept. Used for small table alterations
- `php artisan migrate:fresh` - migrate database. Data is not kept. Used for breaking schema changes.
- `php artisan key:generate` - change the `APP_KEY` in the `.env` file used by `Crypt`.
