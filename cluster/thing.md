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

## 15.9 Supresión de warnings
```
export HADOOP_HOME_WARN_SUPPRESS=1
export HADOOP_ROOT_LOGGER="WARN,DRFA"
```

## 16. YARN
Nuevamente en /opt/hadoop/etc/hadoop, tenemos que modificar el archivo `mapred-site.xml`:

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

Antes de modificar el siguiente, ejecutamos `hadoop classpath` y copiamos el resultado.

Y también tenemos que modificar el archivo `yarn-site.xml`:
```
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>nodo1</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.application.classpath</name>
        <value>
            [el valor de lo copiado en hadoop classpath, que es un churro bastante voluminoso]
        </value>
    </property>

</configuration>
```

Y por último, iniciamos, `start-dfs.sh` y `start-yarn.sh`.

Esto crea una interfaz web accesible por el puerto 8088, con información de yarn.

Por último, hay que ejecutar el comando `mapred --daemon start historyserver` para temas del historial.

* Resumiendo, a partir de ahora cada vez que iniciemos:
`start-dfs.sh`, `start-yarn.sh`, `mapred --daemon start historyserver`.

* Y para pararlo: `stop-dfs.sh`, `stop-yarn.sh`, `mapred --daemon stop historyserver`.

## 17. Ejemplo quijote
Tenemos que poner el archivo quijote.txt en la carpeta que prefiramos y bueno esta parte ya te la sabes cómo va lo de los archivos.
```
$ hdfs dfs -mkdir /libros
$ hdfs dfs -put quijote.txt /libros
$ hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.2.jar wordcount /libros /salida_libros
$ hdfs dfs -get /salida_libros/part-r-00000 palabras.txt
```

## 18. Ejemplo contar palabras (java)
Para la ejecución de archivos java, lo primero que hay que hacer es `export HADOOP_CLASSPATH=/usr/java/jdk1.8.0_331/lib/tools.jar`.

### Compilación
* `javac ContarPalabras.java -cp $(hadoop classpath)`

### Creación del jar
* `jar cf ContarPalabras.jar Contar*.class`

### Ejecución
* `hadoop jar ContarPalabras.jar ContarPalabras /temporal/access_log /salidaLog`

## 19. Ejemplo con Mapper y Reducer en Python
Creamos los dos archivos python nativamente tal y como pone en el pdf (por reducir espacio no se ponen aquí). *Crearlos nativamente en la propia máquina virtual es la mejor manera de evitar problemas de compatibilidad, terminaciones de líneas, encoding, etcétera.*

Después ejecutamos `hadoop jar /opt/hadoop/share/hadoop/tools/lib/hadoop-streaming-3.3.2.jar -file pymap.py -mapper pymap.py -file pyreduce.py -reducer pyreduce.py -input /libros/quijote.txt -output /resultado4`

## 20. Clonación
Para clonar el nodo Hadoop debemos configurar:

* Red: El Adaptador 2 es una red interna con nombre intnet.

Y luego a la hora de clonar hay que configurarlo todo para que no repita nada: En MAC generar nuevas direcciones, y no marcar nada de mantener nombres o UUIDs.

Cambiarle el nombre a nodo2 y escoger "Estado actual de la máquina" (eso sale porque he estado haciendo instantáneas).

## 21. Configuración de la red
A partir de ahora hay que hacer ciertas cosas tanto en nodo2 como en nodo3. Lo único que hay que hacer es cambiar donde se vea nodo2 por nodo3 y ya está. Lo demás es todo igual solo que hay que ejecutarlo dos veces, una en cada nodo.

**Es muy importante que las tres máquinas virtuales estén conectadas a la red interna para esta parte.**

### Nombre

Lo primero que vamos a hacer es cambiarle los nombres, y para que esto sea permanente lo vamos a apuntar en `/etc/hostname`.

De esta manera, accedemos mediante `sudo gedit /etc/hostname` y escribimos nodo2 o nodo3 sustituyendo a nodo1 en el nodo correspondiente.

### Sysconfig
En cada uno de los tres nodos, hacemos `sudo gedit /etc/sysconfig/network`. Nos aparecerá como texto `# created by anaconda`. Lo borramos. Escribimos lo siguiente:
```
NETWORKING=yes
HOSTNAME=nodoX (siendo X 1, 2 o 3 dependiendo del nodo)
```

### Hosts
Hacemos lo propio con /etc/hosts, que en cada uno de los tres nodos debe tener el mismo contenido:

```
127.0.0.1     localhost localhost.localdomain localhost4 localhost4.localdomain4
::1           localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.101 nodo1
192.168.0.102 nodo2
192.168.0.103 nodo3
```
Y luego nos dirigimos a la configuración de red y le asignamos la IPv4 manual correspondiente al nodo (101, 102 o 103, como se ve en el archivo /etc/hosts). Máscara: 255.255.255.0, Puerta de enlace: 192.168.0.1.

Reiniciamos los tres nodos para que se guarde la confi. Después comprobamos con ping que todos los nodos están conectados entre sí y que todo se ha hecho correctamente.

### SSH
En cada uno de los nodos (los tres) hay que hacer lo siguiente:

1. Borrar el contenido del fichero /home/hadoop/.ssh
2. Nos conectamos al nodo2 y al nodo3 mediante ssh desde el nodo1 para añadirlos a la lista de known hosts.
3. El vídeo recomienda tres pestañas, una para cada nodo, conectándose como se ha dicho.
4. Nos metemos en la carpeta .ssh de cada nodo.
5. Ejecutamos `ssh-keygen` en cada uno de los nodos.

Ahora en el nodo1 hacemos `cp id_rsa.pub authorized_keys` y lo hacemos circular:
```
$ scp authorized_keys nodo2:/home/hadoop/.ssh
```
Y allí hacemos `cat id_rsa.pub >> authorized_keys` para que se añada al final del fichero. Esto lo pasamos al nodo3 con el mismo comando de arriba, y este último se pasa al nodo1 y al nodo2.

*Nota: Para que SSH funcione correctamente, hay que ejecutar `chmod 600 authorized_keys` en el propio directorio .ssh, para que los permisos los tenga el propio nodo. Si no se hace así, SSH se pone nervioso y no funciona.

### Ficheros de configuración de los nodos
Hay que borrar según qué carpetas: en los nodos 2 y 3 hay que borrar el `/datos/namenode`, porque eso solo está en el nodo maestro. También hay que borrar `/datos/datanode/current` en los nodos *worker*, porque es una copia y los vamos a recrear.

El nodo1 (nodo maestro) es al revés: hay que borrar el datanode.

Ahora hay que configurar ciertos archivos:

#### hdfs-site.xml
Cambiar `dfs.replication` de 1 a 2. (Porque ahora tenemos dos nodos).

Para transferirlo a los nodos worker se hace mediante `scp hdfs-site.xml nodox:/opt/hadoop/etc/hadoop/`.

El resto de archivos no hay que tocarlos, se quedan igual.

#### workers
Aquí se pone
```
nodo2
nodo3
```
Los nombres (identificadores) de los nodos worker.

### Últimos pasos
1. Quitar el cortafuegos. En CentOS 7 es más complicado, pero se puede hacer así:
```
$ sudo systemctl stop firewalld // lo detiene para esta sesión.
$ sudo systemctl disable firewalld // previene que se inicie en siguientes sesiones.

// Para comprobar el estado (por si otro servicio lo encendiera):
sudo firewall-cmd --state
```
He rehusado esconderlo ante otros servicios porque me parece una medida demasiado drástica, pero sería con `sudo systemctl mask --now firewalld`. 

Esto nuevamente debemos hacerlo en los tres nodos.

2. Por último, formateamos con `hdfs namenode -format` (nos preguntará si sobreescribimos, que sí)

## 22. Arranque
El arranque arranca más procesos de forma automática. Luego el jps visualiza distintos según el nodo. Aquí no hay que modificar nada respecto al arranque tradicional.

En un maestro estarían: ResourceManager, JobHistoryServer, SecondaryNameNode, Jps y NameNode.

En un nodo esclavo estarían: NodeManager, Jps y DataNode.

Y bueno, ya es cuestión de hacer pruebas y la gracia es ver su funcionamiento en el modo gráfico que mola mucho.

## 23. Ejercicios prueba

### Primer ejercicio

* `hdfs dfs -mkdir /practicas` y tal.
* `hdfs dfs -put cite75_99.txt /practicas` para meter el archivo.
* Creamos un archivo `MyJob.java` según el contenido del pdf.
* Exportamos la librería para el classpath: `export HADOOP_CLASSPATH=$JAVA_HOME/lib/tools.jar`
* Compilamos: `hadoop com.sun.tools.javac.Main MyJob.java` y creamos el JAR `jar cvf MyJob.jar My*`

Finalmente ejecutamos el job: `hadoop jar MyJob.jar MyJob /practicas/cite75_99.txt /resultado7`

### Segundo ejercicio: streaming mediante comandos de linux

* `hadoop jar /opt/hadoop/share/hadoop/tools/lib/hadoop-streaming-3.3.2.jar -input /practicas/cite75_99.txt -output /resultado8 -mapper 'cut -f 2 -d ,' -reducer 'uniq'`: con reducer
* `hadoop jar /opt/hadoop/share/hadoop/tools/lib/hadoop-streaming-3.3.2.jar -D mapred.reduce.tasks=0 -input /practicas/cite75_99.txt -output /resultado9 -mapper 'wc -l'`: sin reducer

Todo esto se puede ver muy bien en la interfaz web el proceso y tal y mola mucho.

### Tercer ejercicio: Python

* `hadoop jar/opt/hadoop/share/hadoop/tools/lib/hadoop-streaming-3.3.2.jar -D mapred.job.reducers=1 -input /practicas/cite75_99.txt -output /resultado11 -mapper 'rand.py 1' -file rand.py`

**Segundo error: En el papel pone `-mapper 'rand.py'` pero esto lleva a error a no ser que se incluya el 1. Seguramente no esté haciendo ni lo mismo, pero no sé a qué se debe el error.**

## 24. Gestión del cluster
* El comando `yarn` tiene muchas opciones muy útiles seguro para quien sepa lo que está haciendo.
* `yarn application` muestra las aplicaciones activas.
* `yarn container` muestra los contenedores que han ejecutado las aplicaciones.
* `yarn node` muestra información de los nodos con los que estamos trabajando.
* `yarn applicationattempt` muestra los intentos de ejecución de cada aplicación.

Cada una tiene una serie de opciones muy interesantes. Por ejemplo, `-list`, que muestra una lista.

Mencionar que solo se mostrarán datos activos; es decir, aplicaciones que se están ejecutando en el momento concreto en que se usa el comando. También se pueden chapar aplicaciones con `yarn application -kill <id>`, para aplicaciones que vemos que van a tardar mucho o que no van bien.

## 25. Yarn Scheduler
También es muy interesante.
* `mapred queue -list` muestra las colas de dicho scheduler, con lo que se está ejecutando y tal.

La configuración es importante hacerla bien: El Capacity Scheduler.

### Capacity Scheduler
* `gedit capacity-scheduler.xml` en el fichero donde están todos los xml a configurar.

Una vez dentro tenemos que modificar según qué propiedades, que lo haremos varias veces durante estas prácticas.
```
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,prod,desa</value>
    <description>
        lo que ponga aquí que no nos interesa.
    </description>
</property>
```

Y luego hay que hacer un poquito de matemáticas: por cada nivel del árbol de colas (mirarse la teoría que viene muy bien explicado):
```
<property>
    <name>yarn.scheduler.capacity.root.[nombrecola].capacity</name>
    <value>[numero entre el 1 y el 100]</value>
    <description>Capacidad de la cosa esta.</description>
</property>
```

Y entre todas las colas tienen que sumar 100. O entre todas las subcolas por sí solas. Esas cosas.

En este caso se queda una cosa así:
```
  <property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,prod,desa</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>30</value>
    <description>Default queue target capacity.</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.prod.capacity</name>
    <value>50</value>
    <description>Default queue target capacity.</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.desa.capacity</name>
    <value>20</value>
    <description>Default queue target capacity.</description>
  </property>
```

* Para actualizar las cosas (necesario), `yarn rmadmin -refreshQueues`.

### Subcolas
Te muestro tal cual un ejemplo de subcolas para que lo veas. Hemos añadido dos subcolas a prod.

```
  <property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,prod,desa</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.prod.queues</name>
    <value>warehouse,batch</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>30</value>
    <description>Default queue target capacity.</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.prod.capacity</name>
    <value>50</value>
    <description>Default queue target capacity.</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.prod.warehouse.capacity</name>
    <value>40</value>
    <description>Default queue target capacity.</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.prod.batch.capacity</name>
    <value>60</value>
    <description>Default queue target capacity.</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.desa.capacity</name>
    <value>20</value>
    <description>Default queue target capacity.</description>
  </property>
```

* Muy importante: Debido a la estructura hay colas funcionando que vamos a actualizar así que no podemos refrescar las colas: tendremos que reiniciar el clúster. (Stop y luego Start como aparece un poco más arriba).

Por último, como demostración del funcionamiento de las colas ejecutamos `hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.2.jar wordcount -Dmapred.job.queue.name=warehouse /practicas/cite75_99.txt /salida16`.

### Ejemplo
* El ejemplo indica {default,10},{rrhh,50},{marketing,20},{ventas,20} - lo cual es fácil de hacer. En el mismo archivo que hemos estado modificando hasta ahora (capacity-scheduler.xml):
```
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,rrhh,marketing,ventas</value>
    <description>Lo que ponga aquí.</description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>10</value>
    <description>Capacidad de la cola default</description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.rrhh.capacity</name>
    <value>50</value>
    <description>Capacidad de la cola rrhh</description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.marketing.capacity</name>
    <value>20</value>
    <description>Capacidad de la cola marketing</description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.ventas.capacity</name>
    <value>20</value>
    <description>Capacidad de la cola ventas</description>
</property>
```

Debido a la magnitud del cambio (básicamente que estamos cambiando cosas en ejecución), tendremos que reiniciar el clúster nuevamente.

## 26. Instalación de Hive

1. Nos descargamos la última versión de Hive en [la página de Hive](hive.apache.org). En mi caso estoy utilizando la versión 3.1.3.

2. Una vez tenemos el archivo, lo descomprimimos en /opt/hadoop (nos colocamos allí): `tar xvf /home/hadoop/Descargas/apache-hive-3.1.3.bin.tar.gz` y luego lo renombramos para facilidad de uso con `mv apache-hive-3.1.3-bin/ hive`.

3. Después tenemos que modificar el .bashrc, añadiendo `export HIVE_HOME=/opt/hadoop/hive` antes del path y luego añadiendo `$HIVE_HOME/bin` al path.

4. Ahora hay que meterse en la configuración de los xml, para lo cual  `cd hive/conf` (estando en /opt/hadoop/hive/conf).

5. Como veremos casi todo son templates que no podemos usar directamente, sino que debemos copiarlas a un archivo estándar eliminándoles la terminación template.

### hive-default.xml
* `cp hive-default.xml.template hive-site.xml`, en el mismo directorio. A este le hemos cambiado el nombre porque sí.

Hay que añadir un par de propiedades al principio del documento:
```
  <property>
    <name>system:java.io.tmpdir</name>
    <value>/tmp/hive/java</value>
  </property>
  <property>
    <name>system:user.name</name>
    <value>${user.name}</value>
  </property>
```

Además debemos solucionar un error localizado en la línea 3224 en la columna 96: hay un caracter extraño en una descripción. Al ser en una descripción, podemos borrarlo sin ningún problema.

### hive-env.sh.template
* `cp hive-env.sh.template hive-env.sh`

Entramos y definimos dos variables:
```
export HADOOP_HOME=/opt/hadoop
export HIVE_CONF_DIR=/opt/hadoop/hive/conf
```

### hive-exec-log4j2.properties
* `cp hive-exec-log4j2.properties.template hive-exec-log4j2.properties`

### hive-log4j2.properties
* `cp hive-log4j2.properties.template hive-log4j2.properties`

### beeline-log4j2.properties
* `cp beeline-log4j2.properties.template beeline-log4j2.properties`

## 27. Configuración de HDFS para Hive
Hay que crear unos directorios (o tenerlos creados) en HDFS para que no falle la cosa.

* `hdfs dfs -mkdir /tmp`, con permisos `hdfs dfs -chmod g+w /tmp`.
* `hdfs dfs -mkdir -p /user/hive/warehouse`, con permisos `hdfs dfs -chmod g+w /user/hive/warehouse`

Creamos una carpeta /opt/hadoop/hive/bbdd, y dentro lanzamos el comando `schematool -dbType derby -initSchema`, que inicializa el schema en la carpeta correspondiente de la base de datos por defecto.

**Es posible que de error de schematool: command not found. Se debería solucionar cerrando la terminal y abriendo otra nueva.**

Imprime un montón de líneas vacías. Eso es un poco raro.

Debería aparecer `schemaTool completed`, finalmente.

## 28. Creación de cosas en la base de datos hive (un poco lo que vamos haciendo ahí dentro)
Como curiosidad, se crea un directorio con la base de datos dentro de la carpeta warehouse en HDFS, por ejemplo: `/user/hive/warehouse/ejemplo.db/` y ahí dentro pues carpetas para las tablas y tal. Y dentro pues ficheros numéricos que al hacerle cat tiene el contenido de parte de la tabla.

En este apartado no voy a explicar mucho de hive porque ya lo hemos hecho antes.

* `hive> create database ejemplo;`
* `hive> use ejemplo;` -- muy importante estar en la base de datos concreta, no te mete automáticamente.
* `hive> create table if not exists t1 (name string);`
* `hive> insert into t1 values ('mi nombre');`
* `hive> create table t2 (codigo integer);`
* `hive> insert into t2 values(10);`

### Tablas internas

Ahora con un ejemplo más complejo, vamos a crear un archivo de texto en otro lugar con el formato `[nombre],[edad]`, con una coma en medio para separar los valores. Uno por línea, a decisión propia. Por ejemplo, sería algo así (desde fuera de hive): `gedit empleados.txt`
```
juan,28
maría,33
...
```

Y para introducir estos datos pues creamos una tabla tal que así: `hive> create table empleados(nombre string, edad integer) row format delimited fields terminated by ',';`

Ahora, para cargar los datos, se ejecuta `load data local inpath '/home/hadoop/empleados.txt into table empleados;`. Asumimos que se sabe lo que significa este comando así que no voy a explicarlo.

Para propósitos ilustrativos, se dirá que hive lee todos los contenidos de la carpeta asociada (/user/hive/warehouse/ejemplo.db/empleados) y que al borrar la tabla se borra el contenido de esa carpeta.

### Tablas externas
Por otro lado tenemos las tablas externas.

* `hive> drop table empleados;` (vamos a recrearla)
* Introducimos el archivo en el sistema en `hdfs dfs -put empleados.txt /prueba`
* `hive> create external table empleados (nombre string, edadinteger) row format delimited fields terminated by ',' location '/user/hive/datos/empleados'` (carpeta que se crea automáticamente. Location determina la localización de los datos)
* `hive> load data inpath '/prueba/empleados.txt'`.

La diferencia con una tabla interna radica en que al dropear la tabla los datos permanecen en la localización designada en `location`.

## 29. Ejercicio
```
hive> CREATE TABLE IF NOT EXISTS empleados_internal
    > (
    >   name string,
    >   work_place ARRAY<string>,
    >   sex_age STRUCT<sex:string,age:int>,
    >   skills_score MAP<string,int>,
    >   depart_title MAP<STRING,ARRAY<STRING>>
    > )
    > COMMENT 'This is an internal table'
    > ROW FORMAT DELIMITED
    > FIELDS TERMINATED BY '|'
    > COLLECTION ITEMS TERMINATED BY ','
    > MAP KEYS TERMINATED BY ':';
```
Y cargamos los datos con `hive> LOAD DATA LOCAL INPATH '/home/hadoop/ejercicios/empleados.txt' OVERWRITE INTO TABLE empleados_internal;` (el archivo empleados.txt está disponible en el curso, el que vamos a usar ahora lo vamos a guardar en la carpeta ejercicios -- pero vamos lo que queráis no soy vuestro padre)

Y luego para la tabla externa igual
```
hive> CREATE EXTERNAL TABLE IF NOT EXISTS empleados_external
    > (
    >   name string,
    >   work_place ARRAY<string>,
    >   sex_age STRUCT<sex:string,age:int>,
    >   skills_score MAP<string,int>,
    >   depart_title MAP<STRING,ARRAY<STRING>>
    > )
    > COMMENT 'This is an external table'
    > ROW FORMAT DELIMITED
    > FIELDS TERMINATED BY '|'
    > COLLECTION ITEMS TERMINATED BY ','
    > MAP KEYS TERMINATED BY ':'
    > LOCATION '/ejemplo/empleados';
```
Y luego cargamos con `LOAD DATA LOCAL INPATH '/home/hadoop/ejercicios/empleados.txt' OVERWRITE INTO TABLE empleados_external;`.

Por último, y pasando un poco de las comprobaciones (lo dejo al lector que debería ir siguiendo la cosa) dropeamos las dos tablas. Los datos de la externa se quedarán, no así los de la interna.

## 30. Beeline
Lo primero que tenemos que hacer antes de empezar a trabajar con beeline es modificar una propiedad muy concreta de beeline para poder realizar ejecuciones "anónimas". Tenemos que dirigirnos al fichero `/opt/hadoop/hive/conf/hive-site.xml` y cambiar la propiedad "hive.server2.enable.doAs" por "false".

**Tercer error: A mí me ha salido aleatoriamente. No sé muy bien por qué funciona o por qué deja de funcionar. *Temas raros de puertos, supongo.***

### El ejercicio de deslizamientos de tierra
Pues ya es cuestión de hacerlo. Ahí no me voy a meter, se deja como ejercicio para el lector.

Sí debo advertir que la ultimísima parte de Excel no funciona del todo correctamente en Excel 365 (o no he encontradoals opciones), porque parece que la interfaz web está un poco limitada respecto a la versión de escritorio, de la que carezco.

## 31. Hue

No funciona. Siguiente.

## 32. Sqoop

Sqoop básicamente nos permite transferir información de bases de datos relacionales a hdfs, y viceversa.

Usaremos la versión descargada desde [esta página](http://archive.apache.org/dist/sqoop/1.4.7/sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz). Preveo que me va a dar muchos problemas, pero aquí hemos venido a jugar, órdago a la grande y me quedo con mi caja.

Una vez descargada, nos dirigimos a la carpeta `/opt/hadoop`, descomprimimos el archivo con `tar xvf /home/hadoop/Descargas/sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz` y lo renombramos la carpeta con `mv sqoop-1.4.7.bin__hadoop-2.6.0/ sqoop/`.

Ahora hay que editar el archivo .bashrc, añadiendo la variable `export SQOOP_HOME=/opt/hadoop/sqoop` y el `:$SQOOP_HOME/bin` añadido al $PATH.

### Configuración de Sqoop
Nos dirigimos al directorio /opt/hadoop/sqoop/conf, y generamos un archivo a partir de plantilla con `cp sqoop-env-template.sh sqoop-env.sh`.

En ese tenemos que cambiar estas líneas, añadiéndoles la ruta:
* `export HADOOP_COMMON_HOME=/opt/hadoop`
* `export HADOOP_MAPRED_HOME=/opt/hadoop`
* `export HIVE_HOME=/opt/hadoop/hive`

Reiniciar la consola para que los cambios de .bashrc tengan efecto, si no lo has hecho antes (cerrar y abrir).

Ahora al ejecutar `sqoop-version` nos saldrán mensajes de error de cosas que no tenemos instaladas pero por lo demás todo bien

En cuanto a la ejecución no se verá en este documento porque a) está centrado en temas de configuración de clúster y b) ya hemos usado sqoop anteriormente.

Lo de Oracle no funciona. Pasamos a otra cosa.
<!--
### Oracle Express
[He seguido esta guía](https://oracle-base.com/articles/21c/oracle-db-21c-xe-rpm-installation-on-oracle-linux-7-and-8), referente a la parte de RHEL 7.

**Error no se qué Lang** https://mvnrepository.com/artifact/commons-lang/commons-lang/2.6

// OLD
[Sería muy conveniente seguir esta guía.](https://oraxedatabase.blogspot.com/2020/04/como-instalar-oracle-database-20c.html)

Y luego para que el sqoop funcione hay que tener instalado `commons-lang-2.6.jar` y `ojdbc8.jar`, en la carpeta /opt/hadoop/sqoop/lib. **Importante**.

Como prueba de funcionamiento, ejecutamos `sqoop-import --connect jdbc:oracle:thin:@nodo1:1521:XE --username HR --password HR --table DEPARTMENTS --target-dir /empleados --as-textfile`. No funciona. Me da error de conexión. Me pego un tiro. Fallo. Me pego otro tiro. Vuelvo a fallar. Error de conexión. Intento arreglarlo. No lo consigo. Me pego otro fallo. Esta vez vuelvo a fallar. Repita *ad nauseam*.

VALE! La solución es no generar el usuario HR:HR y hacerlo directamente desde root. NO es la solución, pero eh. Ahí está la cosa.

Efectivamente, no era la solución.
-->
## 33. ZooKeeper
Seguir la guía y vídeos del curso. Hay que hacer `scp` a todo lo que modifiquemos.

En este momento por problemas de configuración he borrado el nodo1 y lo he recreado a partir del nodo2, porque estoy hasta las narices.

Luego lo he borrado para descargarme la versión correcta, que no es la última sino la que dice en la guía; borrando cualquier rastro de todo después del punto 30.

Lo que hay que hacer en este caso es seguir la guía, habiéndose descargado exactamente la versión que se usa en el curso, que es la 3.4.11.

# 34. Spark
Esto ya lo conocemos. Debido a conflictos con Zookeeper, se revierte a una versión pre-Zookeeper.

* Importante descargarse la versión con hadoop "provisto por el usuario" (es decir, que venga sin hadoop adjunto).

Después del procedimiento habitual (rpm en /opt/hadoop y mv carpeta para renombrarla) ya visto en herramientas anteriores, modificamos el .bashrc añadiéndole dos variables:

* `export SPARK_DIST_CLASSPATH=$(hadoop classpath)`
* `export SPARK_HOME=/opt/hadoop/spark`
* `export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin`
* `export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop`

**Si te aparece un error del tipo `Error initializing SparkContext`, comprueba que la conexión en la que estás es la que está configurada para el clúster.**

**Si te aparece un error al cargar pyspark del tipo 'python3' no se reconoce comando, es que tienes que instalar python3. Lo mejor es hacerlo mediante yum.**

Luego, para el ejemplo de los puertos.csv, hay que ejecutar los siguientes comandos:
```
$ hdfs dfs -mkdir /spark
$ hdfs dfs -put /home/hadoop/puertos.csv /spark
$ spark-shell
scala> val v1 = sc.textFile("/spark/puertos.csv")
```

Y bueno luego ejecutar las cosas.

El programa python da un fallo de lectura de python3 o no sé qué historia.

### Modo Standalone
Para este hay que descargarse el paquete en el que va incluido hadoop. En mi caso, `spark-3.2.1-bin-hadoop3.2.tgz`. Por ejemplo lo vamos a hacer en la carpeta `/home/hadoop/spark_standalone` y tal.

* Nuevamente tenemos que hacer el mv para cambiar de nombre a la carpeta.
* Después arrancamos `start-master.sh`, que está en sbin en la carpeta que hayamos creado. Esto nos crea una interfaz web accesible mediante 8080. Ahí dice que está escuchando el nodo1 por el puerto 7077.
* Así que ahora arrancamos `start-slave.sh nodo1:7077`.

Para acceder a la shell de esta versión concreta lo hacemos desde `bin/spark-shell`

# 35. HBase
Procedimiento habitual. Instalamos el paquete. Yo me he instalado la versión 2.4.12. En /opt/hadoop/hbase.

Editamos `/opt/hadoop/hbase/conf/hbase-site.xml`, el cual NO está vacío, y colocamos lo siguiente, manteniendo lo que ya haya y se corresponda:
```
  <property>
    <name>hbase.cluster.distributed</name>
    <value>false</value>
  </property>
  <property>
    <name>hbase.tmp.dir</name>
    <value>./tmp</value>
  </property>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///opt/hadoop/hbase/data/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/hadoop/hbase/data/hbase</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
```

Y luego, localizado en bin, ejecutamos `./start-hbase.sh`

Podemos también añadir HBASE_HOME al PATH tal cual lo hemos hecho en ocasiones anteriores.

Ahora lo que hay que hacer es juguetear con los comandos de HBase.

Una nota quizá de cierta importancia es el uso del comando `put`, que hace las veces de INSERT y de UPDATE también.

### Semidistribuido 
Para esto tenemos que realizar una instalación semidistribuida de HBase.

1. Modificar el archivo hbase-site.xml que ya hemos modificado antes.

```
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>

<property>
  <name>hbase.rootdir</name>
  <value>hdfs://nodo1:9000/hbase</value>
</property>

<property>
  <name>hbase.zookeeper.quorum</name>
  <value>nodo1,nodo2,nodo3</value>
</property>
```

2. En el archivo hbase-env.sh hay que modificar lo siguiente.
```
# Tell HBase whether it should manage it's own instance of ZooKeeper or not.
export HBASE_MANAGES_ZK=false
export HBASE_DISABLE_HADOOP_CLASSPATH_LOOKUP=true
```

Sigue sin funcionar, así que pasamos del tema.

## 36. Ambari
Por lo visto ha sido retirado al ático. Igualmente lo vamos a hacer, pero parecía tanto y resulta que ahora no vale de nada.

Vamos a ir directos a por la versión 2.2.1, que es la versión que se usa en la guía. La instalación de la última versión difiere lo suficiente, tanto es así que tan solo está la opción de compilar la fuente. O yo no he encontrado la opción de cliente binario o como sea.

**Error 403 Forbidden**

Pues me cago en la leche. Por lo visto Cloudera ha pasado a repositorios privados para Cloudera y HDP (que no sé lo que es, pero me lo imagino) y hacen falta credenciales. Por eso da acceso forbidden. Voy a limitarme a ver los vídeos, y después buscaré una guía.