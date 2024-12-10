# P9-Apache-e-Virtual-Host

**1º configuraciṕon de docker-compose.yml**
```
version: '3.8'

services:
  web:
    image: php:7.4-apache
    container_name: apache_server
    ports:
      - "8080:80"
    volumes:
      - ./www:/var/www
      - ./confApache:/etc/apache2
    networks:
      apache_red:
        ipv4_address: 172.39.4.2

  dns:
    image: ubuntu/bind9
    container_name: dns_server
    ports:
      - "57:53"
    volumes:
      - ./confDNS/conf:/etc/bind
      - ./confDNS/zonas:/var/lib/bind
    networks:
      apache_red:
        ipv4_address: 172.39.4.3

networks:
  apache_red:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.39.0.0/16
```

**2º configurar en bin9 varios archivos el primero named.conf**
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
```

**3º configurar archivo named.conf.local**
```
zone "asircastelao.int" {
    type master;
    file "/var/lib/bind/db.asircastelao.int";
    allow-query { any; };
};
```

**4º configurar archivo named.conf.options**
```
options {
    directory "/var/cache/bind";
    recursion yes;
    allow-query { any; };
    dnssec-validation no;
    forwarders {
        8.8.8.8;
        1.1.1.1;
    };
    listen-on { any; };
    listen-on-v6 { any; };
};
```


**5º configurar archivo db.asircastelao.int**
```
$TTL    604800
@       IN      SOA     ns.asircastelao.int. some.email.address.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
@       IN      NS      ns.asircastelao.int.
ns      IN      A       172.39.4.3
fabulasmaravillosas  IN  A       172.39.4.2
fabulasoscuras       IN  A       172.39.4.2
```

**6º Creamos los archivos html**
Primero creaemos las carpetas donde iran las dos paginas webs

```
sudo mkdir /var/www/fabulosasmaravillosas
sudo mkdir /var/www/fabulasoscuras
```
```
 Y dentro los index.html

 fabulosasmaravillosas

<!DOCTYPE html>
<html>
<head>
    <title>Fabulas Maravillosas</title>
</head>
<body>
    <h1>Bienvenido a Fabulas Maravillosas</h1>
</body>
</html>


 fabulasoscuras

 <!DOCTYPE html>
<html>
<head>
    <title>Fabulas Oscuras</title>
</head>
<body>
    <h1>Bienvenido a Fabulas Oscuras</h1>
</body>
</html>

```
**7º DNS de las webs**
Dentro de la carpeta tanto de /etc/apache2/sites-avaliables como de /etc/apache2/sites-enabled vamos crear dos archivos:

fabulasmaravillosas.conf
```
<VirtualHost *:8080>
    ServerAdmin webmaster@localhost
    ServerName fabulasmaravillosas.asircastelao.int 
    ServerAlias www.fabulasmaravillosas.asircastelao.int 
    DocumentRoot /var/www/fabulasmaravillosas
</VirtualHost>
```

fabulasoscuras.conf
```
<VirtualHost *:8080>
    ServerAdmin webmaster@localhost
    ServerName fabulasoscuras.asircastelao.int
    ServerAlias www.fabulasoscuras.asircastelao.int
    DocumentRoot /var/www/fabulasoscuras
</VirtualHost>
```

**8º DNS**
Desactivaremos el DNS automático en el adaptador de red apartado IPv4.