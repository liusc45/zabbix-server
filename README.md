# Zabbix Server Docker Installation Guide

## Prerequisites
- Docker and Docker Compose installed
- A domain name (if using SSL)
- Root or sudo access to your server

## Installation Steps

### 1. Clone the Repository
```bash
git clone https://github.com/liusc45/zabbix-server.git
cd zabbix-server
```

### 2. Start Docker Services
Launch Zabbix, PostgreSQL, and Nginx containers:
```bash
docker-compose up -d
```

### 3. Nginx Configuration

#### a. Configure Nginx
Create and edit the Nginx configuration file:
```bash
sudo nano /etc/nginx/sites-available/zabbix
```

Add the following configuration:
```nginx
server {
    server_name babbix.poecdn.cloud;
    
    location / {
        proxy_pass http://localhost:8080;
        
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/babbix.poecdn.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/babbix.poecdn.cloud/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    if ($host = babbix.poecdn.cloud) {
        return 301 https://$host$request_uri;
    }
    
    server_name babbix.poecdn.cloud;
    listen 80;
    return 404;
}
```

#### b. Enable the Site
Create a symbolic link:
```bash
sudo ln -s /etc/nginx/sites-available/zabbix /etc/nginx/sites-enabled/zabbix
```

#### c. Verify Configuration
Test Nginx configuration:
```bash
sudo nginx -t
```

#### d. Apply Changes
Restart Nginx service:
```bash
sudo systemctl restart nginx
```

### 4. SSL Configuration (Optional)
Configure HTTPS using Certbot:
```bash
sudo certbot --nginx -d babbix.poecdn.cloud
```

### 5. Access Zabbix Interface
- HTTP: `http://localhost:8080`
- HTTPS: `https://babbix.poecdn.cloud` (if SSL configured)

### 6. Default Credentials
Access the Zabbix web interface with:
- Username: `Admin`
- Password: `zabbix`

### 7. Container Management
Stop all containers:
```bash
docker-compose down
```

## Security Notes
- Change the default admin password after first login
- Configure firewall rules to restrict access to necessary ports
- Regularly update containers for security patches