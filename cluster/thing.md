# Guía de lo que he ido haciendo para montar un clúster Hadoop mediante máquinas virtuales y tal

Hola. Esta guía indica los pasos que he estado siguiendo de una manera más sucinta mediante un curso para crear un clúster Hadoop y no sé qué historias. No esperéis información detallada o exhaustiva. Solo voy a decir lo que he ido haciendo.

El enfoque principal es llevar la máquina a un estado concreto, solucionando los posibles errores que se presenten por el camino, no explicar la teoría ni ná de eso. Para eso mejor seguir el curso.

## Cuestiones importantes
Se asumen conocimientos informáticos suficientes como para que si quieres hacer las cosas de otra manera sepas qué partes tienes que cambiar.

## 1. Requisitos

* Descargarse VirtualBox. Yo he usado la versión 6.1.32.
* Descargarse CentOS. Yo he usado la versión 7.9.2009 para arquitectura x86-64.

## 2. Instalación

Después de instalar VirtualBox (asumo que hay un mínimo de conocimiento informático entre el lector) crear una nueva máquina virtual con las siguientes especificaciones:

* Nombre: nodo1
* Sistema operativo: Linux - Red Hat 64 bits.
* Memoria: 4 GB
* Disco: uno nuevo, dinámico, VDI, de 60 GB.
* Memoria de vídeo: 32 MB (esto se pone luego) en la configuración.

El resto se puede dejar tal como está. Es importante que la primera interfaz web sea NAT, pero lo demás se deja igual.

## 3. Instalación de CentOS

Se mete como disco en la máquina virtual el .iso descargado y se instala CentOS. Yo he usado los siguientes parámetros/cambios (ajustar el resto del documento al gusto si no se han usado):

* Español de España como idioma
* Selección de Software: Escrtorio Gnome (para que al entrar tengamos una interfaz gráfica)

El resto lo he dejado tal cual está. Es posible que te obligue a entrar en la configuración de disco, pero es entrar y salir, para que se active el particionado automático.

### Configuración de usuarios:
* Contraseña de root: apetecan
* Usuario: hadoop, Contraseña: gallifante. Importante marcarlo como administrador también (si no, el proceso posterior de instalación de Guest Additions será más engorroso).

Terminamos aceptando el acuerdo de licencia y conectando a la red por defecto (luego configuraremos una red interna).

## 4. Lo del Guest Additions
Lo del Guest Additions trae cola, pero siguiendo [esta útil guía](https://www.youtube.com/watch?v=XyEnoLWUrKE) y tal y pascual.

* Le damos a Dispositivos > Insertar imagen de las Guest Additions.
* Ejecutamos automáticamente lo que entra (no va a funcionar)
* Y ahora abrimos una terminal y ejecutamos `sudo yum -y install kernel-header kernel-devel gcc`
* Reiniciamos.
* Expulsamos el CD y lo volvemos a introducir para que se autoejecute la instalación. Que volverá a no funcionar.
* `sudo yum update kernel`
* Reiniciamos. Se nos abrirá un GRUB con las distintas versiones del kernel; no tenemos que hacer nada porque la versión que se inicia por defecto es la más actualizada. Así que ningún problema.
* Repetimos el proceso de expulsar y cargar el disco nuevamente.

Ya por fin podemos redimensionar pantallas, ajustar, fluir con el ratón, ya va todo perfecto. Siguiente paso.

## 5. Hadoop.
A partir de ahora haremos frecuentes accesos a internet: verifica que estás conectado a la red. *No sé por qué narices se enciende desconectado.*

* Descargamos Hadoop desde hadoop.apache.org (la última versión). En mi caso la versión 3.3.2. Nos descargamos en binary tal cual.
* Creamos una carpeta para Hadoop:
```
$ cd /opt
$ sudo mkdir hadoop
```
* Cambiamos los permisos para poder hacer cosas sin ser root: `sudo chown hadoop /opt/hadoop`
* Estando colocados en /opt/hadoop/, descomprimimos hadoop en esa carpeta: `tar xvf /home/hadoop/Descargas/hadoop-3.3.2.tar.gz`

Como somos tontos no nos hemos dado cuenta que se crea una carpeta dentro de la carpeta, así que ahora hay que arreglar eso también:
* `mv hadoop-3.3.2/* .` mueve lo de dentro de la carpeta fuera.
* `rm -r hadoop-3.3.2` borra la carpeta y sus contenidos.

## 6. Java
Es un percal. Nos metemos en root (`su - root`)Nos dirigimos a [la página de descarga](https://www.oracle.com/es/java/technologies/javase/javase8u211-later-archive-downloads.html) y nos descargamos el rpm x64 de 8u331.

*Te pedirá usuario y contraseña. Accede [aquí](http://bugmenot.com/view/oracle.com) y usa estos, porque f^<k oracle.*

* Localizados en la carpeta de descargas, ejecutamos `rpm -ivh jdk-8u331-linux-x64.rpm`.
* Después para usar la versión correcta, entramos en root (`su - root`) escribimos `alternatives --config java`. Deberían aparecer tres versiones, siendo la 3ª la que hemos instalado ahora mismo. Pulsamos `3`.

* Si al ejecutar `javac` nos lo reconoce tanto en root como en hadoop, es que va bien.

## 7. Variables de entorno.
Nuevamente como hadoop (salimos de root) entramos en la carpeta /home/hadoop y hacemos `gedit .bashrc` para abrir las variables de entorno.

Y ahí introducimos lo siguiente:
```
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
Guardamos y para actualizar ejecutamos `. ./.bashrc`

* Ejecutamos `alternatives --config java` y tenemos muy en cuenta la dirección de lo que tenemos activo (muestro mi caso):
```
+ 3      /usr/java/jdk1.8.0_331-amd64/bin/java
```
Nuevamente abrimos .bashrc, mediante gedit e introducimos la siguiente línea antes de PATH: `export JAVA_HOME=/usr/java/jdk1.8.0_331-amd64` (hasta antes del bin)

Para comprobar que funciona, ejecutamos `hadoop version`, debería mostrar la versión y otras cosas.

## 7'9. PEQUEÑO E IMPORTANTE APUNTE
Como soy imbécil no he cambiado el nombre de host, que es muy importante para la parte de a continuación. Por suerte, CentOS está hecho a prueba de idiotas y lo que podemos hacer es `sudo gedit /etc/hostname` y cambiar el nombre que aparece por `nodo1`, y reiniciar por si acaso.

Por otro lado es importante cambiar el dueño de la carpeta Hadoop: `chown -R hadoop:hadoop hadoop`

## 8. Algo de los hosts
* `ifconfig` (suponiendo que estemos conectados) nos mostrará nuestra IP en la interfaz en la que estamos conectados. Es importante apuntarla para escribirla en hosts.
* `gedit /etc/hosts`, añadamos la línea `<IP> nodo1`: `10.0.2.15 nodo1`.

## 9. Ejemplo de uso de hadoop
Para ir abriendo boca, nos dirigimos a `/opt/hadoop/share/hadoop/mapreduce/`, y vamos a pasar los archivos xml a una carpeta temporal:

```
$ mkdir /tmp/entrada
$ cp /opt/hadoop/etc/hadoop/*.xml /tmp/entrada
$ hadoop jar hadoop-mapreduce-examples-3.3.2.jar grep /tmp/entrada /tmp/salida 'kms[a-z]+'
```
En /tmp/salida aparecen los sospechosos habituales: `_SUCCESS` y `part-r-00000`.

**Primer error: No me sale lo mismo que en el vídeo o que en el pdf.**

## 10. SSH
* `ssh-keygen` en la carpeta raíz del usuario (/home/hadoop).
* `cd .ssh`
* `cp id_rsa.pub authorized_keys`.
* `ssh nodo1` para conectarnos a nosotros mismos y añadirnos a la lista de `known_hosts`.

# 11. Modificación de los archivos .xml
Nos vamos a `/opt/hadoop/etc/hadoop` que es donde están los xml

### core-site.xml
Abrimos core-site.xml mediante el editor de preferencia y cambiamos esta parte:

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://nodo1:9000</value>
    </property>
</configuration>
```

### hdfs-site.xml
Abrimos hdfs-site.xml mediante el editor de preferencia y cambiamos esta parte:

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/datos/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/datos/datanode</value>
    </property>
<configuration>
```

### Formateo
Formateamos.

1. Crear las carpetas, desde root (su - root):
```
$ cd /
$ sudo mkdir /datos
$ sudo mkdir /datos/datanode
$ sudo mkdir /datos/namenode
$ sudo chown -R hadoop:hadoop datos
```
Y con eso a funcionar, con `hdfs namenode -format`, para formatear el sistema de ficheros que vamos a estar usando y tal.

## 12. Inicio
Se enciende con `start-dfs.sh` (nuevamente, verificar que estamos conectados o dará error ssh 22, porque no hay internet).

En `jps` debería aparecernos lo siguiente: 
```
6439 NameNode
6600 DataNode
6954 Jps
6797 SecondaryNameNode
```
Que son los procesos que se tienen que ejecutar.

Para comprobar que funciona: `hdfs dfs -ls /` no debería dar error. Accedemos a `nodo1:9870` y aparece una página de tiñe verdoso que nos indica información sobre el sistema de archivos: "Namenode information".

## 13. Un ejemplo de trabajo con ficheros
```
$ hdfs dfs -ls /
$ echo Hola >> prueba.txt
$ hdfs dfs -mkdir /temporal
$ hdfs dfs -put prueba.txt /temporal
$ hdfs dfs -ls /temporal
```
Y dentro está el archivo. Esto también se puede ver explorando el sistema de archivos gráficamente en lo de nodo1:9870, que tiene un apartado para eso.

Hacer lo mismo con el access_log (que está en la carpeta de las prácticas). La diferencia aquí es que por el tamaño, se puede ver gráficamente que access_log está en varios bloques mientras que un "Hola" pues no, porque es de menos de 128 MB que es el tamaño del bloque.

También se puede hacer esto:
```
$ hdfs dfs -cat /temporal/prueba.txt
Hola
$ hdfs dfs -mkdir /temporal1
$ hdfs dfs -cp /temporal/prueba.txt /temporal1/prueba.txt
$ hdfs dfs -rm /temporal/prueba.txt
$ hdfs dfs -get /temporal1/prueba.txt /home/hadoop/test.txt
```

## 14. Seguir el ejercicio para cosas
Realmente no tiene mayor relevancia, pero para dejar la cosa tal y como está mejor ponerlo así.

```
$ hdfs dfs -mkdir /datos
$ echo "Esto es una prueba" > /tmp/prueba.txt
$ hdfs dfs -put /tmp/prueba.txt /datos
$ cd /datos/datanode/current/BP-1838032218-10.0.2.15-1652873974572/current/finalized/subdir0/subdir0 -- Cambia solo el número del bloque BP
$ cat blk_1073741831 -- ea.
```

Ahora generamos un archivo grande de 1 G lleno de ceros.
```
$ dd if=/dev/zero of=/tmp/fic_grande.dat bs=1024 count=1000000
$ hdfs dfs -put /tmp/fic_grande.dat /datos
```
Este archivo estará en múltiples bloques.

Podemos generar otro.
```
$ hdfs dfs -mkdir /practicas
$ hdfs dfs -cp /datos/prueba.txt /practicas/prueba.txt
$ hdfs dfs -ls /practicas
$ hdfs dfs -rm /practicas/prueba.txt
```

### Ahora vamos a generar un proceso Hadoop
Primero tenemos que generar dos archivos de texto cualesquiera (un gedit, un touch, una cosa) y guardarlos en /tmp. Luego los meteremos en la carpeta practicas del hdfs y a partir de ahí realizaremos el ejemplo.

```
$ cd /opt/hadoop/share/hadoop/mapreduce
$ hdfs dfs -put /tmp/palabras.txt /practicas
$ hdfs dfs -put /tmp/palabras1.txt /practicas
$ hadoop jar hadoop-mapreduce-examples-3.3.2.jar wordcount /practicas /salida1
```
Y aparecerán los sospechosos habituales: `part-r-00000` y `_SUCCESS`, con, en este caso, el conteo de palabras y vacío, respectivamente.

## 15. Administración y Snapshots
En cuanto a comandos de administración tenemos:
* `hdfs dfsadmin -report` muestra un informe del sistema.
* `hdfs fsck /` muestra el estado del sistema de ficheros a partir del directorio indicado. Indica la salud a partir del nº de bloques subreplicados (con menos replicación de la indicada) (con opciones para listar distintas partes como `-files`, `-blocks` y `-locations`)
* `hdfs dfsadmin -printTopology` imprime la topología de nodos.
* `hdfs dfsadmin -listOpenFiles` muestra los ficheros 

Para hacer una snapshot vamos a hacer lo siguiente:

```
$ echo Esto es una prueba > f1.txt
$ hdfs dfs -put f1.txt /datos
$ hdfs dfsadmin -allowSnapshot /datos --Permite la creación de snapshots en/de la carpeta /datos.
$ hdfs dfs -createSnapshot /datos snap1
```
Y esto lo que hace es que si ahora borramos f1 podemos recuperarlo directamente con un copia-pega de los datos de la snapshot.

### El ejercicio de las snapshots
```
$ echo Ejemplo de Snapshot > /tmp/f1.txt
$ hdfs dfs -mkdir /datos4
$ hdfs dfs -put /tmp/f1.txt /datos4
$ hdfs dfsadmin -allowSnapshot /datos4
$ hdfs dfs -createSnapshot /datos4 s1
$ hdfs dfs -rm /datos4/f1.txt
$ hdfs dfs -cp /datos4/.snapshot/s1/f1.txt /datos4/ -- Este es el método.
```

## 16. Chapter 5 lo vamos viendo adios