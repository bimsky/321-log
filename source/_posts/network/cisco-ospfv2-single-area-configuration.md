---
draft: true
comments: true
toc: true
title: 'Cisco: OSPFv2 Single Area Configuration (point to point and broadcast multiaccess)'
date: 2020-09-08T13:06:00Z
updated: 
category:
- network
- ensa
tags:
- cisco
photos: https://res.cloudinary.com/bimagv/image/upload/v1608573469/banner/cisco-ensa_cavb6w.png
excerpt: Belajar mengimplementasikan OSPF pada jaringan point to pont dan broad...
---
# **Objectives**

Kalau di materi [sebelumnya](https://8log.js.org/2020/08/16/network/cisco-ospfv2-single-area-point-to-point/), kita sudah mengimplementasikan OSPF single area dengan topologi point to point, sekarang kita akan mencoba menerapkannya ke jaringan `point to point and broadcast multiaccess`.

# **Overview**

Sekarang kita akan mencoba untuk mengkonfigurasi Routing Dynamic OSPF, untuk yang ingin tahu cara melihat dan menganalisis konfigurasi OSPF sebelumnya sudah dijelaskan di [sini](https://8log.js.org/2020/09/08/network/cisco-ospfv2-single-area-verify/).

# **Lab**

![](/images/2020-08-09_sel_07-21-22.png)

Pada materi kali ini akan sangat berguna sekali untuk melatih kemampuan kita dalam mengkonfigurasi router dengan OSPF dengan beberapa ketentuan yang disediakan sebagai berikut.

# **Requirements**

Use process ID 10 for OSPF activation on all routers.

* Activate OSPF using network statements and inverse masks on the routers in the Headquarters network.
* Activate OSPF by configuring the interfaces of the network devices in the Data Service network, where required.
* Configure router IDs on the multiaccess network routers as follows:
  * BC-1: 6.6.6.6
  * BC-2: 5.5.5.5
  * BC-3: 4.4.4.4
* Configure OSPF so that routing updates are not sent into networks where they are not required.
* Configure router BC-1 with the highest OSPF interface priority so that it will always be the designated router of the multiaccess network.
* Configure a default route to the ISP cloud using the exit interface command argument.
* Automatically distribute the default route to all routers in the network.
* Configure the OSPF routers so that the Gigabit Ethernet interface cost will be 10 and the Fast Ethernet cost will be 100.
* Configure the OSPF cost value of P2P-1 interface Serial0/1/1 to 50.
* Configure the hello and dead timer values on the interfaces that connect P2P-1 and BC-1 to be twice the default values.

# **Tabel Address**

![](/images/2020-08-09_sel_07-23-29.png)

![](/images/2020-08-09_sel_07-23-44.png)

# **1. Mengemplementasikan OSPF di Router2 Headquarters**

Pertama-tama kita akan mengerjakan network di **Headquarters** yang berupa point to point.

Kita telah mengetahui di materi [kemarin]() bahwa di jaringan point to point tidak ada election DR/BDR, maka kita tidak perlu repot menyetel Router ID untuk router2 P2P ini.

![](/images/2020-09-09_rab_12-55-44.png)

Untuk itu mari kita langsung saja jalankan OSPF routing process di mulai pada router P2P-1 ini

    P2P-1(config)#router ospf 10

## 1.A. Mencari router yang directly connected dengan P2P-1

Kita akan mencari router interface yang directly connected alias terhubung secara langsung dengan Router P2P ini, langsung saja jalankan `do show ip route connected`

     C   10.0.0.0/30  is directly connected, Serial0/1/0
     C   10.0.0.8/30  is directly connected, Serial0/1/1
     C   10.0.0.12/30  is directly connected, Serial0/2/0

Tampak yang IP yang muncul yaitu berupa network2 yang terkoneksi langsung ke router P2P-1

Sedangkan kalau kita menggunakan command `do show ip route` saja

    Gateway of last resort is not set
    
         10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
    C       10.0.0.0/30 is directly connected, Serial0/1/0
    L       10.0.0.1/32 is directly connected, Serial0/1/0
    C       10.0.0.8/30 is directly connected, Serial0/1/1
    L       10.0.0.9/32 is directly connected, Serial0/1/1
    C       10.0.0.12/30 is directly connected, Serial0/2/0
    L       10.0.0.13/32 is directly connected, Serial0/2/0

Maka akan menampilkan juga interface2 milik router P2P-1 itu sendiri yang ada pada baris entri kode **L** alias **Local** yang kemudian meneruskan informasi ke network **C** Alias (directly) **Connected**, itu sekedar informasi.

Dari sini kita telah menemukan network2 yang directly connected, yaitu `10.0.0.0/30`, `10.0.0.8/30`, dan `10.0.0.12/30` yang juga sudah ada keterangannnya di topologi

![](/images/2020-09-09_rab_14-01-51.png)

## 1.B. Menyetel P2P-1 dengan Network statements dan inverse mask

Kita telah melakukan cara ini pada materi [sebelumnya](https://8log.js.org/2020/08/16/network/cisco-ospfv2-single-area-point-to-point/#2-B-Menyetel-R1-dengan-Network-statements-dan-Wilcard-Mask). Sekarang yang kita butuhkan adalah command

> Router(config-router)# **network** network-address wildcard-mask **area** area-id

Jadi kita mendaftarkan network2nya satu-persatu, lalu memasukkan wilcard mask (kalau lupa menghitung wilcard mask caranya di [materi yang lalu](https://8log.js.org/2020/08/16/network/cisco-ospfv2-single-area-point-to-point/#2-A-Cara-Mengkonversikan-subnet-ke-desimal) dan ingat karena single area berarti area 0.

    P2P-1(config-router)#network 10.0.0.0 0.0.0.3 area 0
    P2P-1(config-router)#network 10.0.0.8 0.0.0.3 area 0
    P2P-1(config-router)#network 10.0.0.12 0.0.0.3 area 0

Lanjut.

## 1.C. Mengatur interface cost pada Router OSPF P2P-1

Sekarang saatnya kita akan mengatur dahulu pada router OSPF ini sehingga pada interface GigabitEthernet dan FastEthernet disetel cost yang telah ditentukan, yaitu

* **GigabitEthernet=10**
* **FastEthernet=100**

perintahnya ialah `auto-cost reference-bandwidth <bandwidth>` \[ **Gbps** | **Mbps** \]

Sementara itu ...

_"apa itu OSPF cost?"_

_"kenapa ada OSPF cost?"_

_"kenapa digunakan?"_

_"dan apa itu reference-bandwith?"_

_"apa cost semacam ini hanya ada di OSPF saja?"_

Oke, cara belajar kita adalah **_learning by doing_** sembari ngoprek, sembari belajar :)

Sebelum itu kita harus tahu fungsi dari penerapan dari cost di OSPF ini yang tidak lain dan tidak bukan ialah untuk **metric** :) Yang merupakan suatu nilai yang digunakan untuk menuju ke destination.

![](/images/2020-09-09_rab_15-17-27.png)

Simpelnya metrik ini membantu router memilih rute terbaik dimana nilai metrik setiap routing protokol berbeda-beda.

Untuk OSPF sendiri untuk nilai metric nya menggunakan bandwith.

> Pada **Dynamic Routing Protocol lain** contoh seperti **RIP**, ia memakai `Hop` untuk metricnya dan **EIGRP** memakai `Bandwidth`, `Delay`, `Reliability`, dan `Load`

Sama seperti EIGRP, OSPF (juga) mendasarkan metriknya secara default pada `bandwidth link`, sehingga OSPF dinilai lebih baik dibanding RIP yang mengandalkan metrik `hop-count`.

Oke.

Lanjut.

Kalau seperti itu maka kita musti tahu berapa bandwith yang dipakai dahulu, karena logikanya sebuah paket akan lebih banyak overhead jika melewati medium yang kapasitas bandwithnya kecil.

Contoh saat melintas Serial link yang bandwithnya ialah 56Kbps maka paket OSPF akan lebih lama sampainya daripada melintasi Ethernet link yang bandwithnya 100Mbps.

Dan sekarang kita akan lihat bandwith dan cost ini **berbanding terbalik**, karena bandwith yang lebih tinggi seperti Ethernet link (100Mbps) akan mempunyai _cost yang lebih kecil_ dimana ini merupakan **rute terbaik**. Sedangkan bandwith yang lebih rendah seperti Serial link (56Kbps) akan memiliki _cost yang lebih tinggi_.

Sekedar info berikut default nilai cost pada beberapa interface

| Interface Type | bandwidth | Metric Calculation | Cost |
| :---: | :---: | :---: | --- |
| Ethernet Link | 10Mbps | 100000000/10000000 = 10 | 10 |
| FastEthernet Link | 100Mbps | 100000000/100000000 = 1 | 1 |
| Serial Link | 1544Kbps(default) | 100000000/1544000 = 64.76 | 64 |

### Konfigurasi interface cost menggunakan Auto Cost Reference Bandwith

Kalau mengikuti defaultnya, maka reference bandwith berjumlah 100 Mbps

    BimaRR# show ip ospf | include Reference
     Reference bandwidth unit is 100 mbps

Terlihat terlalu kecil.

Namun, menurut requirement diawal:

> **_Configure the OSPF routers so that the Gigabit Ethernet interface cost will be 10 and the Fast Ethernet cost will be 100._**

Kalau memilih reference bandwith 10000 Mbps

![](/images/2020-10-09_kam_00-06-57.png)

Berarti kita boleh mempunyai kabel dengan kapasitas bandwithnya sekitar 100 Mbps untuk FastEthernet dan 1 Gbps (1000Mbps) untuk GigabitEthernet

Namun, jika kita memilih reference bandwith 1000 Mbps  
![](/images/2020-10-09_kam_00-03-17.png)

Sedangkan reference bandwith 1000 terlihat lebih standar untuk kebutuhan link berkapasitas 10 Mbps untuk FastEthernet dan 100 Mbps untuk GigabitEthernet.

Dan sebagai pembanding sekarang kita beralih ke tabel berikut

### **Cost of common lines**

| Interface Type | bandwidth | Metric Calculation | Cost |
| :---: | :---: | :---: | --- |
| 56 Kbps line | 56Kbps | 100000000/56000 = 1785.71 | 1785 |
| 64 Kbps line | 64Kbps | 100000000/64000 = 1562.5 | 1562 |
| 128 Kbps line | 128Kbps | 100000000/128000 = 781.25 | 781 |
| 512 Kbps line | 512Kbps | 100000000/512000 = 195.31 | 195 |
| 1 Mbps line | 1Mbps | 100000000/1000000 = 100 | 100 |
| 10 Mbps line | 10Mbps | 100000000/10000000 = 10 | 10 |
| 100 Mbps line | 100Mbps | 100000000/100000000 = 1 | 1 |
| 1 Gbps line | 1Gbps | 100000000/100000000 0= 0.1 | 1 |
| 10 Gbps line | 10Gbps | 100000000/10000000000 = 0.01 | 1 |

Mungkin konsumsi bandwith untuk cost requirements tidak terlalu tinggi (FastEthernet 1 Mbps dan GigabitEthernet 10 Mbps), maka kita akan memilih yang standar saja, yaitu `auto-cost bandwith-reference 1000`

    P2P-1(config-router)# auto-cost reference-bandwidth 1000
    % OSPF: Reference bandwidth is changed.
            Please ensure reference bandwidth is consistent across all routers.

Lihat pesan tersebut, saat kita merubah reference bandwith pada satu router OSPF akan menyuruh kita untuk mengubahnya juga untuk setiap router juga. Untuk itu kita nanti akan mengkonfigurasinya kepada semua router.

Dan ada satu requirement lagi terkait OSPF cost ini, yaitu

> **_Configure the OSPF cost value of P2P-1 interface Serial0/1/1 to 50._**

Jadi selain mengubah `auto-cost reference-bandwith` yang mana OSPF akan mengkalkulasi sendiri kira-kira satu interface akan menggunakan bandwith berapa, kita juga akan menkonfigurasi berapa nilai cost yang akan digunakan OSPF untuk interface serial 0/1/1 pada router ini

    P2P-1(config)#int serial 0/1/1
    P2P-1(config-if)#ip ospf cost 50

Keknya panjang amat :)

Biar tambah panjang, terakhir sekalian ...

### Contoh perintah verifikasi

Selanjutnya apabila nanti ingin melihat cost dan bandwithnya kira-kira contohnya seperti ini perintahnya.

Untuk melihat bandwith pakai perintah `show interfaces FastEthernet 0/0 | include BW`

    BimaRR# show interfaces FastEthernet 0/0 | include BW
      MTU 1500 bytes, BW 100000 Kbit/sec, DLY 100 usec

Dan muncullah `100000 Kbit/sec`

Nanti dari sini OSPF akan menghitung costnya

> **Cost = Reference bandwidth (default 100 Mbps) / Interface bandwidth in bps.**
>
> atau **Cost = 10^8 / interface bandwidth in bps**

Cost = `100.000 kbps reference bandwidth / 100.000 interface bandwidth = 1`

Dan perintah verifikasi untuk menampilkan cost dengan `show ip ospf interface FastEthernet 0/0 | include Cost`

    BimaRR# show ip ospf interface FastEthernet 0/0 | include Cost
      Process ID 1, Router ID 192.168.1.1, Network Type BROADCAST, Cost: 1

Nah, itulah hasil costnya, yaitu 1.

(^_^)

## 1.D. Mengatur Hello dan Dead timer values antara P2P-1 dan BC-1

Setelah itu lanjut ke paket OSPF Hello dan Dead interaval, OSPF menggunakan paket kedua paket ini untuk dapat memeriksa apakah router neighbornya masih hidup atau tidak.

![](/images/2020-10-09_kam_16-16-59.png)

Sekarang kita akan menyetel nilai hello dan dead timer pada interface S0/2/0 atau antara P2P-1 dan BC-1 yang merupakan proses checkup antara network point to point dan network multi point.

* **Hello interval** ini menentukan seberapa sering kita mengirim paket hello
* Sedangkan **Dead interval** menentukan berapa lama router harus menunggu paket hello sebelum menyatakan router neighbor mati

Requirementnya:

> Configure the hello and dead timer values on the interfaces that connect P2P-1 and BC-1 to be twice the default values.

Dua kali nilai default, nilai defaultnya ialah Hello 10 seconds dan Dead timernya 40 second, maka dua kali nilai tersebut yakni 20 seconds dan 80 seconds.

    P2P-1(config)#interface serial 0/2/0
    P2P-1(config-if)#ip ospf hello-interval 20
    P2P-1(config-if)#ip ospf dead-interval 80

### Contoh perintah verifikasi

Untuk dapat mengetahui niai Dead dan hello interval dengan menggunakan perintah `show ip ospf interface <interface> | include intervals`

    BimaRR #show ip ospf interface FastEthernet 0/0 | include intervals
      Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5

## 1.E Lakukan konfigurasi pada router P2P-2

**_Pada section 1.A sampai 1.D_** sembari mengkonfigurasi router P2P-1 sembari juga menjelaskan detailnya, karena cukup sayang kalau tidak dicatat :)

Pada bagian ini, saya asumsikan kita telah mengetahui langkah dalam mengkonfigurasi router tersebut, sekarang kita tinggal implementasikan pembelajaran tadi ke router P2P-2

### Dimulai mengaktifkan process OSPF di P2P-2

    P2P-2(config)#router ospf 10

### Mengkonfigurasi Network2 di P2P-2

    P2P-2(config-router)#do show ip route connected
     C   10.0.0.0/30  is directly connected, Serial0/1/0
     C   10.0.0.4/30  is directly connected, Serial0/1/1
     C   192.168.1.0/24  is directly connected, GigabitEthernet0/0/0
     C   192.168.2.0/24  is directly connected, GigabitEthernet0/0/1
    
    P2P-2(config-router)#network 10.0.0.0 0.0.0.3 area 0
    12:52:56: %OSPF-5-ADJCHG: Process 10, Nbr 10.0.0.13 on Serial0/1/0 from LOADING to FULL, Loading Done
    .0
    
    P2P-2(config-router)#network 10.0.0.4 0.0.0.3 area 0
    P2P-2(config-router)#network 192.168.1.0 0.0.0.255 area 0
    P2P-2(config-router)#network 192.168.2.0 0.0.0.255 area 0

### Mempassivekan interface G0/0/0 dan G0/0/1 di P2P-2

![](/images/2020-10-09_kam_14-13-03.png)

    P2P-2(config-router)#passive-interface g0/0/0
    P2P-2(config-router)#passive-interface g0/0/1

### Mengubah auto cost reference bandwith di P2P-2

    P2P-2(config-router)#auto-cost reference-bandwidth 1000
    % OSPF: Reference bandwidth is changed.
            Please ensure reference bandwidth is consistent across all routers.

## 1.F Lakukan konfigurasi pada router P2P-3

### Dimulai mengaktifkan process OSPF di P2P-3

    P2P-3(config)#router ospf 10

### Mengkonfigurasi Network2 di P2P-3

    P2P-3(config-router)#do show ip route connected
     C   10.0.0.4/30  is directly connected, Serial0/1/0
     C   10.0.0.8/30  is directly connected, Serial0/1/1
     C   192.168.3.0/28  is directly connected, GigabitEthernet0/0/0
    
    P2P-3(config-router)#network 10.0.0.4 0.0.0.3 area 0
    13:04:28: %OSPF-5-ADJCHG: Process 10, Nbr 192.168.2.1 on Serial0/1/0 from LOADING to FULL, Loading Done
    
    P2P-3(config-router)#network 10.0.0.8 0.0.0.3 area 0
    13:04:43: %OSPF-5-ADJCHG: Process 10, Nbr 10.0.0.13 on Serial0/1/1 from LOADING to FULL, Loading Done
    
    P2P-3(config-router)#network 192.168.3.0 0.0.0.15 area 0

### Mempassivekan interface G0/0/0 di P2P-3

    P2P-3(config-router)#passive-interface g0/0/0

### Mengubah auto cost reference bandwith di P2P-3

    P2P-3(config-router)#auto-cost reference-bandwidth 1000
    % OSPF: Reference bandwidth is changed.
            Please ensure reference bandwidth is consistent across all routers.

# **2. Mengemplementasikan OSPF di Router2 Data Services**

Sekarang kita akan mengkonfigurasi Router2 pada multiaccess network ini yang mana router2 pada network ini akan melakukan OSPF election DR/BDR, maka nanti kita akan menyetel sebuah Router ID pada setiap Router BC ini.

sesuai requirement:

> Configure router IDs on the multiaccess network routers as follows:
>
> * BC-1: 6.6.6.6
> * BC-2: 5.5.5.5
> * BC-3: 4.4.4.4

![](/images/2020-10-09_kam_14-29-50.png)

Selain itu kita lihat pada **ISP Cloud**, nanti disitu kita akan melakukan routing ke ISP Cloud dengan `default static route` yang nantinya akan `di-redistribute` oleh BC-1 ke semua router2 OSPF pada network point to point.

## 2.A. Lakukan konfigurasi pada router BC-1

### Dimulai mengaktifkan process OSPF di BC-1

    BC-1(config)#router ospf 10

### Memberi Router ID di BC-1

    BC-1(config-router)#router-id 6.6.6.6

### Mempassivekan interface S0/1/1

Berdasarkan soal requirements:

> **Configure OSPF so that routing updates are not sent into networks where they are not required.**

dalam kasus ini interface yang kearah ISP cloud tentu tidak membutuhkan paket update dari OSPF, maka kita akan passivekan interface tersebut

    BC-1(config-router)#passive-interface s0/1/1

### Meredistribute default static route ke router2 yang menjalankan OSPF

Berdasarkan soal requirements:

> **Automatically distribute the default route to all routers in the network.**

Jadi, kita pada BC-1 ini nanti akan di konfigurasi default static route pada interface yang mengarah ke ISP Cloud, maka hal yang dilakukan untuk network point to point adalah meredistribute informasi tersebut dengan perintah

    BC-1(config-router)#default-information originate

### Mengubah auto cost reference bandwith di BC-1

    BC-1(config-router)#auto-cost reference-bandwidth 1000
    % OSPF: Reference bandwidth is changed.
            Please ensure reference bandwidth is consistent across all routers.

### Mengkonfigurasi Network2 di BC-1

Sesuai requirements pada soal:

> **Activate OSPF by configuring the interfaces of the network devices in the Data Service network, where required.**

Sekarang kita akan memakai cara yang berbeda untuk memasukkan network2 pada router BC-1 ini, yaitu dengan mengkonfig OSPF dengan command `ip ospf 10 area 0` di interfacesnya.

Untuk itu kita tampilkan dahulu interface2 di router BC-1

    BC-1(config)#do show ip interface brief
    Interface              IP-Address      OK? Method Status                Protocol 
    GigabitEthernet0/0/0   10.0.1.1        YES NVRAM  up                    up 
    GigabitEthernet0/0/1   unassigned      YES NVRAM  administratively down down 
    Serial0/1/0            10.0.0.14       YES NVRAM  up                    up 
    Serial0/1/1            64.0.100.2      YES NVRAM  up                    up 
    Vlan1                  unassigned      YES unset  administratively down down

dan interface router yang akan menjadi OSPF neighbor adjacency adalah `GigabitEthernet0/0/0` dan `Serial0/1/0`

    BC-1(config)#int g0/0/0
    BC-1(config-if)#ip ospf 10 area 0
    
    BC-1(config-if)#int s0/1/0
    BC-1(config-if)#ip ospf 10 area 0

### Mengubah OSPF interface priority pada BC-1 G0/0/0

Sesuai requirements pada soal:

> **Configure router BC-1 with the highest OSPF interface priority so that it will always be the designated router of the multiaccess network.**

kita akan menjadikan interface neighbor adjacency (yang menuju ke multiaccess network) BC-1 yaitu G0/0/0 menjadi high priority

    BC-1(config-if)#int g0/0/0
    BC-1(config-if)#ip ospf priority 255

### Mengatur Hello dan Dead timer values antara BC-1 dan P2P-1

![](/images/2020-10-09_kam_16-16-59.png)

Sebelumnya kita sedah mengkonfigurasi pada interface S0/2/0 P2P-1, sekarang saatnya menyetel pada interface S0/1/0 BC-1 supaya nanti kedua router tersebut (tepatnya kedua network tersebut) bisa menjadi neighbor adjacency

    BC-1(config)#int s0/1/0
    BC-1(config-if)#ip ospf hello-interval 20
    BC-1(config-if)#ip ospf dead-interval 80
    15:05:11: %OSPF-5-ADJCHG: Process 10, Nbr 10.0.0.13 on Serial0/1/0 from LOADING to FULL, Loading Done

### Melakukan routing ke ISP Cloud router dengan Default static route

Kalau pakai statements `Static default route` ini nanti kita menggunakan destination network address 0.0.0.0 dan subnet mask 0.0.0.0 pada saat melakukan routing seperti statements quad zero mask.

> _Proses routing untuk static default route ini adalah nantinya router melakukan proses pencarian gateway yang akan digunakan oleh router untuk mengirimkan semua paket IP untuk network destination yang tidak diketahui di routing table, sehingga akan diforward ke route 0.0.0.0/0._

Sesuai requiremts di soal:

> Configure a default route to the ISP cloud using the exit interface command argument.

berikut adalah default static route dengan perintah exit interface, yakni dengan melakukan route dengan memasukkan interfacenya langsung (S0/1/1) sebagai media bagi router ISP Cloud.

    BC-1(config)#ip route 0.0.0.0 0.0.0.0 s0/1/1
    %Default route without gateway, if not a point-to-point interface, may impact performance

Sampai langkah ini Router ISP Cloud sudah bisa memping ke BC-1

    ISP(config)#do ping 64.0.100.2
    
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 64.0.100.2, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 1/8/27 ms

## 2.B. Lakukan konfigurasi pada router BC-2

### Dimulai mengaktifkan process OSPF di BC-2

    BC-2(config)#router ospf 10

### Memberi Router ID di BC-2

    BC-2(config-router)#router-id 5.5.5.5

### Mempassivekan interface G0/0/0 di BC-2

    BC-2(config-router)#passive-interface g0/0/0

### Mengubah auto cost reference bandwith di BC-2

    BC-2(config-router)#auto-cost reference-bandwidth 1000
    % OSPF: Reference bandwidth is changed.
    Please ensure reference bandwidth is consistent across all routers.

### Mengkonfigurasi Network2 di BC-2

    BC-2(config-router)#do show ip int br
    Interface              IP-Address      OK? Method Status                Protocol 
    GigabitEthernet0/0/0   192.168.4.1     YES manual up                    up 
    GigabitEthernet0/0/1   10.0.1.2        YES manual up                    up 
    Serial0/1/0            unassigned      YES unset  administratively down down 
    Serial0/1/1            unassigned      YES unset  administratively down down 
    Vlan1                  unassigned      YES unset  administratively down down
    
    BC-2(config-router)#int g0/0/0
    BC-2(config-if)#ip ospf 10 area 0
    BC-2(config-if)#int g0/0/1
    BC-2(config-if)#ip ospf 10 area 0

## 2.C. Lakukan konfigurasi pada router BC-3

### Dimulai mengaktifkan process OSPF di BC-3

    BC-3(config)#router ospf 10

### Memberi Router ID di BC-3

    BC-3(config-router)#router-id 4.4.4.4

### Mempassivekan interface G0/0/0 di BC-3

    BC-3(config-router)#passive-interface g0/0/0

### Mengubah auto cost reference bandwith di BC-3

    BC-3(config-router)#auto-cost reference-bandwidth 1000
    % OSPF: Reference bandwidth is changed.
            Please ensure reference bandwidth is consistent across all routers.

### Mengkonfigurasi Network2 di BC-3

    BC-3(config-router)#do show ip int br
    Interface              IP-Address      OK? Method Status                Protocol 
    GigabitEthernet0/0/0   192.168.5.1     YES manual up                    up 
    GigabitEthernet0/0/1   10.0.1.3        YES manual up                    up 
    Serial0/1/0            unassigned      YES unset  administratively down down 
    Serial0/1/1            unassigned      YES unset  administratively down down 
    Vlan1                  unassigned      YES unset  administratively down down
    
    BC-3(config-router)#int g0/0/0
    BC-3(config-if)#ip ospf 10 area 0
    BC-3(config-if)#int g0/0/1
    BC-3(config-if)#ip ospf 10 area 0

Sampai sini konfigurasi antara network Head Quarters dan Data Services sudah terjalin, dan ping ke internal server sudah berhasil.

Oke.

Mantap.

# **Referensi**

* [https://www.computernetworkingnotes.com/ccna-study-guide/ospf-metric-cost-calculation-formula-explained.html](https://www.computernetworkingnotes.com/ccna-study-guide/ospf-metric-cost-calculation-formula-explained.html "https://www.computernetworkingnotes.com/ccna-study-guide/ospf-metric-cost-calculation-formula-explained.html")
* [h](https://www.cisco.com/c/m/en_us/techdoc/dc/reference/cli/nxos/commands/ospf/auto-cost-ospf.html "https://www.cisco.com/c/m/en_us/techdoc/dc/reference/cli/nxos/commands/ospf/auto-cost-ospf.html")[https://networklessons.com/ospf/ospf-reference-bandwidth](https://networklessons.com/ospf/ospf-reference-bandwidth "https://networklessons.com/ospf/ospf-reference-bandwidth")
* [https://networklessons.com/ospf/ospf-hello-and-dead-interval](https://networklessons.com/ospf/ospf-hello-and-dead-interval "https://networklessons.com/ospf/ospf-hello-and-dead-interval")