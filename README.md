# MPI-bubble-sort
mengeksekusi program bubble sort secara paralel menggunakan MPI

#Device Dan Tools Yang di gunakan dalam mengeksekusi
1.	Linux Mint
  -	Linux mint master
  -	Linux mint slave1
  -	Linux mint slave2
  -	Linux mint salve3
2.	MPI (Master dan Slave)
3.	SSH (Master dan Slave)
4.	Codingan Bubble Short python

#Topology : 

![image](https://github.com/renn31/MPI-bubble-sort/assets/128461789/75118ea2-a078-42ba-a95e-d0f9d10661f1)
Pada percobaan ini digunakan empat komputer, dimana salah satunya sebagai komputer master, yang bertanggung jawab untuk mengoordinasikan dan mengontrol seluruh proses. Sementara itu, tiga komputer lainnya dijadikan sebagai slave, dengan tugas untuk menjalankan perintah-perintah dari komputer master. Penting untuk memastikan bahwa keempat komputer ini sudah terintegrasi dalam satu jaringan yang sama.	

#Konfigurasi file /etc/hosts

Lakukan pada master dan slave:

Edit file /etc/hosts melalui nano. Tambahkan isinya dengan beberapa IP dan aliasnya. Di bawah ini sebagai contoh. sesuaikan IP nya dengan komputer masing-masing. Untuk mengecek IP gunakan perintah ifconfig.

Tambahkan baris berikut dengan format :

```bash
[IP_address1] [hostname1]
[IP_address2] [hostname2] 
[IP_address3] [hostname3] 
[IP_address4] [hostname4] 
```

sesuaikan dengan komputer yang akan dijalankan, contoh:

```bash
[10.8.141.217] [master]
[10.8.143.246] [slave1] 
[10.8.143.239] [slave2] 
[10.8.143.46] [slave3]
```
#Buat User Baru

lakukan di master dan slave:

Buat user baru di master dan slave dengan perintah berikut: contohnya akan membuat user dengan nama mpiusr. Nama user harus sama pada kompuer master dan slave.

```bash
Sudo adduser <mpiusr>
````
jadikan mpiusr memiliki hak akses superuser

```bash
Sudo usermod –aG sudo mpiusr
```

masuk ke user
```bash
su – mpiusr
```

#Konfigurasi SSH

lakukan di master dan slave:
```bash
sudo apt install openssh-server
```
Perintah tersebut akan menginstal perangkat lunak OpenSSH Server pada sistem agar dapat menggunakan layanan SSH untuk mengakses dan mengelola sistem secara remote dengan aman.

lakukan di master:
```bash
ssh-keygen -t rsa
```
Perintah ini akan membuat kunci SSH baru. Lewatkan seluruh input. Setelah melalui tahap tersebut akan ada folder .ssh dan di dalamnya terdapat file id_rsa dan id_rsa.pub.

Salin isi dari file id_rsa.pubke file authorized_keyske semua slave menggunakan perintah berikut:
```bash
cd .ssh
cat id_rsa.pub | ssh <nama user>@<host> "mkdir .ssh; cat >> .ssh/authorized_keys"
```
Lakukan penyalinan perintah berulang-ulang dari master ke slave dengan mengubah <host>  menjadi nama host masing-masing slave. Dengan membagikan kunci SSH, master akan dapat mengakses server slave jarak jauh dengan aman tanpa perlu memasukkan kata sandi setiap kali.

#konfigurasi NFS

Lakukan di master dan slave:

•	Buat shared folder

```bash
mkdir fix
```

lakukan di master:

•	Install NFS Server
```bash
sudo apt install nfs-kernel-server
```
Perintah ini akan menginstall paket nfs-kernel-server pada master agar dapat berbagi direktori atau sistem berkas dengan slave.

•	Konfigurasi file /etc/exports

Edit file /etc/exports dengan editornano sudonano /etc/exports.

tambahkan baris berikut:
```bash
<lokasi shared folder> *(rw,sync,no_root_squash,no_subtree_check)
```
Sesuaikan<lokasi shared folder>denganlokasi folder yg telah dibuat:
```bash
/home/mpiusr/fix *(rw,sync,no_root_squash,no_subtree_check)
```
Lakukan perintah berikut untuk memastikan bahwa perubahan konfigurasi yang dilakukan dalam file /etc/exports diterapkan tanpa harus memulai ulang layanan NFS.
```bash
Sudo exportfs –a
```
Jalankan perintah ini untuk memuat ulang layanan server NFS dan menerapkan perubahan konfigurasi terbaru dalam file konfigurasi /etc/exports.

```bash
Sudo systemctl restart nfs-kernel-server
```

lakukan di slave:

•	Install NFS 
```bash
sudo apt install nfs-common
```
Paket nfs-common akan di install, memungkinkan untuk mengakses dan menggunakan berkas yang dibagikan oleh master NFS yang telah dikonfigurasi dengan benar.

•	lakukan mounting dengan perintah berikut:

```bash
sudo mount <server host>:<lokasi shared folder di master><lokasi shared folder di slave>
```
sesuaikan <server host>, <lokasi shared folder di master> dan<lokasi shared folder di slave>. contohnya:
```bash
sudo mount master:/home/mpiusr/fix /home/mpiusr/fix
```

#MPI

lakukan di master dan slave:

•	install mpi dengan perintah beriku
```bash
sudo apt install openmpi-bin libopenmpi-dev
```

•	lakukan testing untuk pengeksekusian
lakukan di master:

Buat file python di folder fix. Misal test.py
```bash
touch test.py
```
Kemudian edit file menggunakan perintah nano dengan mengisi file tersebut dengan perogram python sederhana, misalnya:
```bash
Print("Selamat Datang Linux Lover <3")
```
Gunakan perintah berikut untuk mengeksekusi program tersebut:
```bash
mpirun -np <jumlahprosesor> -host <daftar host> python3 test.py
```
Sesuaikan dengan progrm yang akan dijalankan
```bash
mpirun -np 3 -host master,slave1,slave2,slave3 python3 test.py
```

#Eksekusi program MPI

lakukan di master:
```bash
sudo apt install python3-pip
pip install mpi4py
```

lakukan di maser:

Buat program mpibubble sort menggunakan bahasa pemrograman python.
```bash
nano bubble4.py
```
isi bubble4.py dengan program bubble sort contohnya:

```bash
import time
def bubble_sort(arr):
    arr_len = len(arr)

    for i in range(arr_len - 1):
        flag = 0

        for j in range(0, arr_len - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                flag = 1

        if flag == 0:
            break

    return arr
time_start=time.time()

arr = [10, 6, 8, 3, 2, 7, 5, 4, 1, 9]
sorted_arr = bubble_sort(arr)
print("List sorted with bubble sort in ascending order:", sorted_arr)

time_end=time.time()

waktu_eksekusi=time_end-time_start

print ("waktu eksekusi=",waktu_eksekusi)
```

Gunakan perintah berikut untuk mengeksekusi program tersebut:
```bash
mpirun -np <jumlahprosesor> -host <daftar host> python3 test.py
```
Sesuaikan dengan program yang akan dijalankan:
```bash
mpirun -np 3 -host master, slave1,slave2,slave3  python3 bubble4.py
```


