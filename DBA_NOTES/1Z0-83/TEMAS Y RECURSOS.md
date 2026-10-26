# GUÍA COMPLETA PARA ESTUDIAR 1Z0-083

## BIBLIOTECA TÉCNICA
Estos manuales oficiales son la referencia principal para múltiples temas:
- [Documentación oficial Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/19/books.html).
- [Manual de administración de DB multitenant](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/).
- [Manual para RMAN, backups y restauraciones](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/)
- [Manual para upgrades de versión y actualizaciones](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/)
- [Instalaciones y software (Grid, DBCA, Oracle Restart...)](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/)
- [Manual para tuning y rendimiento](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/)
- [Tuning para SQL](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgsql/)
- [Nuevas características de 19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/newft/)

# [PARTE I. ARQUITECTURA MULTITENANT](obsidian://open?vault=DBA&file=DBA_NOTES%2F1Z0-83%2FPARTE%20I.%20ARQUITECTURA%20MULTITENANT)

## 1. CREACIÓN DE CBDs Y PDBs

| Exam Topic                         | Recursos gratuitos                                                                                                                                                                                                                                                                                   |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Configure and create a CDB         | [Database Administrator's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/) · [Vídeo: crear CDB con DBCA](https://www.youtube.com/results?search_query=Oracle+19c+create+CDB+DBCA+tutorial)                                                                               |
| Create a new PDB from the CDB seed | [Creating PDBs](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/creating-pdbs.html) · [Vídeo: crear PDB desde PDB$SEED](https://www.youtube.com/results?search_query=Oracle+19c+create+PDB+from+PDB%24SEED)                                                                      |
| Explore the structure of PDBs      | [Introduction to Multitenant Architecture](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/introduction-to-the-multitenant-architecture.html) · [Vídeo: CDB, PDB, root y seed](https://www.youtube.com/results?search_query=Oracle+19c+CDB+PDB+root+seed+architecture+explained) |

Contenido que debes dominar: estructura de una CDB, `CDB$ROOT`, `PDB$SEED`, creación de PDB, archivos de datos y vistas para identificar contenedores.

## 2. Manage CDBs and PDBs

|Exam Topic|Recursos gratuitos|
|---|---|
|Manage PDB service names and connections|[Administering PDBs](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/administering-pdbs.html) · [Oracle Net Services](https://docs.oracle.com/en/database/oracle/oracle-database/19/netag/) · [Vídeo: PDB services y listener](https://www.youtube.com/results?search_query=Oracle+19c+PDB+services+listener+connections)|
|Manage startup, shutdown and availability of CDBs and PDBs|[Administering a CDB](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/administering-a-cdb.html) · [Vídeo: startup, shutdown, open, close y SAVE STATE](https://www.youtube.com/results?search_query=Oracle+19c+PDB+startup+shutdown+SAVE+STATE)|
|Change the different modes and settings of PDBs|[Administering PDBs](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/administering-pdbs.html) · [Vídeo: PDB open modes](https://www.youtube.com/results?search_query=Oracle+19c+PDB+read+write+read+only+restricted+mode)|
|Evaluate the impact of parameter value changes|[Database Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/) · [Vídeo: parámetros de CDB/PDB](https://www.youtube.com/results?search_query=Oracle+19c+PDB+initialization+parameters+ALTER+SYSTEM)|
|Performance management in CDBs and PDBs|[Monitoring CDBs and PDBs](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/monitoring-cdbs-and-pdbs.html) · [Performance Tuning Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/) · [Vídeo: monitorización CDB/PDB](https://www.youtube.com/results?search_query=Oracle+19c+CDB+PDB+performance+monitoring)|
|Control CDB and PDB resource usage with Oracle Resource Manager|[Database Administrator's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/) · [Vídeo: Resource Manager para PDB](https://www.youtube.com/results?search_query=Oracle+19c+Resource+Manager+CDB+PDB+tutorial)|

Contenido que debes dominar: servicios, conexiones, estados de apertura, persistencia de estados, parámetros y su ámbito, monitorización, planes y directivas de Resource Manager.

## 3. Backup and Duplicate

|Exam Topic|Recursos gratuitos|
|---|---|
|Perform Backup and Recover CDBs and PDBs|[Backup and Recovery User's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/) · [Vídeo: RMAN backup y recovery de CDB/PDB](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+backup+recover+CDB+PDB)|
|Duplicate an active PDB|[Creating PDBs](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/creating-pdbs.html) · [Vídeo: clonar PDB activa](https://www.youtube.com/results?search_query=Oracle+19c+clone+active+PDB+database+link)|
|Duplicate a Database|[Duplicating Databases with RMAN](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-duplicating-databases.html) · [Vídeo: RMAN DUPLICATE](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+DUPLICATE+DATABASE+tutorial)|

Contenido que debes dominar: backup, restore y recover; requisitos de clonación de PDB; duplicación basada en backup y desde una base de datos activa; conexiones y almacenamiento de origen y destino.

## 4. Upgrading and Transporting CDBs and Regular PDBs

|Exam Topic|Recursos gratuitos|
|---|---|
|Upgrade an Oracle Database|[Database Upgrade Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/) · [Vídeo: AutoUpgrade y DBUA](https://www.youtube.com/results?search_query=Oracle+19c+AutoUpgrade+DBUA+database+upgrade+tutorial)|
|Transport Data|[Database Utilities](https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/) · [Transporting Data](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/transporting-data.html) · [Vídeo: Data Pump y transportable tablespaces](https://www.youtube.com/results?search_query=Oracle+19c+Data+Pump+transportable+tablespaces+tutorial)|

Contenido que debes dominar: preparación y comprobaciones de upgrade, AutoUpgrade, tareas posteriores, Data Pump, transporte de tablespaces y movimiento de PDB entre CDB compatibles.

## 5. Manage Application PDBs

|Exam Topic|Recursos gratuitos|
|---|---|
|Explain the purpose of application root and application seed|[Multitenant Architecture](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/introduction-to-the-multitenant-architecture.html) · [Vídeo: application root y seed](https://www.youtube.com/results?search_query=Oracle+19c+application+root+application+seed+explained)|
|Define and create application PDBs|[Creating Application Containers and Seeds](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/creating-removing-application-containers-seeds-with-sql-plus.html) · [Vídeo: crear application PDB](https://www.youtube.com/results?search_query=Oracle+19c+create+application+container+application+PDB)|
|Install, upgrade and patch applications|[Administering Application Containers](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/administering-application-containers-with-sql-plus.html) · [Vídeo: application maintenance](https://www.youtube.com/results?search_query=Oracle+19c+application+container+install+upgrade+patch)|
|Create and administer Application PDBs|[Administering Application Containers](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/administering-application-containers-with-sql-plus.html) · [Vídeo: administrar application PDB](https://www.youtube.com/results?search_query=Oracle+19c+administer+application+PDB)|
|Clone PDBs and Application containers|[Creating PDBs](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/creating-pdbs.html) · [Vídeo: clonar application container](https://www.youtube.com/results?search_query=Oracle+19c+clone+application+container+PDB)|
|Plug and unplug operations with PDBs and application containers|[Creating PDBs](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/creating-pdbs.html) · [Vídeo: unplug y plug de PDB](https://www.youtube.com/results?search_query=Oracle+19c+PDB+unplug+plug+tutorial)|
|Comparing Local Undo Mode and Shared Undo Mode|[Multitenant Administrator's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/) · [Vídeo: Local Undo frente a Shared Undo](https://www.youtube.com/results?search_query=Oracle+19c+local+undo+shared+undo+mode)|

Contenido que debes dominar: application root, application seed, sincronización de aplicaciones, mantenimiento y parcheado de aplicaciones, clonación, unplug/plug y los dos modos de undo.

## 6. Manage Security in Multitenant Databases

|Exam Topic|Recursos gratuitos|
|---|---|
|Manage Security in Multitenant databases|[Managing Security for a Multitenant Environment](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/managing-security-for-a-multitenant-environment.html) · [Vídeo: common users y local users](https://www.youtube.com/results?search_query=Oracle+19c+common+users+local+users+privileges)|
|Manage PDB lockdown profiles|[Multitenant Security](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/managing-security-for-a-multitenant-environment.html) · [Vídeo: PDB lockdown profiles](https://www.youtube.com/results?search_query=Oracle+19c+PDB+lockdown+profiles+tutorial)|
|Audit Users in CDBs and PDBs|[Configuring Audit Policies](https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/configuring-audit-policies.html) · [Vídeo: Unified Auditing](https://www.youtube.com/results?search_query=Oracle+19c+Unified+Auditing+CDB+PDB)|
|Manage other types of policies in application containers|[Database Security Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/) · [Vídeo: políticas de seguridad en application containers](https://www.youtube.com/results?search_query=Oracle+19c+application+container+security+policies)|

Contenido que debes dominar: usuarios y privilegios comunes/locales, lockdown profiles, Unified Auditing y el ámbito de las políticas de seguridad en los contenedores.

## 7. Recovery and Flashback

|Exam Topic|Recursos gratuitos|
|---|---|
|Restore and Recovering Databases with RMAN|[Backup and Recovery User's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/) · [Vídeo: RMAN restore y recover](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+restore+recover+database+step+by+step)|
|Perform CDB and PDB flashback|[Backup and Recovery User's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/) · [Multitenant Administrator's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/) · [Vídeo: Flashback CDB/PDB](https://www.youtube.com/results?search_query=Oracle+19c+Flashback+Database+PDB+tutorial)|

Contenido que debes dominar: recuperación de CDB/PDB, diferencia entre restore y recover, Flashback Database, Flashback PDB y requisitos de restore points.

Los manuales de estos siete primeros bloques son los recursos oficiales de referencia para arquitectura multitenant, backup/recovery, upgrades y seguridad.

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

+2

# Parte II. Backup and Recovery Workshop

## 8. Backup Strategies and Terminology

|Exam Topic|Recursos gratuitos|
|---|---|
|Perform Full and Incremental Backups and Recoveries|[RMAN Backup Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-backup-concepts.html) · [Vídeo: backups full e incrementales](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+full+incremental+backup+recovery)|
|Compress and Encrypt RMAN Backups|[RMAN Backup Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-backup-concepts.html) · [Vídeo: compresión y cifrado RMAN](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+backup+compression+encryption+tutorial)|
|Use a media manager|[Configuring RMAN for Use with a Media Manager](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/configuring-rman-client-basic.html) · [Vídeo: RMAN SBT tape media manager](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+SBT+media+manager+configuration)|
|Create multi-section backups of very large files|[RMAN Backup Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-backup-concepts.html) · [Vídeo: RMAN multisection backup](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+multisection+backup+large+datafile)|
|Create duplexed backup sets|[RMAN Backup Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-backup-concepts.html) · [Vídeo: RMAN duplexed backup sets](https://www.youtube.com/results?search_query=Oracle+RMAN+duplexed+backup+sets+CONFIGURE+COPIES)|
|Create archival backups|[RMAN Backup Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-backup-concepts.html) · [Vídeo: RMAN archival backup](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+archival+backup+KEEP+FOREVER)|
|Backup of recovery files|[Managing the Fast Recovery Area](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/managing-the-fast-recovery-area.html) · [Vídeo: backup de FRA y recovery files](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+backup+recovery+area+FRA)|
|Backup non database files|[Backing Up Non-Database Files](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/backing-up-non-database-files.html) · [Vídeo: RMAN backup non-database files](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+backup+non-database+files)|
|Back up ASM meta data|[Backing Up and Recovering ASM Metadata](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/backing-up-and-recovering-asm-metadata.html) · [Vídeo: backup ASM metadata](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+backup+ASM+metadata+md_backup)|

Qué debes estudiar: nivel 0 y nivel 1 de backups incrementales, image copies frente a backup sets, `CONFIGURE COMPRESSION`, cifrado, canales SBT, `SECTION SIZE`, duplexación con `CONFIGURE BACKUP COPIES`, `KEEP`, Fast Recovery Area, archivos externos y metadatos ASM.

Práctica recomendada: crea un backup completo y uno incremental, comprueba sus piezas y practica la recuperación. Después configura compresión y cifrado en una base de datos de laboratorio. Para los ejercicios de media manager, ASM y backups de archivo, estudia también las dependencias de software y hardware que no están disponibles en todos los entornos.

## 9. Restore and Recovery Concepts

|Exam Topic|Recursos gratuitos|
|---|---|
|Employ the best Oracle Database recovery technology for your failure situation|[Overview of Backup and Recovery](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/overview-of-backup-and-recovery.html) · [Vídeo: elegir técnica de recuperación](https://www.youtube.com/results?search_query=Oracle+19c+DBA+recovery+scenarios+RMAN+recovery+strategy)|
|Describe and use recovery technology for Crash, Complete, and Point-in-time recovery|[Performing Complete Database Recovery](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/performing-complete-database-recovery.html) · [Point-in-Time Recovery](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-performing-flashback-dbpitr.html) · [Vídeo: crash recovery y PITR](https://www.youtube.com/results?search_query=Oracle+19c+instance+crash+recovery+complete+recovery+point+in+time+recovery)|

Qué debes estudiar:

- Diferencias entre recuperación de instancia tras un crash, recuperación completa y recuperación incompleta.
    
- Papel de redo, archived redo logs y checkpoints.
    
- Cuándo elegir RMAN, Flashback o recuperación manual.
    
- Recuperación a un punto en el tiempo (PITR), recuperación de tablespaces y recuperación de archivos dañados.
    
- Cómo decidir una técnica según el tipo de fallo y los backups disponibles.
    

La guía oficial de Backup and Recovery presenta los escenarios de recuperación y los procedimientos específicos para cada tipo.

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

## 10. Using Flashback Technologies

|Exam Topic|Recursos gratuitos|
|---|---|
|Configure your Database to support Flashback|[Configuring Flashback Database](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/configuring-flashback-database.html) · [Vídeo: configurar Flashback Database](https://www.youtube.com/results?search_query=Oracle+19c+configure+Flashback+Database+DB_FLASHBACK_RETENTION_TARGET)|
|Perform flashback operations|[Performing Flashback and Database Point-in-Time Recovery](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/performing-flashback-dbpitr.html) · [Vídeo: Flashback Database, Table y Drop](https://www.youtube.com/results?search_query=Oracle+19c+Flashback+Database+Flashback+Table+Flashback+Drop+tutorial)|

Qué debes estudiar: Fast Recovery Area, modo `ARCHIVELOG`, flashback logging, restore points normales y garantizados, `FLASHBACK DATABASE`, `FLASHBACK TABLE` y las diferencias entre Flashback y recuperación tradicional.

## 11. Duplicating a Database

|Exam Topic|Recursos gratuitos|
|---|---|
|Duplicate Databases|[Duplicating Databases](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-duplicating-databases.html) · [Vídeo: RMAN DUPLICATE desde backup](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+duplicate+database+backup+based+duplication) · [Vídeo: active duplication](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+active+database+duplication+tutorial)|

Qué debes estudiar: duplicación activa frente a duplicación basada en backup, base de datos auxiliar, conexiones RMAN, `SPFILE`, `NOFILENAMECHECK`, ubicación de archivos y requisitos para crear una copia independiente.

## 12. Configuring and Using RMAN

|Exam Topic|Recursos gratuitos|
|---|---|
|Configure RMAN and the Database for Recoverability|[Starting and Configuring RMAN](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/starting-and-configuring-rman.html) · [Vídeo: configurar RMAN](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+configure+retention+policy+controlfile+autobackup)|
|Configure and use an RMAN recovery catalog|[Managing a Recovery Catalog](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/managing-recovery-catalog.html) · [Vídeo: recovery catalog](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+recovery+catalog+create+register+database)|

Qué debes estudiar: configuración persistente de RMAN, políticas de retención, dispositivos y canales, control file autobackup, destino de backups, creación y registro de bases de datos en un recovery catalog y sincronización del catálogo.

## 13. Diagnosing Failures

|Exam Topic|Recursos gratuitos|
|---|---|
|Detect and repair database and database block corruption|[Validating Database Files and Backups](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/validating-database-files-and-backups.html) · [Vídeo: DBVERIFY, RMAN VALIDATE y block recovery](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+VALIDATE+DBVERIFY+block+corruption+recovery)|
|Diagnosing Database Issues|[Oracle Database Utilities — ADRCI](https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/) · [Database Administrator's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/) · [Vídeo: ADRCI y alert log](https://www.youtube.com/results?search_query=Oracle+19c+ADRCI+alert+log+diagnose+database+issues)|

Qué debes estudiar: corrupción lógica y física, `RMAN VALIDATE`, `VALIDATE DATABASE`, `DBVERIFY`, block media recovery, alert log, Automatic Diagnostic Repository (ADR), ADRCI y lectura de mensajes de error.

## 14. Performing Recovery

|Exam Topic|Recursos gratuitos|
|---|---|
|Restore and Recovering Databases with RMAN|[Performing Recovery](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/performing-complete-database-recovery.html) · [Vídeo: recuperación completa con RMAN](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+restore+recover+database+complete+recovery)|
|Perform Non RMAN database recovery|[User-Managed Backup and Recovery](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/user-managed-recovery.html) · [Vídeo: recuperación manual sin RMAN](https://www.youtube.com/results?search_query=Oracle+19c+user+managed+database+recovery+without+RMAN)|

Qué debes estudiar: recuperación mediante RMAN y recuperación gestionada por el usuario, restauración de datafiles, aplicación de archived redo logs, recuperación de control files y requisitos previos para abrir la base de datos.

La recuperación no RMAN debe entenderse como un procedimiento distinto: no se deben mezclar comandos de RMAN y procedimientos manuales sin comprender qué metadatos y archivos se están utilizando.

## 15. Transporting Data

|Exam Topic|Recursos gratuitos|
|---|---|
|Transport Data|[Transporting Data](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/transporting-data.html) · [Database Utilities](https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/) · [Vídeo: Data Pump y transportable tablespaces](https://www.youtube.com/results?search_query=Oracle+19c+transport+data+Data+Pump+transportable+tablespaces)|

Qué debes estudiar: Data Pump Export/Import, transportable tablespaces, transporte entre plataformas compatibles, conversión de endianess cuando corresponda y comprobaciones de consistencia y compatibilidad.

## 16. RMAN Troubleshooting and Tuning

|Exam Topic|Recursos gratuitos|
|---|---|
|Interpret the RMAN message output|[RMAN Error Message Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/errmg/) · [Backup and Recovery Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/rcmrf/) · [Vídeo: interpretar errores RMAN](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+troubleshooting+error+messages+debugging)|
|Diagnose RMAN performance issues|[Tuning RMAN Performance](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/tuning-rman-performance.html) · [Vídeo: RMAN performance tuning](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+performance+tuning+backup+restore+channels+parallelism)|

Qué debes estudiar: mensajes RMAN, errores de canales, problemas de I/O, throughput, `RATE`, `MAXPIECESIZE`, paralelismo, multiplexación y cuellos de botella durante backups y restauraciones.

La guía de Backup and Recovery contiene capítulos específicos de diagnóstico, rendimiento y solución de problemas de RMAN.

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

# Parte III. Deploy, Patch and Upgrade Workshop

## 17. Install Grid Infrastructure and Oracle Database

|Exam Topic|Recursos gratuitos|
|---|---|
|Install Grid Infrastructure for a Standalone server|[Installing Grid Infrastructure for a Standalone Server](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/installing-and-configuring-oracle-grid-infrastructure-for-a-standalone-server.html) · [Vídeo: instalar Grid Infrastructure standalone](https://www.youtube.com/results?search_query=Oracle+19c+Grid+Infrastructure+standalone+server+installation)|
|Install Oracle Database software|[Database Installation Guide for Linux](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/) · [Vídeo: instalación Oracle Database 19c en Linux](https://www.youtube.com/results?search_query=Oracle+Database+19c+Linux+installation+step+by+step)|

Qué debes estudiar: requisitos del sistema operativo, usuarios y grupos, Oracle Base, Oracle Inventory, Oracle Home, preinstalación, instalación por imagen, configuración de red y comprobaciones posteriores.

Para practicar en Linux, consulta también la [lista oficial de requisitos previos](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/operating-system-checklist-for-oracle-database-installation.html).

## 18. Patching Grid Infrastructure and Oracle Database

|Exam Topic|Recursos gratuitos|
|---|---|
|Patch Grid Infrastructure and Oracle Database|[Oracle Database Patch Maintenance](https://docs.oracle.com/en/database/oracle/oracle-database/19/dbptc/) · [Grid Infrastructure Installation and Upgrade Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/cwsol/) · [Vídeo: OPatch y parches de Oracle 19c](https://www.youtube.com/results?search_query=Oracle+19c+OPatch+Grid+Infrastructure+database+patching+RU+tutorial)|

Qué debes estudiar: inventario de parches, OPatch, prerequisitos, conflictos, aplicación de Release Updates (RU), parcheado de Grid Infrastructure y base de datos, `datapatch` y validación del resultado.

Recurso adicional: [Oracle Database Patch Maintenance — métodos de parcheado](https://docs.oracle.com/en/database/oracle/oracle-database/19/dbptc/toc.htm).

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

+1

## 19. Upgrading to Oracle Grid Infrastructure

|Exam Topic|Recursos gratuitos|
|---|---|
|Upgrade Oracle Grid Infrastructure|[Grid Infrastructure Installation and Upgrade Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/cwsol/) · [Vídeo: upgrade Grid Infrastructure 19c](https://www.youtube.com/results?search_query=Oracle+19c+Grid+Infrastructure+upgrade+standalone+Oracle+Restart)|

Qué debes estudiar: compatibilidad de versiones, preparación del entorno, requisitos del upgrade, comprobaciones previas, actualización del Grid Home, validación de servicios y comprobaciones posteriores.

Distingue el upgrade de Grid Infrastructure del upgrade del software de base de datos: pueden estar relacionados, pero tienen procedimientos, homes y validaciones propios.

## 20. Oracle Database 18c: New Features

|Exam Topic|Recursos gratuitos|
|---|---|
|Image and RPM based Database Installation|[Database Installation Guide for Linux 19c: instalación por imagen y RPM](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/) · [Instalación por RPM](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/running-rpm-packages-to-install-oracle-database.html) · [Vídeo: Oracle Database image-based installation](https://www.youtube.com/results?search_query=Oracle+Database+18c+image+based+installation+RPM)|

Qué debes estudiar: diferencias entre instalación tradicional, instalación basada en imagen y paquetes RPM; preparación del sistema, configuración del software y creación de la base de datos.

Importante: este objetivo se refiere expresamente a Oracle 18c. La documentación 19c sirve para estudiar conceptos de instalación que siguen vigentes, pero para una pregunta específica sobre novedades de 18c debes consultar la documentación de esa versión. La documentación histórica de Oracle puede localizarse desde el [portal de documentación](https://docs.oracle.com/en/database/).

## 21. Creating an Oracle Database by using DBCA

|Exam Topic|Recursos gratuitos|
|---|---|
|Create, Delete and Configure Databases using DBCA|[Database Configuration Assistant (DBCA)](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html) · [DBCA mediante response files](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/running-database-configuration-assistant-using-response-files.html) · [Vídeo: DBCA create, delete y configure](https://www.youtube.com/results?search_query=Oracle+19c+DBCA+create+delete+configure+database+tutorial)|

Qué debes estudiar: creación, eliminación y configuración de bases de datos, creación interactiva y silenciosa, response files, configuración de opciones y comprobación del resultado.

## 22. Oracle Restart

|Exam Topic|Recursos gratuitos|
|---|---|
|Configure and use Oracle Restart to manage components|[Installing and Configuring Oracle Grid Infrastructure for a Standalone Server](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/installing-and-configuring-oracle-grid-infrastructure-for-a-standalone-server.html) · [Vídeo: Oracle Restart y SRVCTL](https://www.youtube.com/results?search_query=Oracle+19c+Oracle+Restart+SRVCTL+ASM+listener+database+tutorial)|

Qué debes estudiar: relación entre Oracle Restart, ASM, listeners y bases de datos; alta disponibilidad de una instancia única; registro de recursos y uso de `srvctl` para iniciar, detener, comprobar y administrar componentes.

Oracle Restart forma parte de Grid Infrastructure para un servidor independiente y puede gestionar el reinicio automático de componentes registrados.

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

+1

## 23. Upgrade the Oracle Database

|Exam Topic|Recursos gratuitos|
|---|---|
|Plan for Upgrading an Oracle Database|[Preparing to Upgrade Oracle Database](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/preparing-for-upgrading-oracle-database.html) · [Vídeo: planificar un upgrade](https://www.youtube.com/results?search_query=Oracle+19c+database+upgrade+planning+preupgrade+checks)|
|Upgrade an Oracle Database|[Using AutoUpgrade](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/) · [Vídeo: upgrade con AutoUpgrade](https://www.youtube.com/results?search_query=Oracle+19c+AutoUpgrade+analyze+deploy+upgrade+tutorial)|
|Perform Post-Upgrade tasks|[Post-Upgrade Tasks](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/post-upgrade-tasks-for-oracle-database.html) · [Vídeo: tareas posteriores al upgrade](https://www.youtube.com/results?search_query=Oracle+19c+post+upgrade+tasks+utlrp+datapatch+timezone)|

Qué debes estudiar: planificación, compatibilidad, espacio, backups, herramientas de preupgrade, AutoUpgrade y DBUA, validación de componentes, recompilación de objetos inválidos y comprobaciones posteriores.

Oracle documenta los métodos de upgrade, las tareas previas y las posteriores en su guía de actualización.

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

+1

## 24. Install Grid Infrastructure for a Standalone Server

|Exam Topic|Recursos gratuitos|
|---|---|
|Rapid Home Provisioning|[Fleet Patching and Provisioning: guía oficial](https://docs.oracle.com/en/database/oracle/oracle-database/19/fppad/) · [Instalación mediante Fleet Patching and Provisioning](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/about-deploying-oracle-database-using-rapid-home-provisioning.html) · [Vídeo: Rapid Home Provisioning / Fleet Patching and Provisioning](https://www.youtube.com/results?search_query=Oracle+19c+Rapid+Home+Provisioning+Fleet+Patching+Provisioning+tutorial)|

Qué debes estudiar: imágenes de software (gold images), provisión de Oracle Homes, despliegue, clonación, parcheado y upgrade mediante provisión automatizada.

Oracle cambió el nombre de Rapid Home Provisioning a Oracle Fleet Patching and Provisioning (FPP) en Oracle Database 19c. Por eso encontrarás ambos términos en documentación y vídeos.

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

+1

# Parte IV. Oracle Database 19c: New Features for Administrators

## 25. Using General Overall Database Enhancements

|Exam Topic|Recursos gratuitos|
|---|---|
|Install Oracle Database software|[Installation Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/) · [Vídeo: instalar Oracle 19c](https://www.youtube.com/results?search_query=Oracle+Database+19c+Linux+installation+tutorial)|
|Create, Delete and Configure Databases using DBCA|[DBCA](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/creating-and-configuring-an-oracle-database.html) · [Vídeo: DBCA 19c](https://www.youtube.com/results?search_query=Oracle+19c+DBCA+create+delete+configure+database)|
|Creating CDBs and Regular PDBs|[Creating PDBs](https://docs.oracle.com/en/database/oracle/oracle-database/19/multi/creating-pdbs.html) · [Vídeo: CDB y PDB](https://www.youtube.com/results?search_query=Oracle+19c+create+CDB+PDB+tutorial)|
|Use Miscellaneous 19c New Features|[Oracle Database 19c New Features Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/newft/) · [Vídeo: novedades Oracle 19c para administradores](https://www.youtube.com/results?search_query=Oracle+Database+19c+new+features+for+DBAs)|

Qué debes estudiar: novedades relevantes para administración general, instalación y DBCA, creación de CDB/PDB y cambios que afectan al trabajo diario de un DBA. Dentro de New Features Guide, prioriza los apartados Database Overall y las novedades relacionadas con las tareas que entran en tu examen.

## 26. Using Availability Enhancements

|Exam Topic|Recursos gratuitos|
|---|---|
|Use an RMAN recovery catalog|[Managing a Recovery Catalog](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/managing-recovery-catalog.html) · [Vídeo: RMAN recovery catalog](https://www.youtube.com/results?search_query=Oracle+19c+RMAN+recovery+catalog+tutorial)|
|Use Flashback Database|[Configuring Flashback Database](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/configuring-flashback-database.html) · [Vídeo: Flashback Database](https://www.youtube.com/results?search_query=Oracle+19c+Flashback+Database+restore+point+tutorial)|

Qué debes estudiar: gestión centralizada de metadatos RMAN, recuperación de bases de datos, configuración de Flashback, restore points y cómo utilizar estas tecnologías para reducir el tiempo de recuperación.

## 27. Using Diagnosibility Enhancements

|Exam Topic|Recursos gratuitos|
|---|---|
|Use new Diagnoseability Features|[Oracle Database 19c New Features Guide — Diagnosability](https://docs.oracle.com/en/database/oracle/oracle-database/19/newft/) · [Oracle Database Utilities — ADRCI](https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/) · [Vídeo: novedades de diagnosability en Oracle 19c](https://www.youtube.com/results?search_query=Oracle+19c+diagnosability+enhancements+ADRCI+TFA)|

Qué debes estudiar: mejoras de diagnóstico de 19c, herramientas de diagnóstico, alertas, trazas, ADR y utilidades para recopilar y analizar información sobre problemas.

El New Features Guide organiza las novedades por áreas, incluida Diagnosability. Para este objetivo, estudia las funciones concretas que Oracle identifica como nuevas en 19c, no solo el uso general de ADRCI.

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

+1

# Parte V. Oracle Database: Administration Workshop

## 28. Monitoring and Tuning Database Performance

|Exam Topic|Recursos gratuitos|
|---|---|
|Managing Memory Components|[Managing Memory](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/managing-memory.html) · [Vídeo: SGA, PGA y memoria Oracle](https://www.youtube.com/results?search_query=Oracle+19c+SGA+PGA+memory+management+DBA)|
|Understanding the Automatic Workload Repository (AWR)|[Managing Automatic Workload Repository](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/managing-automatic-workload-repository.html) · [Vídeo: interpretar informes AWR](https://www.youtube.com/results?search_query=Oracle+19c+AWR+report+interpretation+DBA+tutorial)|
|Understanding the Advisory Framework|[Performance Tuning Overview](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/performance-tuning-overview.html) · [Vídeo: Oracle Database performance advisors](https://www.youtube.com/results?search_query=Oracle+19c+performance+advisors+memory+SQL+ADDM)|
|Monitoring Wait Events, Sessions, and Services|[Performance Tuning Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/) · [Vídeo: wait events, sessions y servicios](https://www.youtube.com/results?search_query=Oracle+19c+wait+events+sessions+services+performance+monitoring)|
|Managing Metric Thresholds and Alerts|[Managing Automatic Database Performance Monitoring](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/) · [Vídeo: métricas, thresholds y alertas](https://www.youtube.com/results?search_query=Oracle+19c+metric+thresholds+server+generated+alerts+tutorial)|
|Understanding and Using the Performance Tuning Methodology|[Performance Tuning Methodology](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/instance-tuning-using-performance-views.html) · [Vídeo: Oracle performance tuning methodology](https://www.youtube.com/results?search_query=Oracle+19c+performance+tuning+methodology+DBA)|
|Performing Performance Planning|[Performance Tuning Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/) · [Vídeo: Oracle capacity and performance planning](https://www.youtube.com/results?search_query=Oracle+Database+performance+capacity+planning+DBA)|
|Understanding the Automatic Database Diagnostic Monitor (ADDM)|[Diagnosing Performance Problems Using ADDM](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/diagnosing-performance-problems-using-addm.html) · [Vídeo: interpretar informes ADDM](https://www.youtube.com/results?search_query=Oracle+19c+ADDM+report+interpretation+tutorial)|

Qué debes dominar:

- Componentes de memoria SGA/PGA, parámetros y asesores de memoria.
    
- Qué recoge AWR, snapshots, informes y comparación de periodos.
    
- Cómo interpretar sesiones, servicios, eventos de espera y métricas.
    
- Relación entre AWR, ADDM y los asesores de tuning.
    
- Cómo establecer una línea base, identificar un cuello de botella, formular una hipótesis y validar la solución.
    
- Capacidad, carga de trabajo y planificación de rendimiento.
    

La documentación oficial describe AWR como infraestructura para recopilar y conservar estadísticas de carga de trabajo, que ADDM analiza para detectar posibles problemas de rendimiento y recomendar acciones.

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

+1

Nota de licencia: estudia los conceptos y la interpretación de informes, pero verifica la licencia antes de utilizar AWR, ADDM u otras funciones de Diagnostics Pack en un entorno real.

## 29. Tuning SQL Statements

|Exam Topic|Recursos gratuitos|
|---|---|
|Understanding the Oracle Optimizer|[Query Optimizer Concepts](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgsql/query-optimizer-concepts.html) · [Vídeo: Oracle Cost-Based Optimizer](https://www.youtube.com/results?search_query=Oracle+19c+optimizer+execution+plans+cost+based+optimizer)|
|Using the SQL Tuning Advisor|[SQL Tuning Advisor](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgsql/introduction-to-sql-tuning.html) · [Vídeo: SQL Tuning Advisor](https://www.youtube.com/results?search_query=Oracle+19c+SQL+Tuning+Advisor+tutorial)|
|Managing Optimizer Statistics|[Managing Optimizer Statistics](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgsql/optimizer-statistics-concepts.html) · [Vídeo: DBMS_STATS y estadísticas](https://www.youtube.com/results?search_query=Oracle+19c+DBMS_STATS+optimizer+statistics+tutorial)|
|Using the SQL Access Advisor|[SQL Access Advisor](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgsql/sql-access-advisor.html) · [Vídeo: SQL Access Advisor](https://www.youtube.com/results?search_query=Oracle+19c+SQL+Access+Advisor+indexes+materialized+views)|
|Understanding the SQL Tuning Process|[Introduction to SQL Tuning](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgsql/introduction-to-sql-tuning.html) · [Vídeo: proceso de SQL tuning](https://www.youtube.com/results?search_query=Oracle+19c+SQL+tuning+process+execution+plan+statistics)|

Qué debes dominar:

- Cómo el optimizador elige planes de ejecución y utiliza estadísticas.
    
- Diferencia entre estimaciones de cardinalidad, coste, acceso por índice y full table scan.
    
- Cómo interpretar un plan de ejecución.
    
- Qué analiza SQL Tuning Advisor y cómo presenta recomendaciones.
    
- Recopilación de estadísticas con `DBMS_STATS` y consecuencias de estadísticas obsoletas.
    
- Qué recomienda SQL Access Advisor sobre índices, vistas materializadas y otras estructuras de acceso.
    
- Cómo medir una consulta antes y después de cambiarla.
    

La documentación oficial de Oracle diferencia claramente SQL Tuning Advisor —que analiza sentencias individuales o conjuntos de sentencias— de SQL Access Advisor, que recomienda estructuras de acceso a partir de una carga de trabajo.

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

Oracle Docs

+2

# Cómo utilizar esta guía para estudiar el examen

Ahora tienes los recursos de todos los bloques que has enumerado, pero te recomiendo seguir este procedimiento para no perder tiempo con vídeos redundantes.

1. Documentación oficial primero

Lee la sección concreta del manual indicada para el objetivo. Anota los conceptos, requisitos, comandos y restricciones.

2. Vídeo para visualizar el procedimiento

Utiliza la búsqueda enlazada para localizar una demostración de la operación. Contrasta la versión de Oracle del vídeo con la del objetivo del examen.

3. Laboratorio

Ejecuta la operación en una base de datos de pruebas y documenta los resultados. Para instalación, recuperación y parcheado, practica en un entorno que puedas restaurar.

4. Verificación del Exam Topic

Da el objetivo por estudiado cuando puedas explicar la operación, identificar sus requisitos y restricciones, resolver un escenario y reconocer qué procedimiento sería incorrecto.

## Dos comprobaciones importantes antes de pagar o presentarte al examen

- Versión: los materiales que has enumerado mezclan referencias a Oracle 18c y 19c. Para comandos y procedimientos de administración, utiliza preferentemente documentación 19c, pero consulta las diferencias de versión cuando el objetivo trate expresamente de una novedad de 18c o 19c.
    
- Cobertura: esta guía enlaza documentación oficial gratuita para cada objetivo y búsquedas de vídeos específicas. No significa que cada vídeo cubra el objetivo entero ni que se haya verificado cada vídeo individual. La documentación oficial y los Exam Topics son la referencia para comprobar la cobertura.
    

El siguiente paso lógico es usar tu lista original como checklist y marcar cada objetivo solo cuando hayas completado la lectura y la práctica correspondientes. Así podrás avanzar por los cursos de MyLearn sin pagar por ellos y mantener una trazabilidad clara de qué contenido has estudiado, qué has practicado y qué te falta dominar.