#############################
## 0. Persiapan LAB        ##
#############################

# pada LAB ini kita juga akan menggunakan 2 node, yaitu node1 & node2
# node1 digunakan sebagai server aplikasi
# node2 digunakan sebagai jenkins server untuk CI/CD
# kita juga akan menggunakan git server, dalam hal ini adalah github.com

##################################################
## I. Install aplikasi berbasis Flask di node1  ##
##################################################

# login ke node1 melalui ssh
## install beberapa paket aplikasi yang akan digunakan
sudo apt update
sudo apt install python3-{pip,venv} -y

# lewati langkah update-alternatives berikut [tidak perlu dieksekusi]
#sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 10

## install nginx & supervisor, pada LAB kita, nginx sudah terinstall, jadi di sini kita cukup install supervisor saja
sudo apt install supervisor -y


## berikutnya berpindah ke user project
sudo su - -s /bin/bash project

# buat folder untuk aplikasi flask
mkdir flaskapp
mkdir logs
cd flaskapp

# buat virtualenv dengan python3
python3 -m venv flaskvenv

# load virtualenv tersebut
source flaskvenv/bin/activate

# jika berhasil akan muncul shell prompt seperti berikut
(flaskvenv) project@nest-test-admin1:~/flaskapp$ 

# install flask & gunicord di dalam virtualenv
pip install flask gunicorn


##############################
## II. Buat aplikasi Flask  ##
##############################

# buat sebuah file myapp.py
vim myapp.py

# isikan sebagai berikut

import os
from flask import Flask,render_template

app = Flask(__name__)

@app.route("/")
def hello_world(hostname=None):
    myhost = os.uname()[1]
    appname = os.environ.get('APP_NAME', 'NoName')
    return render_template('index.html', hostname=myhost, appname=appname)


# buat sebuah folder templates
mkdir -p templates/

# buat berkas index.html di dalam folder tersebut
wget -O templates/index.html https://raw.githubusercontent.com/nacita/nest-nginx-workshop/main/python-app/templates/index.html

# coba jalankan secara manual 
export APP_NAME=[username]-belajar-flask
flask run

# jika berhasil akan muncul ip & port
# lalu coba akses port tersebut dengan curl dari terminal lain/browser


################################
## III. Konfigurasi Gunicorn  ##
################################

# buaf file wsgi
nano wsgi.py

# isikan sebagai berikut

# import myapp Flask application
from myapp import app

if __name__ == "__main__":
    app.run(debug=True)


# jalankan gunicorn secara manual
gunicorn -w 4 --bind 0.0.0.0:8000 wsgi:app

# coba akses aplikasi tersebut melalui browser dengan ip-node1:8000

### berikutnya kita jalankan gunicorn menggunakan supervisor
# kembali ke user sebelumnya dengan perintah exit
exit

# buat konfigurasi supervisor
sudo nano /etc/supervisor/conf.d/myapp.conf

# isinya sebagai berikut
[program:flaskapp] 
command=/bin/bash -c 'source /var/www/project/flaskapp/flaskvenv/bin/activate; /var/www/project/.local/bin/gunicorn -w 3 --bind unix:/var/www/project/gunicorn.sock wsgi:app'
environment=APP_NAME="[username]-belajar-flask"
directory=/var/www/project/flaskapp
user=project
group=project
autostart=true 
autorestart=true 
stdout_logfile=/var/www/project/logs/flaskapp.log 
stderr_logfile=/var/www/project/logs/error.log


# restart supervisor
sudo systemctl restart supervisor

# coba pantau log service supervisor
sudo journalctl -f -u supervisor


##############################################
## IV. Nginx reverse proxy untuk Flask App  ##
##############################################

# buat virtualhost baru
sudo nano /etc/nginx/sites-available/flaskapp.conf

# isikan sebagai berikut

upstream flask_server {
server unix:/home/flask[username]/ipc.sock fail_timeout=0;
}

server {
    listen 80;
    server_name flask.[username].ok;

    location / {
        include proxy_params;
        proxy_pass http://flask_server;
    }
}

# buat symlink 
sudo ln -s /etc/nginx/sites-available/flaskapp.conf /etc/nginx/sites-enabled/

# cek konfigurasi nginx, jika ok, lalu restart nginx
sudo nginx -t
sudo systemctl restart nginx



##################################
## V. Install Jenkins di node2  ##
##################################


## install juga Java 
sudo apt update
sudo apt install openjdk-11-jre
java -version

## outputnya kurang lebih sebagai berikut
openjdk version "11.0.12" 2021-07-20
OpenJDK Runtime Environment (build 11.0.12+7-post-Debian-2)
OpenJDK 64-Bit Server VM (build 11.0.12+7-post-Debian-2, mixed mode, sharing)


# untuk melakukan instalasi Jenkins, silakan copas beberapa baris berikut ke terminal node2
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

# pantau log dengan journalctl
sudo journalctl -f -n 300 -u jenkins

# dapatkan initial password dari log tersebut
# password juga bisa didapat dari berkas /var/lib/jenkins/secrets/initialAdminPassword




