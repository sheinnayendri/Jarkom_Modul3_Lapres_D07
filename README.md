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

![image](https://user-images.githubusercontent.com/48936125/100202565-0f834c80-2f34-11eb-8b9a-bb594a4f1113.png)

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
![image](https://user-images.githubusercontent.com/48936125/100203579-5e7db180-2f35-11eb-92cd-e7068313fd80.png

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
