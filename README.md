# ESS UAlberta

## Introduction

The ESS Website is hosted by an ENGG IT server. Engg IT's office can be found on the first floor of Donadeo ICE at 1-300. They have access to our DNS records and house the server locally. To add subdomains or edit any `A` or `CNAME` values, you must contact ENGG IT. You can also setup an account on their helpdesk by talking to them or sending them an email at [enggit@ualberta.ca](mailto:enggit@ualberta.ca).

## SSH

To get SSH access, you need to be on the VPN (Virtual Private Network). VPN access can be provided at the ENGG IT office. There are different methods for different operating systems.

Once you can login to the VPN, you can SSH into the server (provided you have an account on the server. If you don't have an account, ______________).

### macOS and Linux
>To SSH on macOS and Linux, use Terminal and type:\
\
`ssh username@ess.ualberta.ca`

### Windows
>Provided you are on the latest release of Windows 10, you may use:\
\
`ssh username@ess.ualberta.ca`

In the case that [ess.ualberta.ca](https://ess.ualberta.ca) is no longer pointed towards the server (you should talk to ENGG IT to fix this first), the IP address of the server can be used instead. As of 10/DEC/2018, the server is on the IP address `142.244.32.90`

Note: `username` is your UNIX username on the server. (For simplicity purposes, please use your CCID or personal home directory).

## SSL Certification

### Renewing

>The SSL certification can be renewed using the command:\
\
`sudo certbot renew` \
(append `--dry-run` to test it without actually renewing)\
\
Note:  the nginx server needs to be stopped for this to occur. Please refer to the nginx section in the README.

### Adding/Removing Domains to the SSL certificate:

#### Currently Registered Domains: 
    (Please update this list as you add or remove the Currently Registered Domains)

    - www.ess.ualberta.ca
    - ess.ualberta.ca
    - beta.ualberta.ca 

> To add or remove domains to the SLL certification, use the following command:\
\
`sudo certbot certonly --standalone [-d example0.com]`\
\
Example:\
`sudo certbot certonly --standalone -d ess.ualberta.ca -d www.ess.ualberta.ca -d beta.ess.ualberta.ca`\
\
Note:  the nginx server needs to be stopped for this to occur. Please refer to the nginx section in the README.

### Certification Locations (used for nginx) configuration

>`ssl_certificate: /etc/letsencrypt/live/ess.ualberta.ca/fullchain.pem`
>`ssl_certificate_key: /etc/letsencrypt/live/ess.ualberta.ca/privkey.pem`

### Footnotes for SSL

>We are using letsencrypt alongside certbot. The university does not use letsencrypt, however, letsencrypt's ssl certification is adequate for now.

## Nginx

### Introduction

The websites are served using `nginx`. The websites have their own domains (subdomains or otherwise) and are defined in `/etc/nginx/sites-available/`. The sites under `sites-availble` that are enabled can be found under `/etc/nginx/sites-enabled/` using a `symlink` (continue reading to figure out how to enable a site)

### Configuration

>#### default for site: [default](https://ess.ualberta.ca)
    # HTTP — redirect all traffic to HTTPS
    server {
        listen 80;
        listen [::]:80 default_server ipv6only=on;
        return 301 https://$host$request_uri;
    }

    server {
        # Enable HTTP/2
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name ess.ualberta.ca www.ess.ualberta.ca;

        # Use the Let’s Encrypt certificates
        ssl_certificate /etc/letsencrypt/live/ess.ualberta.ca/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/ess.ualberta.ca/privkey.pem;

        # Include the SSL configuration from cipherli.st
        include snippets/ssl-params.conf;

        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-NginX-Proxy true;
            # listen to whatever is on localhost:5000
            proxy_pass http://localhost:5000/;
            proxy_ssl_session_reuse off;
            proxy_set_header Host $http_host;
            proxy_cache_bypass $http_upgrade;
            proxy_redirect off;
        }
    }

>#### default for site: beta.ess.ualberta.ca

    # HTTP — redirect all traffic to HTTPS
    server {
        listen 80;
        listen [::]:80;
        return 301 https://$host$request_uri;
    }

    server {
        # Enable HTTP/2
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name beta.ess.ualberta.ca;

        # Use the Let’s Encrypt certificates
        ssl_certificate /etc/letsencrypt/live/ess.ualberta.ca/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/ess.ualberta.ca/privkey.pem;

        # Include the SSL configuration from cipherli.st
        include snippets/ssl-params.conf;

        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-NginX-Proxy true;
            # listen to whatever is on localhost:5001
            proxy_pass http://localhost:5001/;
            proxy_ssl_session_reuse off;
            proxy_set_header Host $http_host;
            proxy_cache_bypass $http_upgrade;
            proxy_redirect off;
        }
    }

>Generally speaking, if you want to create a new entry, copy the template example for beta.ess.ualberta.ca and modify it to respond to a new domain and a different path. You may use a `webroot` or a `proxy_pass`. Both of the above examples use `proxy_pass`.\
\
The tutorial followed for this can be found at:\
https://code.lengstorf.com/deploy-nodejs-ssl-digitalocean/

## Serving and Node

ess.ualberta.ca is served to `port 5000` using `serve`.

The command issued is as follows:

`sudo nohup serve -l 5000 &`

Where `nohup` re-routes the output to a text file `nohup.out`, `serve` starts serving to `port 5000` using the argument `-l 5000`. The `&` at the end allows you to continue your ssh session with the rest of the command running in the background. This command needs to be run in the build folder under the project directory. To generate the build folder, use:
`yarn build`
in the project directory.

beta.ess.ualbert.ca is currently a live React app running using `react-scripts`

The command issued is as follows:

`sudo nohup yarn start &`

Where `nohup` re-routes the output to a text file `nohup.out`, `yarn start` starts the node app on `port 5001` (as defined in `package.json`). The `&` at the end allows you to continue your ssh session with the rest of the command running in the background. You have to be in the project directory for this.

If you want to quit the terminal or ssh session at this time but want the process to continue even after logging out, use the command

`disown`

once for each backgrounded task. 

As of right now, the projects are held under a specific user's direcotry but these will later be moved to `/websites`
