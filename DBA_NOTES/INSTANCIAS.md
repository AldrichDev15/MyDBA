# INTRODUCCIÓN A LAS INSTANCIAS DE BASES DE DATOS
Una instancia de base de datos es un conjunto de estructuras de memoria que gestionan archivos de base de datos.

Cuando se levanta una instancia, ORACLE asigna memoria llamada SGA (ver [[ARQUITECTURA DE MEMORIA#SGA]]) y arranca uno o varios procesos en segundo plano.

ORACLE Database se puede ejecutar con configuración _singe instance_ o RAC (_Real Application Cluster_).
En _single instance_ la relación entre instancia y base de datos es uno a uno, mientras que en RAC podemos tener varias instancias para una base de datos. 
Una instancia siempre se asocia a una sola base de datos, ya que no existe una configuración donde una única instancia monte simultáneamente dos databases distintas. 
Varias instancias pueden ejecutarse en un mismo ordenador.

Las instancias pueden ser _read/write_ o _read-only_. Por defecto usamos las _read/write_, las cuales pueden procesar DML (Data Manipulation Language) y aceptan conexiones de clientes. Por otro lado, las _read-only_ no, y solo procesan consultas.
En anteriores _releases_ las bases de datos eran siempre _read/write_ a no ser que estuviésemos accediendo a una _standby_. A partir de ORACLE Database 12c Release 2 (12.2), instancias en modo _read-only_ y _read/write_ pueden coexistir para una sola base de datos.
Las instancias read-only tienen las siguientes características:
1. Solo pueden abrir DB que ya han sido abiertas por otra instancia _read/write_. Esto es porque la instancia _read-only_ no realiza tareas de escritura y recuperación
2. Deshabilita procesos en segundo plano como el CKPT (Checkpoint) o el ARCn (Archiver)
3. Al no generar redo, puede montar un hilo de redo (redo thread) deshabilitado o un hilo que no tenga ningún archivo de redo log en línea asociado.
Para cambiar el modo de la DB, debemos cambiar el parámetro de inicialización `INSTANCE_MODE` a `READ_ONLY`. Lo podemos hacer desde sqlplus con
`ALTER SYSTEM SET instance_mode='READ-ONLY' SCOPE=SPFILE;`
o en el PFILE o SPFILE.
# REFERENCIAS
[Administración de instancias y clusters RAC ](https://docs.oracle.com/en/database/oracle/oracle-database/19/racad/administering-database-instances-and-cluster-databases.html#RACAD900)
