# DEBIAN/UBUNTU BASED 
# WITH PHP 7.2

# install nginx server
- sudo apt-get install nginx

# check nginx version 
- sudo nginx -v

# install php and php libraries
- sudo apt-get install php7.2 php7.2-mysql php7.2-fpm php7.2-xml php7.2-gd php7.2-opcache php7.2-mbstring
- sudo apt install zip unzip php7.2-zip

# check php version
- sudo php -v

# install composer
- sudo curl -sS https://getcomposer.org/installer | php
- mv composer.phar /usr/local/bin/composer
- cd /var/www

# clone or create a project
- composer create-project laravel/laravel application --prefer-dist

# add permissions
- sudo chown -R www-data:www-data application/
- sudo chmod -R 775 application/

# change nginx configuration by copy and pasting this code into nginx.conf file
================================================================
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*.conf;
}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
# 
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
================================================================
- sudo nano /etc/nginx/nginx.conf

# before configuring nginx virtual host make sure to register the server name in host 
- sudo nano /etc/hosts

make sure you write it like this the server name depends on how you name it
===================================
127.0.0.1       app.dev
===================================

# configure nginx virtual host
- cd /etc/nginx/sites-available
- sudo nano application.conf

# and copy this code and paste it in application.conf file

===================================================
- server {
		listen 80 default_server;
		listen [::]:80 default_server;
 
		root /var/www/myproj/public;
		
		index index.php index.html index.htm index.nginx-debian.html;
 
		server_name app.dev;
 
		location / {
			try_files $uri $uri/ /index.php?$query_string;
		}
 
		location ~ \.php$ {
			include snippets/fastcgi-php.conf;
			
			# With php-fpm (or other unix sockets);
			fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
# With php-fpm (or other unix sockets);
		}
}
=====================================================
# create a symlink of sites available conf in sites enabled
- sudo ln -s  /etc/nginx/sites-available/application.conf /etc/nginx/sites-enabled

# run nginx test
- sudo nginx -t

# if there is no error you can now restart the nginx service
- sudo service nginx restart

- sudo chmod -R 777 vendor/

# DONE !

# here is the steps if you want to install ssl on your local machine

create req.conf and copy the code below

========================================================
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
C = PH
ST = NCR
L = Caloocan
O = <Company Name>
OU = RnD
CN = *.<domain-name>.dev
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = *.<domain-name>.dev
DNS.2 = <domain-name>.dev
=========================================================
  
# generate ssl certificate
- sudo openssl genrsa -out {project-domain}.key 3072
- sudo openssl req -new -x509 -key {project-domain}.key -sha256 -out {project-domain}.crt -days 1826 -config {path-to-conf}/req.conf -extensions 'v3_req'

# setup your server inside vhost configuration

/etc/nginx/sites-available/application.conf in this scenario

==============================================================
server {
		listen 80;
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        server_name 4bears-new.dev;

        ssl_certificate /var/www/workspace/4bears/new/ssl/4bears-new.dev.crt;
        ssl_certificate_key /var/www/workspace/4bears/new/ssl/4bears-new.dev.key;
        
        ssl_protocols TLSv1.2 TLSv1.1 TLSv1;
 
		root /var/www/workspace/4bears/new/src/public;
		
		index index.php index.html index.htm index.nginx-debian.html;
 
		location / {
			try_files $uri $uri/ /index.php?$query_string;
		}
 
		location ~ \.php$ {
			include snippets/fastcgi-php.conf;
			
			# With php-fpm (or other unix sockets);
			fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
# With php-fpm (or other unix sockets);
		}
}

===================================================================

# import your ssl certificate 
- sudo certutil -d sql:$HOME/.pki/nssdb -A -t "P,," -n "{cerficate name}" -i {cerficate name}.crt

# DONE!
