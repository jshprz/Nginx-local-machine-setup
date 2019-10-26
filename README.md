# DEBIAN/UBUNTU BASED 
# WITH PHP 7^

# install nginx server

- sudo apt-get install nginx

# check nginx version 
- sudo nginx -v

# Install php and php libraries
- sudo apt install software-properties-common
- sudo add-apt-repository ppa:ondrej/php
- sudo apt update
- sudo apt-get install php7.2 php7.2-mysql php7.2-fpm php7.2-xml php7.2-gd php7.2-opcache php7.2-mbstring
- sudo apt install zip unzip php7.2-zip

# Check php version
- sudo php -v

# Install composer and move it into /usr/local/bin/composer.

- curl -sS https://getcomposer.org/installer | php
- mv composer.phar /usr/local/bin/composer
- cd /var/www

# Clone or create a project in this scenario I created laravel application.

- composer create-project laravel/laravel application --prefer-dist

# "application" was the name of the project I created.
# Add permissions to the project my entering those command below

- sudo chown -R www-data:www-data application/
- sudo chmod -R 775 application/

# Change nginx configuration by copy and pasting this code into nginx.conf file

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

# Before configuring nginx virtual host make sure to register the server name in the host which is located in /etc/hosts.

- sudo nano /etc/hosts

# Make sure you write it like this and about the name of server name it any name you want just make sure each of the server name don't have duplicate.

127.0.0.1       app.dev

# Configure nginx virtual host.

- cd /etc/nginx/sites-available
- sudo nano application.conf

# And copy this code and paste it in application.conf file this part would serves as your virtual host.
# Note: you can use default virtual host inside sites-available folder but in my case I create different file.

server {
		listen 80 default_server;
		
		listen [::]:80 default_server;
 
		root /var/www/myproj/public; #The directory of the project where the index file is located
		
		index index.php index.html index.htm index.nginx-debian.html;
 
		server_name app.dev; #Write the name of your server
 
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

# create a symlink of sites available conf in sites enabled.

- sudo ln -s  /etc/nginx/sites-available/application.conf /etc/nginx/sites-enabled

# Run nginx test just to make sure you don't have any error.

- sudo nginx -t

# If there is no error you can now restart the nginx service

- sudo service nginx restart

- sudo chmod -R 777 vendor/

# DONE !

# Here is the steps if you want to install ssl on your local machine. Create req.conf file and copy then paste code below into the file.

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

# Generate ssl certificate.

- cd go to the folder where you want the ssl and certificate and key to be located

- sudo openssl genrsa -out {project-domain}.key 3072

- sudo openssl req -new -x509 -key {project-domain}.key -sha256 -out {project-domain}.crt -days 1826 -config {path-to-conf}/req.conf -extensions 'v3_req'

# If you want your server to have an SSL certificate change the configuration your server inside vhost configuration like this make sure you put correct project root, servername and the path of your ssl certificate and key.

server {
  listen 80;
  server_name application.dev;
  return 301 https://$server_name$request_uri; #will redirect your site to https even though you enter http in the URL
}

server {
 	listen 443 ssl http2;
        listen [::]:443 ssl http2;

        server_name 4bears-old.dev;

        ssl_certificate /var/www/workspace/myproject/ssl/application.dev.crt; #path of the ssl certificate you created
        ssl_certificate_key /var/www/workspace/myproject/ssl/application.dev.key; #path of the ssl key you created
        
        ssl_protocols TLSv1.2 TLSv1.1 TLSv1;

		root /var/www/workspace/myproj/public;
		
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

# import your ssl certificate to enable it.

- cd go to the folder where your certificate and key located
- sudo certutil -d sql:$HOME/.pki/nssdb -A -t "P,," -n "{cerficate name}" -i {cerficate name}.crt

# DONE ENJOY CODING :)
