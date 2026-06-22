# INSTANCIAS DE BASES DE DATOS EN ORACLE

> **Referencia oficial:** [Oracle Database Concepts 19c — Oracle Database Instance](https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/oracle-database-instance.html)

---
## DEFINICIÓN, ESTRUCTURA Y TIPOS DE INSTANCIAS
Una instancia de base de datos es un conjunto de estructuras de memoria que gestionan archivos de base de datos.

Cuando se levanta una instancia, ORACLE asigna memoria llamada SGA (ver [[ARQUITECTURA DE MEMORIA#SGA]]) y arranca uno o varios procesos en segundo plano.

Oracle Database puede ejecutarse en dos configuraciones fundamentales. En **_Single Instance_**, la relación entre instancia y base de datos es estrictamente uno a uno: una instancia gestiona una sola base de datos y viceversa. Esta es la configuración más habitual en entornos de desarrollo o bases de datos de tamaño pequeño y mediano, y su administración es relativamente sencilla.

En cambio, **_RAC (Real Application Clusters)_** permite que varias instancias, ejecutándose en distintos nodos de un clúster, accedan simultáneamente a una misma base de datos. Esto proporciona alta disponibilidad —si un nodo cae, los demás continúan operando— y escalabilidad horizontal, añadiendo más nodos en lugar de potenciar un único servidor. A cambio, requiere Oracle Clusterware, ASM y la opción RAC de Enterprise Edition, lo que eleva considerablemente la complejidad administrativa y el coste de licenciamiento.

En ambos casos se cumple la misma regla fundamental: una instancia siempre se asocia a una sola base de datos. No existe ninguna configuración en la que una única instancia monte simultáneamente dos bases de datos distintas.

Las instancias pueden ser **_read/write_** o **_read-only_**. Por defecto usamos las _read/write_, las cuales pueden procesar DML (Data Manipulation Language) y aceptan conexiones de clientes. Por otro lado, las _read-only_ no, y solo procesan consultas.
En anteriores _releases_ las bases de datos eran siempre _read/write_ a no ser que estuviésemos accediendo a una _standby_. A partir de ORACLE Database 12c Release 2 (12.2), instancias en modo _read-only_ y _read/write_ pueden coexistir para una sola base de datos.
Las instancias _read-only_ tienen las siguientes características:
1. Solo pueden abrir DB que ya han sido abiertas por otra instancia _read/write_. Esto es porque la instancia _read-only_ no realiza tareas de escritura y recuperación
2. Deshabilita procesos en segundo plano como el CKPT (Checkpoint) o el ARCn (Archiver)
3. Al no generar redo, puede montar un hilo de redo (redo thread) deshabilitado o un hilo que no tenga ningún archivo de redo log en línea asociado.

El modo de operación se controla mediante el parámetro de inicialización `INSTANCE_MODE`. Puede configurarse de dos formas: desde SQL\*Plus de manera persistente en el SPFILE, o editando directamente el PFILE.

```bash
# En el PFILE (init<SID>.ora) 
instance_mode = READ_ONLY
```
```sql
-- SQL*Plus
ALTER SYSTEM SET instance_mode = 'READ_ONLY' SCOPE = SPFILE;
```

> **PFILE** (`initSID.ora`) y **SPFILE** (`spfileSID.ora`) sirven para almacenar los parámetros de configuración de una instancia ORACLE. La diferencia principal es que el PFILE es un archivo de texto editable manualmente, mientras que el SPFILE es binario y permite que Oracle guarde automáticamente cambios realizados con `ALTER SYSTEM`. En entornos modernos y de producción se utiliza normalmente SPFILE, mientras que PFILE se emplea sobre todo para tareas de recuperación, diagnóstico o para crear y revisar configuraciones de forma manual.

## CICLO DE VIDA DE UNA INSTANCIA

Una instancia no pasa directamente de apagada a completamente operativa: transita por tres fases intermedias que conviene conocer, especialmente para tareas de mantenimiento.
Para levantar o cerrar una instancia podemos hacerlo manualmente con el comando `STARTUP`, con el _Oracle Enterprise Manager_ (OEM) o con la utilidad `SRVCTL`.

Las opciones de arranque y cierre están restringidas a usuarios con privilegios de administrador (o bien el usuario de OS tiene permisos para conectar como administrador o bien se han otorgado los privilegios administrativos en la misma DB al usuario). 
Los privilegios que permiten acceder a una DB en un estado que no sea `OPEN` son: `SYSDBA`, `SYSOPER`, `SYSBACKUP`, `SYSDG` y `SYSKM`. 
Si nos conectamos como `SYSDBA` estaremos en el esquema `SYS`. Si nos conectamos como `SYSOPER` estaremos en el esquema `PUBLIC`, con unos permisos que serán un subconjunto de los permisos de `SYSDBA`

### MODOS DE INICIO

En la fase _**NOMOUNT**_ (`STARTUP NOMOUNT`), Oracle lee el SPFILE o PFILE y asigna la SGA, pero todavía no accede a ningún archivo de la base de datos.
También abre el fichero `alert.log` y los ficheros de trazas y escribe los parámetros configurados explícitamente en un formato válido.
Es el estado mínimo necesario para, por ejemplo, crear una base de datos desde cero o restaurar un control file. 

En la fase _**MOUNT**_ (`ALTER DATABASE MOUNT`), ORACLE lee los control files y conoce la ubicación de los datafiles y redo logs, pero aún no los abre. Se usa para operaciones como cambiar el modo de archivado, renombrar datafiles o aplicar recuperación incompleta.
Si varias instancias intentan acceder a la misma base de datos, el parámetro `CLUSTER_DATABASE` determina si la base de datos puede funcionar en modo RAC. Cuando `CLUSTER_DATABASE=FALSE` (configuración de una instancia única), solo una instancia puede montar y abrir la base de datos. Cuando `CLUSTER_DATABASE=TRUE`, varias instancias pueden montar y abrir simultáneamente la misma base de datos, compartiendo los mismos archivos de datos. El número máximo de instancias concurrentes viene determinado principalmente por el parámetro `MAXINSTANCES`, que se establece al crear la base de datos y se almacena en el control file.

Finalmente, en la fase _**OPEN**_ (`ALTER DATABASE OPEN`), todos los archivos quedan accesibles (se abren datafiles, redo logs...) y la base de datos está lista para aceptar conexiones de usuario. Por defecto la DB se abre en modo _read/write_.
También se coge espacio de UNDO. Si el parámetro `UNDO_TABLESPACE` no está definido y existen varios tablespaces de UNDO en el momento de abrir la DB, se escoge el primero disponible.
Si al abrir la DB algún fichero de datos no está, o no se superan los tests de consistencia, devolverá un error.

El comando `STARTUP` a secas ejecuta estas tres fases en orden automáticamente.

### MODOS DE CIERRE

Al cerrar una instancia existen cuatro modos, de más seguro a más abrupto.
- `SHUTDOWN NORMAL` espera a que todos los usuarios se desconecten voluntariamente antes de cerrar; en la práctica rara vez se usa porque puede esperar indefinidamente.
- `SHUTDOWN TRANSACTIONAL` no admite nuevas conexiones pero espera a que las transacciones en curso finalicen. 
- `SHUTDOWN IMMEDIATE`, el más habitual en administración, hace rollback de las transacciones activas y cierra ordenadamente sin esperar usuarios.
- `SHUTDOWN ABORT` es equivalente a un corte de corriente: no realiza checkpoint ni rollback, y la siguiente apertura requerirá una recuperación de instancia automática (*instance recovery*) que aplique el redo log pendiente.

 ```sql
SHUTDOWN IMMEDIATE -- el más usado en administración habitual
SHUTDOWN ABORT -- último recurso
 ```

> **⚠️ Atención:** `SHUTDOWN ABORT` no corrompe la base de datos, pero el arranque posterior será más lento por la recuperación automática.

Para ver cuanto tiempo lleva nuestra instancia levantada, podemos usar el siguiente comando

```sql
SELECT STATUS, TO_CHAR(STARTUP_TIME, 'DD-MON-YYYY HH24:MI:SS') AS "Inicio"
FROM V$INSTANCE;
````

La vista `V$INSTANCE` expone metadatos de la instancia activa: nombre, host, modo de apertura, versión y tiempo de arranque, entre otros.

Al cerrar una instancia de forma normal (cualquier modo excepto `ABORT`) se escriben datos de la SGA a los datafiles y al redo log activo y luego los cierra. Los control files siguen abiertos tras el cierre de la DB.

---

##  IDENTIFICACIÓN DE UNA INSTANCIA

### Oracle System Identifier (SID)

El **SID** (*System Identifier*) es el nombre único que identifica una instancia dentro de un host. En Linux y UNIX, Oracle combina el SID con el Oracle Home para construir la clave del segmento de memoria compartida, y lo usa también para localizar los ficheros de parámetros de inicialización (SPFILE/PFILE), que a su vez referencian los control files y demás archivos críticos. Los procesos del sistema operativo también llevan el SID en su nombre, lo que facilita identificarlos.

Las dos variables de entorno fundamentales en Unix/Linux son `ORACLE_SID`, que identifica la instancia, y `ORACLE_HOME`, que apunta al directorio del software:

```bash 
export ORACLE_SID=ORCL
export ORACLE_HOME=/u01/app/oracle/product/19.3.0/dbhome_1
```

> **Nota:** En Windows el SID se registra en el Registro del sistema y el servicio Oracle lleva el nombre `OracleService<SID>`.

### OFA — Optimal Flexible Architecture

OFA es el conjunto de estándares y buenas prácticas definido por Oracle para organizar la instalación del software, las bases de datos y los archivos relacionados de forma escalable, mantenible y segura. Define una jerarquía de directorios con responsabilidades bien separadas.

El **Oracle Base** (`ORACLE_BASE`, típicamente `/u01/app/oracle`) es la raíz de toda la instalación y el directorio home del usuario propietario del software. Bajo él cuelga el **Oracle Home** (`ORACLE_HOME`), que contiene los binarios y librerías de una versión concreta del software; cada nueva instalación de un producto ORACLE requiere su propio Oracle Home. 
El **Oracle Base Config** es un directorio compartido por todos los Oracle Homes de un mismo Oracle Base y almacena ficheros dinámicos específicos de cada instancia, como los ficheros de parámetros o los de contraseñas. Finalmente, el directorio **oradata** alberga los datafiles, control files y redo log files de cada base de datos.

Desde Oracle 18c es posible crear un **Oracle Home de solo lectura** (*read-only Oracle Home*). La idea es separar completamente el software de la configuración de las instancias: los ficheros dinámicos residen en Oracle Base Config, no en Oracle Home, por lo que se puede actualizar o parchear el software sin tocar la configuración de las instancias activas. Esto simplifica los parcheos en producción y facilita compartir un mismo Oracle Home entre varias instancias.


