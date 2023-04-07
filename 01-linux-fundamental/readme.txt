#############################
## 0. Persiapan LAB        ##
#############################

- akses ke https://remotedesktop.google.com/access
- Setup via SSH
- klik Begin
- klik Next
- klik Authorize
- copy kode/perintah di bagian Debian Linux, kirim ke trainer
- tunggu kabar dari trainer terkait PIN untuk akses VM

- untuk akses VM, klik Remote Access, lalu pada remote devices akan ada device baru dengan nama [nama]-nacita-lab
- klik pada device tersebut, lalu masukkan PIN 147258 [atau pin yang diberikan oleh trainer]

# saat pertama kali membuka VM, tampilan desktop GNOME akan muncul pop-up Wellcome 
- klik next beberapa kali hingga muncul tombol Start Using Ubuntu
- jika muncul pop-up lain berupa Upgrade Available, klik Don't Upgrade, klik Ok
- jika muncul pop-up problem detected, klik Cancel saja



#######################################
## 1. Perintah-perintah dasar Linux  ##
#######################################

# untuk memulai, mari kita buka terminal, 
- klik pojok kiri atas: Activities, 
- lalu ketik Terminal
- klik terminal warna hitam
- akan muncul terminal prompt user@hostname


- shell prompt

#     : user root
$     : user biasa

- format prompt shell default
user@hostname:~$ 


# Perintah Dasar Linux
0. man
1. sudo
2. pwd
3. cd
4. ls
5. cat
6. cp
7. mv
8. mkdir
9. rmdir
10. rm
10. touch
11. locate
13. find
14. grep
15. df
16. du
17. head
18. tail
19. diff
20. tar
21. chmod
22. chown
23. jobs
24. kill
25. ping
26. wget
27. uname
28. top
29. history
30. man
31. echo
32. zip, unzip
33. hostname
34. useradd, userdel
35. apt-get
36. nano, vi, jed
37. alias, unaliass
38. su
39. htop
40. ps


Catatan:
- Autocomplete dengan TAB
- Stop perintah yang sedang berjalan dengan Ctrl+C
- Stop perintah yang sedang berjalan secara paksa dengan Ctrl+Z
- Bekukan terminal dengan Ctrl+S, batalkan dengan Ctrl+Q
- Ctrl+A ke awal baris, Ctrl+E ke akhir baris
- pipelining dengan |, output perintah sebelumnya akan digunakan sebagai input untuk perintah berikutnya
- gunakan ; atau && untuk menjalankan beberapa perintah sekaligus dalam 1 baris



###########################################
## 2. Package Management  (debian based) ##
###########################################

- Apa itu dependency (ketergantungan paket)? 

sudo apt install apt-rdepends springgraph
apt-rdepends --dotty htop | springgraph > dependency-map.png

- ekstensi paket berbasis Debian *.deb
- Apa itu linux repository

## Repository
- sebuah repository merupakan lumbung paket, tempat tersimpan berbagai paket yang dapat digunakan oleh pengguna untuk dipasang di komputer/perangkat berbasis Linux, tiap distri memiliki repository resminya masing-masing. 
- pada repository berbasis Debian/Ubuntu memiliki beberapa komponen repository
- pada Ubuntu terdapat beberapa komponen antara lain`
  * Main - Canonical-supported free and open-source software.
  * Universe - Community-maintained free and open-source software.
  * Restricted - Proprietary drivers for devices.
  * Multiverse - Software restricted by copyright or legal issues.
- untuk mengetahui lebih detail terkait masing-masing komponen silakan merujuk ke --> https://help.ubuntu.com/community/Repositories
- konfigurasi repository di Debian/Ubuntu berada di berkas /etc/apt/sources.list dan tambahannya /etc/apt/souces.list.d/*.list
sudo nano /etc/apt/sources.list

- format penulisannya konfigurasi repository memiliki aturan yang jelas
  * type - apakah berupa "binary" (deb), atau "source" (deb-src)
  * URI - merupakan alamat repository, beberapa protokol yang dapat digunakan cdrom, ftp, http, https, smb, nfs, mirror protocol
  * Distribution - nama distro atau nama kode rilis distro 
  * Components - komponen berupa main, universe, restricted, universe (atau yang lainnya)
  * Comments - komentar menjelaskan tentang repository agar memudahkan referensi (opsional)

deb http://archive.ubuntu.com/ubuntu focal main restricted
# type URI                           distro Components

## repository dengan mirror protocol
- silakan baca melalui --> https://wiki.samsul.web.id/linux/Memanfaatkan.Mirror.Protocol.repo.Ubuntu

## Repository PPA (Personal Package Archives)
- merupakan semacam repository yang dapat dibuat oleh siapa saja, khususnya developer atau maintainer paket agar dapat mendistribusikan paketnya secara mandiri

## apt

- install paket
sudo apt install nmap

- hapus paket
sudo apt remove nmap

- update index paket
sudo apt update

- upgrade paket
sudo apt upgrade


## aptitude
- update index paket
sudo aptitude update

- install paket
sudo aptitude install nmap

- aptitude TUI (text user insterface)
sudo aptitude

- install paket melalui aptitude TUI
  * pilih kategori Not Installed Packages -> golang -> golang-1.18
  * tekan + untuk menandai paketnya
  * tekan g untuk preview, tekan g lagi untuk memulai proses instalasi
  * setelah proses instalasi selesai, tekan q lalu enter untuk keluar


- status paket di aptitude
i: Installed package
c: Package not installed, but package configuration remains on the system
p: Purged from system
v: Virtual package
B: Broken package
u: Unpacked files, but package not yet configured
C: Half-configured - configuration failed and requires fix
H: Half-installed - removal failed and requires a fix


## dpkg

- list semua paket
dpkg -l
dpkg -l | grep apache2

- list berkas yang terinstall oleh paket
dpkg -L wget

- cari tahu sebuah berkas dibawa oleh paket apa
dpkg -S /etc/issue.net

- catatan : beberapa berkas hasil generate script tidak akan diketahui oleh perintah tersebut dari paket mana berkas tersebut berasal

- install local berkas *.deb
sudo dpkg -i namafile.deb

- uninstall paket dengan opsi -r
sudo dpkg -r namapaket

- catatan: uninstall paket menggunakan sangat TIDAK direkomendasikan pada kasus umum. Sebaiknya gunakan manajer paket yang dapat menangani ketergantungan, seperti apt atau aptitude.


## synaptic
- gui package management

- install synaptic
sudo apt install synaptic

- jalankan synaptic dengan sudo
sudo synaptic



########################
## 3. VIM Text Editor ##
########################


2 mode
- Command 
- Edit

i : untuk masuk ke mode edit
Esc : keluar dari mode edit, masuk ke mode command

:set number                : tampilkan nomor baris
:wq                        : simpan dan keluar
yy                         : copy 1 baris
5yy                        : copy 5 baris
p                          : paste
dd                         : menghapus baris
3dd                        : menghapus 3 baris
/"text"                    : untuk mencari text
n                          : ke text berikutnya yang ditemukan
:%s/text1/text2            : cari dan ganti text1 menjadi text2
:4,7d                      : hapus baris 4 - 7
:g /kata/d                 : hapus baris yang mengandung "kata" 
:g!/kata/d                 : hapus baris yang tidak mengandung "kata"
:g/^$/d                    : hapus semua baris kosong


## vim plugins menggunakan vim-plug

- install vim-plug
mkdir ~/.vim/autoload
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

- install NERDTree plugin
vim ~/.vimrc

- tambahkan baris berikut
call plug#begin()
  Plug 'preservim/nerdtree'
call plug#end()

- jalankan :PlugInstall di dalam VIM

- jika proses instalasi sudah selesai, jalankan dengan mengetikkan :NERDTree



## Upgrade Ubuntu 

- Upgrade 20.04 LTS ke 22.04 LTS 
less /etc/update-manager/release-upgrades

- pastikan ada baris berikut
Prompt=lts

- cek apakah ada paket yang di-hold 
sudo apt-mark showhold

- lakukan unhold paket tersebut (jika ada), ganti pkg1 dan pkg2 dengan nama paket yang di-hold
sudo apt-mark unhold pkg1 pkg2

- update semua paket
sudo apt update 
sudo apt upgrade

- reboot jika ada kernel baru yang terpasang
sudo reboot

- catat jika diperlukan versi Ubuntu & kernel saat ini
uname -mrs
lsb_release -a

- gunakan tmux sebelum melakukan upgrade
tmux

- lakukan proses upgrade ke Ubuntu 22.04 (jangan lupa maksimalkan jendela terminal)
sudo do-release-upgrade


- Tekan ENTER jika muncul teks seperti berikut
To continue please press [ENTER]

- tekan y jika muncul teks seperti berikut, tekan y sekali lagi jika diminta
Fetching and installing the upgrade can take several hours. Once the 
download has finished, the process cannot be canceled. 

 Continue [yN]  Details [d]

- tunggu proses upgrade hingga selesai


- jika semua berjalan lancer, proses reboot akan dilakukan

- setelah reboot, lakukan proses verifikasi
uname -mrs
lsb_release -a

- selamat Anda telah berhasil melakukan upgrade OS Ubuntu Anda