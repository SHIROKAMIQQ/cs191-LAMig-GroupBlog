# Development Installation

This setup is to be done for developers' machines.

This assumes you have:

- A GitHub Account. This is where the repository is hosted online. You may be added as a Collaborator so that you could add changes to the main repository.
- Linux (or WSL if you're on Windows)
- `saln-server/.env` gotten from the developers. This has configurations for database and mailer.

## Git

### Installing Git
Assuming you do not have git on your machine yet, you may want to install it for your terminal. Then, we could also have your machine use your github account.
```bash
sudo apt install git
git config --global user.name "<GITHUB_USERNAME>"
git config --global user.email "<GITHUB_EMAIL>"
```

Then, we recommend you connect to your GitHub account via SSH so that you won't need to enter your password every time. Here is GitHub's documentation for that: 

- [Generating a new SSH Key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

### Cloning Repository
With your GitHub and Git now set up, you could now clone the repository into your machine:
```bash
git clone git@github.com:SHIROKAMIQQ/SALN-App.git
```

**Remember to put in `saln-server/.env`**

**NOTE:** Us developers follow a workflow when creating new updates/features. Initially, you would have your code on a feature branch. Then, once that feature is done, you would make a Pull Request for it to be reviewed by the Lead Developer and then merged with the main branch. 

**NOTE:** The PRODUCTION branch contains the version to be used by the Digital Ocean virtual machine for production.
 
## For Client Side

### Installing npm and Node.js 
This section would need `npm`, the package manager of [Node.js](https://nodejs.org/en/download).
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

### Installing Node.js Dependencies

With npm now installed, we could now install the Node.js dependencies for the client-side.
```bash
cd saln-client
npm install
```

### Running Client
`npm run dev` must be run in the `saln-client` directory.
```bash
cd saln-client
npm run dev
```

You should see a Marko page at (usually) http://localhost:3000

## For Server Side

### Setup MySQL
The database of the app uses MySQL. You may install it via: 
```bash
sudo apt install mysql-server -y
```

- To start MySQL Server: `sudo service mysql start` 
- To check status: `sudo service mysql status` 
- To login locally: `sudo mysql -u root -p` 
- To stop MySQL Server: `sudo service mysql stop`

On mysql, create a database for Laravel to connect to. We will also create a user for Laravel to use. As you could notice, these are also reflected in the `saln-server/.env` file from the developers. 
``` sql
CREATE DATABASE saln_app_DB;
CREATE USER 'saln_user'@'localhost' IDENTIFIED BY 'saln_password';
GRANT ALL PRIVILEGES ON saln_app_DB.* TO 'saln_user'@'localhost';
FLUSH PRIVILEGES;
```


###  Installing Laravel
The app uses the Laravel framework. So, we would need PHP (at least version 8.4) and PHP's package manager, Composer before we install Laravel.
```bash
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
sudo apt install php8.4 php8.4-cli php8.4-common php8.4-mbstring php8.4-xml php8.4-curl php8.4-mysql php8.4-zip php8.4-gd php8.4-intl php8.4-fpm php8.4-bcmath unzip -y
sudo apt install composer -y
```

Now to install the Laravel Framework:
```bash
composer global require laravel/installer
echo 'export PATH="$PATH:$HOME/.config/composer/vendor/bin"' >> ~/.bashrc
source ~/.bashrc
```

We would also need to install other PHP packages needed for the app:
```bash
cd saln-server
composer install
```

To create all the tables from the `database/migrations` folder, or in the case you have database schema changes in `database/migrations`, run:
```bash
php artisan migrate:fresh
```

### Cronjob setup

This part sets up the cron job for scheduled tasks (like deleting 5-day old SALN Forms).

Run this to open up a text editor in your terminal which will edit the cron job file:
```bash
crontab -e
```

Then put on the bottom of the file
```
* * * * * cd /Path/to/SALN-App/saln-server && php artisan schedule:run >> /dev/null 2>&1
```

Then to exit: `Ctrl+O`, `ENTER`, `CtrlX`. To test if the scheduler works, run `php artisan schdule:run`, or more specifically, `php artisan saln:cleanup`.  

### Run Server
`php artisan serve` must be run inside the `saln-server` folder.
```bash
cd saln-server
php artisan serve
```

You should see some Laravel screen at (usually) http://127.0.0.1:8000
