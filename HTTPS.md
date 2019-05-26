### Install Nginx
This provides a guide to adding HTTPS support.
```
sudo apt-get install -y nginx apache2-utils
```

#### Configure Nginx
```
sudo mkdir /etc/nginx/sites-avaiable
sudo /etc/nginx/sites-avaiable/touch kibana
sudo mousepad /etc/nginx/sites-available/kibana &
```

paste this:
```
server {
    listen 80;
 
    server_name 192.168.16.129;
 
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.kibana-user;
 
    location / {
        proxy_pass http://192.168.16.129:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Create basic auth file and user
```
sudo htpasswd -c /etc/nginx/.kibana-user admin
TYPE YOUR PASSWORD
```

Activate the kibana virtual host
```
sudo ln -s /etc/nginx/sites-available/kibana /etc/nginx/sites-enabled/
```

Test the nginx configuration and make sure there is no error
```
sudo nginx -t
```

#### Configure Nginx to run on startup and restart
```
sudo update-rc.d nginx defaults 95 10
sudo -i service nginx restart
```