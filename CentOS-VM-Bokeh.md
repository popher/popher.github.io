Bokeh Server - VirtualBox Instructions
===================


This tutorial will set up a basic web server in a virtual machine to run the awesome **Bokeh**[^Bokeh] server. 

The end server will be a CentOS 7 LNMP[^LNMP] virtual machine with the ability to serve HTML, PHP, and Python applets. 

Values (names, numbers) I include are subjective, and may not be best for your set up.

[TOC]

[^Bokeh]:"Bokeh is a Python interactive visualization library that targets modern web browsers for presentation." [Bokeh](http://bokeh.pydata.org/en/latest/) 

 [^LNMP]:Linux Nginx MySQL(mariaDB) PHP(Python) i.e. [LAMP stack](https://en.wikipedia.org/wiki/LAMP_%28software_bundle%29) 

----------

Setting up a Virtual Machine
-------------

These instructions are for [VirtualBox](https://www.virtualbox.org/), but presumably would be somewhat applicable to other system. The host operating system is Windows 10, but that does not matter.

 1. Download and install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
 2. Download a [CentOS 7 ISO](https://www.centos.org/download/)[^CentOS] 
 3.  In VirtualBox, create a new machine.
	 4. Name - CentOS7Server
	 5. Type: Linux
	 6. Version Red-Hat (64-bit)
 7.  Give the machine sufficient RAM (e.g. 8GB if you have 32GB).
 8. Create a new hard disk (e.g. dynamically allocated 16GB, but you can go larger).
 9. Once created, right click - settings:
	 10. Network - Adapter 1 - Enable - NAT
	 11. Network - Adaptor 2 - Enable - Host-Only Adapter[^Networking]
 12. Start
 13. Install CentOS...

[^CentOS]:Whilst I prefer CentOS 7, there is no reason you couldn't go for another linux distribution like Ubuntu.

[^Networking]:This set up (one for NAT, one for Host-Only) is useful if you want to set the machine up for local testing within the guest and the host, but not for allowing the web server to serve the wider internet. 

----------

Setting up CentOS 
-------------
CentOS installation is fairly trivial. 

I opted for the Server with GUI option, your choice. At this stage you may also wish to add things like the Development Tools group, etc. Anything can be added later if needed.

Also, change the hostname during installation (it can be fixed later if need be). 
I use `server1.example.com` - just make sure your host and guest later have their hosts files fixed.

Once CentOS is up and running, do the fairly standard[^sudo]:
```
    sudo yum update
```
[^sudo]:You can run all the code as a normal user with sudo permissions.

After rebooting, install the virtualbox guest additions to get a better experience. Reboot again to check it's all working fine.

You might also want to install some other packages at this point, including jpeg support for Pillow.
```
sudo yum install libtiff-devel libjpeg-devel libzip-devel freetype-devel lcms2-devel libwebp-devel tcl-devel tk-devel
```

At this point you can start to set up the server specific software. Most of LNMP instructions come from [here](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-centos-7).

### Hosts and IP addresses
In a terminal check the ip address of your host only adaptor:
```
ip a
```
Check the output for an IP address. It will be something like
`192.168.56.101`
The final two blocks of numbers may vary, but not much. This is the IP address of your guest (CentOS) which your host (Windows) can later communicate with. It's a local-only IP address (192.168.x.x), so no one else can see this from the wider web unless you forward ports explicitly to allow it.

In your guest (CentOS), edit the hosts file to reflect this:
```
sudo vi /etc/hosts
```
Modify this file to read:

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 server1.example.com server1
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.56.101 server1.example.com server1
```

Obviously change the second full IP address to whatever your guest (centos) is.
In your host (Windows), [modify the hosts file to reflect the same](http://www.howtogeek.com/howto/27350/beginner-geek-how-to-edit-your-hosts-file/).


### Installing Nginx
These instructions from [here](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)  and [here](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7) also.

Add the Nginx repo
```
sudo vi /etc/yum.repos.d/nginx.repo
```
Insert:
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```
Save and close.

Then install and start nginx:
```
sudo yum install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

And you can check if this has worked by going to your web browser in your guest to
`http://127.0.0.1` 
and also/obviously,
`http://server1.example.com` 

You will need to modify the firewall rules on your Guest before you can see this from your host:
```
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```
This code above allows HTTP (port 80) and HTTPS (port 443) through the firewall.
Once you have made those changes, you should be able to access your website from the host PC too.

### Install MariaDB and PHP
These steps are optional. Follow the instructions [here](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-centos-7).


## Install Python (2.7)
Unfortunately, I haven't got this working on Python 3 yet - [just under four years left...](https://pythonclock.org/) 

Grab the latest version of Miniconda from [here](http://conda.pydata.org/miniconda.html). 
I like Miniconda/anaconda, but if you want to do it a different way thats your call. 

Download e.g. 
```
wget https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh
chmod +x Miniconda2-latest-Linux-x86_64.sh
bash Miniconda2-latest-Linux-x86_64.sh
```
I recommend you allow it to add the miniconda2 path to your bash profile. 

At this point you will want to update and install a number of python packages:
```
conda update python
conda install numpy numba scipy matplotlib pandas PyTables
conda install bokeh
```
I also love the [DataShader](https://github.com/bokeh/datashader) project.
Due to some packages on Conda being a little outdated, it's best if you grab datashader from github and install it through that[^github]:
```
wget https://github.com/bokeh/datashader/archive/master.zip
unzip master.zip
cd datashader
conda install -c bokeh --file requirements.txt
python setup.py install
```

[^github]:Yes you should probably use git to grab this, but this works too...

## Set up Nginx
I used a mash up of instructions from [Bokeh](http://bokeh.pydata.org/en/0.12.0/docs/user_guide/server.html) and the previous nginx set up instructions. This resulted in some conflicting information. 

First step, SSL.
### SSL Support
This is based on instructions [here](https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04) 

```
sudo mkdir /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out etc/nginx/ssl/nginx.crt
```
This creates a couple of self-signed certificates. Follow the prompt instructions, making sure to at least enter the `Common Name (e.g. server FQDN or YOUR name)` - i.e. `server1.example.com`

### Nginx Conf
The main configuration files are located at:
`/etc/nginx/nginx.conf` 
and 
`/etc/nginx/conf.d/default.conf`

Make a backup of the default.conf file
```
sudo cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.old
```
And then edit the file:
```
sudo vi /etc/nginx/conf.d/default.conf
```

The exact layout for this file will depend what you want to serve. 
The example below is based on PHP being installed, and also wanting to redirect all HTTP requests to HTTPS (i.e. secure only).

``` 
server {
    listen      80;
    server_name server1.example.com;
    return      301 https://$server_name$request_uri;
}

server {
    listen      443 default_server;
    server_name server1.example.com;

    # add Strict-Transport-Security to prevent man in the middle attacks
    add_header Strict-Transport-Security "max-age=31536000";

    ssl on;

    # SSL installation details will vary by platform
    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    # enables all versions of TLS, but not SSLv2 or v3 which are deprecated.
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # disables all weak ciphers
    ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";

    ssl_prefer_server_ciphers on;

    access_log  /var/log/nginx/bokeh.access.log;
    error_log   /var/log/nginx/bokeh.error.log  debug;

    root   /usr/share/nginx/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;

    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location /apps/ {
        proxy_pass http://127.0.0.1:5100;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_buffering off;
    }

    location /apps/static {
        alias /usr/share/nginx/html/static;
    }

}
```


----------


This is quite a long file, so I'll break it down:
```
server {
    listen      80;
    server_name server1.example.com;
    return      301 https://$server_name$request_uri;
}
```
This section above **listens** on port **80** for the server name provided (`server1.example.com`) and [redirects](https://en.wikipedia.org/wiki/HTTP_301) to https.


----------


The next section (abridged):
```
server {
    listen      443 default_server;
    server_name server1.example.com;

    # add Strict-Transport-Security to prevent man in the middle attacks
    add_header Strict-Transport-Security "max-age=31536000";

    ssl on;

    # SSL installation details will vary by platform
    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;
    

    ssl_prefer_server_ciphers on;
```
This tells the server to listen on 443 (HTTPS), much like before. 
It enables SSL and tells Nginx where to find the keys we created earlier. It also sets a few rules for better security. 


----------


The next section:
```
    root   /usr/share/nginx/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;

    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
```
This defines the root of the webserver - i.e. where our files actually are. This is important for the static files...
It also instructs about 404 (etc) error pages, and also how to handle PHP requests (only relevant if you have installed and set up PHP).


----------


This final section
```
    location /apps/ {
        proxy_pass http://127.0.0.1:5100;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_buffering off;
    }

    location /apps/static {
        alias /usr/share/nginx/html/static;
    }
```
This makes Bokeh Server actually work with Nginx. The first bit essentially says that any requests to `https://server1.example.com/apps/` get sent to whatever is listening at `http://127.0.0.1:5100`
Bokeh will (later) be serving apps to 127.0.0.1:5100, and this ties it together. 

There is an additional section which directs requests to `/apps/static` to `/usr/share/nginx/html/static`. I'm not 100% if this is required.


----------
Whenever you make changes to the nginx configuration files, you should restart the service:
```
sudo service nginx restart
```

You can now do a (rudimental) test if that configuration is working properly. 

 1. Go to `http://server1.example.com` in your web browser. It should load the default nginx page, but after redirecting you to `https://server1.example.com`, and, browser depending, have warned you about a self-signed certificate. 
 2. Go to a non-existing page, e.g. `https://server1.example.com/filenothere`. You should get an nginx 404 page.
 3. Go to the bokeh serve page: `http://server1.example.com/apps`. This should return a different error page reading, "An error occurred." This is OK, it is because we aren't yet serving any bokeh apps!

These should do the same in your guest and host web browsers, too.

## Bokeh Server
The next step is to download some example files!
Grab the whole bokeh set from github: (delete the old master.zip file from your downloads first)
```
wget https://github.com/bokeh/bokeh/archive/master.zip
unzip master
cd bokeh-master/examples/app
```

Now it's just a simple single line to serve any one app:
```
bokeh serve crossfilter --port 5100 --host server1.example.com:443 --use-xheaders --prefix=/apps --allow-websocket-origin=server1.example.com
```

That command consists of a number of aspects:
`bokeh serve` is the initial command 
`crossfilter` is the name of the app we wish to serve.
`--port 5100` specifies the port to serve on (see the nginx configuration)
`--host server1.example.com:443` specifies the requesting (incoming) port and host
`--use-xheaders` is justified in the [original docs.](http://bokeh.pydata.org/en/0.12.0/docs/user_guide/server.html)
`--prefix=/apps` tells bokeh to ignore the prefix of "/apps/" (else it returns 404s)
`--allow-websocket-origin=server1.example.com` makes it work (I dont get this)

In your web browser, now navigate to `https://server1.example.com/apps/`
You should be redirected to `https://server1.example.com/apps/crossfilter` (where `crossfilter` is the name of the app you launched)

##Next steps...
Two main things:
1. Serve your own app
2. Serve multiple apps

For the first point, get coding! 

For the second one, I refer to the [demo bokeh site](https://demo.bokehplots.com/). Fortunately, the guys at Bokeh have set up a deploy script for it: [here.](https://github.com/bokeh/demo.bokehplots.com) 
That script relies on [saltstack](https://saltstack.com/) & is based on [Amazon EC2](https://aws.amazon.com/ec2/) web services. I had a quick play with it, but it's another layer of complexity.