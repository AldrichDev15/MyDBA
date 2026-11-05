# CREAR CDB Y PDB
Hay dos métodos para crear una nueva DB:
- Usando DBCA (_**D**ata**B**ase **C**reation **A**ssistant_).
  Es el método recomendado y puede ser lanzado con OUI (_**O**racle **U**niversal **I**nstaller_) o de forma independiente. Tiene opción una opción gráfica o podemos poner la herramienta en modo _silent_ y hacerlo por línea de comandos. 
- Usando la sentencia `CREATE DATABASE`. Solo sirve para _single-instance_ . Es más manual, pero permite crear las DB a partir de scripts.
  Si usamos esta opción tendremos que configurar los discos ASM previo a la creación. También tendremos que construir vistas en el diccionario de datos e instalar algunos paquetes mediante scripts que nos vienen dados. 
  Los pasos son los siguientes:
     1) Elegir la `ORACLE_SID`, usada para identificar instancias en el propio host (caracteres válidos y sensibilidad a mayúsculas y minúsculas depende de la plataforma).
     2) Verificar que las variables de entorno están correctamente especificadas. `ORACLE_SID`, `ORACLE_HOME`, `PATH` son algunos ejemplos.
     3) Decidir un método de autenticación, bien usando un [_password file_](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/getting-started-with-database-administration.html#GUID-26EC8D1B-7A2F-4F0A-9588-D92CBD610858 "You can create a database password file using the password file creation utility, ORAPWD. For some operating systems, you can create this file as part of your standard installation.") o bien entrando al host con un usuario perteneciente a los grupos del OS apropiados.
     4) Crear el _pfile_ usado para la inicialización de la instancia. [Aquí hay un ejemplo]([Creating and Configuring an Oracle Database](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-4ECB5C59-ED5A-4A87-9EEA-6C687AB8986D).
     5) (Solo para Windows) Hay que crear la instancia previo a poder conectarnos usando el comando `oradim`.
     6) Conectar a la DB (`sqlplus /nolog`) autenticándonos por el OS (`CONN / AS SYSDBA`) o con _password file_ (`CONN SYS AS SYSDBA`).
     7) Crear el _spfile_ a partir del _pfile_ (paso recomendado) usando `CREATE SPFILE FROM PFILE;` desde SQL\*Plus. Hay que reiniciar la DB para que el _spfile_ sea el fichero que use la DB.
     8) Arrancar la instancia con `STARTUP NOMOUNT` en SQL\*Plus. En este momento la memoria está asignada y los procesos arrancados, pero la DB en sí no existe.
     9) Lanzamos el comando `CREATE DATABASE`. Tenemos varios ejemplos en [Step 9: Issue the CREATE DATABASE Statement](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-CE58B606-4534-4149-878F-C0DFEF32AA49). La sentencia `CREATE DATABASE` al menos crea _data files_, _control files_, _online redo logs_, el _tablespace SYSTEM_ y _SYSAUX_, el diccionario de datos, establece el juego de caracteres y la zona horaria y monta y abre la DB para su uso.
        Este comando tiene muchas cláusulas para iniciar la creación de la DB.
     10) Lanzar [los scripts](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-FE3078EE-9EF7-4229-BA17-AAD3C6AE7784) que permiten construir las vistas del diccionario de datos, sinónimos, paquetes PL/SQL... También podemos lanzar posteriormente scripts para implementar opciones adicionales.
     11) Los últimos pasos serían configurar la nueva DB para arrancar automáticamente si el host se reinicia y hacer una copia de seguridad inicial de la DB.
Solo es necesario crear la DB una vez, sin importar el número de _data files_ que tenga o el número de instancias que accedan a ella.

Para planificar la creación de la DB, debemos tener en cuenta los siguientes factores:
- El espacio inicial de las tablas y los índices. Considerar también el espacio inicial del [_tablespace_ SYSAUX](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-4EBA1922-B701-4009-B662-6A17B314E2E9 "Oracle strongly recommends that you create a default permanent tablespace. Oracle Database assigns to this tablespace any non-SYSTEM users for whom you do not explicitly specify a different permanent tablespace.") y de si usar o no _tablespace_ para [UNDO](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/managing-undo.html#GUID-2C865CF9-A8B5-4BF1-A451-E8C08D3611F0 "For a default installation, Oracle Database automatically manages undo. There is typically no need for DBA intervention. However, if your installation uses Oracle Flashback operations, you may need to perform some undo management tasks to ensure the success of these operations.").
- La estructura de ficheros del OS que utilizará la DB. Esto puede impactar tremendamente en la base de datos en cuanto a operaciones de E/S (ORACLE recomienda, por ejemplo, situar la _Fast Recovery Area_ en un dispositivo de almacenamiento distinto de los _data files_). Para simplificar esto, usar _**O**racle **M**anaged **F**iles_ y _**A**utomatic **S**torage **M**anager_.
- El [nombre global](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-9698EA9B-013C-40AC-8CFB-12171A4B4974)de la DB con los parámetros de inicialización `DB_NAME` y `DB_DOMAIN`.
- [Seleccionar el juego de caracteres](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-2D16B9A8-99A0-48BD-9DFB-FF180EA2CABE), Oracle tiene por defecto (a partir de 12c R2) y recomienda AL32UTF8 (codificación UTF-8 con el estándar Unicode, soporta la mayoría de los lenguajes). 
  La universalidad de este juego de caracteres normalmente pesa más que otras desventajas que pueda tener como un procesado de texto más lento que con un juego de caracteres que solo use un byte o el requerir un mayor espacio de almacenamiento para textos.
  En una plataforma basada en ASCII, solo es posible elegir un juego de caracteres basado en ASCII para la base de datos.
  Oracle no recomienda usar UTF8 a no ser que necesitemos compatibilidad con versiones anteriores o nos lo especifíquen explícitamente. AL32UTF8 y UTF8 no son compatibles debido a la anchura máxima de sus caracteres (4 bytes vs. 3 bytes).
- Considerar los [usos horarios](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-6925369A-FA41-4FE7-B146-E8F4A231E632 "All databases that share information must use the same time zone data file.") que soportará la DB. El fichero por defecto para esta configuración es `timezlrg_11.dat`.
- El [tamaño de bloque](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-55124C2D-98FF-4305-8F25-58245C9BEC1E "The DB_BLOCK_SIZE initialization parameter specifies the standard block size for the database.") de la DB, con el parámetro `DB_BLOCK_SIZE`. No se puede cambiar, y es el que se usa para el _tablespace_ de SYSTEM (aunque otros _tablespace_ pueden tener otros tamaños de bloque). Si no especificamos, tomaremos por defecto el tamaño de bloque del OS, que es adecuado en general.
  Si elegimos otro tamaño de bloque, asegurarnos que el número es múltiplo del del OS. Un tamaño de bloque más grande es más eficiente en I/O de disco y memoria.
  También debemos considerar el [tamaño de bloque de los _redo logs_](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/managing-the-redo-log.html#GUID-AC858C69-317E-4193-838D-59F8DDEA1927).
- Si queremos el _Oracle home_ en modo _read only_ (a partir de 18c) o _read write_. 
  El _Oracle home_ en modo _read only_ previene la creación y modificación de ficheros en el _Oracle home_. Esto se configura antes de crear el listener y la DB, y sirve para no mezclar los ficheros particulares de cada instancia y los estáticos del software permanecen inmutables. Esto ayuda al actualizar y al 
- Desarrollar una estrategia de recuperación ante desastres que abarque el _control file_, backup de la DB, _redo log_ (_online_ y archivado) etc.

Antes de crear la DB podemos editar parámetros de inicialización del _pfile_ o _spfile_ (recomendado). Los nombres por defecto de los _pfile_ son (UNIX y Linux) `init$ORACLE_SID.ora` en `\$ORACLE_HOME/dbs`. 
Recordar que si estamos usando un _pfile_, para que los campos sean permanentes hay que cambiarlo manualmente en el propio _pfile_, si no, solo se aplicará una vez a esa instancia. En el _pfile_ los parámetros con valores de tipo cadena deben ir entre comillas simples.

El nombre global de una DB está formado por los parámetros `DB_NAME` y `DB_DOMAIN` (opcional, indica ubicación lógica en una estructura de red) y se puede renombrar usando `ALTER DATABASE RENAME GLOBAL_NAME`, aunque para que surta efecto, hay que reiniciar la DB y recrear los _control files_.
El `DB_NAME` no debe superar los 8 caracteres y no puede empezar por un número. Si al arrancar la instancia el parámetro `DB_NAME` no coincide con el nombre que hay en los _control files_, la DB no arranca.

La _**F**ast **R**ecovery **A**rea_ es una ubicación donde Oracle guarda los archivos relacionados con las copias de seguridad y la recuperación y restauración de archivos. Oracle recomienda usar FRA.
Para especificar la FRA usamos los parámetros `DB_RECOVERY_FILE_DEST` (puede ser un directorio, un sistema de archivos o un _disk group_ ASM), y `DB_RECOVERY_FILE_DEST_SIZE` (especifica el tamaño en bytes de la FRA).
En RAC los parámetros anteriores deben ser iguales en todas las instancias, y la FRA debe ser compartida (_disk group_ ASM, directorio compartido con NFS...).
Estos parámetros no se pueden habilitar si tenemos valores en `LOG_ARCHIVE_DEST` y `LOG_ARCHIVE_DUPLEX_DEST` 

Se puede especificar el nombre de los _control files_ (contiene entradas que describen la estructura de la DB) con el parámetro `CONTROL_FILES`. Si no se especifica en el momento de crear la DB, se crea uno por defecto. Oracle recomienda crear al menos dos _control files_ en discos físicos distintos para aumentar la redundancia.

El parámetro `PROCESSES` indica el número máximo de procesos en el OS que se pueden conectar a la DB de forma concurrente.

El parámetro `COMPATIBLE` determina el nivel mínimo de versión con el que la DB debe ser compatible. Esto implica que algunas características de versiones superiores estarán deshabilitadas hasta que el parámetro `COMPATIBLE` suba de versión. Si aumentamos el valor de `COMPATIBLE`, luego no podemos reducirlo fácilmente. El valor por defecto es el numero de _release_ de la última _major release_.
Debe expresarse con 3 valores separados por puntos (por ejemplo, `19.0.0`), si lo cambiamos debemos reiniciar la DB.

## SPFILE
Es un archivo binario que contiene los parámetros de inicialización de la DB. Solo es modificable desde las interfaces que ofrece Oracle y permite que los cambios a nivel de configuración global de la DB permanezcan tras reiniciar la instancia, de forma que no hay que actualizarlos manualmente en el _pfile_.
Si usamos el comando `STARTUP` sin la cláusula `PFILE` Oracle intenta buscar un _spfile_ en la ubicación por defecto. Si no lo encuentra, busca un _pfile_.
Para crear un _spfile_, llevamos un _pfile_ al servidor (en RAC deberemos combinar los _pfile_ de todos los nodos [ver aquí](https://docs.oracle.com/en/database/oracle/oracle-database/19/racad/administering-database-instances-and-cluster-databases.html#RACAD020)) y ejecutamos `CREATE SPFILE=&RUTA1 FROM PFILE=&RUTA2` (no hace falta que la DB esté arrancada).
Se recomienda mantener la ubicación y el nombre por defecto. Si no, deberemos crear una cláusula en un _pfile_ en la ubicación por defecto que nos lleve a dicho _spfile_. Si usamos ASM, Oracle deja el _spfile_ en ASM y utiliza esta técnica.
Si existiese un _spfile_ en la ubicación donde lo creamos, se reescribe sin mensajes de alerta.

Para cambiar los valores de los parámetros de inicialización, usamos `ALTER SYSTEM`. Esto cambia un parámetro en la instancia actual. Para cambiarlos en futuras instancias, al no haber una forma de cambiarlo en disco de forma automática deberíamos cambiarlos en el fichero _pfile_ o usar un _spfile_.
Hay dos tipos de parámetros de inicialización:
- Estáticos: Requieren un reinicio tras el cambio en el _pfile_ o el _spfile_ para que tomen efecto.
- Dinámicos: Los cambios se ejecutan de forma inmediata.
[Aquí hay algunos ejemplos](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-AEC6DDBB-3EF5-4ECF-8735-1A6AA66E78CD).
También tenemos la cláusula `SCOPE` en el comando `ALTER SYSTEM`, que indica el ámbito del efecto. 
Por ejemplo, tenemos `SPFILE`; que no cambia la instancia actual, sino que guarda los cambios para futuras instancias para parámetros tanto estáticos como dinámicos y es permanente. Por otro lado `MEMORY` hace lo opuesto, toma el cambio únicamente para la instancia actual y solo sirve para parámetros dinámicos.
Para combinar ambos, podemos usar la opción `BOTH` que, aunque solo sirve para parámetros dinámicos, permite que estos persistan al modificar el _spfile_. La opción `DEFERRED` es para que parámetros dinámicos se apliquen solo en futuras sesiones.
El comando [`ALTER SYSTEM RESET`](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html#GUID-2004E26A-3C24-4D0C-9EF4-F2854BCD6664) pone el parámetro en su valor por defecto. También podemos añadir la cláusula `SCOPE`, que varía de comportamiento según nos encontremos en una CDB o una PDB.
Para ver el valor actual de un parámetro, usamos la vista `V$SYSTEM_PARAMETER`.

Podemos exportar un _spfile_ siempre que queramos a _pfile_ usando `CREATE PFILE`, siempre y cuando tengamos el rol `SYSDBA`, `SYSOPER` o `SYSBACKUP`. Podemos hacer esto para editar, asegurar una recuperación, arrancar otra DB con propiedades similares en otro servidor...
Se puede hacer esta conversión con los valores que tenga en memoria añadiendo `MEMORY` a la sentencia.
Esto también sirve como estrategia de respaldo, aunque RMAN por defecto hace backup del _spfile_ si hacemos una copia de seguridad. RMAN también permite hacer un backup del _spfile_ específicamente.

Si el _spfile_ se nos daña, podemos recrearlo desde la memoria `CREATE SPFILE FROM MEMORY` o desde un _pfile_ válido con `CREATE SPFILE FROM PFILE`.
Si ninguno de estos métodos nos sirve, podemos crear un nuevo _pfile_ a partir del _alert log_, puesto que lista los parámetros de inicialización al arrancar la instancia.

SI un error ocurre al leer o escribir algún parámetro del servidor en una actualización de parámetros, el error se escribe en el _alert log_, y los siguientes cambios en el _spfile_ serán ignorados.
Para solucionar esta situación apagamos la instancia e intentamos recuperar el _spfile_.

Para ver los parámetros podemos usar `SHOW PARAMETERS`, `V$PARAMETER` o `V$PARAMETER2` para ver los de la sesión actual y `SHOWS PPARAMETERS`, `V$SPPARAMETER` para los del _spfile_, y `V$SYSTEM_PARAMETER` y `V$SYSTEM_PARAMETER2` para los de la instancia actual.

## SERVICIOS DE DB
Un servicio de DB representa una única DB. Puede ser _single-instance_ o RAC. Un servicio global de DB es un servicio proporcionado por múltiples DB sincronizadas mediante replicación de datos.
Los servicios de DB dividen cargas de trabajo de una DB en grupos disjuntos.