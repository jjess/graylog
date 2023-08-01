# Instalación graylog sobre docker

Documento oficial de graylog para docker:

https://go2docs.graylog.org/5-0/downloading_and_installing_graylog/docker_installation.htm


Pero para usar OpenSearch en lugar de ElasticSearch :

https://github.com/Graylog2/docker-compose/blob/main/open-core/docker-compose.yml



Para las passwords de GrayLog instalar:

```
sudo apt install pwgen
```

```
pwgen -N 1 -s 96
```

Para la password de root de graylog:

```
echo -n admin | shasum -a 256
```

o bien:

```
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```

Crear los directorios para datos persistentes:

```
sudo ln -s /var/lib/docker/volumes/graylog_mongodb_data     /opt/graylog/mongodb_data 
sudo ln -s /var/lib/docker/volumes/graylog_os_data          /opt/graylog/os_data
sudo ln -s /var/lib/docker/volumes/graylog_graylog_data     /opt/graylog/graylog_data
sudo ln -s /var/lib/docker/volumes/graylog_graylog_journal  /opt/graylog/graylog_journal
```


```
source ./env
sudo docker-compose --env-file ./.env up
```

Pero por alguna razón no pilla bien las variables de entorno, así que 
las ponemos dentro de docker-compose.yml

Entramos a graylog con el navegador:


http://192.168.56.201:9001/


admin : admin

Para pruebas en esta web explican mejor cómo enviar mensajes de prueba
con netcat:

https://www.techtarget.com/searchitoperations/tutorial/Centrally-manage-IT-logs-with-this-Graylog-tutorial


Creamos un input con las siguientes características:

* raw plaintext tcp
* port 555
* no tls
* NULL frame delimiter: YES

Creamos un mensaje de prueba desde la virtualización ubuntuserver3:

```
echo 'First CLI Log Message!' | nc -N localhost 5555
```

Y también funciona desde el host (freebsd):

```
echo 'Message from host bhyve (freebsd)' | nc -N 192.168.56.201 5555
```

También se puede crear un input de tipo UDP. Pero entonces el envío de mensajes
es de la forma:

```
echo 'First CLI Log Message!' | nc -4u -w1  localhost 5555
```

Desde el host (freebsd) es:

```
echo '3 mensaje desde freebsd' | nc -4u -w1 192.168.56.201 5555
```






