## From Tutorial to actually running letsencrypt

## Certbot
On my server I needed to add `jessie-backports` to the `/etc/apt/sources.list`.
```bash
echo "deb http://httpredir.debian.org/debian jessie-backports main contrib non-free" >> '/etc/apt/sources.list'
```
and run this afterwards.
```bash
sudo apt-get install certbot -t jessie-backports
```
## Nginx 
Make sure the file `.well-known` is reachable via `location`.
```bash
certbot certonly -a webroot --webroot-path=/usr/share/www/itoaster -d itoaster.net -d www.itoaster.net
```
this will create the directory in your webroot thats why it needs to be reachable. 

**Cert Location** `/etc/letsencrypt/live/itoaster.net/`

Its also a good idea to allow Diffie Hellmann keys so lets generate one:
```bash
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```
Lets take a look at the server config file under `sites-available`.
```conf
server {
        listen 80;
        # Block some agents
        if ($blockedagent) {
                return 403;
        }
        # Disable all except GET HEAD and POST
        if ($request_method !~ ^(GET|HEAD|POST)$) {
                return 444;
        }
        server_name itoaster.net www.itoaster.net;
        return 301 https://$host$request_uri;
}

server {
        # SSL configuration
        #
        # Block some agents
        if ($blockedagent) {
                return 403;
        }
        # Disable all except GET HEAD and POST
        if ($request_method !~ ^(GET|HEAD|POST)$) {
                return 444;
        }

        listen 443 ssl;
        listen [::]:443 ssl;
        server_name itoaster.net www.itoaster.net;
        ssl_certificate /etc/letsencrypt/live/itoaster.net/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/itoaster.net/privkey.pem;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/ssl/certs/dhparam.pem;
        ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_stapling on;
        ssl_stapling_verify on;
        add_header Strict-Transport-Security max-age=15768000;


        root /usr/share/www;
        index index.html;

        access_log /usr/share/www/logs/access.log;
        error_log /usr/share/www/logs/error.log error;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        location ~ /.well-known {
                allow all;
        }

        location ~ /\.ht {
                deny  all;
        }
}
```

### References
* [How To Secure Nginx with Let's Encrypt on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04)
* [Certbot for Debian Jessie](https://certbot.eff.org/#debianjessie-nginx)
* [Hardening NGINX](http://www.tecmint.com/nginx-web-server-security-hardening-and-performance-tips/)
* [Debian Backports](https://wiki.debian.org/Backports)
