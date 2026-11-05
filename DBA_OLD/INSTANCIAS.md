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
3. Al no generar redo, puede montar un hilo de redo (redo thread) deshabilitado o un hilo que no tenga ningún archivo _online redo log_ asociado.

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

---
##  IDENTIFICACIÓN DE UNA INSTANCIA
### OFA — Optimal Flexible Architecture
OFA es el conjunto de estándares y buenas prácticas definido por Oracle para organizar la instalación del software, las bases de datos y los archivos relacionados de forma escalable, mantenible y segura. Define una jerarquía de directorios con responsabilidades bien separadas.

El **Oracle Base** (`ORACLE_BASE`, típicamente `/u01/app/oracle`) es la raíz de toda la instalación y el directorio home del usuario propietario del software. Bajo él cuelga el **Oracle Home** (`ORACLE_HOME`), que contiene los binarios y librerías de una versión concreta del software; cada nueva instalación de un producto ORACLE requiere su propio Oracle Home. 
El **Oracle Base Config** es un directorio compartido por todos los Oracle Homes de un mismo Oracle Base y almacena ficheros dinámicos específicos de cada instancia, como los ficheros de parámetros o los de contraseñas. Finalmente, el directorio **oradata** alberga los datafiles, control files y redo log files de cada base de datos.

Desde Oracle 18c es posible crear un **Oracle Home de solo lectura** (*read-only Oracle Home*). La idea es separar completamente el software de la configuración de las instancias: los ficheros dinámicos residen en Oracle Base Config, no en Oracle Home, por lo que se puede actualizar o parchear el software sin tocar la configuración de las instancias activas. Esto simplifica los parcheos en producción y facilita compartir un mismo Oracle Home entre varias instancias.

### Oracle System Identifier (SID)
El **SID** (*System Identifier*) es el nombre único que identifica una instancia dentro de un host. En Linux y UNIX, Oracle combina el SID con el Oracle Home para construir la clave del segmento de memoria compartida, y lo usa también para localizar los ficheros de parámetros de inicialización (SPFILE/PFILE), que a su vez referencian los control files y demás archivos críticos. Los procesos del sistema operativo también llevan el SID en su nombre, lo que facilita identificarlos.

Las dos variables de entorno fundamentales en Unix/Linux son `ORACLE_SID`, que identifica la instancia, y `ORACLE_HOME`, que apunta al directorio del software:

```bash 
export ORACLE_SID=ORCL
export ORACLE_HOME=/u01/app/oracle/product/19.3.0/dbhome_1
```

> **Nota:** En Windows el SID se registra en el Registro del sistema y el servicio Oracle lleva el nombre `OracleService<SID>`.

---
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

Al **cerrar una base de datos** de forma normal (cualquier modo excepto `ABORT`), ORACLE escribe los datos pendientes de la SGA en los datafiles y _online redo log_ files. Después cierra los datafiles y los _online redo log files_, mientras que los controlfiles permanecen abiertos hasta que la instancia se apaga completamente.

En un cierre mediante `SHUTDOWN ABORT`, ORACLE no realiza este proceso ordenado de escritura de los buffers de la SGA a los datafiles y redo log files. Por ello, al reiniciar la base de datos es necesaria una recuperación de instancia automática (Instance Recovery).

Tras el cierre de la DB, se pasa al **estado `UNMOUNT`**, que disocia la DB de la instancia. En este momento se cierran los controlfiles, mientras la instancia sigue en memoria.

El paso final del cierre consiste en el **apagado de la instancia**. Se libera la memoria SGA y la finalizan los procesos en segundo plano. Si quedan remanentes de una instancia anterior en memoria o algunos procesos no han finalizado correctamente, puede que no sea posible iniciar una nueva instancia. En ese caso, puede ser necesario realizar un `SHUTDOWN ABORT` o limpiar manualmente los recursos y procesos que hayan quedado huérfanos.
Si durante la limpieza de procesos ORACLE detecta errores internos graves, puede decidir abortar la instancia para proteger la integridad de la base de datos. El parámetro `INSTANCE_ABORT_DELAY_TIME` permite retrasar ese ABORT automático durante cierto tiempo para diagnosticar el problema.

Para ver cuanto tiempo lleva nuestra instancia levantada, podemos usar el siguiente comando

```sql
SELECT STATUS, TO_CHAR(STARTUP_TIME, 'DD-MON-YYYY HH24:MI:SS') AS "Inicio"
FROM V$INSTANCE;
````

La vista `V$INSTANCE` expone metadatos de la instancia activa: nombre, host, modo de apertura, versión y tiempo de arranque, entre otros.

---

## CHECKPOINTS
El término tiene dos significados relacionados:
- Puede referirse al **proceso de checkpoint**, durante el cual Oracle actualiza la información de recuperación y fuerza la escritura a disco de bloques modificados de la Database Buffer Cache.
- Puede referirse a la **posición de checkpoint**, una estructura lógica representada por un SCN (_System Change Number_) que indica desde qué punto de los redo logs debe comenzar una recuperación de instancia. Este SCN viene determinado por el bloque modificado (_dirty buffer_) más antiguo presente en la Database Buffer Cache y se almacena en los controlfiles y en las cabeceras de los datafiles.

Sus principales finalidades son:
- Reducir el tiempo necesario para realizar una recuperación de instancia.
- Mantener actualizada la información de recuperación almacenada en los controlfiles y datafiles.
- Garantizar que, durante un cierre consistente de la base de datos, los bloques modificados pendientes se escriban en los datafiles.
### CKPT (_Checkpoint Process_)
Es el proceso que escribe los SCN en las cabeceras de los data files y el control file.

Hay varios tipos de checkpoints.
Los distintos tipos de checkpoint son mecanismos internos del motor de Oracle que controlan la sincronización entre memoria y disco y determinan cuándo se puede avanzar la checkpoint position (SCN de recuperación). Algunos de ellos son:
- **Thread checkpoints**: Garantizan que todos los bloques modificados asociados a un determinado _redo thread_ han sido escritos a disco hasta una posición específica del redo. En una base de datos ORACLE convencional (sin RAC), existe un único redo thread, por lo que en la práctica equivale a un checkpoint de toda la base de datos. En ORACLE RAC, cada instancia posee su propio redo thread y puede realizar checkpoints independientes sobre él. El conjunto de los thread checkpoints de todas las instancias constituye un **database checkpoint**.
- **Checkpoints de tablespaces y datafiles**: Afectan únicamente a un tablespace o datafile concreto en lugar de a toda la base de datos. Su objetivo es garantizar que todos los cambios pendientes asociados a dichos archivos han sido escritos a disco antes de realizar determinadas operaciones administrativas, como poner un tablespace en modo `READ ONLY`, ponerlo en `OFFLINE NORMAL`, iniciar una copia de seguridad o redimensionar un datafile.
- **Checkpoints incrementales**: Son checkpoints parciales y continuos que ORACLE ejecuta durante el funcionamiento normal de la base de datos. Su objetivo es avanzar gradualmente la posición de checkpoint y evitar que se acumulen demasiados bloques modificados en memoria. De este modo se reduce el tiempo necesario para una recuperación de instancia y se evitan grandes picos de E/S durante los cambios de _online redo log_.

>Nota: Recuerdo que los checkpoints no generan SCN de transacción, sino que actualizan la posición de checkpoint, que es un SCN que indica hasta qué punto los cambios ya están persistidos en los datafiles. Los distintos tipos de checkpoint (thread, tablespace, incremental, etc.) son mecanismos o "políticas" mediante los cuales ORACLE decide cuándo avanzar esa posición para reducir el coste de la recuperación.

---
## RECUPERACIÓN DE UNA INSTANCIA
Es el proceso de aplicar los registros del _online redo log_ a los _datafiles_ para reconstruir los cambios realizados después del último checkpoint.
Ocurre de forma automática cuando un administrador abre una DB que se cerró de forma inconsistente.

### PROPÓSITO DE _INSTANCE RECOVERY_ Y CUANDO APLICARLO
Pretende asegurar que la DB esté en un estado consistente.

Un _redo thread_ es un registro de todos los cambios generados en una instancia (en RAC hay un _redo thread_ por instancia). 

Al confirmar (_COMMIT_) una transacción, LGWR (_Log Writer process_) escribe todo el redo que aún no ha sido escrito desde el redo log buffer (es decir, las entradas de redo que permanecen en memoria en ese momento) y lo escribe en el _online redo log_. Además, junto con ese conjunto de cambios, registra también el SCN asociado a la transacción. 
Sin embargo, DBW (_Database Writer process_) escribe los bloques de datos modificados en los datafiles cuando resulta más eficiente hacerlo. Por esta razón, pueden existir temporalmente cambios no confirmados en los datafiles, mientras que cambios ya confirmados pueden no haberse escrito todavía en dichos datafiles.

_Instance recovery_ solo usa _online redo logs_ y _datafiles_ en estado _online_ para asegurar la consistencia.

Cuándo se necesite depende del estado de los _redo threads_. Estos están en estado _open_ en el control file cuando la DB se levanta en modo _read/write_. Se marcan como _closed_ cuando la instancia se apaga de forma consistente. 
Si en el control file el _redo thread_ aparece _open_ pero la instancia está apagada, necesitaremos hacer _instance recovery_.

Se realiza en las siguientes situaciones:
- Se abre la DB tras haber fallado la instancia (o todas las instancias, si hablamos de RAC). Esto se llama **crash recovery**. ORACLE recupera los _redo threads_ de todas las instancias terminadas juntas.
- Solo algunas instancias de una DB en RAC fallan. La _instance recovery_ es realizada por una instancia superviviente.
La _instance recovery_ la hace el proceso _SMON_ de forma automática, aplicando el _online redo log_.

### IMPORTANCIA DE LOS CHECKPOINTS
La recuperación de la instancia usa los checkpoints para saber qué cambios deben aplicarse a los _datafiles_. La posición del checkpoint garantiza que **cualquier cambio confirmado con un SCN menor al SCN del checkpoint ha sido escrito en los _datafiles_**.
En la _instance recovery_, se deben aplicar los cambios entre el SCN del checkpoint y el final del _redo thread_. Aunque haya cambios confirmados en ese intervalo, los únicos cambios que es seguro que están en disco son los cambios que tienen un SCN menor al del SCN del checkpoint.

### FASES DE LA _INSTANCE RECOVERY_
1) **_cache recovery / rolling forward_**: Vuelve a aplicar todos los cambios de los _online redo logs_ a los _datafiles_. Como los _online redo logs_ contienen datos de _undo_, también se generan los correspondientes segmentos de _undo_. Esto permite dejar la DB en el mismo estado que tenía justo antes del fallo, incluyendo transacciones confirmadas y no confirmadas. 
2) **rollback**: En la fase de _rollback_, Oracle utiliza el _undo_ reconstruido para deshacer únicamente las transacciones que no habían hecho _COMMIT_, garantizando la consistencia de la base de datos.
Tras un fallo, ORACLE marca todas las transacciones activas como terminadas y comienza a deshacerlas mediante el proceso _SMON_. Sin embargo, para no retrasar el acceso a la DB, si una nueva consulta necesita leer un bloque afectado por una de esas transacciones, ORACLE deshace ese bloque en ese mismo momento utilizando la información de _UNDO_, sin esperar a que _SMON_ complete el rollback general. Esto permite que la DB esté disponible mucho antes.

## FICHEROS DE PARAMETRIZACIÓN
Para levantar una instancia, ORACLE debe leer un **SPFILE (_server parameter file_)** o un **PFILE (_text initialization parameter file_)** - legacy -. 
Para crear una DB manualmente, debemos levantar una instancia con fichero de parametrización y luego usar la sentencia `CREATE DATABASE` (la instancia y el fichero de parámetros pueden existir antes incluso que la DB).

Los parámetros de inicialización afectan al arranque de la propia instancia y muchos no los tendremos que tocar porque son adecuados por defecto.
- **SPFILE**: Solo ORACLE lee y escribe en estos archivos. Además, es único para la base de datos y debe ser accesible por la(s) instancia(s) que la ejecutan. Es un binario que **NO** se puede editar con un editor de código. Si alteramos según qué parámetros, el cambio no se verá reflejado hasta que no reiniciemos la instancia. Se puede construir uno con la sentencia `CREATE SPFILE` a partir de un PFILE o con el _Database Configuration Assistant_.
- **PFILE**: Es un fichero de texto que contiene los parámetros. Es una característica legacy. Pueden existir múltiples copias de los PFILE (una por cada equipo desde el que se inicia la DB), lo que dificulta mantener una configuración consistente. El SPFILE elimina este problema al centralizar la configuración. Para aplicar cambios permanentes en las instancias deberemos escribirlos manualmente en el PFILE.

### MODIFICACIÓN DE LOS PARÁMETROS
Existen dos tipos: dinámicos y estáticos.
- **Dinámicos**: Se categorizan en parámetros a nivel de sesión (afectan a la sesión actual. Ejemplo: `NLS_DATE_FORMAT`) o a nivel de sistema (afectan a la DB y a todas sus sesiones. Ejemplo: `MEMORY_TARGET`).
- **Estáticos**: Cambiarlos requiere modificar en el fichero de parámetros y necesitan de reinicio de la instancia para aplicarse. ("No modificables" en: [REFERENCIA DE PARÁMETROS OFICIAL](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/changing-parameter-values-in-a-parameter-file.html#REFRN00102)). Se pueden cambiar solo para la instancia o la DB. (Ejemplos: `DB_BLOCK_SIZE`, `DB_NAME`, `COMPATIBLE`...).

Si usamos SPFILE, el _scope_ define el alcance del cambio del parámetro dinámico a nivel de sistema. 
- **`SCOPE=MEMORY`**: El cambio se aplica inmediatamente en la instancia que esté en ejecución pero no en el SPFILE. Si reiniciamos la instancia, el cambio se pierde.
- **`SCOPE=SPFILE`**: El cambio se guarda en el SPFILE, pero hasta que la instancia no se reinicie no se aplica.
- **`SCOPE=BOTH`**: Se aplica inmediatamente y se guarda en el SPFILE.
Estos cambios se escriben en el alert log y se validan antes de aplicarse.

## FICHEROS DE DIAGNÓSTICO
### ADR (Automatic Diagnostic Repository)
Consiste en un repositorio donde se almacenan los directorios de traza y diagnóstico. 
Destacan como características principales la estructura unificada de directorios, la consistencia en el formato y el set de herramientas que tiene. 
Se localiza fuera de la DB y la instancia puede escribir el él antes incluso de que la DB esté creada.
ADR detecta y registra automáticamente errores críticos (por ejemplo, `ORA-600`). Cada tipo de error se identifica mediante una clave (una cadena que describe el problema), y cada vez que ese error ocurre se crea un incidente con un identificador único. Cuando se genera un incidente, ORACLE envía una alerta al OEM y recopila información de diagnóstico. Si el mismo problema ocurre repetidamente en poco tiempo, ADR limita la generación de volcados de diagnóstico para evitar sobrecargar el sistema, aunque sigue registrando el error en el _alert log_.

La estructura de directorios es `$ORACLE_BASE/diag/rdbms/<db_name>/<instance_name>/*`. Luego ahí tenemos carpetas como `cdump`, `alert`o `incident`.

### ALERT LOG
Es un XML con la información cronológica de los mensajes y los errores de la DB (errores internos ORA-0600, de corrupción ORA-1578, _deadlocks_ ORA-0060, mensajes de operaciones administrativas). Se encuentra en el subdirectorio `alert`del ADR. El subdirectorio `trace`contiene uno similar pero en texto plano.
También existe el **DDL log**, que es similar pero muestra mensajes detallados relativos a operaciones y sentencias de DDL. Se guarda bajo `log/ddl`en el ADR

### FICHEROS DE TRAZA
Se utilizan para investigar problemas y aplicar tuning y configuraciones adecuadas de una instancia. Son generados por los procesos de ORACLE y contienen información sobre su estado, actividad, errores... En ADR se almacenan en el subdirectorio `.../trace` y suelen tener extensión `*.trc`. Su nombre identifica el SID, el proceso que los genera y el número del proceso en OS. Algunos trace files tienen un archivo asociado con extensión `*.trm`, que almacena metadatos utilizados por ORACLE para facilitar la búsqueda y navegación dentro de los archivos de trazas.
Si estos ficheros son muy grandes se van separando (hasta en 5), y si son demasiado grandes se borra el más antiguo.

### DUMPS DE DIAGNÓSTICO
Se generan si ocurre un incidente en el directorio `incident`del ADR. Contienen el identificador del incidente en el nombre.