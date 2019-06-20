`copy from somewhere`

Start with the installation of NGINX

sudo apt-get install nginx
Create your cert and key

First create a temporary directory and move the files to their final resting place once they have been built (the first cd is just to make sure we are in our home directory to start with):

cd
mkdir temp
cd temp
Generate a new key, you will be asked to enter a passphrase and confirm:

openssl genrsa -des3 -out server.pkey 2048
Remove the passphrase by doing this, we do this because we don’t won’t to have to type this passphrase after every restart.

openssl rsa -in server.pkey -out server.key
Next we need to create a signing request which will hold the data that will be visible in your final certificate:

openssl req -new -key server.key -out server.csr
This will generate a series of prompts like this: Enter the information as requested. And finally we self-sign our certificate.

openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
We only need two of the files in the working directory, the key and the certificate. But before we can use them they need to have their ownership and access rights altered:

sudo chown root:www-data server.crt server.key
sudo chmod 640 server.crt server.key
And then we put them in a sensible place:

sudo mkdir /etc/ssl/nginx
sudo chown www-data:root /etc/ssl/nginx
sudo chmod 710 /etc/ssl/nginx
sudo mv server.crt server.key /etc/ssl/nginx/
We now have the key and certificate on the final location. We can now tell nginx where the files are and how they will behave.

Create the nginx site configuration file

We create a new configuration file

sudo vi /etc/nginx/sites-available/odoo8
with the following content:

IMPORTANT: This file will use all incoming server names on port 80 and port 443. If you want to use it on a specific webaddress change the servername _; in a servername yourwebaddress.com; on both places in the server listening to port 80 and the one listening to port 443.

upstream odoo8 {
server 127.0.0.1:8069 weight=1 fail_timeout=0;
}

upstream odoo8-im {
server 127.0.0.1:8072 weight=1 fail_timeout=0;
}

## http redirects to https ##
server {
listen 80;
server_name _;

# Strict Transport Security
add_header Strict-Transport-Security max-age=2592000;
rewrite ^/.*$ https://$host$request_uri? permanent;
}

server {
# server port and name
listen 443;
server_name _;

# Specifies the maximum accepted body size of a client request,
# as indicated by the request header Content-Length.
client_max_body_size 200m;

# add ssl specific settings
keepalive_timeout 60;
ssl on;
ssl_certificate /etc/ssl/nginx/server.crt;
ssl_certificate_key /etc/ssl/nginx/server.key;

# limit ciphers
ssl_ciphers HIGH:!ADH:!MD5;
ssl_protocols SSLv3 TLSv1;
ssl_prefer_server_ciphers on;

# increase proxy buffer to handle some OpenERP web requests
proxy_buffers 16 64k;
proxy_buffer_size 128k;

#general proxy settings
# force timeouts if the backend dies
proxy_connect_timeout 600s;
proxy_send_timeout 600s;
proxy_read_timeout 600s;
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;

# set headers
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;

# Let the OpenERP web service know that we’re using HTTPS, otherwise
# it will generate URL using http:// and not https://
proxy_set_header X-Forwarded-Proto https;

# by default, do not forward anything
proxy_redirect off;
proxy_buffering off;

location / {
proxy_pass http://odoo8;
}

location /longpolling {
proxy_pass http://odoo8-im;
}

# cache some static data in memory for 60mins.
# under heavy load this should relieve stress on the OpenERP web interface a bit.
location /web/static/ {
proxy_cache_valid 200 60m;
proxy_buffering on;
expires 864000;
proxy_pass http://odoo8;
}
}

We then will enable the new site configuration by creating a symbolic link in the /etc/nginx/sites-enabled directory.

sudo ln -s /etc/nginx/sites-available/odoo8 /etc/nginx/sites-enabled/odoo8
Change the OpenERP server configuration file

We now need to re-configure the openerp server in a way that non-encrypted services are not accessible from the outside world.

We will change the /etc/odoo-server.conf so that it will only except requests from nginx.

Just open then file and add 127.0.0.1 to the xmlrpc and netrpc interface lines as shown below.

sudo vi /etc/odoo-server.conf

xmlrpc_interface = 127.0.0.1
netrpc_interface = 127.0.0.1
Try the new configuration

Restart the services to load the new configurations

sudo service odoo-server restart
sudo service nginx restart
You should not be able to connect to the web client on port 8069 and the GTK client should not connect on either the NetRPC (8070) or XMLRPC (8069) services.

Your ODOO server should be available now.
