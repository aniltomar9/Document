
## Deploying MERN Stack Project on Hostinger VPS




- Preparing the VPS Environment
- Setting Up the MongoDB Database
- Deploying the Express and Node.js Backend
- Deploying the React Frontends
- Configuring Nginx as a Reverse Proxy
- Setting Up SSL Certificates
### 1. Preparing the VPS Environment


Log in to Your VPS in Terminal 

```bash
 ssh root@your_vps_ip
```

Update and Upgrade Your System

```bash
  sudo apt update
```
```bash
  sudo apt upgrade -y
```

Install Node.js and npm ( if not pre-installed)

```bash
  curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
```
```bash
  sudo apt-get install -y nodejs
```

 Install NVM (Optional)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
```

Source NVM script to add it to the current shell session


```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

 Install Node.js using NVM

```bash
nvm install 20
```
```bash
nvm use 20
```
```bash
nvm alias default 20
```

 Verify installation
```bash
node -v
```
```bash
npm -v
```



Install Git 
```bash
  sudo apt install -y git
```


###  2. Setting Up the MongoDB Database

If you want to setup MongoDB on VPS Follow this Guide: [click here](https://github.com/aniltomar9/Document/blob/main/MongoDB_Setup_on_VPS.md)

### 3. Deploying the Express and Node.js Backend

Clone Your Backend Repository

```bash
 mkdir /var/www
```

```bash
 cd /var/www
```
```bash
 git clone https://github.com/yourusername/your-repo.git
```
```bash
 cd your-repo/backend
```

Install Dependencies

```bash
 npm install
```
Create .env file & configure Environment Variables

```bash
 nano .env
```

add environment variables then save and exit (Ctrl + X, then Y and Enter).


Installing pm2 to Start Backend

```bash
 npm install -g pm2
```
```bash
 pm2 start server.js --name project-backend
```
Start Backend on startup
```bash
 pm2 startup
```
```bash
 pm2 save
```
Allowing backend port in firewall 

```bash
 sudo ufw status
```
If firewall is disable then enable it using 
```bash
 sudo ufw enable
```
```bash
 sudo ufw allow 'OpenSSH'
```
```bash
 sudo ufw allow 8000
```
```bash
 sudo ufw allow 3000
```
```bash
 sudo ufw allow 50001
```

### 4. Deploying the React Frontends

Creating Build of React Applications
```bash
 cd path-to-your-first-react-app
```
```bash
 npm install
```
If you have ".env" file in your project

Create .env file and paste the variables
```bash
 nano .env
```
Create build of project
```bash
 npm run build
```

Repeat for the second or mulitiple React app.

Install Nginx

```bash
 sudo apt install -y nginx
```

adding Nginx in firewall

```bash
 sudo ufw status
```
```bash
 sudo ufw allow 'Nginx Full'
```


Configure Nginx for React Frontends


```bash
 nano /etc/nginx/sites-available/default
```

```bash
server {
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;  # Point to your server
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}
server {
    if ($host = yourdomain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name yourdomain.com;
    return 404; # managed by Certbot


}
```
Save and exit (Ctrl + X, then Y and Enter).

Create a similar file for the second or multiple React app.

```bash
 nano /etc/nginx/sites-available/admin.yourdomain
```

```bash
server {
    server_name admin.yourdomain.com;

    location / {
        proxy_pass http://localhost:8000;  # Point to your admin server
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}
server {
    if ($host = admin.yourdomain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name admin.yourdomain.com;
    return 404; # managed by Certbot


}
```

Create symbolic links to enable the sites.

```bash
ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
```

```bash
ln -s /etc/nginx/sites-available/admin.yourdomain /etc/nginx/sites-enabled/
```

Test the Nginx configuration for syntax errors.

```bash
nginx -t
```

```bash
systemctl restart nginx
```

### 5. Configuring Nginx as a Reverse Proxy

Update Backend Nginx Configuration

```bash
nano /etc/nginx/sites-available/api.yourdomain
```
```bash
server {
    if ($host = api.yourdomain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name api.yourdomain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;


}

server {
    listen 443 ssl;
    server_name api.yourdomain.com;
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem; # managed by Certbot


    location / {
        proxy_pass http://localhost:50001;  # Ensure the backend is still running locally on port 50001
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

}

```

Create symbolic links to enable the sites.

```bash
ln -s /etc/nginx/sites-available/api.yourdomain /etc/nginx/sites-enabled/
```

Restart nginx

```bash
systemctl restart nginx
```

### Connect Domain Name with Website

Point all your domain & sub-domain on VPS IP address by adding DNS records in your domain manager 

Now your website will be live on domain name

### 6. Setting Up SSL Certificates 

Install Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
```

Obtain SSL Certificates

```bash
certbot --nginx -d yourdomain.com -d www.yourdomain.com -d yourdomain.com -d api.yourdomain.com
```

Verify Auto-Renewal

```bash
certbot renew --dry-run
```

If you still need help in deployment:

Contact us on email : greatstackdev@gmail.com
