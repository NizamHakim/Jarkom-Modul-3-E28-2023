# Jarkom-Modul-3-E28-2023
## Kelompok E28
- Shafa Nabilah Hanin / 5025211222
- Nizam Hakim Santoso / 5025211209

# Lapres Praktikum 3
## Daftar Isi
- [Topologi](#Topologi)
- [Network Configuration](#network-configuration)
- [Files](#files)
- [Soal Praktikum](#soal-praktikum)
  - [No 0](#no-0)
  - [No 1](#no-1)
  - [No 2](#no-2)
  - [No 3](#no-3)
  - [No 4](#no-4)
  - [No 5](#no-5)
  - [No 6](#no-6)
  - [No 7](#no-7)
  - [No 8](#no-8)
  - [No 9](#no-9)
  - [No 10](#no-10)
  - [No 11](#no-11)
  - [No 12](#no-12)
  - [No 13](#no-13)
  - [No 14](#no-14)
  - [No 15](#no-15)
  - [No 16](#no-16)
  - [No 17](#no-17)
  - [No 18](#no-18)
  - [No 19](#no-19)
  - [No 20](#no-20)
  - [Grimoire](#grimoire)
 
## Topologi
Berikut adalah topologi yang digunakan pada praktikum modul 3

![](/images/topologi.png)

## Network Configuration
- **Aura - Router**
  ```
  auto eth0
  iface eth0 inet dhcp

  auto eth1
  iface eth1 inet static
	  address 192.220.1.0
	  netmask 255.255.255.0

  auto eth2
  iface eth2 inet static
	  address 192.220.2.0
	  netmask 255.255.255.0

  auto eth3
  iface eth3 inet static
	  address 192.220.3.0
	  netmask 255.255.255.0

  auto eth4
  iface eth4 inet static
	  address 192.220.4.0
	  netmask 255.255.255.0
  ```
- **Himmel - DHCP**
  ```
  auto eth0
  iface eth0 inet static
	  address 192.220.1.1
	  netmask 255.255.255.0
	  gateway 192.220.1.0
	  up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **Heiter - DNS**
  ```
  auto eth0
  iface eth0 inet static
	  address 192.220.1.2
	  netmask 255.255.255.0
	  gateway 192.220.1.0
	  up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **Denken - DB**
  ```
  auto eth0
  iface eth0 inet static
	  address 192.220.2.1
	  netmask 255.255.255.0
	  gateway 192.220.2.0
	  up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **Eisen - LB**
  ```
  auto eth0
  iface eth0 inet static
	  address 192.220.2.2
	  netmask 255.255.255.0
	  gateway 192.220.2.0
	  up echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
- **Sein, Stark, Revolte, Richter - Clients**
  ```
  auto eth0
  iface eth0 inet dhcp
  ```
- **Lawine, Linie, Lugner - PHP Worker**

  - **Lawine**
    ```
    auto eth0
    iface eth0 inet dhcp
    hwaddress ether 7e:c0:67:43:b9:7b
    ```
  - **Linie**
    ```
    auto eth0
    iface eth0 inet dhcp
    hwaddress ether 36:94:f5:f8:91:5e
    ```
  - **Lugner**
    ```
    auto eth0
    iface eth0 inet dhcp
    hwaddress ether ba:5a:55:c3:fd:a6
    ```
- **Frieren, Flamme, Fern - Laravel Worker**

  - **Frieren**
    ```
    auto eth0
    iface eth0 inet dhcp
    hwaddress ether be:87:1b:45:a5:cf
    ```
  - **Flamme**
    ```
    auto eth0
    iface eth0 inet dhcp
    hwaddress ether 36:4d:cf:6b:28:9f
    ```
  - **Fern**
    ```
    auto eth0
    iface eth0 inet dhcp
    hwaddress ether 42:55:10:8f:52:77
    ```
## Files
Sebelum menjawab soal - soal yang diberikan, pastikan file - file ini ada pada `/root` masing - masing node  
- **Aura - Router**  
  - init.sh
    ```sh
    # iptables
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.220.0.0/16

    # dhcp relay
    apt-get update
    apt-get install isc-dhcp-relay -y

    cp /root/isc-dhcp-relay /etc/default/isc-dhcp-relay
    cp /root/sysctl.conf /etc/sysctl.conf

    service isc-dhcp-relay start
    service isc-dhcp-relay restart
    ```
  - isc-dhcp-relay
    ```
    SERVERS="192.220.1.1"
    INTERFACES="eth1 eth3 eth4"
    OPTIONS=""
    ```
  - sysctl.conf
    ```
    net.ipv4.ip_forward=1
    ```
- **Himmel - DHCP**
  - init.sh
    ```sh
    apt-get update
    apt-get install isc-dhcp-server -y
    clear
    dhcpd --version
    echo "init complete"

    cp /root/isc-dhcp-server /etc/default/isc-dhcp-server
    cp /root/dhcpd.conf /etc/dhcp/dhcpd.conf
    rm /var/run/dhcpd.pid

    service isc-dhcp-server restart
    service isc-dhcp-server status
    ```
  - dhcpd.conf
    ```
    option domain-name "example.org";
    option domain-name-servers ns1.example.org, ns2.example.org;

    default-lease-time 600;
    max-lease-time 7200;

    ddns-update-style none;
    authoritative;

    subnet 192.220.1.0 netmask 255.255.255.0 {

    }

    subnet 192.220.2.0 netmask 255.255.255.0 {

    }

    subnet 192.220.3.0 netmask 255.255.255.0 {
        range 192.220.3.16 192.220.3.32;
        range 192.220.3.64 192.220.3.80;
        option routers 192.220.3.0;
        option broadcast-address 192.220.3.255;
        option domain-name-servers 192.220.1.2;
        default-lease-time 180;
        max-lease-time 5760;

        host Lawine_PHP {
            hardware ethernet 7e:c0:67:43:b9:7b;
            fixed-address 192.220.3.1;
        }
        host Linie_PHP {
            hardware ethernet 36:94:f5:f8:91:5e;
            fixed-address 192.220.3.2;
        }
        host Lugner_PHP {
            hardware ethernet ba:5a:55:c3:fd:a6;
            fixed-address 192.220.3.3;
        }
    }

    subnet 192.220.4.0 netmask 255.255.255.0 {
        range 192.220.4.12 192.220.4.20;
        range 192.220.4.160 192.220.4.168;
        option routers 192.220.4.0;
        option broadcast-address 192.220.4.255;
        option domain-name-servers 192.220.1.2;
        default-lease-time 720;
        max-lease-time 5760;

        host Frieren_Laravel {
            hardware ethernet be:87:1b:45:a5:cf;
            fixed-address 192.220.4.1;
        }
        host Flamme_Laravel {
            hardware ethernet 36:4d:cf:6b:28:9f;
            fixed-address 192.220.4.2;
        }
        host Fern_Laravel {
            hardware ethernet 42:55:10:8f:52:77;
            fixed-address 192.220.4.3;
        }
    }
    ```
  - isc-dhcp-server
    ```
    INTERFACESv4="eth0"
    INTERFACESv6=""
    ```
- **Heiter - DNS**
  - init.sh
    ```sh
    apt-get update
    apt-get install bind9 -y
    clear
    echo "init complete"

    cp /root/named.conf.options /etc/bind/named.conf.options
    cp /root/named.conf.local /etc/bind/named.conf.local

    mkdir /etc/bind/jarkom
    cp /root/riegel.canyon.e28.com /etc/bind/jarkom/riegel.canyon.e28.com
    cp /root/granz.channel.e28.com /etc/bind/jarkom/granz.channel.e28.com

    service bind9 start
    service bind9 restart
    ```
  - named.conf.options
    ```
    options {
        directory "/var/cache/bind";

        forwarders {
                192.168.122.1;
        };

        allow-query{any;};
        listen-on-v6 { any; };
    };
    ```
  - named.conf.local
    ```
    zone "riegel.canyon.e28.com" {
        type master;
        file "/etc/bind/jarkom/riegel.canyon.e28.com";
    };

    zone "granz.channel.e28.com" {
        type master;
        file "/etc/bind/jarkom/granz.channel.e28.com";
    };
    ```
  - granz.channel.e28.com
    ```
    ;
    ; BIND data file for local loopback interface
    ;
    $TTL    604800
    @       IN      SOA    granz.channel.e28.com. root.granz.channel.e28.com. (
                            2023110101    ; Serial
                            604800        ; Refresh
                            86400         ; Retry
                            2419200       ; Expire
                            604800 )      ; Negative Cache TTL
    ;
    @               IN      NS      granz.channel.e28.com.
    @               IN      A       192.220.3.1 ; IP Lawine
    www             IN      CNAME   granz.channel.e28.com.
    ```
  - riegel.canyon.e28.com
    ```
    ;
    ; BIND data file for local loopback interface
    ;
    $TTL    604800
    @       IN      SOA     riegel.canyon.e28.com. root.riegel.canyon.e28.com. (
                            2023110101    ; Serial
                            604800        ; Refresh
                            86400         ; Retry
                            2419200       ; Expire
                            604800 )      ; Negative Cache TTL
    ;
    @               IN      NS      riegel.canyon.e28.com.
    @               IN      A       192.220.4.1 ; IP Frieren
    www             IN      CNAME   riegel.canyon.e28.com.
    ```
- **Denken - DB**
  - init.sh
    ```sh
    apt-get update
    apt-get install mariadb-server -y
    clear
    echo "init complete"

    cp /root/my.cnf /etc/mysql/my.cnf

    service mysql start
    service mysql restart
    ```
  - my.cnf
    ```
    # The MariaDB configuration file
    #
    # The MariaDB/MySQL tools read configuration files in the following order:
    # 1. "/etc/mysql/mariadb.cnf" (this file) to set global defaults,
    # 2. "/etc/mysql/conf.d/*.cnf" to set global options.
    # 3. "/etc/mysql/mariadb.conf.d/*.cnf" to set MariaDB-only options.
    # 4. "~/.my.cnf" to set user-specific options.
    #
    # If the same option is defined multiple times, the last one will apply.
    #
    # One can use all long options that the program supports.
    # Run program with --help to get a list of available options and with
    # --print-defaults to see which it would actually understand and use.

    #
    # This group is read both both by the client and the server
    # use it for options that affect everything
    #
    [client-server]

    # Import all .cnf files from configuration directory
    !includedir /etc/mysql/conf.d/
    !includedir /etc/mysql/mariadb.conf.d/

    [mysqld]
    skip-networking=0
    skip-bind-address
    ```
  - mysql.txt
    ```
    mysql -u root -p

    CREATE USER 'kelompoke28'@'%' IDENTIFIED BY 'passworde28';
    CREATE USER 'kelompoke28'@'localhost' IDENTIFIED BY 'passworde28';
    CREATE DATABASE dbkelompoke28;
    GRANT ALL PRIVILEGES ON *.* TO 'kelompoke28'@'%';
    GRANT ALL PRIVILEGES ON *.* TO 'kelompoke28'@'localhost';
    FLUSH PRIVILEGES;
    ```
- **Eisen - LB**
  - init.sh
    ```sh
    apt-get update
    apt-get install nginx -y
    apt-get install htop -y
    apt-get install apache2-utils -y
    clear
    echo "init complete"

    cp /root/lb/roundrobin-php /etc/nginx/sites-available/php-block
    cp /root/lb/roundrobin-laravel /etc/nginx/sites-available/laravel-block

    ln -s /etc/nginx/sites-available/php-block /etc/nginx/sites-enabled/
    ln -s /etc/nginx/sites-available/laravel-block /etc/nginx/sites-enabled/

    service nginx start
    service nginx restart
    service nginx status

    echo "load balancer changed to round robin"
    ```
  - create-user.sh
    ```sh
    mkdir /etc/nginx/rahasisakita
    htpasswd -c /etc/nginx/rahasisakita/.htpasswd netics
    ```
  - use_rr.sh
    ```sh
    cp /root/lb/roundrobin-php /etc/nginx/sites-available/php-block
    cp /root/lb/roundrobin-laravel /etc/nginx/sites-available/laravel-block
    service nginx restart
    service nginx status
    echo "load balancer changed to round robin"
    ```
  - use_weightrr.sh
    ```sh
    cp /root/lb/weightedroundrobin-php /etc/nginx/sites-available/php-block
    service nginx restart
    service nginx status
    echo "load balancer changed to weighted round robin"
    ```
  - use_leastconn.sh
    ```sh
    cp /root/lb/leastconn-php /etc/nginx/sites-available/php-block
    cp /root/lb/leastconn-laravel /etc/nginx/sites-available/laravel-block
    service nginx restart
    service nginx status
    echo "load balancer changed to least connection"
    ```
  - use_iphash.sh
    ```sh
    cp /root/lb/iphash-php /etc/nginx/sites-available/php-block
    service nginx restart
    service nginx status
    echo "load balancer changed to ip hash"
    ```
  - use_generichash.sh
    ```sh
    cp /root/lb/generichash-php /etc/nginx/sites-available/php-block
    service nginx restart
    service nginx status
    echo "load balancer changed to generic hash"
    ```
  Buat directory `/root/lb` kemudian masukkan file - file ini di dalamnya  
  - roundrobin-php
    ```
    #Default menggunakan Round Robin
    upstream phpworker  {
            server 192.220.3.1; #IP Lawine
            server 192.220.3.2; #IP Linie
            server 192.220.3.3; #IP Lugner
    }

    server {
            listen 8000;
            server_name _;

            allow 192.220.3.69;
            allow 192.220.3.70;
            allow 192.220.4.167;
            allow 192.220.4.168;
            deny all;

            location /its {
                    proxy_pass https://www.its.ac.id/;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

            location / {
                    proxy_pass http://phpworker;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;

                    auth_basic "Private Property";
                    auth_basic_user_file /etc/nginx/rahasisakita/.htpasswd;
            }

            location ~ /\.ht {
                    deny all;
            }

            error_log /var/log/nginx/lb_error.log;
            access_log /var/log/nginx/lb_access.log;
    }
    ```
  - weightedroundrobin-php
    ```
    upstream phpworker  {
            server 192.220.3.1 weight=4; #IP Lawine
            server 192.220.3.2 weight=2; #IP Linie
            server 192.220.3.3 weight=1; #IP Lugner
    }

    server {
            listen 8000;
            server_name _;

            location / {
                    proxy_pass http://phpworker;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

            error_log /var/log/nginx/lb_error.log;
            access_log /var/log/nginx/lb_access.log;
    }
    ```
  - leastconn-php
    ```
    #Least Connection
    upstream phpworker  {
            least_conn;
            server 192.220.3.1; #IP Lawine
            server 192.220.3.2; #IP Linie
            server 192.220.3.3; #IP Lugner
    }

    server {
            listen 8000;
            server_name _;

            location / {
                    proxy_pass http://phpworker;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

            error_log /var/log/nginx/lb_error.log;
            access_log /var/log/nginx/lb_access.log;
    }
    ```
  - iphash-php
    ```
    # IP Hash
    upstream phpworker  {
            ip_hash;
            server 192.220.3.1; #IP Lawine
            server 192.220.3.2; #IP Linie
            server 192.220.3.3; #IP Lugner
    }

    server {
            listen 8000;
            server_name _;

            location / {
                    proxy_pass http://phpworker;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

            error_log /var/log/nginx/lb_error.log;
            access_log /var/log/nginx/lb_access.log;
    }
    ```
  - generichash-php
    ```
    # Generic hash
    upstream phpworker  {
            hash $request_uri consistent;
            server 192.220.3.1; #IP Lawine
            server 192.220.3.2; #IP Linie
            server 192.220.3.3; #IP Lugner
    }

    server {
            listen 8000;
            server_name _;

            location / {
                    proxy_pass http://phpworker;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

            error_log /var/log/nginx/lb_error.log;
            access_log /var/log/nginx/lb_access.log;
    }
    ```
  - roundrobin-laravel
    ```
    #Default menggunakan Round Robin
    upstream laravelworker  {
            server 192.220.4.1:8001; #IP Frieren
            server 192.220.4.2:8002; #IP Flamme
            server 192.220.4.3:8003; #IP Fern
    }

    server {
            listen 8080;
            server_name _;

            location / {
                    proxy_pass http://laravelworker;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

            error_log /var/log/nginx/lb_error.log;
            access_log /var/log/nginx/lb_access.log;
    }
    ```
  - leastconn-laravel
    ```
    #Least Connection
    upstream laravelworker  {
            least_conn;
            server 192.220.4.1:8001; #IP Frieren
            server 192.220.4.2:8002; #IP Flamme
            server 192.220.4.3:8003; #IP Fern
    }

    server {
            listen 8080;
            server_name _;

            location / {
                    proxy_pass http://laravelworker;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

            error_log /var/log/nginx/lb_error.log;
            access_log /var/log/nginx/lb_access.log;
    }
    ```
- **Sein, Stark, Revolte, Richter - Clients**
  - init.sh
    ```sh
    apt-get update
    apt-get install lynx -y
    apt-get install apache2-utils -y
    apt-get install jq -y
    clear
    ab -V
    echo "init complete"
    ```
  Buat directory `/root/phptest` kemudian masukkan file - file ini di dalamnya
  - 1000-100.loadtest.sh
    ```sh
    ab -A netics:ajke28 -n 1000 -c 100 -g out.data http://192.220.2.2:8000/
    ```
  - 200-10.loadtest.sh
    ```sh
    ab -A netics:ajke28 -n 200 -c 10 -g out.data http://192.220.2.2:8000/
    ```
  - 100-10.loadtest.sh
    ```sh
    ab -A netics:ajke28 -n 100 -c 10 -g out.data http://192.220.2.2:8000/
    ```
  - switch-ip.sh
    ```sh
    echo 'auto eth0
    iface eth0 inet static
            address 192.220.4.167
            netmask 255.255.255.0
            gateway 192.220.4.0
            up echo nameserver 192.220.1.2 > /etc/resolv.conf' > /etc/network/interfaces
    ```
  - revert-ip.sh
    ```sh
    echo 'auto eth0
    iface eth0 inet dhcp' > /etc/network/interfaces
    ```
  Buat directory `/root/laraveltest` kemudian masukkan file - file ini di dalamnya
  - register.json
    ```json
    {
        "username": "username",
        "password": "password"
    }
    ```
  - register-frieren.sh
    ```sh
    ab -n 100 -c 10 -p register.json -T application/json http://192.220.4.1:8001/api/auth/register
    ```
  - login.json
    ```json
    {
        "username": "username",
        "password": "password"
    }
    ```
  - login-frieren.sh
    ```sh
    ab -n 100 -c 10 -p login.json -T application/json http://192.220.4.1:8001/api/auth/login
    ```
  - login-lb.sh
    ```sh
    ab -n 100 -c 10 -p login.json -T application/json http://192.220.2.2:8080/api/auth/login
    ```
- **Lawine, Linie, Lugner - PHP Worker**
  - init.sh
    ```
    apt-get update
    apt-get install wget -y
    apt-get install unzip -y
    apt-get install nginx -y
    apt-get install php -y
    apt-get install php php-fpm -y
    apt-get install htop -y
    clear
    echo "init complete"

    cp /root/default /etc/nginx/sites-available/default
    cp -R /root/modul-3/* /var/www/html

    service nginx start
    service nginx status
    service nginx restart
    service php7.3-fpm start
    service php7.3-fpm restart
    service php7.3-fpm status
    ```
  - download.sh
    ```sh
    wget -O x.zip "https://drive.google.com/u/0/uc?id=1ViSkRq7SmwZgdK64eRbr5Fm1EGCTPrU1&export=download"
    unzip x.zip
    rm x.zip
    ```
  - default
    ```
    server {
            listen 80 default_server;
            listen [::]:80 default_server;

            root /var/www/html;
            index index.php index.html index.htm index.nginx-debian.html;
            server_name granz.channel.e28.com;

            location / {
                    try_files $uri $uri/ =404;
            }

            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;

                    fastcgi_pass unix:/run/php/php7.3-fpm.sock;
            }

            location ~ /\.ht {
                    deny all;
            }
    }
    ```
- **Frieren, Flamme, Fern - Laravel Worker**
  - init.sh
    ```sh
    apt-get update
    apt-get install mariadb-client -y
    apt-get install nginx -y
    apt-get install git -y
    apt-get install wget -y

    clear
    echo "init complete"

    cp /root/clone.txt /var/www/clone.txt

    service nginx start
    service nginx restart
    ```
  - addPHP8.sh
    ```sh
    # DO NOT RUN MANUALLY!!
    apt-get update
    apt-get install -y lsb-release ca-certificates apt-transport-https software-pr$curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/$sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://$apt-get update
    apt-get install php8.0-mbstring php8.0-xml php8.0-cli php8.0-common php8.0-int$
    wget https://getcomposer.org/download/2.0.13/composer.phar
    chmod +x composer.phar
    mv composer.phar /usr/bin/composer

    apt-get install git -y
    composer -V
    ```
  - enable-site.sh
    ```sh
    cp /root/laravel-block /etc/nginx/sites-available/laravel
    ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
    chown -R www-data.www-data /var/www/laravel-praktikum-jarkom/storage
    service nginx restart
    service php8.0-fpm restart
    ```
  - show-databases.sh
    ```sh
    mariadb --host=192.220.2.1 --port=3306 --user=kelompoke28 --password=passworde28 dbkelompoke28 -e "SHOW DATABASES;"
    ```
  - cat-log.sh
    ```sh
    cat /var/www/laravel-praktikum-jarkom/storage/logs/laravel.log
    ```
  - env
    ```
    APP_NAME=Laravel
    APP_ENV=local
    APP_KEY=base64:tbEA5XsOoSgrNOuxc3Td1BlgZ2EMkqdmwrsA0ouJwKA=
    APP_DEBUG=true
    APP_URL=http://localhost

    LOG_CHANNEL=stack
    LOG_DEPRECATIONS_CHANNEL=null
    LOG_LEVEL=debug

    DB_CONNECTION=mysql
    DB_HOST=192.220.2.1
    DB_PORT=3306
    DB_DATABASE=dbkelompoke28
    DB_USERNAME=kelompoke28
    DB_PASSWORD=passworde28

    BROADCAST_DRIVER=log
    CACHE_DRIVER=file
    FILESYSTEM_DISK=local
    QUEUE_CONNECTION=sync
    SESSION_DRIVER=file
    SESSION_LIFETIME=120

    MEMCACHED_HOST=127.0.0.1

    REDIS_HOST=127.0.0.1
    REDIS_PASSWORD=null
    REDIS_PORT=6379

    MAIL_MAILER=smtp
    MAIL_HOST=mailpit
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null
    MAIL_FROM_ADDRESS="hello@example.com"
    MAIL_FROM_NAME="${APP_NAME}"

    AWS_ACCESS_KEY_ID=
    AWS_SECRET_ACCESS_KEY=
    AWS_DEFAULT_REGION=us-east-1
    AWS_BUCKET=
    AWS_USE_PATH_STYLE_ENDPOINT=false

    PUSHER_APP_ID=
    PUSHER_APP_KEY=
    PUSHER_APP_SECRET=
    PUSHER_HOST=
    PUSHER_PORT=443
    PUSHER_SCHEME=https
    PUSHER_APP_CLUSTER=mt1

    VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
    VITE_PUSHER_HOST="${PUSHER_HOST}"
    VITE_PUSHER_PORT="${PUSHER_PORT}"
    VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
    VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

    JWT_SECRET=PaEcKSh0sQjyJiYzYLgn1X8DegdPAF93VSD2zk9s4WVrT38JE7nRpki47xdPvyAo
    ```
  - clone.txt
    ```
    # cd /var/www
    git clone https://github.com/martuafernando/laravel-praktikum-jarkom.git

    # cd /var/www/laravel-praktikum-jarkom
    composer install
    cp /root/env .env
    php artisan migrate:fresh
    php artisan db:seed --class=AiringsTableSeeder
    php artisan key:generate
    php artisan config:cache
    php artisan storage:link
    php artisan jwt:secret
    php artisan config:clear
    ```
  - laravel-block
    [port] 8001 untuk Frieren, 8002 untuk Flamme, dan 8003 untuk Fern  
    ```
    server {

        listen [port];

        root /var/www/laravel-praktikum-jarkom/public;

        index index.php index.html index.htm;
        server_name _;

        location / {
                try_files $uri $uri/ /index.php?$query_string;
        }

        # pass PHP scripts to FastCGI server
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
        }

    location ~ /\.ht {
                deny all;
        }

        error_log /var/log/nginx/laravel_error.log;
        access_log /var/log/nginx/laravel_access.log;
    }
    ```

  - config-1.sh

  ```
  echo '[www]
  user = www-data
  group = www-data
  listen = /run/php/php8.0-fpm.sock
  listen.owner = www-data
  listen.group = www-data
  php_admin_value[disable_functions] = exec,passthru,shell_exec,system
  php_admin_flag[allow_url_fopen] = off

  ; Choose how the process manager will control the number of child processes.

  pm = dynamic
  pm.max_children = 5
  pm.start_servers = 2
  pm.min_spare_servers = 1
  pm.max_spare_servers = 3' > /etc/php/8.0/fpm/pool.d/www.conf

  service php8.0-fpm restart
  ```

  - config-2.sh

  ```
  echo '[www]
  user = www-data
  group = www-data
  listen = /run/php/php8.0-fpm.sock
  listen.owner = www-data
  listen.group = www-data
  php_admin_value[disable_functions] = exec,passthru,shell_exec,system
  php_admin_flag[allow_url_fopen] = off

  ; Choose how the process manager will control the number of child processes.

  pm = dynamic
  pm.max_children = 25
  pm.start_servers = 5
  pm.min_spare_servers = 3
  pm.max_spare_servers = 10' > /etc/php/8.0/fpm/pool.d/www.conf

  service php8.0-fpm restart
  ```

  - config-3.sh

  ```
  echo '[www]
  user = www-data
  group = www-data
  listen = /run/php/php8.0-fpm.sock
  listen.owner = www-data
  listen.group = www-data
  php_admin_value[disable_functions] = exec,passthru,shell_exec,system
  php_admin_flag[allow_url_fopen] = off

  ; Choose how the process manager will control the number of child processes.

  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 8
  pm.min_spare_servers = 5
  pm.max_spare_servers = 15' > /etc/php/8.0/fpm/pool.d/www.conf

  service php8.0-fpm restart
  ```

  - config-4.sh

  ```
  
  echo '[www]
  user = www-data
  group = www-data
  listen = /run/php/php8.0-fpm.sock
  listen.owner = www-data
  listen.group = www-data
  php_admin_value[disable_functions] = exec,passthru,shell_exec,system
  php_admin_flag[allow_url_fopen] = off

  ; Choose how the process manager will control the number of child processes.

  pm = dynamic
  pm.max_children = 75
  pm.start_servers = 10
  pm.min_spare_servers = 5
  pm.max_spare_servers = 20' > /etc/php/8.0/fpm/pool.d/www.conf

  service php8.0-fpm restart
  ```

## Soal Praktikum
### No 0
> Setelah mengalahkan Demon King, perjalanan berlanjut. Kali ini, kalian diminta untuk melakukan register domain berupa riegel.canyon.yyy.com untuk worker Laravel dan granz.channel.yyy.com untuk worker PHP (0) mengarah pada worker yang memiliki IP [prefix IP].x.1

#### Answer:  
Konfigurasi domain riegel.canyon.e28.com diatur pada file `/root/riegel.canyon.e28.com` yang kemudian akan di copy menuju `/etc/bind/jarkom/riegel.canyon.e28.com` melalui `init.sh`  

**riegel.canyon.e28.com**  
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     riegel.canyon.e28.com. root.riegel.canyon.e28.com. (
                        2023110101    ; Serial
                        604800        ; Refresh
                        86400         ; Retry
                        2419200       ; Expire
                        604800 )      ; Negative Cache TTL
;
@               IN      NS      riegel.canyon.e28.com.
@               IN      A       192.220.4.1 ; IP Frieren
www             IN      CNAME   riegel.canyon.e28.com.
```

Sedangkan konfigurasi domain granz.channel.e28.com diatur pada file `/root/granz.channel.e28.com` yang kemudian akan di copy menuju `/etc/bind/jarkom/granz.channel.e28.com` melalui `init.sh`  

**granz.channel.e28.com**  
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA    granz.channel.e28.com. root.granz.channel.e28.com. (
                        2023110101    ; Serial
                        604800        ; Refresh
                        86400         ; Retry
                        2419200       ; Expire
                        604800 )      ; Negative Cache TTL
;
@               IN      NS      granz.channel.e28.com.
@               IN      A       192.220.3.1 ; IP Lawine
www             IN      CNAME   granz.channel.e28.com.
```

#### Testing:  
Test domain menggunakan ping melalui Sein_Client.  

**riegel.canyon.e28.com**  
![](/images/ping-riegel.canyon.e28.com.png)  

**granz.channel.e28.com**  
![](/images/ping-granz.channel.e28.com.png)  

### No 1
> Lakukan konfigurasi sesuai dengan peta yang sudah diberikan. Semua CLIENT harus menggunakan konfigurasi dari DHCP Server.

#### Answer:  
Konfigurasi dilakukan sesuai dengan [Network Configuration](#network-configuration).  
Konfigurasi DHCP Server pada node Himmel diatur pada file `/root/dhcpd.conf` dan `/root/isc-dhcp-server` yang kemudian akan dicopy ke `/etc/dhcp/dhcpd.conf` dan `/etc/default/isc-dhcp-server` melalui `init.sh`  

**dhcpd.conf**
```
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

ddns-update-style none;
authoritative;

subnet 192.220.1.0 netmask 255.255.255.0 {

}

subnet 192.220.2.0 netmask 255.255.255.0 {

}

subnet 192.220.3.0 netmask 255.255.255.0 {
    range 192.220.3.16 192.220.3.32;
    range 192.220.3.64 192.220.3.80;
    option routers 192.220.3.0;
    option broadcast-address 192.220.3.255;
    option domain-name-servers 192.220.1.2;
    default-lease-time 180;
    max-lease-time 5760;

    host Lawine_PHP {
        hardware ethernet 7e:c0:67:43:b9:7b;
        fixed-address 192.220.3.1;
    }
    host Linie_PHP {
        hardware ethernet 36:94:f5:f8:91:5e;
        fixed-address 192.220.3.2;
    }
    host Lugner_PHP {
        hardware ethernet ba:5a:55:c3:fd:a6;
        fixed-address 192.220.3.3;
    }
}

subnet 192.220.4.0 netmask 255.255.255.0 {
    range 192.220.4.12 192.220.4.20;
    range 192.220.4.160 192.220.4.168;
    option routers 192.220.4.0;
    option broadcast-address 192.220.4.255;
    option domain-name-servers 192.220.1.2;
    default-lease-time 720;
    max-lease-time 5760;

    host Frieren_Laravel {
        hardware ethernet be:87:1b:45:a5:cf;
        fixed-address 192.220.4.1;
    }
    host Flamme_Laravel {
        hardware ethernet 36:4d:cf:6b:28:9f;
        fixed-address 192.220.4.2;
    }
    host Fern_Laravel {
        hardware ethernet 42:55:10:8f:52:77;
        fixed-address 192.220.4.3;
    }
}
```
IP Dynamic untuk clients pada subnet 192.220.3.0 dan 192.220.4.0 dan IP Static untuk WebServer pada subnet 192.220.3.0 dan 192.220.4.0  

**isc-dhcp-server**
```
INTERFACESv4="eth0"
INTERFACESv6=""
```
eth0 adalah link Himmel yang mengarah ke router. INTERFACESv6 tidak disetting karna tidak digunakan.  

Kemudian karena Client dan DHCP Server berada pada subnet yang berbeda, jangan lupa lakukan konfigurasi DHCP Relay pada Aura_Router.  
Konfigurasi DHCP Relay diatur pada file `/root/isc-dhcp-relay` dan `/root/sysctl.conf` yang kemudian akan dicopy ke `/etc/default/isc-dhcp-relay` dan `/etc/sysctl.conf` melalui `init.sh`  

**isc-dhcp-relay**  
```
SERVERS="192.220.1.1"
INTERFACES="eth1 eth3 eth4"
OPTIONS=""
```
SERVERS adalah IP dari DHCP server. eth1 adalah link dari Aura_Router ke DHCP server sedangkan eth3 dan eth4 adalah link dari router menuju Client dan WebServer yang dikonfigurasi menggunakan DHCP.  

**sysctl.conf**
```
net.ipv4.ip_forward=1
```
Digunakan agar Aura_Router bisa memforward DHCP request menuju DHCP server  

#### Testing:  
Test bisa dilakukan menggunakan command `ip a` pada Sein_Client  

![](/images/ipa-sein.png)

Bisa dilihat bahwa IP Sein pada eth0 adalah 192.220.4.16

### No 2
> Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.16 - [prefix IP].3.32 dan [prefix IP].3.64 - [prefix IP].3.80

#### Answer:  
Range IP diatur pada `/root/dhcpd.conf` pada subnet 192.220.3.0 bagian,
```
range 192.220.3.16 192.220.3.32;
range 192.220.3.64 192.220.3.80;
```
#### Testing:  
Test bisa dilakukan menggunakan command `ip a` pada Revolte_Client dan Richter_Client    

**Revolte_Client**  

![](/images/ipa-revolte.png)  

**Richter_Client**  

![](/images/ipa-richter.png)

Bisa dilihat bahwa IP Revolte pada eth0 adalah 192.220.3.18 dan IP Richter pada eth0 adalah 192.220.3.19

### No 3
> Client yang melalui Switch4 mendapatkan range IP dari [prefix IP].4.12 - [prefix IP].4.20 dan [prefix IP].4.160 - [prefix IP].4.168

#### Answer:  
Range IP diatur pada `/root/dhcpd.conf` pada subnet 192.220.4.0 bagian,
```
range 192.220.4.12 192.220.4.20;
range 192.220.4.160 192.220.4.168;
```
#### Testing:  
Test bisa dilakukan menggunakan command `ip a` pada Sein_Client dan Stark_Client    

**Sein_Client**  

![](/images/ipa-sein.png)  

**Stark_Client**  

![](/images/ipa-stark.png)  

Bisa dilihat bahwa IP Sein pada eth0 adalah 192.220.4.16 dan IP Stark pada eth0 adalah 192.220.4.13

### No 4
> Client mendapatkan DNS dari Heiter dan dapat terhubung dengan internet melalui DNS tersebut

#### Answer:  
DNS pada node yang dikonfigurasi menggunakan DHCP server diatur pada file `/root/dhcpd.conf` pada bagian,
```
option domain-name-servers 192.220.1.2;
```
IP yang dituliskan pada option domain-name-servers akan dimasukkan ke `/etc/resolv.conf` masing - masing node.  

Sedangkan agar dapat terhubung dengan internet maka perlu ditambahkan forwarder menuju 192.168.122.1 pada DNS Server Heiter yang diatur pada file `/root/named.conf.options`
```
options {
        directory "/var/cache/bind";

        forwarders {
                192.168.122.1;
        };

        allow-query{any;};
        listen-on-v6 { any; };
};
```

#### Testing:  
Test dilakukan dengan melakukan ping google.com pada Sein_Client, jika sudah terhubung dengan DNS dengan forwarder seharusnya bisa melakukan ping ke internet  

![](/images/ping-google.png)

### No 5
> Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 3 menit sedangkan pada client yang melalui Switch4 selama 12 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 96 menit

#### Answer:  
Lama peminjaman IP diatur pada file `/root/dhcpd.conf` pada bagian,

Untuk client yang melalui Switch3 (Subnet 192.220.3.0) 
```
default-lease-time 180;
max-lease-time 5760;
```

Untuk client yang melalui Switch4 (Subnet 192.220.4.0)
```
default-lease-time 720;
max-lease-time 5760;
```

#### Testing:  
(none)

### No 6
> Pada masing-masing worker PHP, lakukan konfigurasi virtual host untuk website [berikut](https://drive.google.com/file/d/1ViSkRq7SmwZgdK64eRbr5Fm1EGCTPrU1/view?usp=sharing) dengan menggunakan php 7.3

#### Answer:  
Download dan unzip file yang dibutuhkan dengan menjalankan `/root/download.sh` kemudian copy semua file didalamnya ke directory `/var/www/html` melalui `init.sh`    
```sh
wget -O x.zip "https://drive.google.com/u/0/uc?id=1ViSkRq7SmwZgdK64eRbr5Fm1EGCTPrU1&export=download"
unzip x.zip
rm x.zip
```

Konfigurasi server block dari worker php diatur pada file `/root/default` yang kemudian dicopy ke `/etc/nginx/sites-available/default` melaui `init.sh`  
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name granz.channel.e28.com;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;

                fastcgi_pass unix:/run/php/php7.3-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }
}
```
Karena metode ini melakukan overwrite pada default block nginx maka tidak perlu membuat symbolic link  

#### Testing:  
Test dengan melakukan lynx granz.channel.e28.com pada Sein_Client  

![](/images/lynx-granz.channel.e28.com.png)  

Terhubung ke worker Lawine karena domain granz.channel.e28.com mengarah ke 192.220.3.1  

### No 7
> Kepala suku dari Bredt Region memberikan resource server sebagai berikut: Lawine, 4GB, 2vCPU, dan 80 GB SSD. Linie, 2GB, 2vCPU, dan 50 GB SSD. Lugner 1GB, 1vCPU, dan 25 GB SSD. aturlah agar Eisen dapat bekerja dengan maksimal, lalu lakukan testing dengan 1000 request dan 100 request/second

#### Answer:  
Konfigurasi block Load Balancer pada node Eise diatur pada file `/root/lb/roundrobin-php` yang kemudian akan dicopy ke `/etc/nginx/sites-available/php-block` melalui `init.sh`  
```
#Default menggunakan Round Robin
upstream phpworker  {
        server 192.220.3.1; #IP Lawine
        server 192.220.3.2; #IP Linie
        server 192.220.3.3; #IP Lugner
}

server {
        listen 8000;
        server_name _;

        allow 192.220.3.69;
        allow 192.220.3.70;
        allow 192.220.4.167;
        allow 192.220.4.168;
        deny all;

        location /its {
                proxy_pass https://www.its.ac.id/;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
        }

        location / {
                proxy_pass http://phpworker;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;

                auth_basic "Private Property";
                auth_basic_user_file /etc/nginx/rahasisakita/.htpasswd;
        }

        location ~ /\.ht {
                deny all;
        }

        error_log /var/log/nginx/lb_error.log;
        access_log /var/log/nginx/lb_access.log;
}
```
#### Testing:  
Jalankan command lynx 192.220.2.2:8000 melalui Sein_Client. Tetapi sebelum melakukan lynx, jalankan dulu file `/root/phptest/switch-ip.sh` untuk mengganti IP dari DHCP Dynamic ke 192.220.4.167, kemudian restart node.  

![](/images/switched-sein.png)  

Jalankan lynx sebanyak 3 kali  

![](/images/lb-lawine.png)  

![](/images/lb-linie.png)  

![](/images/lb-lugner.png)  

Testing 1000 request dan 100 request/second dengan menjalankan `/root/phptest/1000-100.loadtest.sh`
```sh
ab -A netics:ajke28 -n 1000 -c 100 -g out.data http://192.220.2.2:8000/
```

![](/images/result-1000-100.png)  

![](/images/htop-1000-100.png)  

### No 8
> Karena diminta untuk menuliskan grimoire, buatlah analisis hasil testing dengan 200 request dan 10 request/second masing-masing algoritma Load Balancer dengan ketentuan sebagai berikut: Nama Algoritma Load Balancer, Report hasil testing pada Apache Benchmark, Grafik request per second untuk masing masing algoritma., Analisis

#### Answer:  
Konfigurasi server block untuk masing - masing algoritma load balancing ada pada `/root/lb`. Untuk menggunakan algoritma tersebut jalankan file - file pada `/root` sebagai berikut.  

- use_rr.sh
- use_weightrr.sh
- use_leastconn.sh
- use_iphash.sh
- use_generichash.sh  

File - file ini akan mengoverwrite `/etc/nginx/sites-available/php-block` dengan algoritmanya masing - masing.  

#### Testing:  
- Round Robin
  Jalankan `use_rr.sh` pada Eisen_LB kemudian jalankan `/root/phptest/200-10.loadtest.sh` pada Sein_Client  

  ![](/images/res-roundrobin.png)  

  ![](/images/roundrobin.png)  

- Weighted Round Robin
  Jalankan `use_weightrr.sh` pada Eisen_LB kemudian jalankan `/root/phptest/200-10.loadtest.sh` pada Sein_Client  

  ![](/images/res-weightedroundrobin.png)  

  ![](/images/weightedroundrobin.png)  

- Least Connection
  Jalankan `use_leastconn.sh` pada Eisen_LB kemudian jalankan `/root/phptest/200-10.loadtest.sh` pada Sein_Client  

  ![](/images/res-leastconnection.png)  

  ![](/images/leastconnection.png)  
  
- IP Hash
  Jalankan `use_iphash.sh` pada Eisen_LB kemudian jalankan `/root/phptest/200-10.loadtest.sh` pada Sein_Client  

  ![](/images/res-leastconnection.png)  

  ![](/images/leastconnection.png)  
  
- Generic Hash
  Jalankan `use_generichash.sh` pada Eisen_LB kemudian jalankan `/root/phptest/200-10.loadtest.sh` pada Sein_Client  

  ![](/images/res-generichash.png)  

  ![](/images/generichash.png)  
  
- Grafik

  ![](/images/grafik-request-per-second.png)  

### No 9
> Dengan menggunakan algoritma Round Robin, lakukan testing dengan menggunakan 3 worker, 2 worker, dan 1 worker sebanyak 100 request dengan 10 request/second, kemudian tambahkan grafiknya pada grimoire

#### Answer:  
Ganti algoritma load balancing dengan menjalankan `/root/use_rr.sh` pada Eisen_LB. Kemudian lakukan test untuk 3 worker, 2 worker, dan 1 worker. Pengurangan worker dapat dilakukan dengan mengcomment worker yang ingin dimatikan pada `/root/lb/roundrobin-php`, jangan lupa untuk menjalankan `/root/use_rr.sh` setiap pengurangan jumlah worker.  

#### Testing:
Test dilakukan dengan menjalankan `/root/phptest/100-10.loadtest.sh` pada Sein_Client    
- 3 Worker  

  ![](/images/res-3worker.png)  

  ![](/images/3worker.png)  

- 2 Worker  

  ![](/images/res-2worker.png)  

  ![](/images/2worker.png)  
  
- 1 Worker  

  ![](/images/res-1worker.png)  

  ![](/images/1worker.png)  
  
### No 10
> Selanjutnya coba tambahkan konfigurasi autentikasi di LB dengan dengan kombinasi username: “netics” dan password: “ajkyyy”, dengan yyy merupakan kode kelompok. Terakhir simpan file “htpasswd” nya di /etc/nginx/rahasisakita/

#### Answer:  
Penambahan user bisa dilakukan dengan menjalankan `/root/create-user.sh` pada Eisen_LB  
```
mkdir /etc/nginx/rahasisakita
htpasswd -c /etc/nginx/rahasisakita/.htpasswd netics
```
Masukkan password `ajke28` untuk use netics  

#### Testing:  
Jalankan lynx 192.220.2.2:8000 pada Sein_Client, seharusnya akan diminta untuk memasukkan user dan password untuk autentikasi  

![](/images/permission-denied.png)  

![](/images/auth.png)  

### No 11
> Lalu buat untuk setiap request yang mengandung /its akan di proxy passing menuju halaman https://www.its.ac.id

#### Answer:  
Proxy passing diatur pada file `/root/lb/roundrobin-php` pada Eisen_LB, yaitu pada bagian berikut,  
```
location /its {
        proxy_pass https://www.its.ac.id/;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;                
        proxy_set_header    Host $http_host;
}
```

#### Testing:  
Jalankan lynx 192.220.2.2:8000/its pada Sein_Client, seharusnya akan diarahkan menuju https://www.its.ac.id  

![](/images/its.png)  

### No 12
> Selanjutnya LB ini hanya boleh diakses oleh client dengan IP [Prefix IP].3.69, [Prefix IP].3.70, [Prefix IP].4.167, dan [Prefix IP].4.168

#### Answer:  
Permission untuk akses load balancer diatur pada file `/root/lb/roundrobin-php` yaitu pada bagian,
```
allow 192.220.3.69;
allow 192.220.3.70;
allow 192.220.4.167;
allow 192.220.4.168;
deny all;
```

#### Testing:  
Sebelumnya kita menggunakan `/root/phptest/switch-ip.sh` pada Sein_Client untuk mengganti IP menjadi 192.220.4.167, sekarang kembalikan IP menjadi menggunakan DHCP dengan menjalankan `/root/phptest/revert-ip.sh`, kemudian restart node. Gunakan `ip a` untuk mengecek IP setelah direstart.  

IP sekarang adalah 192.220.4.14

![](/images/sein-revert.png)  

Jalankan lynx 192.220.2.2:8000, seharusnya akan error karena IP tidak sesuai  

![](/images/forbidden.png)  

### No 13
> Semua data yang diperlukan, diatur pada Denken dan harus dapat diakses oleh Frieren, Flamme, dan Fern

#### Answer: 
Melakukan konfigurasi pada Database Server (Denken) sesuai dengan [Files](#files). 

Melakukan konfigurasi mysql sebagai berikut dengan yyy adalah nama kelompok:
```
mysql -u root -p

CREATE USER 'kelompoke28'@'%' IDENTIFIED BY 'passworde28';
CREATE USER 'kelompoke28'@'localhost' IDENTIFIED BY 'passworde28';
CREATE DATABASE dbkelompoke28;
GRANT ALL PRIVILEGES ON *.* TO 'kelompoke28'@'%';
GRANT ALL PRIVILEGES ON *.* TO 'kelompoke28'@'localhost';
FLUSH PRIVILEGES;
```

Konfigurasi berikut ini dilakukan pada `/etc/mysql/my.cnf` karena database akan diakses oleh 3 worker. 
```
# The MariaDB configuration file
#
# The MariaDB/MySQL tools read configuration files in the following order:
# 1. "/etc/mysql/mariadb.cnf" (this file) to set global defaults,
# 2. "/etc/mysql/conf.d/*.cnf" to set global options.
# 3. "/etc/mysql/mariadb.conf.d/*.cnf" to set MariaDB-only options.
# 4. "~/.my.cnf" to set user-specific options.
#
# If the same option is defined multiple times, the last one will apply.
#
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.

#
# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]

# Import all .cnf files from configuration directory
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mariadb.conf.d/

[mysqld]
skip-networking=0
skip-bind-address

```

Restart pada MySQL `service mysql restart`

Untuk memeriksa apakah database dapat diakses oleh worker lakukan instalasi mariadb-client pada worker seperti berikut ini:
```
apt-get install mariadb-client -y
```


#### Testing:  
Melakukan koneksi dengan perintah berikut:
```
mariadb --host=192.220.2.1 --port=3306 --user=kelompoke28 --password=passworde28 dbkelompoke28 -e "SHOW DATABASES;"
```

Hasilnya adalah sebagai berikut:

![](/images/test-database.png)

### No 14
> Frieren, Flamme, dan Fern memiliki Riegel Channel sesuai dengan quest guide berikut. Jangan lupa melakukan instalasi PHP8.0 dan Composer

#### Answer:  
Melakukan instalasi php 8.0 dengan perintah berikut:
```
apt-get update
apt-get install -y lsb-release ca-certificates apt-transport-https software-pr$curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/$sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://$apt-get update
apt-get install php8.0-mbstring php8.0-xml php8.0-cli php8.0-common php8.0-int$
php --version
```

Melakukan instalasi composer dengan perintah berikut:
```
    wget https://getcomposer.org/download/2.0.13/composer.phar
    chmod +x composer.phar
    mv composer.phar /usr/bin/composer
    composer -V
```

Untuk mempersiapkan web yang akan di-deploy, lakukan instalasi git terlebih dahulu:
```
apt-get install git -y
```

Clone repository yang akan di-deploy pada direktori `/var/www`
```
git clone https://github.com/martuafernando/laravel-praktikum-jarkom.git
```

Install vendor berikut pada direktori laravel-praktikum-jarkom
```
composer install
```

Lakukan konfigurasi pada `.env` seperti berikut ini:
```
APP_NAME=Laravel
    APP_ENV=local
    APP_KEY=base64:tbEA5XsOoSgrNOuxc3Td1BlgZ2EMkqdmwrsA0ouJwKA=
    APP_DEBUG=true
    APP_URL=http://localhost

    LOG_CHANNEL=stack
    LOG_DEPRECATIONS_CHANNEL=null
    LOG_LEVEL=debug

    DB_CONNECTION=mysql
    DB_HOST=192.220.2.1
    DB_PORT=3306
    DB_DATABASE=dbkelompoke28
    DB_USERNAME=kelompoke28
    DB_PASSWORD=passworde28

    BROADCAST_DRIVER=log
    CACHE_DRIVER=file
    FILESYSTEM_DISK=local
    QUEUE_CONNECTION=sync
    SESSION_DRIVER=file
    SESSION_LIFETIME=120

    MEMCACHED_HOST=127.0.0.1

    REDIS_HOST=127.0.0.1
    REDIS_PASSWORD=null
    REDIS_PORT=6379

    MAIL_MAILER=smtp
    MAIL_HOST=mailpit
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null
    MAIL_FROM_ADDRESS="hello@example.com"
    MAIL_FROM_NAME="${APP_NAME}"

    AWS_ACCESS_KEY_ID=
    AWS_SECRET_ACCESS_KEY=
    AWS_DEFAULT_REGION=us-east-1
    AWS_BUCKET=
    AWS_USE_PATH_STYLE_ENDPOINT=false

    PUSHER_APP_ID=
    PUSHER_APP_KEY=
    PUSHER_APP_SECRET=
    PUSHER_HOST=
    PUSHER_PORT=443
    PUSHER_SCHEME=https
    PUSHER_APP_CLUSTER=mt1

    VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
    VITE_PUSHER_HOST="${PUSHER_HOST}"
    VITE_PUSHER_PORT="${PUSHER_PORT}"
    VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
    VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

    JWT_SECRET=PaEcKSh0sQjyJiYzYLgn1X8DegdPAF93VSD2zk9s4WVrT38JE7nRpki47xdPvyAo
```

Setelah mengatur `.env`, jalankan perintah berikut pada worker
```
php artisan migrate:fresh
php artisan db:seed --class=AiringsTableSeeder
php artisan key:generate
php artisan config:cache
php artisan storage:link
php artisan jwt:secret
php artisan config:clear
```

Untuk melakukan deploymet pada masing-masing worker, tambahkan virtual host pada /etc/nginx/sites-available/laravel seperti di bawah ini:
```
server {

    listen [port];

    root /var/www/laravel-praktikum-jarkom/public;

    index index.php index.html index.htm;
    server_name _;

    location / {
            try_files $uri $uri/ /index.php?$query_string;
    }

    # pass PHP scripts to FastCGI server
    location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
    }

location ~ /\.ht {
            deny all;
    }

    error_log /var/log/nginx/laravel_error.log;
    access_log /var/log/nginx/laravel_access.log;
}
```

Dengan port masing-masing worker:
```
192.220.4.1:8001; #Frieren
192.220.4.2:8002; #Flame
192.220.4.3:8003; #Fern
```
Setelah selesai, buat symlink untuk melakukan enable pada site dengan perintah:
```
ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
```

Kemudian, untuk memastikan bahwa server web (diasumsikan berjalan sebagai www-data) memiliki izin yang diperlukan untuk mengelola dan mengakses direktori penyimpanan maka jalankan perintah berikut:
```
chown -R www-data.www-data /var/www/laravel-praktikum-jarkom/storage
```

Jalankan php-fpn dengan perintah berikut:
```
service php8.0-fpm restart
```
#### Testing:  

![](/images/lynx-frieren.png)

### No 15
> Riegel Channel memiliki beberapa endpoint yang harus ditesting sebanyak 100 request dengan 10 request/second. Tambahkan response dan hasil testing pada grimoire. POST /auth/register

#### Answer:  
Untuk menyelesaikan soal ini perlu dilakukan Apache Benchmark pada salah satu worker, dimana kami di sini menggunakan Frieren. Selanjutnya, akan dilakukan testing pada client. Di sini menggunakan json yang akan dikirim pada endpoint api/auth/register:
```
{
    "username": "username",
    "password": "password"
}
```

#### Testing:  
Lakukan perintah berikut:

```
ab -n 100 -c 10 -p register.json -T application/json http://192.220.4.1:8001/api/auth/register
```

Hanya ada 1 request saja yang diproses sedangkan 99 lainnya gagal.

![](/images/benchmark-register.png)

![](/images/htop-register.png)

### No 16
> Riegel Channel memiliki beberapa endpoint yang harus ditesting sebanyak 100 request dengan 10 request/second. Tambahkan response dan hasil testing pada grimoire. POST /auth/login

#### Answer:
Untuk menyelesaikan soal ini perlu dilakukan Apache Benchmark pada salah satu worker, dimana kami di sini menggunakan Frieren. Selanjutnya, akan dilakukan testing pada client. Di sini menggunakan json yang akan dikirim pada endpoint api/auth/login:

```
{
    "username": "username",
    "password": "password"
}  
```

#### Testing:

Lakukan perintah berikut:

```
ab -n 100 -c 10 -p login.json -T application/json http://192.220.4.1:8001/api/auth/login
``` 

Terdapat 60 request yang diproses sedangkan 40 lainnya gagal.

![](/images/htop-login.png)

![](/images/benchmark-login.png)

### No 17
> Riegel Channel memiliki beberapa endpoint yang harus ditesting sebanyak 100 request dengan 10 request/second. Tambahkan response dan hasil testing pada grimoire. GET /me

#### Answer:
Dapatkan token dengan menjalankan perintah berikut:

```
curl -X POST -H "Content-Type: application/json" -d @login.json http://192.220.4.1:8001/api/auth/login > /root/login_output.txt
```

Jalankan perintah berikut untuk set token secara global:

```
token=$(cat /root/login_output.txt | jq -r '.token')
```

#### Testing:  
Lakukan perintah berikut:

```
ab -n 100 -c 10 -H "Authorization: Bearer $token" http://192.220.4.1:8001/api/me
```

Terdapat 60 request yang diproses sedangkan 40 lainnya gagal.

![](/images/htop-me.png)

![](/images/benchmark-me.png)

### No 18
> Untuk memastikan ketiganya bekerja sama secara adil untuk mengatur Riegel Channel maka implementasikan Proxy Bind pada Eisen untuk mengaitkan IP dari Frieren, Flamme, dan Fern

#### Answer:
Di sini akan diimplementasikan Load Balancing dengan membagi beban secara merata kepada setiap workernya
Lakukan konfigurasi pada `/etc/nginx/sites-available/laravel-block` di Load Balancer seperti berikut ini:

```
upstream laravelworker  {
            server 192.220.4.1:8001; #IP Frieren
            server 192.220.4.2:8002; #IP Flamme
            server 192.220.4.3:8003; #IP Fern
    }

    server {
            listen 8080;
            server_name _;

            location / {
                    proxy_pass http://laravelworker;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

            error_log /var/log/nginx/lb_error.log;
            access_log /var/log/nginx/lb_access.log;
    }
```

#### Testing:  
Lakukan testing pada client dengan perintah berikut:

```
ab -n 100 -c 10 -p login.json -T application/json http://192.220.2.2:8080/api/auth/login
```

Terdapat 80 request yang diproses sedangkan 20 lainnya gagal.

![](/images/htop-lb.png)

![](/images/benchmark-lb.png)

Frieren

![](/images/frieren-handle.png)

Flamme

![](/images/flamme-handle.png)

Fern

![](/images/fern-handle.png)

### No 19
> Untuk meningkatkan performa dari Worker, coba implementasikan PHP-FPM pada Frieren, Flamme, dan Fern. Untuk testing kinerja naikkan pm.max_children, pm.start_servers, pm.min_spare_servers, pm.max_spare_servers
sebanyak tiga percobaan dan lakukan testing sebanyak 100 request dengan 10 request/second kemudian berikan hasil analisisnya pada Grimoire

#### Answer:

pm.max_children: Menentukan jumlah maksimum pekerja PHP (proses anak) yang dapat berjalan secara bersamaan. Nilai ini sebaiknya disesuaikan dengan kapasitas sumber daya server. Jika terlalu rendah, server mungkin tidak dapat menangani banyak permintaan secara bersamaan, sementara jika terlalu tinggi, dapat menyebabkan kelebihan beban dan kekurangan sumber daya.

pm.start_servers: Menentukan jumlah pekerja PHP yang akan dimulai secara otomatis ketika PHP-FPM pertama kali dijalankan atau direstart. Ini membantu dalam mengoptimalkan performa pada saat server pertama kali dimulai.

pm.min_spare_servers: Menentukan jumlah minimum pekerja PHP yang tetap berjalan saat server berjalan. Ini membantu menjaga agar server tetap responsif terhadap permintaan bahkan saat lalu lintas rendah.

pm.max_spare_servers: Menentukan jumlah maksimum pekerja PHP yang dapat berjalan tetapi tidak menangani permintaan. Jumlah ini disesuaikan dengan kebutuhan untuk menangani lonjakan lalu lintas tanpa menambahkan terlalu banyak sumber daya ketika beban rendah.

Terdapat 4 konfigurasi proses package manager pada masing-masing worker:

Konfigurasi 1:

```
echo '[www]
user = www-data
group = www-data
listen = /run/php/php8.0-fpm.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

; Choose how the process manager will control the number of child processes.

pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3' > /etc/php/8.0/fpm/pool.d/www.conf

service php8.0-fpm restart
```  

Konfigurasi 2

```
echo '[www]
user = www-data
group = www-data
listen = /run/php/php8.0-fpm.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

; Choose how the process manager will control the number of child processes.

pm = dynamic
pm.max_children = 25
pm.start_servers = 5
pm.min_spare_servers = 3
pm.max_spare_servers = 10' > /etc/php/8.0/fpm/pool.d/www.conf

service php8.0-fpm restart
```

Konfigurasi 3

```
echo '[www]
user = www-data
group = www-data
listen = /run/php/php8.0-fpm.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

; Choose how the process manager will control the number of child processes.

pm = dynamic
pm.max_children = 50
pm.start_servers = 8
pm.min_spare_servers = 5
pm.max_spare_servers = 15' > /etc/php/8.0/fpm/pool.d/www.conf

service php8.0-fpm restart
```

Konfigurasi 4

```
echo '[www]
user = www-data
group = www-data
listen = /run/php/php8.0-fpm.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

; Choose how the process manager will control the number of child processes.

pm = dynamic
pm.max_children = 75
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20' > /etc/php/8.0/fpm/pool.d/www.conf

service php8.0-fpm restart
```

#### Testing:  
Script 1

![](/images/config-1.png)

Script 2

![](/images/config-2.png)

Script 3

![](/images/config-2.png)

Script 4

![](/images/config-4.png)


### No 20
> Nampaknya hanya menggunakan PHP-FPM tidak cukup untuk meningkatkan performa dari worker maka implementasikan Least-Conn pada Eisen. Untuk testing kinerja dari worker tersebut dilakukan sebanyak 100 request dengan 10 request/second.

#### Answer:  
Karena ternyata hasil yang diberikan juga tidak cukup untuk meningkatkan performa worker. Oleh karena itu, ditambahkan algoritma Least-connection. Algoritma ini akan melakukan prioritas pembagian dari beban kinerja yang paling rendah. Node master akan mencatat semua beban dan kinerja dari semua node dan akan melakukan prioritas dari beban yang paling rendah. Sehingga diharapkan tidak ada server dengan beban yang rendah.

Lakukan konfigurasi pada `/etc/nginx/sites-available/laravel-block` seperti di bawah ini:
```
upstream laravelworker  {
            least_conn;
            server 192.220.4.1:8001; #IP Frieren
            server 192.220.4.2:8002; #IP Flamme
            server 192.220.4.3:8003; #IP Fern
    }

    server {
            listen 8080;
            server_name _;

            location / {
                    proxy_pass http://laravelworker;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

            error_log /var/log/nginx/lb_error.log;
            access_log /var/log/nginx/lb_access.log;
    }
```
#### Testing:  
Lakukan perintah: 
```
ab -n 100 -c 10 -p login.json -T application/json http://www.riegel.canyon.e28.com/api/auth/login
```

Semua request berhasil diproses

![](/images/benchmark-leastconnection.png)  

### Grimoire
[Google Docs](https://docs.google.com/document/d/1lvomvX1JvBtyktqLWGej73DYIFzhHMV7gt0fMgIox_U/edit)
