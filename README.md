## flask-nginx
### Uwsgi config
[uwsgi]\
module = wsgi:app\
master = true\
processes = 5\
socket = app.sock\
chmod-socket = 660\
vacuum = true\
die-on-term = true\
post-buffering = 8192\
logto= /var/log/uwsgi/%n.log

### Nginx config

server {                                                                                             
        listen 80 default_server;                                                                    
        listen [::]:80 default_server;                                                              
                                                                                                                                          
        root /var/www/html;                                                                                                                 
        index index.html index.htm index.nginx-debian.html;                                                                                                                                      
        server_name _;                                                                                                                                                                  
        location / {                                                                                
                include uwsgi_params;                                                                  
                client_max_body_size 100M;                                                              
                uwsgi_pass unix:/var/www/app.sock;                                                                            
        }                                                                                                                                                                                                                                                      
}                                                                                                      


### App service config

Add to  `/etc/systemd/system/app.service`\
[Unit]\
Description=uWSGI instance to serve flask project\
After=network.target\
[Service]\
User=root\
Group=www-data\
WorkingDirectory=<project_location>\
Environment="PATH=<environment_location>" # <project_location>/.env/bin\
ExecStart=<uwsgi_ocation> --ini <configuration_file>    #/var/www/.env/bin/uwsgi --ini app.ini\
[Install]\
WantedBy=multi-user.target
