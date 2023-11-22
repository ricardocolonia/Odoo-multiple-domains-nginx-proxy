# Odoo-multiple-domains-nginx-proxy
Nginx configuration for using multiple domains on odoo

Installing Nginx

sudo apt install nginx

sudo systemctl status nginx
Install the Certbot package

sudo apt install certbot

Next, we will use the command below to create a strong key that will provide a high level of security:

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

Obtaining the SSL certificate

sudo mkdir -p /var/lib/letsencrypt/.well-known
sudo chgrp www-data /var/lib/letsencrypt 
sudo chmod g+s /var/lib/letsencrypt

sudo nano /etc/nginx/snippets/letsencrypt.conf
# Put this content:
location ^~ /.well-known/acme-challenge/ { allow all;
 root /var/lib/letsencrypt/; default_type
"text/plain"; try_files $uri =404;
} 

sudo nano /etc/nginx/snippets/ssl.conf 
# Put this content:
ssl_dhparam /etc/ssl/certs/dhparam.pem;
ssl_session_timeout 1d; ssl_session_cache
shared:SSL:50m; ssl_session_tickets off;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHEECDSAAES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCMSHA384:ECDHERSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHEECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSAAES256-
SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-
SHA:ECDHERSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHERSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-
SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-
SHA256:AES128SHA:AES256-SHA:DES-CBC3-SHA:!DSS'; ssl_prefer_server_ciphers on;
ssl_stapling on; ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s; resolver_timeout 30s;
add_header Strict-Transport-Security "max-age=15768000; includeSubdomains; preload"; add_header X-FrameOptions SAMEORIGIN; add_header X-Content-Type-Options nosniff; 

sudo nano /etc/nginx/sites-available/YOURWEBSITE.COM
# Put this content:
server { listen 80;
 server_name YOURWEBSITE.COM www.YOURWEBSITE.COM;
include snippets/letsencrypt.conf;
} 

sudo ln -s /etc/nginx/sites-available/YOURWEBSITE.COM /etc/nginx/sites-enabled/

sudo systemctl restart nginx

Acquiring the SSL Certificate

sudo certbot certonly --agree-tos --email admin@YOURWEBSITE.COM --webroot -w /var/lib/letsencrypt/ -d YOURWEBSITE.COM -d www.YOURWEBSITE.COM 

Auto-renew your SSL Certificate

sudo nano /etc/cron.d/certbot 
Then we want to APPEND the following string to the END of the listing in that file:

--renew-hook "systemctl reload nginx"

Note: You DO want to include the double quotes. If you have made the edit correctly, the entire line in the
cerbot file should look something like this:

0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e
'sleep int(rand(3600))' && certbot -q renew --renew-hook "systemctl reload nginx" 

Modifying your Nginx configuration to access your Odoo installation
with the SSL certificate
So now that we have our SSL certificate, we can update our Nginx domain definition for secure access
and get it talking to our Odoo installation.

sudo nano /etc/nginx/sites-available/YOURWEBSITE.COM

Use yourwebsite.com template
