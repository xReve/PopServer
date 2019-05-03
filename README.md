# GUIA USUARI
# INSTAL·LACIÓ I CONFIGURACIÓ SERVIDOR POP IMPLEMENTANT DOCKERS I AWS
## ERIC ESCRIBA


### PAS a PAS

* En primer pas abans creem un docker de prova el qual utilitzarem per treballar i extreure dades. 

```
docker run --name popserver -h popserver -it fedora:24 /bin/bash

```

### Configuració POP

* Primer que tot hem d'instal·lar el paquet **dovecot** el qual incoropora el servidor **POP**.

```
dnf -y install dovecot
```

* Una vegada instal·lat, editarem el fitxer de configuració per tal de que ens obri el server **POP**.

vim `/etc/dovecot/dovecot.conf`


* Editarem la següent linia, la qual previament estava comentada, la descomentem:

```
protocols = imap pop3 lmtp
```

* Abans d'iniciar el servei prepararem els certificats perquè el servidor pugi treballar amb mode segur **SSL**.

* Borrem els següents certificats:

`/etc/pki/dovecot/certs/dovecot.pem`

`/etc/pki/dovecot/private/dovecot.pem`

* Ara crearem els certificats autosignats amb la següent ordre:

```
/usr/libexec/dovecot/mkcert.sh
```

* I ara si que ja podem iniciar el nostre servidor:

```
/usr/sbin/dovecot
```

### COMPROVACIÓ SERVIDOR ACTIU

```
[root@popserver dovecot]# telnet localhost 110
Trying ::1...
Connected to localhost.
Escape character is '^]'.
+OK Dovecot ready.
AUTH
+OK
PLAIN
.
```
 
### CREACIÓ USUARIS i COMPROVACIÓ

```
useradd pere
passwd pere

useradd marta
passwd marta

[root@popserver dovecot]# tail -2 /etc/passwd
marta:x:1001:1001::/home/marta:/bin/bash
pau:x:1002:1002::/home/pau:/bin/bash

[root@popserver dovecot]# ll /var/spool/mail/
total 0
-rw-rw----. 1 marta mail 0 May  3 08:58 marta
-rw-rw----. 1 pere  mail 0 May  3 08:57 pere
```

### CREACIÓ IMATGES DOCKER

* Per què aquest process tingui una funcionalitat fa falta automatizar-l'ho.

* Creem una imatge amb un **Dockerfile** i tots els seus fitxers de configuració adients:

```
docker build -t eescriba/m11eric:v1 .
```

* Creem la xarxa en que aquest servidor ha de funcionar:

```
docker network create popnet
```

* Finalment activem el contenidor en mode **detach** 

```
[root@ip-172-31-0-77 servidorPop]# docker run --rm --name popserver -h popserver --network popnet -d eescriba/m11eric:v1
d87af2d6a8fce3532063c11ef8096979724ee84a4730690cbdf79f7de1b96323

[root@ip-172-31-0-77 servidorPop]# docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS               NAMES
d87af2d6a8fc        eescriba/m11eric:v1   "/opt/docker/start..."   2 seconds ago       Up 1 second         110/tcp, 995/tcp    popserver
```


### AWS

* Pel que respecta a amazon hem d'obrir els ports **110** i **995** perquè ens pugem connectar.

* Per fer això hem d'anar a la configuració del grup de la instància i editar les regles de **INBOUND**



### ORDRES DOCKERHUB

* Creem la versió latest de la imatge del servidor **POP**

[root@ip-172-31-0-77 servidorPop]# docker tag eescriba/m11eric:v1 eescriba/m11eric:latest
[root@ip-172-31-0-77 servidorPop]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
eescriba/m11eric    latest              fbee003c62f1        12 minutes ago      524 MB
eescriba/m11eric    v1                  fbee003c62f1        12 minutes ago      524 MB

* Pujem la imatge al **DOCKERHUB**

```
[root@ip-172-31-0-77 servidorPop]# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: eescriba
Password: 
Login Succeeded
```
```
docker push eescriba/m11eric:v1

docker push eescriba/m11eric:latest
```


### COMPROVACIÓ TELNET DESDE LOCAL

* Comprovem que hi ha connexió al docker de amazon desde local:

```
[root@i24 ~]# telnet 35.177.0.165 110
Trying 35.177.0.165...
Connected to 35.177.0.165.
Escape character is '^]'.
+OK Dovecot ready.
AUTH
+OK
.
```

```
[root@i24 ~]# telnet 35.177.0.165 995
Trying 35.177.0.165...
Connected to 35.177.0.165.
Escape character is '^]'.
USER pere
Connection closed by foreign host.
```

**No hem deixa autenticar**

