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

--

# **How to Host Multiple Node.js Applications on a DigitalOcean Droplet**

Hosting multiple Node.js applications on a single DigitalOcean droplet is a cost-effective way to manage your projects. In this guide, I'll show you how to host three backend servers, each running on different ports (5000, 5001, and 5002), using tools like NVM, PM2, and Nginx.

## **1. Access Your Droplet**

To start, you'll need to SSH into your DigitalOcean droplet. This is done by using the `ssh` command with your droplet’s IP address and root password.

```bash
ssh root@<ip>
<password>
```

## **2. Update Your System**

Once logged in, it’s a good practice to update your system to ensure all packages are up-to-date.

```bash
apt update
apt upgrade -y
```

## **3. Install Node.js and npm Using NVM**

Node Version Manager (NVM) is a great tool for managing multiple Node.js versions. Let’s install NVM and use it to set up Node.js.

1. **Install NVM:**

   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
   source ~/.bashrc
   ```

2. **Verify the Installation and List Available Node.js Versions:**

   ```bash
   nvm help
   nvm list-remote
   ```

3. **Install the Required Node.js Version:**

   ```bash
   nvm install v20.10.0
   ```

4. **Set the Node.js Version to Use:**

   ```bash
   nvm use v20.10.0
   ```

5. **Check the Installed Version:**

   ```bash
   nvm current
   ```

With Node.js installed, npm is also automatically available.

## **4. Clone Your Projects**

Now that Node.js is ready, clone each of your backend projects to the droplet.

### **Application 1 (Port 5000)**

```bash
git clone https://github.com/your_username/app1_repository.git
cd app1_repository
npm install
```

### **Application 2 (Port 5001)**

```bash
git clone https://github.com/your_username/app2_repository.git
cd app2_repository
npm install
```

### **Application 3 (Port 5002)**

```bash
git clone https://github.com/your_username/app3_repository.git
cd app3_repository
npm install
```

## **5. Set Up PM2**

PM2 is a process manager for Node.js applications. It allows your application to run in the background and will automatically restart it if it crashes.

### **Start Application 1 on Port 5000**

```bash
pm2 start /path/to/app1/app.js --name "app1" --watch -- --port 5000
```

### **Start Application 2 on Port 5001**

```bash
pm2 start /path/to/app2/app.js --name "app2" --watch -- --port 5001
```

### **Start Application 3 on Port 5002**

```bash
pm2 start /path/to/app3/app.js --name "app3" --watch -- --port 5002
```

PM2 ensures each app remains running even if the terminal session ends, and the `--watch` flag will monitor your files for changes.

## **6. Set Up Nginx**

Nginx will serve as a reverse proxy to route incoming HTTP requests to your Node.js applications.

1. **Install Nginx:**

   ```bash
   apt install nginx -y
   ```

2. **Create Nginx Configuration Files for Each App:**

### **Configuration for Application 1**

```bash
nano /etc/nginx/sites-available/app1
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name app1.your_domain_or_ip;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### **Configuration for Application 2**

```bash
nano /etc/nginx/sites-available/app2
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name app2.your_domain_or_ip;

    location / {
        proxy_pass http://localhost:5001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### **Configuration for Application 3**

```bash
nano /etc/nginx/sites-available/app3
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name app3.your_domain_or_ip;

    location / {
        proxy_pass http://localhost:5002;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

3. **Enable the Configurations:**

Create symbolic links for each configuration file to enable them.

```bash
ln -s /etc/nginx/sites-available/app1 /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/app2 /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/app3 /etc/nginx/sites-enabled/
```

4. **Test the Configuration:**

```bash
nginx -t
```

5. **Restart Nginx:**

```bash
systemctl restart nginx
```

## **7. Adjust the Firewall**

Ensure your firewall allows traffic through Nginx.

```bash
ufw allow 'Nginx Full'
```

## **8. Access Your APIs**

Each API should now be accessible via your domain name or IP address.

- Application 1: `http://app1.your_domain_or_ip`
- Application 2: `http://app2.your_domain_or_ip`
- Application 3: `http://app3.your_domain_or_ip`

## **9. Set Up SSL with Let's Encrypt**

Finally, secure your APIs with SSL using Let's Encrypt.

1. **Install Certbot:**

   ```bash
   apt install certbot python3-certbot-nginx -y
   ```

2. **Obtain SSL Certificates for Each Application:**

### **SSL for Application 1**

```bash
certbot --nginx -d app1.your_domain_or_ip
```

### **SSL for Application 2**

```bash
certbot --nginx -d app2.your_domain_or_ip
```

### **SSL for Application 3**

```bash
certbot --nginx -d app3.your_domain_or_ip
```

Follow the prompts to complete the SSL setup. Afterward, your APIs will be accessible via HTTPS.

- Application 1: `https://app1.your_domain_or_ip`
- Application 2: `https://app2.your_domain_or_ip`
- Application 3: `https://app3.your_domain_or_ip`

## **Conclusion**

Congratulations! You've successfully hosted multiple Node.js applications on a single DigitalOcean droplet. With PM2 managing your applications and Nginx handling incoming requests, your server is now robust and capable of running multiple apps simultaneously.

Happy coding!
