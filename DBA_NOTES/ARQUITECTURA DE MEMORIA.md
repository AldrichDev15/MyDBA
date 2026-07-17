# ARQUITECTURA DE MEMORIA EN LAS BASES DE DATOS EN ORACLE

> **Referencia oficial:** [Oracle Database Concepts 19c — Memory Architecture](https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/memory-architecture.html#GUID-913335DF-050A-479A-A653-68A064DCCA41)

---

## INTRODUCCIÓN

Cuando se inicia una instancia, Oracle asigna un área de memoria en la que almacena:

- El código ejecutable de Oracle.
- Información sobre todas las sesiones conectadas.
- Datos almacenados en caché, como bloques de datos y registros de _redo_.
- Información compartida entre procesos, como bloqueos (_locks_) y el estado de las operaciones o consultas en ejecución.

La base de datos Oracle consta de varias áreas de memoria:

- **SGA (_System Global Area_)**: Es un grupo de estructuras de memoria compartida que contienen datos y controlan toda la instancia. La comparten tanto los procesos servidor como los procesos en segundo plano.
- **PGA (_Program Global Area_)**: Es una región de memoria usada exclusivamente por un proceso Oracle. La PGA se crea cuando un proceso comienza.
- **UGA (_User Global Area_)**: Se refiere a la memoria asociada a la sesión de usuario.
- **Áreas de software**: Almacena el código que se está ejecutando o se puede ejecutar.

La administración de la memoria implica mantener tamaños óptimos a medida que las necesidades cambian. En ese sentido tenemos varias opciones de administración:

- **AMM (_Automatic Memory Management_)**: Especificas el tamaño total de la memoria de Oracle y se distribuye sola entre SGA y PGA.
- **ASMM (_Automatic Shared Memory Management_)**: Pones un tamaño para la SGA y Oracle ajusta lo que consumen los distintos componentes. Para la PGA tienes la opción de fijar un tamaño agregado (`PGA_AGGREGATE_TARGET`) o de gestionar tú mismo las _work areas_ una a una.
- **MMM (_Manual Memory Management_)**: Todo manual.

## UGA (_User Global Area_)

Memoria de sesión: variables de sesión como la información del login, entre otra información necesaria para la sesión. También almacena el estado de los paquetes PL/SQL (_package state_), es decir, los valores de sus variables globales. Por defecto, estas variables son privadas de cada sesión y conservan su valor mientras la sesión permanezca activa. En conexiones _Dedicated Server_, la UGA se almacena en la PGA del proceso servidor. En conexiones _Shared Server_, se almacena en la SGA ya que distintos procesos pueden atender la misma sesión.

## PGA (_Program Global Area_)

Es memoria específica para un proceso o hilo que no es compartida. Almacena las estructuras temporales y variables de sesión que ese proceso necesita para ejecutarse. La _instance PGA_ es la suma de todas las PGA de los procesos de la instancia, y su tamaño puede gestionarse mediante parámetros como `PGA_AGGREGATE_TARGET`, que establece un objetivo de memoria para el conjunto de todas ellas.

Se divide en varias áreas:

- **Área SQL privada (_Private SQL Area_):** Almacena la información necesaria para ejecutar una sentencia SQL o PL/SQL en una sesión concreta. Incluye los valores de las variables _bind_ (`:valor`), el estado de ejecución de la consulta y la memoria temporal utilizada durante su ejecución. Un **cursor** es el identificador asociado a un área SQL privada que permite gestionar el estado de ejecución de la sentencia. A su vez se divide en:
    
    - **Área de tiempo de ejecución (_run-time area_)**: Contiene la información del estado de la ejecución de la consulta. Al procesar una ejecución, lo primero que hace Oracle es crear esta área. Si la operación contiene DML, el _run-time area_ se libera cuando se cierra la sentencia.
    - **Área persistente (_persistent area_)**: Contiene los valores de las _bind variables_. Esta área solo se libera cuando se cierra el cursor.
    
    La aplicación (a través del proceso cliente) gestiona la apertura y cierre de los cursores, que están asociados a las áreas SQL privadas. El número máximo de cursores abiertos por sesión está limitado por el parámetro `OPEN_CURSORS`. Es recomendable cerrar los cursores que ya no se utilicen para liberar memoria y evitar alcanzar dicho límite.
- **Área de trabajo (_Work Area_)**: Consiste en una asignación privada para ejecutar operaciones que requieren un uso intensivo de memoria (_sorts_, _hash joins_, _bitmap merges_...). Si la cantidad de datos a procesar por los operadores no cabe en un área de trabajo, Oracle los divide en partes más pequeñas y las guarda en disco temporalmente (esto reduce mucho el rendimiento, por lo que hay que ajustar bien el valor de estas áreas). Estas áreas de trabajo se dimensionan automáticamente, aunque también podemos establecer nosotros el valor.

### PGA en _Shared_ y _Dedicated Servers_

La asignación de la PGA depende de si estamos usando conexiones tipo _dedicated_ o _shared_. Si es _shared server_, la _persistent area_ está en la SGA (el _run-time area_, en cambio, permanece siempre en la PGA, tanto en _dedicated_ como en _shared_).

## SGA (_System Global Area_)

Es una memoria de lectura/escritura que, junto a los procesos en segundo plano de Oracle, forma la instancia. Son los procesos servidor —que actúan en nombre de los usuarios— los que pueden leer información de la SGA, y varios procesos escriben en ella durante las operaciones de la base de datos. Se asigna la SGA al arrancar una instancia y cada instancia tiene la suya. Se libera al apagar dicha instancia.

La SGA se compone de varias partes llamadas componentes SGA. Todos los componentes asignan memoria en espacios unitarios contiguos llamados gránulos, a excepción del _redo log buffer_. El tamaño de los gránulos es específico por plataforma y lo determina el tamaño de la SGA.

Los componentes más importantes son:

### DB Buffer Cache

Mantiene copias de los bloques de datos leídos de los _datafiles_. Este componente logra varios objetivos:

- **Optimizar las operaciones I/O**: Cuando una transacción modifica datos, Oracle actualiza los bloques en la DB buffer cache y registra los cambios en el _redo log buffer_. Tras un COMMIT, el proceso LGWR escribe el contenido del _redo log buffer_ en el _online redo log_, garantizando la recuperación de la transacción. Los bloques de datos no se escriben inmediatamente en los datafiles; esta tarea la realiza posteriormente el proceso DBW mediante escrituras diferidas (_lazy writes_) para mejorar el rendimiento al escribir varias modificaciones de una sentada.
- **Mantener los bloques de datos de acceso frecuente en caché y el resto en disco**: **Flash Cache** permite ampliar la DB buffer cache utilizando almacenamiento flash (por ejemplo, SSD o memoria flash), reduciendo el número de accesos a los discos convencionales. Se configura mediante los parámetros `DB_FLASH_CACHE_FILE` y `DB_FLASH_CACHE_SIZE` (solo disponible en Solaris y Oracle Linux).

La base de datos usa algoritmos internos para gestionar los buffers en caché. Los buffers pueden estar en uno de estos estados, mutuamente excluyentes:

- **_Unused_**: Disponible para su uso pues nunca ha sido usado o actualmente no está en uso.
- **_Clean_**: El buffer contiene una copia consistente de un bloque de datos que ya está sincronizada con el disco. Como no ha sido modificado, no necesita checkpoint antes de reutilizarse.
- **_Dirty_**: Contiene datos modificados en memoria y no escritos en disco, por lo que es necesario hacer checkpoint del bloque antes de reutilizarlo.

Además del estado, cada buffer tiene un **modo de acceso** independiente: _pinned_ (fijado) o _free_ (libre). Mientras una sesión está accediendo a un buffer, este se fija (_pinned_) para que no sea expulsado de memoria mientras se usa; mientras permanece fijado, ninguna otra sesión puede modificarlo.

Cuando un cliente solicita datos, Oracle recupera los bloques de la DB buffer cache en uno de estos dos modos:

- **_Current mode_**: Recupera los datos tal y como están en memoria, aunque tengan datos no confirmados. Se usa en operaciones de modificación (`INSERT`, `DELETE`...).
- **_Consistent mode_**: Recupera una versión consistente de los bloques de datos. Si estos bloques de datos no se han confirmado, Oracle usa el UNDO para reconstruir una copia consistente (_consistent read clone_). Es el modo que utilizan las consultas.

Esto permite que sesiones puedan consultar datos mientras otras modifican dichos bloques, sin bloquearse entre ellas.

Una **_buffer I/O_** (también llamada _logical I/O_) se refiere al acceso a un bloque que ya se encuentra en la DB buffer cache. Si el bloque no está en memoria, Oracle realiza primero una I/O física para cargarlo desde la Flash Cache o el disco al buffer cache y, a continuación, realiza la I/O lógica para acceder a él.

Para que la base de datos sea eficiente debe escoger bien qué buffers cachea en memoria y cuáles lee desde disco. Para ello usa los siguientes algoritmos:

- **Algoritmo basado en LRU (_Least Recently Used_), a nivel de bloque**: Algoritmo por defecto. Tiene dos categorías para los bloques: calientes o fríos. Se reutilizan los bloques fríos. Oracle tiene varias listas LRU para mejorar la concurrencia.
- **Algoritmo basado en temperatura, a nivel de objeto**: Introducido en 12c (12.1.0.2) para mejorar el rendimiento de los escaneos de tablas grandes (_automatic big table caching_). En lugar de decidir qué bloques conservar individualmente, Oracle evalúa la frecuencia de acceso de cada objeto y decide qué porcentaje de sus bloques mantener en memoria, evitando el fenómeno de _thrashing_ (carga y expulsión continua de bloques cuando una tabla no cabe completamente en la caché).

> **Nota:** salvo que se indique lo contrario, el resto de este apartado (touch count, escrituras de DBW...) describe el comportamiento del algoritmo LRU por defecto, no el algoritmo basado en temperatura.

El proceso **Database Writer (DBW)** escribe periódicamente en disco los buffers sucios (_dirty_) y fríos (_cold_) de la DB buffer cache. DBW escribe buffers en las siguientes situaciones:

- Cuando un proceso servidor no encuentra suficientes buffers limpios para cargar nuevos bloques en la caché.
- Cuando el número de buffers libres disminuye por debajo de un umbral interno. En ese caso, los procesos servidor solicitan a DBW que escriba buffers sucios al disco para liberar espacio.
- Utilizando el algoritmo LRU: cuando un buffer sucio llega al extremo _cold_ de la lista LRU, Oracle lo mueve a una cola de escritura y DBW lo escribe en el disco (preferiblemente mediante escrituras de varios bloques a la vez). Así se evita que los buffers sucios ocupen el extremo de la LRU e impidan reutilizar la caché.
- Cuando Oracle necesita avanzar un checkpoint (la posición en el hilo de _redo_ desde la que debe comenzar la _instance recovery_), escribiendo en disco los bloques modificados necesarios para ello.
- Cuando un tablespace se cambia a modo READ ONLY o se deja OFFLINE (se asegura de que no quedan cambios pendientes en memoria).

Cuando el número de buffers libres es bajo, Oracle necesita reutilizar algunos buffers de la DB buffer cache. El algoritmo depende de si la Flash Cache está habilitada:

- **Flash Cache deshabilitada**: Oracle reutiliza directamente los buffers limpios, sobrescribiéndolos. Si más adelante vuelve a necesitar ese bloque, tendrá que leerlo de nuevo desde el disco.
- **Flash Cache habilitada**: Antes de reutilizar un buffer limpio, DBW puede copiar su contenido a la Flash Cache. De este modo, el buffer de memoria queda libre para reutilizarse, pero se conserva una referencia (_buffer header_) en memoria que indica dónde está almacenado el bloque en la Flash Cache. Si posteriormente vuelve a necesitar ese bloque, podrá leerlo desde la Flash Cache en lugar de desde el disco.

Cuando un proceso necesita un bloque, Oracle lo busca en este orden:

1. Busca el bloque completo en la DB buffer cache: si lo encuentra, realiza una lectura lógica (_cache hit_).
2. Si no está en memoria, busca su cabecera (_buffer header_) en la Flash Cache: si la encuentra, recupera el bloque desde la Flash Cache mediante una lectura física optimizada.
3. Si tampoco está en la Flash Cache (_cache miss_): lee el bloque desde el datafile (lectura física), lo copia a la buffer cache y realiza la lectura lógica desde memoria.

El **_buffer cache hit ratio_** mide la frecuencia con la que la base de datos encuentra en caché un bloque solicitado sin necesidad de leerlo desde disco.

Oracle puede realizar lecturas físicas desde datafiles o tempfiles. Las lecturas desde un datafile copian primero el bloque a la DB buffer cache y posteriormente realizan una lectura lógica. En cambio, las lecturas desde un tempfile se producen cuando Oracle necesita utilizar espacio temporal por falta de memoria (por ejemplo, en ordenaciones o _hash joins_) y acceden directamente al tempfile, sin pasar por la DB buffer cache ni generar una lectura lógica.

Oracle mide la frecuencia con la que se accede a los buffers de la lista LRU mediante un contador llamado _touch count_. En lugar de mover continuamente los buffers dentro de la lista LRU cada vez que se accede a ellos, Oracle simplemente incrementa este contador lógico cuando un buffer es fijado (_pinned_).

> **Nota:** Oracle no mueve físicamente los bloques en memoria. Lo que cambia es la posición de los punteros que los referencian en la lista LRU.

Cuando un buffer se fija (_pin_), Oracle comprueba cuándo se incrementó por última vez su _touch count_:

- Si han pasado más de 3 segundos, incrementa el contador.
- Si no han pasado 3 segundos, el contador permanece igual.

Esta regla evita que muchos accesos consecutivos a un mismo bloque se contabilicen como múltiples usos independientes. Por ejemplo, si una sesión inserta varias filas seguidas en un mismo bloque, Oracle lo considera un único acceso.

Si un buffer llega al extremo frío (_cold_) de la lista LRU pero tiene un _touch count_ elevado, se lo devuelve al extremo caliente (_hot_). Si su _touch count_ es bajo, el buffer termina siendo expulsado de la caché.

Un **_buffer pool_** es un conjunto de buffers. La DB buffer cache puede dividirse en uno o varios buffer pools. Todos gestionan los bloques prácticamente con el mismo algoritmo de reemplazo (LRU y _touch count_), pero permiten separar los objetos según el uso que se hace de ellos. Oracle permite configurar manualmente distintos buffer pools y asignar tablas, índices u otros segmentos a cada uno para controlar cómo permanecen en la caché.

Los principales son:

- **_Default pool_**: pool por defecto. Si no se configura nada, toda la buffer cache corresponde a este pool.
- **_Keep pool_**: pensado para objetos que se acceden con frecuencia, para reducir lecturas desde disco. No utiliza un algoritmo especial; simplemente se le suele asignar más memoria.
- **_Recycle pool_**: para objetos poco utilizados. Permite que sus buffers se reutilicen rápidamente y evita que ocupen espacio innecesario en la caché.

Si una base de datos utiliza tablespaces con tamaños de bloque distintos al tamaño estándar, Oracle crea un buffer pool independiente para cada tamaño de bloque (por ejemplo, 2 KB, 4 KB o 16 KB).

Oracle utiliza un algoritmo complejo para gestionar los _full table scans_. Por defecto, cuando debe leer bloques desde el disco, no los coloca al principio de la lista LRU (_hot_), sino aproximadamente en el medio. De esta forma, los bloques que ya eran muy utilizados pueden permanecer en la parte _hot_ de la caché y no tienen que volver a leerse desde el disco. El problema aparece con un _full table scan_, que lee secuencialmente todos los bloques de una tabla hasta su _High Water Mark_ (HWM). Si la tabla es más grande que la DB buffer cache, un escaneo completo podría llenar la caché con bloques que probablemente solo se van a utilizar una vez, expulsando bloques más útiles.

- Si la tabla ocupa un porcentaje pequeño en función del total de la caché, se carga en caché.
- Si la tabla es de tamaño medio, Oracle decide dinámicamente si merece la pena cachearla, teniendo en cuenta cuándo fue el último escaneo de la tabla, el estado de envejecimiento (_aging_) de la buffer cache y el espacio libre disponible.
- Si la tabla es muy grande, normalmente utiliza un **_Direct Path Read_**, leyendo los bloques directamente a la PGA sin pasar por la SGA, para no llenar la DB buffer cache.

Si para una tabla de tamaño medio Oracle decide usar la DB buffer cache, coloca los bloques en el extremo _cold_ de la lista LRU para que puedan reutilizarse rápidamente y no desplacen los bloques más utilizados.

Desde Oracle 12c (12.1.0.2), Oracle puede detectar automáticamente si toda la base de datos cabe en memoria. Si es así, trata todas las tablas como pequeñas y las cachea. Una excepción son los objetos LOB definidos con el atributo `NOCACHE`, que no se almacenan en la DB buffer cache.

Durante un _full table scan_, Oracle puede utilizar varios procesos de ejecución en paralelo para acelerar la consulta. Aunque las tablas grandes suelen leerse mediante _direct path reads_ hacia la PGA, si la instancia dispone de suficiente memoria Oracle puede optar por almacenar los bloques en la DB buffer cache para reutilizarlos en consultas posteriores (habitual en entornos _Data Warehouse_, donde hay un gran volumen de datos y poca concurrencia).

El atributo `CACHE` (`ALTER TABLE ... CACHE`) indica a Oracle que los bloques de esa tabla se deben cargar en la DB buffer cache (no se cargan al ejecutar el comando: hace falta consultar la tabla para que se cacheen). Para estas tablas, Oracle no fuerza ni fija los bloques en caché: siguen envejeciendo con el mismo algoritmo que cualquier otro bloque. Ejecutar este comando con cuidado, puesto que si lo ponemos en una tabla grande, un _full table scan_ puede hacer que se llene la DB buffer cache de bloques que no son tan útiles.

Con `ALTER TABLE ... STORAGE (BUFFER_POOL KEEP)` se puede indicar que los bloques de una tabla se almacenen en el _keep pool_ en lugar del _default pool_. Esto no modifica el algoritmo de gestión de la caché (LRU + _touch count_), sino únicamente el pool donde se almacenan. El objetivo es aislar los objetos más importantes y reducir la probabilidad de que sus bloques sean reemplazados por los de otros objetos.

`ALTER DATABASE FORCE FULL DATABASE CACHING` activa un modo en el que Oracle considera que todos los objetos de la base de datos, incluidos los LOB `NOCACHE`, son candidatos para almacenarse en la DB buffer cache. Esto no fuerza a que la base de datos entera se cargue en memoria de golpe: los objetos se siguen cacheando solo a medida que se accede a ellos, pero Oracle deja de aplicar la política conservadora habitual y considera candidata a cualquier tabla, siempre que la memoria disponible lo permita. Disponible desde Oracle 12c (12.1.0.2).

Se recomienda usar `FORCE FULL DATABASE CACHING` únicamente cuando la DB buffer cache de cada instancia sea mayor que el tamaño de la base de datos, de forma que esta pueda mantenerse prácticamente completa en memoria (aplica tanto a RAC como a _single instance_). En RAC, cuando la carga de trabajo está bien distribuida entre instancias, también puede utilizarse si la suma de las cachés de todas las instancias (dejando margen para bloques duplicados) supera el tamaño de la base de datos.

###  Reo Log Buffer
