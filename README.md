# sails

a [Sails](http://sailsjs.org) application

##Pre-Requisites 
Provision virtual machine from one of the following
- [Digital Ocean](https://www.digitalocean.com/?refcode=6cb7608c5544)
- [Linode](https://www.linode.com/?r=61064a879e7e2243e086b6cd1d7ca404c0e56b32)

##Linux Distro
Ubuntu 14 LTS 

##Linux Image
LEMP or equivalent

##Steps 

Setup directory structure for SSL Cert storage
```bash
mkdir -p /etc/nginx/ssl/domain.com
```

Generate a key.csr (Certificate Signing Request)
```bash
openssl req -newkey rsa:2048 -x509 -sha256 -days 365 -nodes -keyout /etc/nginx/ssl/domain.com/key.csr
```
Fill out the required fields
```text
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:California
Locality Name (eg, city) []:Long Beach
Organization Name (eg, company) [Internet Widgits Pty Ltd]:SoftwareDev, LLC
Organizational Unit Name (eg, department) []:IT
Common Name (eg, FQDN) []:domain.com
Email Address []:info@mydomain.com
```
Acquire browser compatible development SSL cert for your domain from one of the following <sup>1</sup>

- [Comodo 90 Day Trial SSL](https://ssl.comodo.com/free-ssl-certificate.php)
- [GeoCerts 30 Day Trial SSL](https://www.geocerts.com/ssl/trial)
- [RapidSSL 30 Day Trial SSL](https://forms.rapidssl.com/websurveys/servlet/ActionMultiplexer?Action_ID=ACT2000&WSD_surveyInfoID=659&WSD_mode=3&toc=L07FY-659-03-26)
- Symantec 30 Day Trial SSL <sup>2</sup>

Copy certificate.crt to SSL domain directory <sup>3</sup> 
```bash
cp certificate.crt /etc/nginx/ssl/domain.com/certificate.crt
```

Create Nginx VirtualHost for domain.com
```bash
cd /etc/nginx/sites-available
sudo touch mydomain.com
sudo pico mydomain.com
```
Add the following to mydomain.com
```text
server {
    listen 80;
    server_name domain.com;
    return 301 https://domain.com$request_uri;
}

server {
    listen 443 ssl;
    server_name domain.com;
    root /home/yourusername/domain.com;

    # SSL Certs
    ssl_certificate /etc/nginx/ssl/domain.com/bundle.crt;
    ssl_certificate_key /etc/nginx/ssl/domain.com/server.key;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    charset utf-8;

    location / {
        proxy_pass http://localhost:1337;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;         
    }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;
}
```

<kbd>CTRL</kbd>+<kbd>X</kbd> to exit and save

Create symlink for sites-enabled
```bash
cd /etc/nginx/sites-enabled
sudo ln -s domain.com /etc/nginx/sites-available/domain.com
```

Restart Nginx process
```text
sudo service nginx restart 
```

Footnotes:

<sup>1</sup> Only allowed one free trial cert per domain name, even if it's a subdomain (ie me.mydomain.com or you.mydomain.com)

<sup>2</sup> Total BS, don't waste your time

<sup>3</sup> The SSL provider may provide an intermediate.crt, and you'd have to combine your domain cert with it, into one file bundle.crt
```bash
cat certificate.crt intermediate.crt >> bundle.crt
```
