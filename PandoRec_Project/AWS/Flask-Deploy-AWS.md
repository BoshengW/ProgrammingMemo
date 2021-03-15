# Flask + Angular Deployment in AWS (EC2)
#### Step 1. Prerequisite 
1. Generate a AWS EC2 instance
2. install Nginx web server 
3. install gunicorn for Gateway services


#### Step 2. Create Gunicorn as Service
In Production Env, we prefer to running the web-application at backstage. So we need to pack flask application as a **socket(.sock)** and create a gunicorn.service
```
# 1. In AWS EC2, move to /etc/systemd/system folder, which used for managing backstage process

cd /etc/systemd/system
touch app.service
vim app.service

# 2. Edit app.service
========== In app.service (INI format) ==================
[Unit]
Description=PandoRec Recommendation System application
After=network.target

[Service]
User=ubuntu 
Group=www-data

WorkingDirectory=<location of your app.py>
Environment="PATH=<virtual env folder>/bin" // used for config virtual env

// packing flask application into a socket
ExecStart=/home/ubuntu/PandoRec_MovieRecom/Flask/restapi/pandorec/bin/gunicorn --workers 3 --bind unix:app.sock -m 007 app:app

[Install]
WantedBy=multi-user.target


# 3. reload new backstage service
sudo systemctl daemon-reload
sudo service gunicorn3 start
sudo service gunicorn3 status // check running status

# 4. if success, in your working directory will create a app.sock file
```


#### Step 3. Packing Angular application
Generate **dist** folder for deployment angular code.
**Note**: In this part, remember set aot & build optimizer on, since this will help you check if your source code have any potential issue for production environment.
```
# move to your Angular application folder

ng build --prod //make sure aot is on
```

#### Step 4. Setup Nginx Web Server
**What is Web Server**: 
- used for collecting requests from client-side, and allocating requests into different server based on load-balancer. Finally collecting data from backserver and send back to client side.

```
# 1. move to /etc/nginx/sites-enabled folder
cd /etc/nginx/sites-enabled //this folder used for allocating port for each application

# 2. register Angular application
mv <Angular dist folder> /var/www/<your folder name>

================== Nginx sites-enabled/default(or create a new files)==========
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        index index.html index.htm index.nginx-debian.html;
        server_name _;
        location / {
                root /var/www/angular-deploy/dist/movie-recom;
                try_files $uri $uri/ =404;
        }
}

# 3. register Flask application socket
================== Nginx sites-enabled/default(or create a new files)==========
server {
        listen 8080;
        server_name <Ipv4 IP address of EC2>;
        location / {
                include proxy_params;
                proxy_pass http://unix:/home/ubuntu/PandoRec_MovieRecom/Flask/restapi/app.sock;
        }
}

# 4. start nginx process
sudo systemctl daemon-reload
sudo systemctl reload nginx
sudo systemctl restart nginx
sudo systemctl status nginx // check running status
```

#### Finally
You can check <IP address>:<Port number> to see if your application running well. Enjoy!!!




