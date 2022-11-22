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
### Nginx config with php too
server {
        listen 443 default_server;
        listen [::]:443 default_server;

        root /var/www/html;

        index index.html index.php index.htm index.nginx-debian.html;

        server_name  ocr.eca-assurances.com;
        ssl                  on;
        ssl_certificate      /home/telcrm.pem.cer;
        ssl_certificate_key  /home/telcrm-key.pkey;

        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_send_timeout 300;

        location / {
                root /var/www/ai-doc/client/dist;
                proxy_read_timeout 120s;

                try_files $uri $uri/ /index.html;
        }

        location /maarch-file {
                location ~ \.php$ {
                       include snippets/fastcgi-php.conf;
                       include fastcgi_params;
                       fastcgi_pass unix:/run/php/php7.3-fpm.sock;
                       fastcgi_send_timeout 300;
                       fastcgi_read_timeout 300;
                }
                root /var/www/ai-doc;
                try_files $uri $uri/ =404;
        }

        location /api {
                include uwsgi_params;
                client_max_body_size 100M;
                client_body_timeout 300s;
                uwsgi_pass unix:/var/www/ai-doc/api/app.sock;
                uwsgi_read_timeout 300s;

                #proxy_buffers 8 32k;
                #proxy_buffer_size 64k;
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
