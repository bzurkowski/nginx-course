# Installation

Switch to root mode:

```bash
sudo -s
```

## Installing NGINX from Ubuntu packages

### Why?

* fast & easy
* provided by the community
* additional compiled-in modules
* basic configuration under the hood (init script, NGINX start on system boot, log rotation etc.)

### How?

```bash
add-apt-repository ppa:nginx/stable -y
apt-get -y update
apt-get -y install nginx
```

## Installing NGINX from source

### Why?

* up to date with security updates
* total control over NGINX instance
* ability to compile in third party modules
* ability to setup compilation-time options

### How?

Ensure system's package database and installed programs are up to date:

```bash
apt-get update
apt-get upgrade
```

Install make and all dependent packages:

```bash
apt-get install libpcre3-dev build-essential libssl-dev
```


[Download](http://nginx.org/en/download.html#stable_versions) the latest stable release of NGINX:

```bash
cd /opt/
wget http://nginx.org/download/nginx-1.3.7.tar.gz # fetch the release
tar -zxvf nginx-1.3.7.tar.gz                      # extract it
cd /opt/nginx-1.3.7/                              # go to folder
```

Configure build options:

```bash
./configure \
--user=nginx                              \
--group=nginx                             \
--prefix=/etc/nginx                       \
--sbin-path=/usr/sbin/nginx               \
--conf-path=/etc/nginx/nginx.conf         \
--pid-path=/var/run/nginx.pid             \
--lock-path=/var/run/nginx.lock           \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module            \
--with-http_stub_status_module            \
--with-http_ssl_module                    \
--with-pcre                               \
--with-file-aio                           \
--with-ipv6                               \
--with-http_realip_module                 \
--without-http_scgi_module                \
--without-http_uwsgi_module               \
--without-http_fastcgi_module
```

Add 3rd party modules:

```bash
--add-module=/path/to/module1/source \
--add-module=/path/to/module2/source
```

For a [full list](http://wiki.nginx.org/InstallOptions) of options type: `./configure --help`

Build and install:

```bash
make
make install
```

Nginx is now installed in `/opt/nginx`.

Add user and group for NGINX:

```bash
adduser --system --no-create-home --disabled-login --disabled-password --group nginx
```

Check NGINX version:

```bash
nginx -V
```

`nginx -v` - prints version of installed NGINX's release
`nginx -V` - additionally prints configuration's options (inclued modules, paths etc.)


### Creating an Init script to manage NGINX easily

```bash
wget -O init-deb.sh http://www.linode.com/docs/assets/1139-init-deb.sh
mv init-deb.sh /etc/init.d/nginx
chmod +x /etc/init.d/nginx
/usr/sbin/update-rc.d -f nginx defaults
```
The script allows to start, stop, and restart NGINX just like any other server daemon:

```
# start it
/etc/init.d/nginx start
# stop
/etc/init.d/nginx stop
# restart
/etc/init.d/nginx restart
# reload
/etc/init.d/nginx reload
```

## Recompilation

For NGINX installed with apt-get.

### Use cases

* adding/removing 3rd party modules
* setting compilation-time options eg. paths

### How?

Download extra modules:

```bash
mkdir /opt/httpupload
cd /opt/httpupload
wget https://github.com/vkholodkov/nginx-upload-module/archive/2.2.zip
unzip 2.2.zip
# Gives us directory: /opt/httpupload/nginx-upload-module-2.2
```

Ensure that the deb-src directive is not commented out in NGINX's PPA:

```bash
cat /etc/apt/sources.list.d/nginx-stable-trusty.list
```

It should print out uncommented entries:

```bash
deb http://ppa.launchpad.net/nginx/stable/ubuntu trusty main
deb-src http://ppa.launchpad.net/nginx/stable/ubuntu trusty main
```

Update package manager:

```bash
apt-get update
```

Install package creation tools:

```bash
apt-get install -y dpkg-dev
```

Create directory for rebuilt NGINX's release:

```bash
mkdir /opt/rebuildnginx
cd /opt/rebuildnginx
```

Download NGINX's source:

```bash
apt-get source nginx
```

Install build dependencies:

```bash
apt-get build-dep nginx
```

Adjust modules in `/opt/rebuildnginx/nginx-1.6.2/debian/rules` file:

```bash
--add-module=/opt/httpupload/nginx-upload-module-2.2
```

Rebuild NGINX packages:

```bash
dpkg-buildpackage -b
```

Remove current NGINX's release:

```bash
apt-get remove nginx
```

Install extended release:

```bash
dpkg --install nginx-full_1.6.2-5+trusty0_amd64.deb
```

Use `nginx -V` to check if proper version and flags has been installed.

Optionally, Block further updates from apt-get (or else it will overwrite your custom NGINX packages):

```bash
apt-mark hold nginx-full
```

Before next recompiilation unlock using:

```bash
apt-mark unhold nginx-full
```

## Links

* [https://github.com/equivalent/scrapbook2/blob/master/archive/blogs/2014-02-instaling-nginx-1-4-4-on-ubuntu-from-source.md](https://github.com/equivalent/scrapbook2/blob/master/archive/blogs/2014-02-instaling-nginx-1-4-4-on-ubuntu-from-source.md)
* [https://www.digitalocean.com/community/tutorials/how-to-compile-nginx-from-source-on-a-centos-6-4-x64-vps](https://www.digitalocean.com/community/tutorials/how-to-compile-nginx-from-source-on-a-centos-6-4-x64-vps)
* [https://serversforhackers.com/compiling-third-party-modules-into-nginx](https://serversforhackers.com/compiling-third-party-modules-into-nginx)
* [https://www.linode.com/docs/websites/nginx/installing-nginx-on-ubuntu-12-04-lts-precise-pangolin/](https://www.linode.com/docs/websites/nginx/installing-nginx-on-ubuntu-12-04-lts-precise-pangolin/)
* [https://jamie.curle.io/posts/compiling-nginx-ubuntu/](https://jamie.curle.io/posts/compiling-nginx-ubuntu/)
* [http://www.geoffstratton.com/2014/03/ubuntu-recompile-nginx-apt-get/](http://www.geoffstratton.com/2014/03/ubuntu-recompile-nginx-apt-get/)
* [https://longhandpixels.net/blog/2014/02/install-nginx-debian-ubuntu](https://longhandpixels.net/blog/2014/02/install-nginx-debian-ubuntu)
