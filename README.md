# Jarkom_Modul3_Lapres_D07
Laporan Resmi Modul 3 Praktikum Jaringan Komputer 2020
#
1. Naufal Adam Kuncoro (05111740000155)
2. Sheinna Yendri (05111840000038)
#

## Pengerjaan Soal
1. [Soal1](#soal1)
2. [Soal2](#soal2)
3. [Soal3-6](#soal3-6)
4. [Soal7-11](#soal7-11)
5. [Soal12](#soal12)
#

### Soal1
#### Setting Topologi
#
- Membuat file **topologi.sh** yang berisi:
```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.78.33 eth1=daemon,,,switch1 eth2=daemon,,,switch3 eth3=daemon,,,switch2 mem=256M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=160M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T TUBAN -e linux ubd0=TUBAN,jarkom umid=TUBAN eth0=daemon,,,switch2 mem=128M &

# Klien
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch1 mem=64M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch1 mem=64M &
xterm -T BANYUWANGI -e linux ubd0=BANYUWANGI,jarkom umid=BANYUWANGI eth0=daemon,,,switch3 mem=64M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch3 mem=64M &
```

- Kemudian setelah masuk ke UML, pada router SURABAYA lakukan setting sysctl dengan mengetikkan perintah ```nano /etc/sysctl.conf```. Hilangkan tanda pagar (#) pada bagian ```net.ipv4.ip_forward=1```. Lalu mengetikkan ```sysctl -p``` untuk mengaktifkan perubahan yang ada. Dengan mengaktifkan fungsi IP Forward ini maka Linux nantinya dapat menentukan jalur mana yang dipilih untuk mencapai jaringan tujuan.

- Lalu dilakukan setting IP pada setiap UML dengan mengetikkan ```nano /etc/network/interfaces``` Lalu setting sebagai berikut:

**SURABAYA (Sebagai Router / DHCP Relay)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.78.34
netmask 255.255.255.252
gateway 10.151.78.33

auto eth1
iface eth1 inet static
address 192.168.0.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address 192.168.1.1
netmask 255.255.255.0

auto eth3
iface eth3 inet static
address 10.151.79.65
netmask 255.255.255.248
```

**MALANG (Sebagai DNS Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.79.66
netmask 255.255.255.248
gateway 10.151.79.65
```

**MOJOKERTO (Sebagai Proxy Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.79.67
netmask 255.255.255.248
gateway 10.151.79.65
```

**TUBAN (Sebagai DHCP Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.79.68
netmask 255.255.255.248
gateway 10.151.79.65
```

- Kemudian restart network dengan mengetikkan ```service networking restart``` di setiap UML.

- Setelah itu melakukan export proxy pada setiap UML dengan sintaks seperti di bawah ini:
```
export http_proxy=”http://DPTSI-563019-73c30:fb3e7@proxy.its.ac.id:8080”
export https_proxy=”http://DPTSI-563019-73c30:fb3e7@proxy.its.ac.id:8080”
export ftp_proxy=”http://DPTSI-563019-73c30:fb3e7@proxy.its.ac.id:8080”
```
- Serta memberikan perintah ```apt-get update``` juga pada setiap UML untuk melakukan update.

### Soal2
#### SURABAYA menjadi DHCP Relay
#
**Pada UML SURABAYA**
- Install isc-dhcp-relay dengan perintah ```apt-get install isc-dhcp-relay```.
- Kemudian isikan IP Servers dengan IP **TUBAN** sebagai DHCP Servernya, yaitu ```10.151.79.68```.
- Kemudian isikan interfaces dengan ```eth1 eth2```.
- Kosongkan berikut-berikutnya:

![image](https://user-images.githubusercontent.com/48936125/100202680-393c7380-2f34-11eb-8f37-672b1904c4ba.png)

### Soal3-6
**Pada UML TUBAN**
- Install isc-dhcp-server dengan perintah ```apt-get install isc-dhcp-server```.
- Kemudian mengedit file dengan perintah ```nano /etc/default/isc-dhcp-server```. Bagian interface diisikan ```eth0```.

![image](https://user-images.githubusercontent.com/48936125/100202718-478a8f80-2f34-11eb-98dd-a3d4b927b52a.png)

- Kemudian mengedit file dengan perintah ```nano /etc/dhcp/dhcpd.conf```. Ditambahkan sesuai gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/100210125-35f9b580-2f3d-11eb-8ddf-7727d09186d2.png)

- ```range 192.168.0.10 192.168.0.100;``` dan ```range 192.168.0.110 192.168.0.200;``` merupakan jawaban soal no 3 (SUBNET 1)
- ```range 192.168.1.50 192.168.1.70;``` merupakan jawaban soal no 4 (SUBNET 3)
- ```option domain-name-servers 10.151.79.66, 202.46.129.2;``` merupakan jawaban soal no 5
- ```default-lease-time 300;``` dan ```max-lease-time 300;``` merupakan jawaban soal no 6 (SUBNET 1)
- ```default-lease-time 600;``` dan ```max-lease-time 600;``` merupakan jawaban soal no 6 (SUBNET 3)

- Ketika service isc-dhcp-server di-restart akan berhasil:

![image](https://user-images.githubusercontent.com/48936125/100202646-2d50b180-2f34-11eb-85b9-9952c47adf95.png)

Untuk testing dilakukan pada 4 UML klien, yaitu mengubah file ```/etc/network/interfaces``` menjadi:
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```

Kemudian dilakukan restart network: ```service networking restart```.
Maka ketika dijalankan ```ifconfig``` dan ```cat /etc/resolv.conf``` akan terlihat pada setiap UML sebagai berikut:

**Pada UML GRESIK (SUBNET 1)**

![image](https://user-images.githubusercontent.com/48936125/100203579-5e7db180-2f35-11eb-92cd-e7068313fd80.png)

![image](https://user-images.githubusercontent.com/48936125/100203570-5aea2a80-2f35-11eb-9d4e-1e83f95b57bd.png)


**Pada UML SIDOARJO (SUBNET 1)**

![image](https://user-images.githubusercontent.com/48936125/100203594-65a4bf80-2f35-11eb-8789-07e56eb10091.png)

![image](https://user-images.githubusercontent.com/48936125/100203661-79502600-2f35-11eb-8eb6-4570c512505b.png)


**Pada UML BANYUWANGI (SUBNET 3)**

![image](https://user-images.githubusercontent.com/48936125/100203751-94bb3100-2f35-11eb-98f8-6da985290666.png)

![image](https://user-images.githubusercontent.com/48936125/100203763-98e74e80-2f35-11eb-98d3-5c35e13bc306.png)


**Pada UML MADIUN (SUBNET 3)**

![image](https://user-images.githubusercontent.com/48936125/100203807-a69cd400-2f35-11eb-89da-d56a2bf4377b.png)

![image](https://user-images.githubusercontent.com/48936125/100203827-aac8f180-2f35-11eb-98c6-a93b120081ca.png)


- Karena sudah sesuai, maka berarti setting DHCP sudah berhasil.

### Soal7-11
- Pada proxy server, yaitu UML **MOJOKERTO** akan diinstall squid terlebih dahulu dengan perintah ```apt-get install squid```.
- Kemudian mengedit file dengan perintah ```nano /etc/squid/squid.conf``` menjadi gambar berikut untuk menjawab soal no 7-11:

![image](https://user-images.githubusercontent.com/48936125/100204709-c4b70400-2f36-11eb-8617-a5a7cfbd459b.png)

- Untuk soal no7, perlu dilakukan pendaftaran username dan password dengan cara meng-install apache-utils terlebih dahulu dengan perintah ```apt-get install apache2-utils```.
- Kemudian masukkan akun baru dengan ```htpasswd -c /etc/squid/passwd userta_d07``` dan masukkan password yang sesuai yaitu ```inipassw0rdta_d07```.
- Pada file ```squid.conf``` ditambahkan perintah berikut:
```
auth_param basic program /usr/lib/squid/ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow USERS
```
- Sehingga saat diakses muncul autentikasi sebagai berikut:

![image](https://user-images.githubusercontent.com/48936125/100205153-5292ef00-2f37-11eb-9721-26039b411a27.png)

- Untuk soal no8-9, ditambahkan file baru yaitu dengan perintah ```nano /etc/squid/acl.conf``` ditambahkan perintah seperti gambar berikut:

![image](https://user-images.githubusercontent.com/48936125/100205316-8b32c880-2f37-11eb-8cdc-73b11daab57a.png)

- Pada baris pertama, dilakukan setting sesuai no8, yaitu pada hari Selasa-Rabu pujul 13.00-18.00, sedangkan pada baris kedua dan ketiga, dilakukan setting sesuai no9, yaitu pada hari Selasa-Kamis pukul 21.00-09.00.

- Pada file ```/etc/squid/squid.conf``` juga ditambahkan baris berikut:
```
include /etc/squid/acl.conf

http_access allow AVAILABLE_WORKING
http_access allow AVAILABLE_WORKING_2
http_access allow AVAILABLE_WORKING_3
```

- Kemudian untuk soal no10, ditambahkan perintah pada ```/etc/squid/squid.conf``` dengan perintah:
```
acl redi dstdomain google.com
deny_info http://monta.if.its.ac.id redi
http_reply_access deny redi
```

- Untuk soal no11, untuk mengubah default error page, kita dapat memindahkan custom error page kita ke folder ```/usr/share/squid/errors/en/```, jadi setelah kita download dengan perintah ```wget 10.151.36.202/ERR_ACCESS_DENIED```, kita copy-kan ke folder tersebut. Mengenai isi default folder tersebut dapat dipindah dahulu ke folder temporary lainnya dengan perintah ```mv /usr/share/squid/errors/en /usr/share/squid/errors/en1```, kemudian meng-copy file kita ke folder tersebut dengan perintah ```cp -r ERR_ACCESS_DENIED /usr/share/squid/errors/en/```.

### Soal12
- Perlu dilakukan setting DNS Server terlebih dahulu pada UML **MALANG**, yaitu dengan instalasi ```apt-get install bind9 -y```.
- Kemudian ```nano /etc/bind/named.conf.local```
- Tambahkan kode seperti gambar berikut:

![image](https://user-images.githubusercontent.com/48936125/100206290-b833ab00-2f38-11eb-8af6-69603fccde70.png)

- Kemudian buat folder baru: ```mkdir /etc/bind/jarkom```
- Dan copy file db.local ke folder yang baru saja dibuat dan mengganti namanya sesuai domain yang diinginkan: ```cp /etc/bind/db.local /etc/bind/jarkom/janganlupa-ta.d07.pw```
- Kemudian buka file ```janganlupa-ta.d07.pw``` dengan perintah: ```nano /etc/bind/jarkom/janganlupata-d07.pw```. Edit sesuai gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/100206420-e1543b80-2f38-11eb-910b-9a3e2b2613bb.png)

- Kemudian setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

- Maka saat proxy domain diubah menjadi ```janganlupa-ta.d07.pw```, proxy berarti diakses.

![image](https://user-images.githubusercontent.com/48936125/100206497-fb8e1980-2f38-11eb-8e1e-fde43982d6f2.png)
