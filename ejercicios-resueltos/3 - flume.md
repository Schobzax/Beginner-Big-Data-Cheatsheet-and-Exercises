# Ejercicios de Flume

Vamos a configurar un agente con las siguientes características:
* Agente a1
* Puerto 44444
* Channel en memoria
* Sink por consola

Antes de nada debemos comprobar que telnet está instalado pues lo requeriremos para estos ejercicios.
```
$ yum install telnet telnet-server -y --Para instalarlo
$ sudo chmod 777 /etc/xinetd.d/telnet --Para dar permisos
$ vi /etc/xinetd.d/telnet --Accedemos al archivo
disable=no --Actualizamos esta variable
$ cat /etc/xinetd.d/telnet --Comprobamos el contenido
$ sudo service xinetd start --Iniciamos el servicio
$ sudo chkconfig telnet on --Hacemos que el servicio se inicie automáticamente
$ sudo chkconfig xinetd on --Lo mismo
```

Habiendo realizado la configuración previa, pasamos a configurar el agente.

1. Configuramos un archivo "example.conf" en `/home/cloudera`, cuyo contenido es el siguiente:
```
# Nombra los componentes del agente
a1.sources = r1 --La fuente usada por este agente se nombra r1.
a1.sinks = k1 --Se nombra k1 al sumidero.
a1.channels = c1 --Se nombra c1 al canal.
--Importante destacar que puede ser más de uno.

# Describe/configura la fuente
a1.sources.r1.type = netcat --El tipo de la fuente es netcat, así que los datos vendrán por un puerto de red.
a1.sources.r1.bind = localhost --Se usa el dominio localhost.
a1.sources.r1.port = 44444 --El puerto que se usa, por tanto, es localhost:44444

# Describe el sumidero
a1.sinks.k1.type = logger --Los datos aparecerán por consola.

# Usa un canal que guarda los eventos en memoria
a1.channels.c1.type = memory --Guarda los eventos en un buffer en memoria
a1.channels.c1.capacity = 1000 --Capacidad de 1000
a1.channels.c1.transactionCapacity = 100 -- Capacidad de transacción de 100

# Ata la fuente y sumidero al canal
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

2. Ahora abrimos un Shell y ejecutamos el comando que arranca el agente flume.
```
$ flume-ng agent --conf conf --conf-file /home/cloudera/example.conf --name a1 Dflume.root.logger=INFO,console
```
* `flume-ng` lanza flume
* `agent` lanza un agente flume
* `--conf` determina que se usará una configuración
* `--conf-file` determina el archivo de configuración que se utilizará mediante una ruta.
* `--name` es el nombre del agente. Debe coincidir con el nombre usado con el del archivo (a1, en este caso).
* Lo siguiente es una configuración del `logger`.

Recibiremos un montón de información, sobre todo de cuestiones de información y librerías. En cualquier caso, finaliza el agente iniciando sink, source y channel y cargando su configuración.

3. Ahora abrimos una terminal distinta e iniciamos una conexión telnet con el puerto 44444.
```
$ telnet localhost 44444
Trying 127.0.0.1...
Connected to localhost.
Escape character is `^]`.
```

Ahora todo lo que escribamos en la terminal telnet debe aparecer en la otra terminal.

Las líneas que veremos tienen este formato:

```
22/04/18 10:14:25 INFO sink.LoggerSink: Event: { headers:() body: <hex codes> <Textolim> }
```

`<hex codes>` se corresponde en contenido con lo que viene después en código hexadecimal. Sin embargo, la línea se corta ante una longitud de 16 caracteres en la parte de `<Textolim>`.

Ahora vamos a pasar a otra cosa: dejar ficheros en un directorio (spool-dir) como fuente.

4. Creamos el directorio spool y le damos permisos. También tenemos que hacer lo propio con los directorios checkpoint y datadir.
```
$ sudo mkdir -p /var/log/apache/flumeSpool
$ sudo chmod 777 /var/log/apache/flumeSpool
$ sudo mkdir -p /mnt/flume/checkpoint
$ sudo mkdir -p /mnt/flume/data
$ sudo chmod 777 /mnt/flume/checkpoint
$ sudo chmod 777 /mnt/flume/data
```
5. Creamos un segundo archivo de configuración, en el que cambiamos la fuente (`source`) por un spooldir, tal que así:
```
a1.sources.r1.type = spooldir
a1.sources.r1.spoolDir = /var/log/apache/flumeSpool --determina la ruta del archivo.
a1.sources.r1.fileHeader = true
```
6. Cargamos flume nuevamente, con el siguiente comando.
```
flume-ng agent --conf conf --conf-file /home/cloudera/example2.conf --name a1 Dflume.root.logger=DEBUG,console Dorg.apache.flume.lo.printconfig=true Dorg.apache.flume.log.rawdata=true
```
7. Para comprobar su funcionamiento, abrimos otra shell, nos posicionamos en la carpeta que hemos designado como spoolDir, y creamos ficheros.

Lo que veremos en la consola donde tenemos ejecutando el flume tendrá el siguiente formato:

`22/04/18 10:54:53 INFO sink.LoggerSink: Event: { headers:{file=/var/log/apache/flumeSpool/<filename>} body: <hex code> <shortcontent> }`

Como podemos comprobar, es muy parecido al caso anterior. Igualmente la longitud tiene limitaciones, muestra el contenido en hexadecimal, pero la diferencia es que ahora tenemos algo en el header: la ruta del archivo.

Vamos a añadir por último un paso adicional: ahora los datos irán del spool-dir que hemos configurado a HDFS.

8. Creamos los directorios necesarios en HDFS.
```
$ hadoop fs -mkdir /flume
$ hadoop fs -mkdir /flume/events
```

9. Creamos un último archivo de configuración que cambia el sink por HDFS con el Path del directorio HDFS el que acabamos de crear.
```
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /flume/events
```
10. Ejecutamos el agente.
```
$ flume-ng agent --conf conf --conf-file /home/cloudera/example3.conf --name a1 Dflume.root.logger=DEBUG,console Dorg.apache.flume.log.printconfig=true Dorg.apache.flume.log.rawdata=true
```
11. Ahora nos vamos al directorio spool en la segunda shell (si no la hemos cerrado seguirá ahí mismo) y creamos otro archivo.

El resultado de este segundo archivo no aparecerá por consola en este caso, sino que podremos verlo en HDFS, en la carpeta que hemos designado (/flume/events)
```
$ hadoop fs -ls /flume/events
Found 1 items
-rw-r--r--   1 cloudera supergroup        250 2022-04-18 11:12 /flume/events/FlumeData.1650273141710.tmp
```
El contenido es el del archivo que hemos escrito con una gran cantidad de carácteres especiales indescifrables, muchos con interrogaciones debido al encoding.

12. Vamos a cambiar la configuración de este último agente para que nos muestre más información.
```
a1.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H%M/%S
a1.sinks.k1.hdfs.filePrefix = events-
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = minute
a1.sinks.k1.hdfs.useLocalTimeStamp = true --Importante, si no el resto de las configuraciones de tiempo no funcionan y dan error.
```
Estas líneas crean subcarpetas divididas por timestamp, y cambian el nombre del archivo que se crea para mostrar más información.

Al acceder a la carpeta /flume/events mediante HDFS, vemos que se han creado las carpetas correspondientes: `/flume/events/22-04-18`, tal y como se ha determinado en la configuración.

La carpeta final que vemos se corresponde con el filepath que hemos configurado: `/flume/events/22-04-18/1130/00/events-.1650274434170`, el contenido del cual está lleno de carácteres especiales.

13. Vamos a proceder a hacer legible este último archivo, cambiando una última vez la configuración del agente:
```
a1.sinks.k1.hdfs.writeFormat = Text
a1.sinks.k1.hdfs.fileType = DataStream
```

Finalmente, el contenido de ese último fichero es exactamente el contenido del último fichero creado.