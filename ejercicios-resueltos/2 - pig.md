# Ejercicios Pig

Por convención, las palabras reservadas se escribirán en mayúsculas. Esto es una convención y no es necesario, pero ayuda a distinguirlo de valores variables o identificadores.

1. El comando para lanzar pig es el siguiente:
    ```
    $ pig -x local
    ```
A continuación se nos informa varias veces de que tres librerías están desactualizadas y que deberíamos usar otras distintas. Esto cae fuera del ámbito del curso así que tendremos que lidiar con ello. Se abrirá una terminal `grunt>` para escribir los comandos.

2. Cargar los datos en pig en una variable llamada "data", con los nombres de las columnas siendo (key, campana, fecha, tiempo, display, accion, cpc, pais, lugar). Los tipos de las columnas deben ser chararray excepto accion y cpc que son int.
    ```
    grunt> data = LOAD 'ejercicios/pig/datos_pig.txt'
    >> AS (key:chararray,
    >>     campana:chararray,
    >>     fecha:chararray,
    >>     tiempo:chararray, 
    >>     display:chararray, 
    >>     accion:int, 
    >>     cpc:int, 
    >>     pais:chararray, 
    >>     lugar:chararray);
    --La ruta donde se haya pig por defecto es /home/cloudera así que debemos trabajar a partir de ahí.
    ```
Este comando carga una variable con los datos del archivo de texto. Dado que por defecto el delimitador es TAB y el archivo separa los datos por TAB no tenemos por qué usar un delimitador personalizado.

3. Usa el comando DESCRIBE para ver el esquema de la variable "data".
    ```
    grunt> DESCRIBE data;
    data: {key: chararray,campana: chararray,fecha: chararray,tiempo: chararray,display: chararray,accion: int,cpc: int,pais: chararray,lugar: chararray}
    ```
Este comando nos saca el esquema de la variable que acabamos de crear.

4. Seleciona las filas de "data" que provengan de USA
    ```
    grunt> usa = FILTER data BY pais == 'USA';
    --Ahora para comprobar que el filtro ha funcionado vamos a mostrar los datos de la variable usa.
    grunt> DUMP usa;
    --Esto lleva a una ejecución de un trabajo arduo, lento y proceloso, pero finalmente se imprimen los datos.
    ```
El por qué de que tarde tanto radica en las diferencias entre map y reduce que se darán en más detalle en Spark, pero el hecho es que hasta que no hacemos un DUMP (que sería una acción) no se ejecuta el resto de acciones previas (que serían transformaciones, por así decirlo).

5. Listar los datos que contengan en su key el sufijo surf:
    ```
    grunt> keysurf = FILTER data BY key MATCHES '.*surf';
    --La expresión regular usada nos permite ver las tuplas cuya KEY termine en "surf" (en eso consiste un sufijo).
    --Nuevamente al hacer dump se realiza el Job y finalmente se imprimen los datos.
    ```

6. Crear una variable llamada "ordenado" que contenga las columnas de data en el siguiente orden: (campana, fecha, tiempo, key, display, lugar, accion, cpc):
    ```
    grunt> ordenado = FOREACH data GENERATE campana, fecha, tiempo, key, display, lugar, accion, cpc;
    --Se genera una nueva "tabla" a partir de la "tabla" data con las columnas generadas en el orden en el que se indican.
    --En su esencia es un reordenamiento de las columnas, pero nada nos hubiera impedido cambiar el contenido, presentar datos calculados adicionales, o algo similar.
    ```

7. Guardar el contenido de la variable "ordenado" en una carpeta en el local file system de tu MV llamada "resultado" en la ruta /home/cloudera/ejercicios/pig. Comprobar el contenido de la carpeta.
    ```
    grunt> STORE ordenado INTO '/home/cloudera/ejercicios/pig/resultado';
    --Insistimos en que la carpeta donde se almacenen los datos *No debe existir previamente*.
    ```
Para comprobar los contenidos debemos salir de pig. Eso se hace con `quit;`.
    ```
    $ ls /home/cloudera/ejercicios/pig/resultado
    part-m-00000  _SUCCESS
    --Se nos muestra que tenemos el resultado y un archivo que dice que hemos tenido éxito.
    ```
El archivo _SUCCESS está vacío, y el archivo part-m-00000 contiene los datos tal y como se verían si hubiéramos hecho un `dump ordenado;`, pero solo los datos. Nada de la parte inicial sobre la ejecución del job.

Está separado por el delimitador por defecto, tab. Nuevamente podemos configurar ese aspecto usando el comando correspondiente.