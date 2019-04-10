Configuration sample
redirect http requests to https
proxy requests to odoo
in /etc/odoo.conf set:

proxy_mode = True
in /etc/nginx/sites-enabled/odoo.conf set:

#odoo server
upstream odoo {
 server 127.0.0.1:8069;
}
upstream odoochat {
 server 127.0.0.1:8072;
}

# http -> https
server {
   listen 80;
   server_name odoo.mycompany.com;
   rewrite ^(.*) https://$host$1 permanent;
}

server {
 listen 443;
 server_name odoo.mycompany.com;
 proxy_read_timeout 720s;
 proxy_connect_timeout 720s;
 proxy_send_timeout 720s;

 # Add Headers for odoo proxy mode
 proxy_set_header X-Forwarded-Host $host;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_set_header X-Forwarded-Proto $scheme;
 proxy_set_header X-Real-IP $remote_addr;

 # SSL parameters
 ssl on;
 ssl_certificate /etc/ssl/nginx/server.crt;
 ssl_certificate_key /etc/ssl/nginx/server.key;
 ssl_session_timeout 30m;
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
 ssl_prefer_server_ciphers on;

 # log
 access_log /var/log/nginx/odoo.access.log;
 error_log /var/log/nginx/odoo.error.log;

 # Redirect requests to odoo backend server
 location / {
   proxy_redirect off;
   proxy_pass http://odoo;
 }
 location /longpolling {
     proxy_pass http://odoochat;
 }

 # common gzip
 gzip_types text/css text/less text/plain text/xml application/xml application/json application/javascript;
 gzip on;
}
Odoo as a WSGI Application
It is also possible to mount Odoo as a standard WSGI application. Odoo provides the base for a WSGI launcher script as odoo-wsgi.example.py. That script should be customized (possibly after copying it from the setup directory) to correctly set the configuration directly in odoo.tools.config rather than through the command-line or a configuration file.

However the WSGI server will only expose the main HTTP endpoint for the web client, website and webservice API. Because Odoo does not control the creation of workers anymore it can not setup cron or livechat workers

Cron Workers
To run cron jobs for an Odoo deployment as a WSGI application requires

a classical Odoo (run via odoo-bin)
connected to the database in which cron jobs have to be run (via odoo-bin -d)
which should not be exposed to the network. To ensure cron runners are not network-accessible, it is possible to disable the built-in HTTP server entirely with odoo-bin --no-xmlrpc or setting xmlrpc = False in the configuration file
LiveChat
The second problematic subsystem for WSGI deployments is the LiveChat: where most HTTP connections are relatively short and quickly free up their worker process for the next request, LiveChat require a long-lived connection for each client in order to implement near-real-time notifications.

This is in conflict with the process-based worker model, as it will tie up worker processes and prevent new users from accessing the system. However, those long-lived connections do very little and mostly stay parked waiting for notifications.

The solutions to support livechat/motifications in a WSGI application are:

deploy a threaded version of Odoo (instread of a process-based preforking one) and redirect only requests to URLs starting with /longpolling/ to that Odoo, this is the simplest and the longpolling URL can double up as the cron instance.
deploy an evented Odoo via odoo-gevent and proxy requests starting with /longpolling/ to the longpolling port.
Serving Static Files
For development convenience, Odoo directly serves all static files in its modules. This may not be ideal when it comes to performances, and static files should generally be served by a static HTTP server.

Odoo static files live in each module's static/ folder, so static files can be served by intercepting all requests to /MODULE/static/FILE, and looking up the right module (and file) in the various addons paths.

Security
For starters, keep in mind that securing an information system is a continuous process, not a one-shot operation. At any moment, you will only be as secure as the weakest link in your environment.

So please do not take this section as the ultimate list of measures that will prevent all security problems. It's only intended as a summary of the first important things you should be sure to include in your security action plan. The rest will come from best security practices for your operating system and distribution, best practices in terms of users, passwords, and access control management, etc.

When deploying an internet-facing server, please be sure to consider the following security-related topics:

Always set a strong super-admin admin password, and restrict access to the database management pages as soon as the system is set up. See Database Manager Security.
Choose unique logins and strong passwords for all administrator accounts on all databases. Do not use 'admin' as the login. Do not use those logins for day-to-day operations, only for controlling/managing the installation. Never use any default passwords like admin/admin, even for test/staging databases.
Use appropriate database filters ( --db-filter) to restrict the visibility of your databases according to the hostname. See dbfilter.
Once your db_filter is configured and only matches a single database per hostname, you should set list_db configuration option to False, to prevent listing databases entirely (this is also exposed as the --no-database-list command-line option)
Make sure the PostgreSQL user (--db_user) is not a super-user, and that your databases are owned by a different user. For example they could be owned by the postgres super-user if you are using a dedicated non-privileged db_user. See also Configuring Odoo.
Keep installations updated by regularly installing the latest builds, either via GitHub or by downloading the latest version from https://www.odoo.com/page/download or http://nightly.odoo.com
Configure your server in multi-process mode with proper limits matching your typical usage (memory/CPU/timeouts). See also Builtin server.
Run Odoo behind a web server providing HTTPS termination with a valid SSL certificate, in order to prevent eavesdropping on cleartext communications. SSL certificates are cheap, and many free options exist. Configure the web proxy to limit the size of requests, set appropriate timeouts, and then enable the proxy mode option. See also HTTPS.
Whenever possible, host your public-facing demo/test/staging instances on different machines than the production ones. And apply the same security precautions as for production.
If you are hosting multiple customers, isolate customer data and files from each other using containers or appropriate "jail" techniques.
Setup daily backups of your databases and filestore data, and copy them to a remote archiving server that is not accessible from the server itself.
Database Manager Security
Configuring Odoo mentioned admin_passwd in passing.

This setting is used on all database management screens (to create, delete, dump or restore databases).

If the management screens must not be accessible, or must only be accessible from a selected set of machines, use the proxy server's features to block access to all routes starting with /web/database except (maybe) /web/database/selector which displays the database-selection screen.

If the database-management screen should be left accessible, the admin_passwd setting must be changed from its admin default: this password is checked before allowing database-alteration operations.

It should be stored securely, and should be generated randomly e.g.

$ python -c 'import base64, os; print(base64.b64encode(os.urandom(24)))'
which will generate a 32 characters pseudorandom printable string.

Supported Browsers
Odoo is supported by multiple browsers for each of its versions. No distinction is made according to the browser version in order to be up-to-date. Odoo is supported on the current browser version. The list of the supported browsers by Odoo version is the following:

Odoo 8: IE9, Mozilla Firefox, Google Chrome, Safari, Microsoft Edge
Odoo 9: IE11, Mozilla Firefox, Google Chrome, Safari, Microsoft Edge
Odoo 10: Mozilla Firefox, Google Chrome, Safari, Microsoft Edge
[1] to have multiple Odoo installations use the same PostgreSQL database, or to provide more computing resources to both software.
[2] technically a tool like socat can be used to proxy UNIX sockets across networks, but that is mostly for software which can only be used over UNIX sockets
[3] or be accessible only over an internal packet-switched network, but that requires secured switches, protections against ARP spoofing and precludes usage of WiFi. Even over secure packet-switched networks, deployment over HTTPS is recommended, and possible costs are lowered as "self-signed" certificates are easier to deploy on a controlled environment than over the internet.
