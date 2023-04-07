#############################
## 0. Persiapan LAB        ##
#############################

# install beberapa paket berikut
sudo apt install software-properties-common ca-certificates lsb-release apt-transport-https

# tambahkan repo ppa ondred/ppa
sudo add-apt-repository ppa:ondrej/php -y

# lakukan proses update index repository
sudo apt update 


#############################
## 1. Instalasi MariaDB    ##
#############################


## install MariaDB 

# install dengan apt
sudo apt install mariadb-server

# inisiasi konfigurasi awal MariaDB
sudo mysql_secure_installation

# tekan ENTER jika diminta password saat ini
# Pilih Yes jika diminta untuk membuat password
# ketikka password 2x
# tekan ENTER untuk setiap pertanyaan berikutnya

# masuk ke shell MariaDB dengan mengetikkan perintah, berikut lalu ketikkan password yang tadi dibuat
sudo mysql -u root -p

# jika berhasil akan muncul prompt seperti berikut
MariaDB [(none)]>

# buat database dengan format studyjam_[nama];
CREATE DATABASE studyjam_samsul;

# buat user dan beri akses ke database tersebut
grant all privileges on studyjam_samsul.* to 'samsul'@'127.0.0.1' identified by 'rahasia';
flush privileges;

# keluar dari prompt dengan mengetikkan \q
# database siap digunakan


#####################################
## 2. Install & konfigurasi Nginx  ##
#####################################

## Install & konfigurasi Nginx
sudo apt install nginx

# cek limit file
ulimit -n

## sunting berkas konfigurasi utama nginx.conf
vim /etc/nginx/nginx.conf

# perbarui nilai worker_connections dengan hasil output ulimit di atas
# hapus tanda # pada baris 8 yang berisi multi_accept
# hapus tanda # pada baris 20 yang berisi server_tokens off;
# hapus TLSv1 TLSv1.1 pada baris 32 yang berawalan ssl_protocols

# aktifkan gzip compression (baris 46-53)
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_types text/plain text/css application/json application/javascript text/xm    l application/xml application/xml+rss text/javascript;

# simpan, lalu keluar

## buat konfigurasi untuk security dengan nama security.conf

vim /etc/nginx/snippets/security.conf

# isinya sebagai berikut

# mulai security headers
add_header X-XSS-Protection          "1; mode=block" always;
add_header X-Content-Type-Options    "nosniff" always;
add_header Referrer-Policy           "no-referrer-when-downgrade" always;
add_header Content-Security-Policy   "default-src 'self' http: https: ws: wss: data: blob: 'unsafe-inline'; frame-ancestors 'self';" always;
add_header Permissions-Policy        "interest-cohort=()" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

# . files
location ~ /\.(?!well-known) {
    deny all;
}
# selesai security headers


## buat berkas virtualhost di folder sites-available
vim /etc/nginx/sites-available/php-[nama]

# tambahkan teks berikut
server {
    listen 80;
    server_name php.nacita.ok;
    root /var/www/project/latihan1;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-project.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
    include /etc/nginx/snippets/security.conf;

}


# buat symlink konfigurasi virtualhost tersebut ke /etc/nginx/sites-enabled

cd /etc/nginx/sites-enabled
ln -s /etc/nginx/sites-available/project-[nama]


# berikutnya restart nginx

systemct restart nginx

#############################
## 3. PHP & PHP-FPM        ##
#############################


## Install & konfigurasi php
sudo apt install php8.1 php8.1-{fpm,gd,mbstring,mcrypt,mysql,odbc,common,soap,xml,xmlrpc,cli,redis,cli,dev,imap,imagick}

# jika ada pertanyaan "which sevice should be restarted?" pilih No/Cancel

# install composer
wget https://github.com/composer/composer/releases/download/2.5.1/composer.phar -O /usr/bin/composer
chmod +x /usr/bin/composer

# Buat user project dengan directory /var/www/project dan ubah shell ke /bin/false
sudo useradd -m -d /var/www/project -s /bin/false project

# Buat password
sudo passwd project

# untuk berpindah ke user project
sudo su - -s /bin/bash project

# keluar dari user project
exit

## php-fpm 
# Copy file `www.conf` menjadi `project.conf` dan tambahkan code berikut untuk merubah user servie run di directory `/etc/php/php8.1.fpm/pool.d/project.conf`
cd /etc/php/php8.1.fpm/pool.d
cp www.conf project.conf


# edit berkas project.conf
#line 4
[project] 
#line 23 dan 24
user = project
group = project 
#line 33
listen = /run/php/php8.1-project.sock 
#line 48 dan 49
llisten.owner = project 
listen.group = project 

# simpan, lalu restart php-fpm
systemctl restart php8.1-fpm
systemctl status php8.1-fpm

## update konfigurasi nginx
# edit file `nginx.conf` pada directory `/etc/nginx/nginx.conf`pada line 1 dengan 

user www-data project;

# simpan, lalu restart nginx
nginx -t
systemctl restart nginx

## cek php
# buat sebuah file index.php di /var/www/project/latihan1 dengan isi sebagai berikut

<?php
phpinfo();
?>

# simpan, lalu akses melalui browser 

#############################
## 4. Instalasi WordPress  ##
#############################

## siapkan sebuah database
# login ke mysql
sudo mysql -u root -p

## buat database
MariaDB [(none)]> CREATE DATABASE wp_samsul;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON wp_samsul.* TO 'wp_user'@'127.0.0.1' identified BY 'wp_rahasia';
MariaDB [(none)]> FLUSH PRIVILEGES;

## siapkan wordpress
# berpindah ke user project
sudo su - -s /bin/bash project

# download WordPress, lalu ekstrak, dan letakkan di folder /var/www/project/wordpress
wget https://wordpress.org/latest.tar.gz -O wordpress.tar.gz

# ekstrak 
tar -xvzf wordpress.tar.gz


# keluar dari user project
exit







# buat konfigurasi virtualhost nginx
sudo vim /etc/nginx/sites-available/wp-[nama]

# tambahkan teks berikut
server {
    listen 80;
    server_name wordpress.nacita.ok;
    root /var/www/project/wordpress;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-project.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # WordPress: deny wp-content, wp-includes php files
    location ~* ^/(?:wp-content|wp-includes)/.*\.php$ {
        deny all;
    }

    # WordPress: deny wp-content/uploads nasty stuff
    location ~* ^/wp-content/uploads/.*\.(?:s?html?|php|js|swf)$ {
        deny all;
    }

    # WordPress: SEO plugin
    location ~* ^/wp-content/plugins/wordpress-seo(?:-premium)?/css/main-sitemap\.xsl$ {}

    # WordPress: deny wp-content/plugins (except earlier rules)
    location ~ ^/wp-content/plugins {
        deny all;
    }

    # WordPress: deny general stuff
    location ~* ^/(?:xmlrpc\.php|wp-links-opml\.php|wp-config\.php|wp-config-sample\.php|readme\.html|license\.txt)$ {
        deny all;
    }
    
    include /etc/nginx/snippets/security.conf;

}


# buat symlink konfigurasi virtualhost tersebut ke /etc/nginx/sites-enabled

cd /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/wp-[nama]


# berikutnya restart nginx
sudo nginx -t
sudo systemct restart nginx









#############################
## 5. Instalasi Laravel    ##
#############################


## Deploy laravel

# masuk ke directory root nginx 
cd /var/www/project


# clone file git laravel 
git clone https://github.com/rappasoft/laravel-boilerplate.git
cd laravel-boilerplate

# update file laravel agar bisa digunakan

composer update

# copy berkas .env.example ke .env

cp .env.example .env

# edit file `.env`, sesuaikan konfigurasinya

# generate key laravel

php artisan key:generate

# lakukan proses migrate database

php artisan migrate

