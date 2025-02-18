# Zabbix Setup with Nginx and Docker Compose

This project sets up a Zabbix server with the web interface using Nginx and PostgreSQL, deployed via Docker Compose.

## Requirements

- Docker
- Docker Compose
- Certbot (for SSL)
- Root access or `sudo` permissions on the server

## Installation Steps

### 1. Clone the Repository

First, clone this repository to your machine:

```bash
git clone https://github.com/liusc45/zabbix-server.git
cd zabbix-server
2. Start the Containers with Docker Compose
Inside the repository directory, run the following command to start the Docker services (Zabbix, PostgreSQL, Nginx, etc.):

docker-compose up -d
This command will download the necessary images and start the containers in the background.

3. Nginx Configuration
Nginx is used to serve the Zabbix web interface. To configure and enable Nginx, follow these steps:

a. Edit the Nginx Configuration

Access the Nginx configuration file for Zabbix:

sudo nano /etc/nginx/sites-available/zabbix
Next, add the following content to set up the reverse proxy and enable SSL:

server {
    # Listen for requests on your domain/IP address.
    server_name babbix.poecdn.cloud;

    location / {
        # Proxy all requests to web running on port 8080
        proxy_pass http://localhost:8080;
        
        # Pass on information about the requests to the proxied service using headers
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/babbix.poecdn.cloud/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/babbix.poecdn.cloud/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = babbix.poecdn.cloud) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    server_name babbix.poecdn.cloud;
    listen 80;
    return 404; # managed by Certbot
}
b. Enable the Site in Nginx

After editing the Nginx configuration, create a symbolic link to enable the site:

sudo ln -s /etc/nginx/sites-available/zabbix /etc/nginx/sites-enabled/zabbix
c. Test the Nginx Configuration

Check if the Nginx configuration is correct with:

sudo nginx -t
d. Restart Nginx

To apply the changes, restart the Nginx service:

sudo systemctl restart nginx
4. SSL Configuration with Certbot (Optional)
If you want to enable HTTPS for the Zabbix web interface, use Certbot to configure the SSL certificate. Run the following command:

sudo certbot --nginx -d babbix.poecdn.cloud
This command will automatically configure SSL for the specified domain.

5. Access Zabbix
Once the containers are running, you can access the Zabbix web interface:

HTTP: http://localhost:8080
HTTPS: https://babbix.poecdn.cloud (if SSL has been configured)
6. Default Zabbix Login
To log in to the Zabbix web interface, use the following credentials:

USER: Admin
PASSWORD: zabbix
7. Stop the Containers
To stop the containers at any time, run:

docker-compose down