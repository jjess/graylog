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


## rsyslogd por UDP

En la máquina linux (ubuntuserver3) configuramos:

/etc/rsyslog.d/graylog.conf:

```
# Double @@ is for TCP whereas a single @ is for UDP
#*.* @@localhost:5140;RSYSLOG_SyslogProtocol23Format
*.* @localhost:5140;RSYSLOG_SyslogProtocol23Format
```

Probamos primero por UDP ya que es mucho más rápido (menos fiable).

El input debe tener las siguientes características:

* syslog udp 
* port: 5140

# Retention

Para ajustar la retención de logs en graylog y que no ocupe mucho.

https://stackoverflow.com/questions/37313445/graylog2-how-to-config-logs-retention-to-1-week

En el menú:

System/Configurations > Configurations
    Index Set Defaults -> Edit Configuration

    Index Rotation Configuration:
        Select Rotation Strategy:   Index Time
        Rotation Period:            P1D (un índice por día)

    Index Retention Configuration:
        Select Rotation Strategy:   Delete Index 
        Max Number of Indices:      8

Con la configuración anterior habrá 8 indices de OpenSearch,
uno por día. Indices más antiguos de 8 dias se borran.


Para la API de graylog basta crear un usuario con role Reader.
Al usuario hay que crearle un token.

La llamada a la API para traer los eventos:

```
curl --location 'http://192.168.56.201:9001/api/events/search' \
--header 'X-Requested-By: postman' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic MXJhZzVqZjV2b3Y5cnAwNDBqNGk1N2RjcmdvNjFodDZ2OXM4YnRzZjhmMzE2N3RldTJtMTp0b2tlbg==' \
--data '{
    "query" : "session opened for user root" ,
    "timerange": {
        "type": "relative",
        "range": 30000
        }
    }'
```



