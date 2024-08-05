- login to droplet

```javascript

ssh root@165.22.212.22
dr0pl3tD!G!T@L0C3@N

```

- update your system

```javascript

apt update
apt upgrade -y

```

- install nodejs and npm using nvm

```javascript

[text](https://github.com/nvm-sh/nvm)

// nvm setup and installation
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc

// nvm help (get all command list)
nvm help

// list all nodejs versions
nvm list-remote

// install the required nodejs version
nvm install v20.10.0

// uninstall the installed nodejs version
nvm uninstall v20.10.0

// display currently activated version of Node
nvm current

// list installed versions, matching a given <version> if provided
nvm ls <version>

// select the nodejs version you want to use
nvm use v20.10.0

// npm will be installed with nodejs it self

```

- clone your backend or frontend project in the specific path

```javascript

// clone the project
git clone https://github.com/your_username/your_repository.git
cd your_repository

// install the dependencies
npm install

```

- setup PM2

```javascript

// install pm2 globally
npm install -g pm2

// start your app with pm2
pm2 start /path/to/your/app.js --name "your_app_name" --watch


```

- setup nginx

```javascript

// install nginx
apt install nginx -y

// create an nginx configuration file for your app
nano /etc/nginx/sites-available/your_app_name

// add following contents

server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
// replace your_domain_or_ip with your domain name or the droplet's IP address
// replace 3000 with the port your Node.js application is running on.

// enable the configuration by creating a symbolic link:
ln -s /etc/nginx/sites-available/your_app_name /etc/nginx/sites-enabled/

// test the Nginx configuration:
nginx -t

// restart Nginx:
systemctl restart nginx

```

- adjust the firewall

```javascript

ufw allow 'Nginx Full'

```

- access Your API

```javascript

Your API should now be accessible at http://your_domain_or_ip.

```

- access Your API

```javascript

Your API should now be accessible at http://your_domain_or_ip.

```

- set Up SSL with Let's Encrypt

```javascript

// install certbot
apt install certbot python3-certbot-nginx -y

// obtain an SSL certificate:
certbot --nginx -d your_domain_or_ip

// follow the prompts to complete the SSL setup.
your API will now be accessible at https://your_domain_or_ip.

```

- This completes the process of hosting your Node.js backend server on a DigitalOcean droplet with Ubuntu using PM2 and exposing the API URL to the public.
