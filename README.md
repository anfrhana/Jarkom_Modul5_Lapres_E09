# Jarkom_Modul5_Lapres_E09
## Melakukan pembagian IP dengan metode CIDR
* Subneting
<img width="501" alt="subnet cidr" src="https://user-images.githubusercontent.com/61228737/103260134-1d167100-49cf-11eb-9889-7c7ca8da0932.png">

* Tree
![image](https://user-images.githubusercontent.com/61223768/103257994-11727c80-49c6-11eb-8955-2f24e82bd36c.png)

## A. Topologi
Berdasarkan gambar topologi di atas, dibuatlah file topologi.sh sebagai berikut:
```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &
uml_switch -unix switch4 > /dev/null < /dev/null &
uml_switch -unix switch5 > /dev/null < /dev/null &
uml_switch -unix switch6 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.70.41 eth1=daemon,,,switch5 eth2=daemon,,,switch3 mem=96M &
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch1 eth1=daemon,,,switch6 eth2=daemon,,,switch5  mem=96M &
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch3 eth1=daemon,,,switch4 eth2=daemon,,,switch2  mem=96M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 mem=128M &
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch1 mem=128M &

# Klien
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch6 mem=96M &
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch4 mem=96M &
```
### Client
## B. Setting interface
Melakukan uncomment pada `net.ipv4.ip_forward=1` di router **SURABAYA, BATU, KEDIRI**. Dilakukan dengan mengetikkan `nano /etc/sysctl.conf`, untuk mengaktifkan perubahan digunakan `sysctl -p`
Melakukan setting interfaces pada masing masing UML dengan setting pada file `/etc/network/interfaces`
### SURABAYA (Router)
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.70.42
netmask 255.255.255.252
gateway 10.151.70.41

auto eth1
iface eth1 inet static
address 192.168.2.1
netmask 255.255.255.252

auto eth2
iface eth2 inet static
address 192.168.5.1
netmask 255.255.255.252
```
### BATU (Router dan DHCP Relay)
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.5.2
netmask 255.255.255.252
gateway 192.168.5.1

auto eth1
iface eth1 inet static
address 192.168.4.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address 10.151.71.81
netmask 255.255.255.248
```
### KEDIRI (Router dan DHCP Relay)
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.70.42
netmask 255.255.255.252
gateway 10.151.70.41

auto eth1
iface eth1 inet static
address 192.168.2.1
netmask 255.255.255.252

auto eth2
iface eth2 inet static
address 192.168.5.1
netmask 255.255.255.252
```
### MALANG (DNS Server)
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.71.82
netmask 255.255.255.248
gateway 10.151.71.81
```
### MADIUN (Web Server)
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.1.2
netmask 255.255.255.248
gateway 192.168.1.1
```
### PROBOLINGGO (Web Server)
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.1.3
netmask 255.255.255.248
gateway 192.168.1.1
```
### SIDOARJO
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```
### GRESIK
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```
## C. Routing
Melakukan setting pada UML SURABAYA
```
ip route add 10.151.71.80/29 via 192.168.5.2
ip route add 192.168.4.0/23 via 192.168.5.2
ip route add 192.168.0.0/22 via 192.168.2.2
```
## D. DHCP
Edit `/etc/default/isc-dhcp-server` pada DHCP Server (MOJOKERTO) tambahkan interface `eth0` untuk INTERFACESv4.
Edit pada `/etc/dhcp/dhcpd.conf` dengan konfigurasi sebagai  berikut:
```
subnet 10.151.71.80 netmask 255.255.255.248 {
}

subnet 192.168.4.0 netmask 255.255.255.0 {
	range 192.168.4.2 192.168.4.254;
	option routers 192.168.4.1;
	option broadcast-address 192.168.4.255;
	option domain-name-servers 10.151.71.82;
	option domain-name-servers 202.46.129.2;
	default-lease-time 600;
	max-lease-time 600;
}

subnet 192.168.0.0 netmask 255.255.255.0 {
	range 192.168.0.2 192.168.0.254;
	option routers 192.168.0.1;
	option broadcast-address 192.168.0.255;
	option domain-name-servers 10.151.71.82;
	option domain-name-servers 202.46.129.2;
	default-lease-time 600;
	max-lease-time 600;
}
```
Lakukan DHCP restart dengan `service isc-dhcp-server`
Lakukan `service networking restart` pada UML GRESIK dan Sidoarjo

## SOAL

**1.  Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi SURABAYA menggunakan iptables, namun Bibah tidak ingin kalian menggunakan MASQUERADE**
Ditambahkan perintah iptables sebagai berikut di SURABAYA:
`iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.70.42`
* Testing
1. Lakukan ping di seluruh UML
2. Apabila hasilnya adalah sebagai berikut, maka konfigurasi berhasil

**2. Kalian diminta untuk mendrop semua akses SSH dari luar Topologi (UML) Kalian pada server yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan**
Ditambahkan perintah iptables sebagai berikut di SURABAYA:
```iptables -N LOGGING
iptables -A FORWARD -p tcp --dport 22 -d 10.151.71.80/29 -i eth0 -j LOGGING
iptables -A LOGGING -m limit --limit 5/min -j LOG --log-prefix "iptables_FORWARD_denied: " --log-level 7
iptables -A LOGGING -j DROP
```
* Testing
1. Pada PUTTY, ketikkan nc 10.151.71.82 22
2. Pada UML MALANG dan MOJOKERTO, ketikkan perintah nc -l -p 22
3. Ketikkan sesuatu
4. Apabila hasilnya seperti dibawah, maka konfigurasi berhasil

**3. Karena tim kalian maksimal terdiri dari 3 orang, Bibah meminta kalian untuk membatasi DHCP dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP**
Ditambahkan perintah iptables sebagai berikut di MALANG (DNS Server) dan MOJOKERTO (DHCP Server):
```
iptables -N LOGGING
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j LOGGING
iptables -A LOGGING -m limit --limit 5/min -j LOG --log-prefix "iptables_FORWARD_denied: " --log-level 7
iptables -A LOGGING -j DROP
```
* Testing
1. Lakukan ping ke IP MALANG atau IP MOJOKERTO di 4 UML
2. Apabila hasilnya seperti dibawah, maka konfigurasi berhasil

**4-5. kemudian kalian diminta untuk membatasi akses ke MALANG yang berasal dari SUBNET SIDOARJO dan SUBNET GRESIK dengan peraturan sebagai berikut: Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin sampai Jumat5. Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap harinya**
```
#No 4
iptables -A INPUT -s 192.168.4.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -s 192.168.4.0/24 -m time --timestart 17:01 --timestop 06:59 -j REJECT
iptables -A INPUT -s 192.168.4.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Sat,Sun -j REJECT
 
#No 5
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 07:01 --timestop 16:59 -j REJECT

```
* Testing
1. Ubah tanggal dan jam hari ini dengan perintah date -s '2020-12-29 08:00:00' (sesuaikan dengan soal)
2. Cek tanggal sudah benar atau belum dengan perintah date
3. Lakukan ping IP MALANG pada UML GRESIK dan SIDOARJO
4. Apabila hasilnya seperti dibawah, maka konfigurasi berhasil

**6. SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80**
```
iptables -A PREROUTING -t nat -d 10.151.71.82 -p tcp --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.1.2:80
iptables -A PREROUTING -t nat -d 10.151.71.82 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.3:80
```
* Testing
1. Pada PUTTY, ketikkan nc 10.151.71.82 22
2. Pada UML MADIUN dan PROBOLINGGO, ketikkan perintah nc -l -p 22
3. Ketikkan sesuatu
4. Apabila hasilnya seperti dibawah, maka konfigurasi berhasil

**7. Bibah ingin agar semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap UML yang memiliki aturan drop**
Pada UML SURABAYA
```
iptables -N LOGGING 
iptables -A FORWARD -j LOGGING 
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IP Tables Packet Dropped: " --log-level 4 
iptables -A LOGGING -j DROP
```
Pada UML MALANG dan MOJOKERTO
```
iptables -N LOGGING
iptables -A INPUT -j LOGGING
iptables -A OUTPUT -j LOGGING
iptables -A LOGGING -j LOG --log-prefix "IP Tables Packet Dropped: " --log-level 4
iptables -A LOGGING -j DROP
```
* Testing
1. Jalankan iptables
