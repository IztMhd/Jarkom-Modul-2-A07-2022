# Jarkom-Modul-2-A07-2022

## Anggota Kelompok

- I Putu Bagus Adhi Pradana (5025201010)
- Izzati Mukhammad (5025201075)
- Muhammad Damas Abhirama (5025201271)

## Nomor 1
WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet

Membuat Topologi Sebagai berikut

![1!](img/1.png)

Konfigurasikan pada node `Ostania` sebagai berikut

    auto eth0
    iface eth0 inet dhcp

    auto eth1
    iface eth1 inet static
      address 192.172.1.1
      netmask 255.255.255.0

    auto eth2
    iface eth2 inet static
      address 192.172.2.1
      netmask 255.255.255.0

    auto eth3
    iface eth3 inet static
      address 192.172.3.1
      netmask 255.255.255.0
      
Lalu pada setiap node lakukan juga konfigurasi sebagai berikut

- WISE

      auto eth0
      iface eth0 inet static
        address 192.172.3.2
        netmask 255.255.255.0
        gateway 192.172.3.1
      
- SSS

      auto eth0
      iface eth0 inet static
        address 192.172.1.2
        netmask 255.255.255.0
        gateway 192.172.1.1
        
- Garden

      auto eth0
      iface eth0 inet static
        address 192.172.1.3
        netmask 255.255.255.0
        gateway 192.172.1.1
        
- Berlint

      auto eth0
      iface eth0 inet static
        address 192.172.2.2
        netmask 255.255.255.0
        gateway 192.172.2.1
        
- Eden

      auto eth0
      iface eth0 inet static
        address 192.172.2.3
        netmask 255.255.255.0
        gateway 192.172.2.1

## Nomor 2
Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses `wise.A07.com` dengan alias `www.wise.A07.com` pada folder wise

    echo 'zone "wise.a07.com" {
        type master;
        file "/etc/bind/wise/wise.a07.com";
    };' > /etc/bind/named.conf.local
    mkdir /etc/bind/wise
    echo "
    \$TTL    604800
    @       IN      SOA     wise.a07.com. root.wise.a07.com. (
                                    2       ; Serial
                            604800          ; Refresh
                            86400           ; Retry
                            2419200         ; Expire
                            604800 )        ; Negative Cache TTL
    ;
    @               IN      NS      wise.a07.com.
    @               IN      A       192.172.2.2 ; IP Wise
    www             IN      CNAME   wise.a07.com.
    " > /etc/bind/wise/wise.a07.com
    service bind9 restart

## Nomor 3
Setelah itu ia juga ingin membuat subdomain eden.wise.A07.com dengan alias `www.eden.wise.A07.com` yang diatur DNS-nya di WISE dan mengarah ke Eden

Pada node `Wise` menambahkan `Eden` sebagai berikut

    echo "
    \$TTL    604800
    @       IN      SOA     wise.a07.com. root.wise.a07.com. (
                                    2       ; Serial
                            604800          ; Refresh
                            86400           ; Retry
                            2419200         ; Expire
                            604800 )        ; Negative Cache TTL
    ;
    @               IN      NS      wise.a07.com.
    @               IN      A       192.172.2.2 ; IP Wise
    www             IN      CNAME   wise.a07.com.
    eden           IN      A       192.172.3.3 ; IP Eden
    www.eden       IN      CNAME   eden.wise.a07.com.
    " > /etc/bind/wise/wise.a07.com
    service bind9 restart

Kemudian menjalankan `ping eden.wise.A07.com` dan `ping www.eden.wise.yyy.com`

## Nomor 4
Buat juga reverse domain untuk domain utama

Pada `Wise` bashrc

    echo '
    zone "wise.a07.com" {
            type master;
            file "/etc/bind/wise/wise.a07.com";
    };

    zone "2.172.192.in-addr.arpa" {
            type master;
            file "/etc/bind/wise/2.172.192.in-addr.arpa";
    };' > /etc/bind/named.conf.local

    echo "
    \$TTL    604800
    @       IN      SOA     wise.a07.com. root.wise.a07.com. (
                                    2       ; Serial
                            604800          ; Refresh
                            86400           ; Retry
                            2419200         ; Expire
                            604800 )        ; Negative Cache TTL
    ;
    2.172.192.in-addr.arpa.   IN      NS      wise.a07.com.
    2                       IN      PTR     wise.a07.com.
    "> /etc/bind/wise/2.172.192.in-addr.arpa
    service bind9 restart

## Nomor 5
Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama

Pada `Wise`

    echo '
    zone "wise.a07.com" {
            type master;
            notify yes;
            also-notify {192.172.3.2;};  //Masukan IP Berlint tanpa tanda petik
            allow-transfer {192.172.3.2;}; // Masukan IP Berlint tanpa tanda petik
            file "/etc/bind/wise/wise.a07.com";
    };

    zone "2.172.192.in-addr.arpa" {
            type master;
            file "/etc/bind/wise/2.172.192.in-addr.arpa";
    };' > /etc/bind/named.conf.local
    service bind9 restart
    
Pada `Berlint`

    apt-get update
    apt-get install bind9 -y
    echo '
    zone "wise.a07.com" {
            type slave;
            masters { 192.172.2.2; }; // Masukan IP Wise tanpa tanda petik
            file "/var/lib/bind/wise.a07.com";
    };
    ' > /etc/bind/named.conf.local
    service bind9 restart
    
`SSS`

    echo "
    nameserver 192.172.2.2
    nameserver 192.172.3.2
    nameserver 192.172.3.3

    " > /etc/resolv.conf    
    
`Garden`

    echo "
    nameserver 192.172.2.2
    nameserver 192.172.3.2
    nameserver 192.172.3.3

    " > /etc/resolv.conf

## Nomor 6
Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.A07.com dengan alias `www.operation.wise.A07.com` yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation

## Nomor 7
Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses `strix.operation.wise.A07.com` dengan alias `www.strix.operation.wise.A07.com` yang mengarah ke Eden
