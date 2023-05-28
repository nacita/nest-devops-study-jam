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
command=/bin/bash -c 'source /var/www/project/flaskapp/flaskvenv/bin/activate; gunicorn -w 3 --bind unix:/var/www/project/gunicorn.sock wsgi:app'
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
server unix:/var/www/project/gunicorn.sock fail_timeout=0;
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

# buka browser dan ketik http://ip-node2:8080, masukkan password tadi
# pilih install recomended 
# serta buat user & password untuk jenkins

## install plugin

# setelah berhasil login, pada panel kiri, klik Manage Jenkins, pilih Manage Plugins
# selanjutnya, klik Available plugins, install beberapa plugin berikut:
#- Blue Ocean
#- Discord Notifier
#- SSH Agent Plugin
# centang pada bagian "Restart Jenkins when installation is complete and no jobs are running" agar jenkins otomatis direstart

## buat jenkins credential
# sebelumnya, buat dulu ssh-keypair baru. keypair ini akan kita gunakan agar dapat melakukan proses clone di dalam proses ci-cd di jenkins
#- pada Jenkins Dashboard, klik Manage Jenkins
#- klik pada Credentials, selanjutnya klik pada domain (globals), klik Add credentials
#- pada kind, pilih SSH Username with private key
#- pada bagian ID, ketikkan [username]-ssh-key
#- pada Username, isikan username 'student' (sesuai username pada server)
#- lalu pada private key, pilih enter directly, lalu masukkan private key yang tadi kita generate
#- jika sudah, klik tombol Save

## accept hostkey
# nantinya untuk proses jenkins akan ada proses clone dan remote menggunakan ssh, pada saat itu akan error jika konfigurasi berikut tidak kita ubah
#- pada Jenkins Dashboard, klik Manage Jenkins
#- klik Configure Global Security, lalu scroll ke bagian bawah pada Git Host Key Verification Configuration 
#- pada  Host Key Verification Strategy, pilih Accept first connection, klik Save

## daftarkan public ke ke repo github
# ada 2 opsi untuk menambahkan public key ke github, 
#- opsi pertama: level akun, referensi --> https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
#- opsi kedua: level project (deploy key), referensi --> https://docs.github.com/en/authentication/connecting-to-github-with-ssh/managing-deploy-keys#set-up-deploy-keys
#- silakan pilih salah satu, untuk LAB ini saya rekomendasikan pilih opsi kedua.


###############################
## VI. Tugas Akhir StudyJam  ##
###############################

# untuk mengerjakan tugas akhir StudyJam ini, berikut beberapa script yang dapat digunakan untuk mengerjakannya:

## script deploy yang dipasang di server

# login ke node1, lalu pada user student, buat file deploy.sh

vim /home/student/deploy.sh

# isinya sebagai berikut, ubah beberapa variable yang diperlukan

##### script deploy.sh start
#!/bin/bash
set -x

app_user=project
app_path=/var/www/project/[username]-nest-flask-ta
log_file=/var/www/project/deploy.log
app_repo="[URL_REPO_GITHUB]"

sudo su - -s /bin/bash $app_user -c "
touch $log_file
# buat folder jika belum ada
if [ ! -d $app_path ] ; then
        echo 'Cloning repository $app_repo to  $app_path' > $log_file
        git clone $app_repo $app_path
        #mkdir -p $app_path
fi

cd $app_path
git pull origin main
source /var/www/project/flaskapp/flaskvenv/bin/activate
pip install -r requirements.txt
echo 'Deployment done' >> $log_file
"
exit 0
##### script deploy.sh end

# jangan lupa beri akses execute, masih ingat perintahnya kan? 



## Buat jenkins pipeline
# selanjutnya, kita kembali ke Jenkins
#- pada Dashboard Jenkins, klik New item, masukkan nama pipeline [username]-flask-tugas-akhir
#- pilih tipenya : Pipeline (urutan ke-2)
#- klik tombol OK
#- berikutnya akan ada konfigurasi yang perlu kita sesuaikan, description silakan boleh diisi boleh juga tidak,
#- scroll ke bawah, pada bagian pipeline isikan script berikut


// Jenkins pipeline start
// mulai deklarasi pipeline
pipeline {
    // jalankan di agent mana pun
    agent any

    // deklarasikan beberapa stage
    stages {
        // stage 1 Pull repo
        stage('Pull') {
          steps {
            // pull repo dari git, perhatikan credentialsId dan GitHub URL
            git branch: 'main', credentialsId: '[username]-ssh-key', url: 'git@github.com:[username-github]/flask-tugas-akhir-nest.git'
          }
        }
        // stage demo, untuk menjalankan beberapa perintah, silakan ubah jika diperlukan
        stage('Hello') {
            steps {
                echo 'Hello World'
                sh "ls -lha"
                sh "pip3 install -r requirements.txt"
                echo "another"
            }
        }
        // stage deploy ini menggunakan ssh agent plugin untuk menjalankan script yang ada di server
        stage('deploy') {
                steps {
                sshagent(credentials : ['[username]-ssh-key']) {
                    sh 'ssh -tt -vv student@[ip-node1] -o StrictHostKeyChecking=no "/home/student/deploy.sh"'
                }
                }
            }
    }
  // ini tambahan, jika Anda ingin membuat notifikasi menggunakan discord Channel, 
  // 
  //post {
  //  always {
  //    discordSend description: "App with build number ${env.BUILD_DISPLAY_NAME} is ${currentBuild.currentResult}", enableArtifactsList: false, footer: "Triggered in branch main with revision blabla", image: '', result: currentBuild.currentResult, scmWebUrl: '', thumbnail: '', link: "${env.RUN_DISPLAY_URL}", title: "${env.JOB_NAME}", webhookURL: "${DISCORD_WEBHOOK_URL}"
  //}
  //}
}
// Jenkins pipeline end

#- sesuaikan beberapa bagian, seperti URL repo, ip-node1 & credential
#- kemudian klik Save
#- jika sudah, klik tombol Build Now untuk mulai menjalankan pipeline
#- jika semuanya berjalan lancar, warna yang akan ditampilkan berwarna Hijau. 
#- jika terdapat error, dan ingin memperbaiki, klik tombol Configure, lalu edit pipeline-nya, Save dan coba jalankan kembali
#- begitu seterusnya, selamat mengerjakan.