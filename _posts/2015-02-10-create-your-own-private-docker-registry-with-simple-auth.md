---
layout: post
title: Prywatne docker registry z prostą autoryzacją
---

## Instalacja prywatnego registry dockera
*pod ubuntu 14.04*


<table>
  <thead>
    <tr>
      <th>paczka</th>
      <th>wersja</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>docker</td>
      <td>1.4.1, build 5bc2ff8</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>docker-registry</td>
      <td>0.9.1</td>
    </tr>
    <tr>
      <td>python</td>
      <td>2.7.6</td>
    </tr>
  </tbody>
</table>

### Zależności

```
$ apt-get -y install build-essential python-dev libevent-dev python-pip liblzma-dev swig libssl-dev
```

### Instalacja przez pip
*dla wersji 0.9.1*
```
$ pip install docker-registry
```
>Jeśli pojawią się problemy przy budowaniu pakietu **M2Crypto** to znaczy że nie mamy **swig**a - można zainstalować przez ```apt-get``` albo bibliotek openssl - paczka **libssl-dev** załatwia sprawę.

### Konfiguracja

Domyślna konfiguracja powinna znajdować się mniej więcej:
```/usr/local/lib/python2.7/dist-packages/config``` w pliku ```config_sample.yml```. Wystarczy jej zmienić nazwę na ```config.yml```.

> Test lokalizacji plików docker-registry: ```gunicorn --access-logfile - --debug -k gevent -b 0.0.0.0:5000 -w 1 docker_registry.wsgi:application``` i sprawdzenie czego brakuje.

> Tutaj również znajduje się plik konfiguracyjny dla mirrora repozytorium: ```config_mirror.yml```

-------

##### Zmiana domyślnej ścieżki /tmp i ustawienie ścieżki dla logów

```
mkdir -p /var/docker-registry /var/log/docker-registry
```

Podmienić w pliku konfiguracyjnym ```config.yml``` ścieżki:

```
    21c21
    <     sqlalchemy_index_database: _env:SQLALCHEMY_INDEX_DATABASE:sqlite:////tmp/docker-registry.db
    ---
    >     sqlalchemy_index_database: _env:SQLALCHEMY_INDEX_DATABASE:sqlite:////var/docker-registry/docker-registry.db
    74c74
    <     storage_path: _env:STORAGE_PATH:/tmp/registry
    ---
    >     storage_path: _env:STORAGE_PATH:/var/docker-registry/registry
    165c165
    <     storage_path: _env:STORAGE_PATH:/tmp/registry
    ---
    >     storage_path: _env:STORAGE_PATH:/var/docker-registry/registry
    221c221
    <     storage_path: _env:STORAGE_PATH:./tmp/test
    ---
    >     storage_path: _env:STORAGE_PATH:./var/docker-registry/test
```

Testowe odpalenie:

```
$ gunicorn --access-logfile - --debug -k gevent -b 0.0.0.0:5000 -w 1 docker_registry.wsgi:application
10/Feb/2015:00:16:31 +0000 WARNING: Cache storage disabled!
10/Feb/2015:00:16:31 +0000 WARNING: LRU cache disabled!
10/Feb/2015:00:16:31 +0000 DEBUG: Will return docker-registry.drivers.file.Storage
```

Konfiguracja i instalacja ```redis-server``` - cache:

```
$ apt-get -y install redis-server
```

W sekcjach *cache* i *cache_lru* w pliku ```config.yml```  podmienić:

```
    cache:
        host: localhost
        port: 6379
        db: 0

    cache_lru:
        host: localhost
        port: 6379
        db: 1
```
Teścik:

```
gunicorn --access-logfile - --debug -k gevent -b 0.0.0.0:5000 -w 1 docker_registry.wsgi:application

10/Feb/2015:00:27:17 +0000 INFO: Enabling storage cache on Redis
10/Feb/2015:00:27:17 +0000 INFO: Redis host: localhost:6379 (db0)
10/Feb/2015:00:27:17 +0000 INFO: Enabling lru cache on Redis
10/Feb/2015:00:27:17 +0000 INFO: Redis lru host: localhost:6379 (db0)
10/Feb/2015:00:27:17 +0000 INFO: Enabling storage cache on Redis
10/Feb/2015:00:27:17 +0000 INFO: Redis config: {'path': '/var/docker-registry/registry', 'host': 'localhost', 'password': None, 'db': 0, 'port': 6379}
10/Feb/2015:00:27:17 +0000 DEBUG: Will return docker-registry.drivers.file.Storage

```

Konfiguracja skryptu Upstart ```/etc/init/docker-registry.conf```:

```
description "Docker Registry"

start on runlevel [2345]
stop on runlevel [016]

respawn
respawn limit 10 5

script
exec gunicorn --access-logfile /var/log/docker-registry/access.log --error-logfile /var/log/docker-registry/server.log -k gevent --max-requests 100 --graceful-timeout 3600 -t 3600 -b localhost:5000 -w 8 docker_registry.wsgi:application
end script
```

Opalany jako service:
```
service docker-registry start
```

#### Konfiguracja ssl i prostej autoryzacji

```
$ apt-get -y install nginx apache2-utils
```

```
$ htpasswd -c /etc/nginx/docker-registry.htpasswd USERNAME
```

```
/etc/nginx/sites-available/docker-registry
```

```
upstream docker-registry {
 server localhost:5000;
}

server {
 listen 8080;
 server_name private.registry.docker.example.com;

 # ssl on;
 # ssl_certificate /etc/ssl/certs/docker-registry;
 # ssl_certificate_key /etc/ssl/private/docker-registry;

 proxy_set_header Host       $http_host;   # required for Docker client sake
 proxy_set_header X-Real-IP  $remote_addr; # pass on real client IP

 client_max_body_size 0; # disable any limits to avoid HTTP 413 for large image uploads

 # required to avoid HTTP 411: see Issue #1486 (https://github.com/dotcloud/docker/issues/1486)
 chunked_transfer_encoding on;

 location / {
     # let Nginx know about our auth file
     auth_basic              "Restricted";
     auth_basic_user_file    docker-registry.htpasswd;

     proxy_pass http://docker-registry;
 }
 location /_ping {
     auth_basic off;
     proxy_pass http://docker-registry;
 }  
 location /v1/_ping {
     auth_basic off;
     proxy_pass http://docker-registry;
 }

}
```

```
$ ln -s /etc/nginx/sites-available/docker-registry /etc/nginx/sites-enabled/docker-registry
```

```
$ servier nginx restart
```


```
$ service docker-registry start
```

Teściki - pierwszy sprawdza czy repozytorium *nadaje*, drugi czy nginx *nadaje* a właściwie wymaga uprawnień, trzeci powinien dać wynik jak pierwszy:
```
$ curl localhost:5000
$ curl localhost:8080
$ curl USERNAME:PASSWORD@localhost:8080
```

#### Podpięcie pod registry

Instalacja dockera:
```
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9;
```
```
/etc/apt/sources.list.d/docker.list
```
```
deb https://get.docker.io/ubuntu docker main
```

```
$ apt-get install lxc-docker
```

```
$ gpasswd -a ${USER} docker
```

```
$ service docker restart
```

```
$ docker login http://localhost:8080

Username: ghost
Password: 
Email: 
Login Succeeded
```

#### Praca z repozytorium

```
$ docker run -t -i ubuntu /bin/bash
Unable to find image 'ubuntu:latest' locally
ubuntu:latest: The image you are pulling has been verified
511136ea3c5a: Pull complete 
27d47432a69b: Pull complete 
5f92234dcf1e: Pull complete 
51a9c7c1f8bb: Pull complete 
5ba9dab47459: Pull complete 
Status: Downloaded newer image for ubuntu:latest
root@04bc2c52881e:/# touch /SUCCESS
root@04bc2c52881e:/# exit
$ docker commit $(docker ps -lq) test-image
fd2659d1f8d017d46fdaa37bc450669a3b6833b97129f57e269f616db680f919
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
test-image          latest              fd2659d1f8d0        14 seconds ago      192.7 MB
ubuntu              latest              5ba9dab47459        12 days ago         192.7 MB
```

#### Konfiguracja SSL

Odkomentować linie w konfiguracji nginx:

```
 ssl on;
 ssl_certificate /etc/ssl/certs/docker-registry;
 ssl_certificate_key /etc/ssl/private/docker-registry;
```

##### Podpisanie własnym CA
*wszędzie ustawienia domyślne*

```
$ mkdir ~/certs && cd ~/certs
$ openssl genrsa -out devdockerCA.key 2048
$ openssl req -x509 -new -nodes -key devdockerCA.key -days 10000 -out devdockerCA.crt
$ openssl genrsa -out dev-docker-registry.com.key 2048
$ ls

devdockerCA.crt  devdockerCA.key  dev-docker-registry.com.key
```
*teraz trzeba uważać na to co się wpisuje w polu **Common Name*** - to samo co w nazwie serwera w konfiguracji **nginx**a - np. *private.registry.docker.example.com*

```
$ openssl req -new -key dev-docker-registry.com.key -out dev-docker-registry.com.csr
$ openssl x509 -req -in dev-docker-registry.com.csr -CA devdockerCA.crt -CAkey devdockerCA.key -CAcreateserial -out dev-docker-registry.com.crt -days 10000
$ ls
devdockerCA.crt  devdockerCA.srl              dev-docker-registry.com.csr
devdockerCA.key  dev-docker-registry.com.crt  dev-docker-registry.com.key
```
Przeniesć...

```
$ cp dev-docker-registry.com.crt /etc/ssl/certs/docker-registry
$ cp dev-docker-registry.com.key /etc/ssl/private/docker-registry
$ mkdir /usr/local/share/ca-certificates/docker-dev-cert
$ cp devdockerCA.crt /usr/local/share/ca-certificates/docker-dev-cert
$ update-ca-certificates
$ service nginx restart
$ service docker restart
```

W pliku ```/etc/hosts``` dodać przekierowanie dla domeny

```
127.0.0.1 private.registry.docker.example.com
```
Test:

```
$ curl https://USERNAME:PASSWORD@private.registry.docker.example.com:8080
```

###### Dodawanie crt na klientach dockera
Na tym etapie logowanie przy pomocy:

```
docker login https://private.registry.docker.example.com:8080
```

Skończy się błędem

```
FATA[0004] Error response from daemon: Server Error: Post https://private.registry.docker.example.com:8080/v1/users/: x509: certificate signed by unknown authority 

```

Dlatego... na kliencie trzeba wgrać nasz nieautoryzowany certyfikat.

```
$ mkdir /usr/local/share/ca-certificates/docker-dev-cert
```

Wrzucić zawartość, wygenerowanego wcześniej *devdockerCA.crt* do np. ```/usr/local/share/ca-certificates/docker-dev-cert/devdockerCA.crt```

```
$ update-ca-certificates
$ service docker restart
```

-------
TODO:

- Instalacja ze źródeł
- instalacja w virtualenv
- konfiguracja redis-server - dodać parametr dla lru cache
- konfiguracja ssl dla certa wydanego przez startssl
- więcej o użyciu z klientem dockera

-------
##### Source:
1. [How To Set Up a Private Docker Registry on Ubuntu 14.04 - digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04)
2. [Deploying your own Private Docker Registry](http://www.activestate.com/blog/2014/01/deploying-your-own-private-docker-registry)
3. [Docker Registry or How to Run your own Private Docker Image Repository](https://blog.codecentric.de/en/2014/02/docker-registry-run-private-docker-image-repository/)
4. [Building private Docker registry with basic authentication](https://medium.com/@deeeet/building-private-docker-registry-with-basic-authentication-with-self-signed-certificate-using-it-e6329085e612)
5. [Setup Your Own Docker Registry on CoreOS](https://www.vultr.com/docs/setup-your-own-docker-registry-on-coreos)
6. [Redis configuration](http://redis.io/topics/config)
7. [Redis lru cache mode](http://redis.io/topics/lru-cache)