# Patrones de captura de datos dinámicos

## Visión general y objetivos del documento

El texto introduce y analiza los **datos en streaming** y los **patrones de captura de datos dinámicos** dentro de arquitecturas _big data_, con especial atención a entornos **distribuidos, escalables y cercanos al tiempo real**. El objetivo principal es comprender cómo se adquieren datos continuos, qué arquitecturas lo permiten y qué patrones de comunicación son más adecuados según el caso de uso .

## Datos en streaming: concepto y características

Los **datos en streaming** se definen como flujos continuos de información generados de forma constante por múltiples fuentes (sensores IoT, redes sociales, mercados financieros, sistemas biométricos, etc.). Se caracterizan por cumplir las **tres V del big data**: volumen, variedad y velocidad.

El documento diferencia claramente entre:

- **Sistemas en streaming**, cuyo objetivo es suministrar datos cuando una aplicación los necesita.
- **Sistemas estrictamente en tiempo real**, que requieren respuestas deterministas en milisegundos (por ejemplo, sistemas embebidos críticos).

## Modelos de datos en streaming

Se presentan los principales **modelos formales de flujos de datos**:

1. **Modelo de serie temporal**: cada evento representa el estado de una variable en un instante concreto (ej. cotizaciones bursátiles).
    
2. **Modelo de caja registradora (cash register)**: los eventos representan incrementos acumulativos de una señal (ej. número de accesos por IP).
    
3. **Modelo de torniquete (turnstile)**: permite incrementos y decrementos, generalizando el modelo anterior (ej. entradas y salidas en sistemas concurridos).

Estos modelos permiten representar matemáticamente flujos infinitos y sirven de base para algoritmos de análisis de streaming .

## Arquitectura de un sistema de tratamiento de datos

Un sistema típico de análisis de datos incluye:

1. **Captura de datos**.
2. **Procesamiento o análisis**.
3. **Almacenamiento**.
4. **Visualización** (opcional).

En arquitecturas de streaming se introduce una **capa intermedia de mensajería (broker)** que desacopla productores y consumidores, amortigua diferencias de velocidad y mejora la fiabilidad del sistema .

## Captura de datos en streaming (data ingestion)

A diferencia del procesamiento batch, la captura de datos en streaming es compleja y requiere arquitecturas especializadas. Intervienen tres actores principales:

- **Productores**: generan los datos.
- **Consumidores**: los procesan o almacenan.
- **Brokers de mensajería**: gestionan la comunicación y garantizan políticas de entrega.

Se introducen las **semánticas de entrega**:

- _At most once_
- _At least once_
- _Exactly once_ (objetivo ideal en sistemas de streaming) .

## Patrones de adquisición de datos

El núcleo del documento es la descripción de los **patrones de comunicación más usados en la captura de datos dinámicos**:

### 1. Patrón petición/respuesta (request/response)

- Comunicación síncrona o asíncrona.
- Muy utilizado en HTTP y APIs REST.
- Fácil de implementar, pero poco escalable en escenarios de alta frecuencia.
- Ejemplos prácticos con Python (`http.client`, `requests`, `aiohttp`) y herramientas como `curl` .

### 2. Patrón petición/acuse (request/acknowledge)

- El cliente solo recibe confirmación de recepción (ACK).
- Común en sistemas de _tracking_ y analítica de comportamiento.
- Reduce latencia y sobrecarga frente al request/response clásico .

### 3. Patrón publicación/subscripción (publish/subscribe)

- El patrón más utilizado en streaming.
- Productores publican mensajes en _topics_ y consumidores se suscriben.
- Requiere un **broker** que desacople producción y consumo.
- Protocolos destacados: **MQTT** (IoT), **AMQP** (entornos financieros), **STOMP** (mensajería basada en texto).
- Permite alta escalabilidad, reutilización de datos y múltiples consumidores .

### 4. Patrón unidireccional (one-way)

- No hay confirmación de recepción.
- Normalmente basado en **UDP**.
- Usado cuando la pérdida de mensajes no es crítica (NTP, ciertos escenarios IoT) .

### 5. Patrón de flujo (stream)

- El productor envía datos continuamente tras una solicitud inicial.
- El consumidor marca el ritmo de consumo.
- Evita saturación y reduce latencia.
- Utilizado por tecnologías como **Apache Spark Streaming**, **Apache Storm** o **Spring Cloud Data Flow** .

## Importancia de la capa de mensajería

La **cola de mensajes** es clave para:

- Desacoplar componentes.
- Aumentar escalabilidad y tolerancia a fallos.
- Simplificar aplicaciones distribuidas.

Se describen plataformas **MOM (Message-Oriented Middleware)** como:

- Apache Kafka
- RabbitMQ
- Amazon Kinesis
- Azure Event Hubs
- Google Pub/Sub .

## Almacenamiento de datos en streaming

El almacenamiento puede ser:

- **Temporal en memoria** (Redis, buffers internos de Kafka).
- **Persistente y distribuido** (HDFS, S3, bases NoSQL como HBase, Cassandra o MongoDB).

Estas soluciones permiten gestionar grandes volúmenes de datos con alta disponibilidad .

## Casos de uso

### Sentilo (patrón petición/respuesta)

- Plataforma IoT del Ayuntamiento de Barcelona.
- API REST basada en HTTP.
- Uso de Redis (memoria) y MongoDB (persistencia).
- Fácil de integrar y adecuada para dispositivos con pocos recursos.
- Limitaciones en escenarios de alta carga si no se distribuye adecuadamente .

### Apache Metron (patrón stream)

- Plataforma de ciberseguridad en tiempo real.
- Uso de Kafka, Storm y Hadoop.
- Alta escalabilidad y robustez para detección de amenazas.
- Gran complejidad de despliegue y elevados requisitos de infraestructura .



---

# Métodos y algoritmos para el procesado de datos en streaming


El documento aborda de forma sistemática los **métodos, técnicas y algoritmos fundamentales para el análisis y procesado de datos en streaming**, centrándose en la capa **algorítmica y metodológica** (stream analytics), más allá de la arquitectura. El eje central es el **tiempo**, que condiciona tanto la segmentación de los datos como la forma de analizarlos, agregarlos y sintetizarlos en entornos donde los datos llegan de forma continua, potencialmente infinita y con recursos limitados.

## 1. Características de los streams de datos

Un stream de datos se caracteriza por:

- **Procesamiento en una sola pasada**: los datos no se almacenan para reprocesarse; se analizan conforme llegan.
    
- **Concept drift**: las características estadísticas de los datos pueden cambiar con el tiempo, obligando a análisis adaptativos.
    
- **Flujo no controlado**: pueden producirse picos de datos que el sistema debe gestionar, incluso descartando información (de forma aleatoria o semántica).
    
- **Limitaciones de dominio y recursos**: grandes volúmenes y altas tasas de llegada hacen inviable el análisis exacto, motivando el uso de **técnicas de síntesis**.

Estas restricciones diferencian claramente el streaming del procesado batch tradicional.

## 2. Técnicas fundamentales para el análisis en flujo

### Tiempo en streaming

Se distinguen tres nociones temporales:

- **Event time**: momento en que se genera el dato.
- **Stream time**: momento en que el sistema lo procesa.
- **Skew time**: diferencia entre ambos, debida a latencias.

Esta distinción es clave para interpretar correctamente los datos y sus retrasos.

### Ventanas de tiempo

Las **ventanas** son la unidad básica de procesamiento y permiten trabajar con subconjuntos finitos de datos:

- **Sliding windows**: ventanas solapadas definidas por longitud y desplazamiento, útiles para análisis continuos como medias móviles.
    
- **Tumbling windows**: ventanas no solapadas, basadas en tiempo o número de registros, donde cada dato se procesa una sola vez.
    
- **Sesiones**: ventanas delimitadas por períodos de inactividad del stream, típicas de comportamientos humanos o eventos intermitentes.

### Triggers

Los **triggers** determinan cuándo se actualizan los resultados:

- **Triggers recurrentes**: actualizan estadísticas de forma periódica o por llegada de eventos (con o sin retraso).
- **Triggers de completitud**: generan resultados finales cuando se considera que una ventana está completa.

### Watermarks

Las **watermarks** modelan el progreso del tiempo y permiten decidir hasta cuándo esperar eventos retrasados. Pueden ser:

- **Perfectas** (teóricas) o
- **Heurísticas** (prácticas), que introducen compromisos entre latencia, completitud y posibles pérdidas de datos.

Triggers y watermarks se combinan para equilibrar **actualización temprana** y **exactitud**.

## 3. Micro-batching vs. procesado continuo

El documento diferencia dos paradigmas:

- **Micro-batching**: los datos se procesan en pequeñas ventanas como si fueran batch (ej. Spark Streaming).
    
- **Procesado continuo**: los datos se procesan evento a evento, con actualizaciones incrementales (ej. Spark Structured Streaming).
    

El procesado continuo supone un cambio de paradigma más profundo, aunque sigue apoyándose en ventanas para agregaciones temporales.

## 4. Procesado continuo con Apache Structured Streaming

Se introduce la **dualidad stream–tabla**, donde:

- Un stream puede verse como un **changelog** de una tabla.
- Una tabla puede reconstruirse o emitirse como un stream.

Esta dualidad permite:

- **Transformaciones y agregaciones** similares a SQL, usando ventanas como agrupaciones temporales.
    
- **Joins**:
    
    - _Stream–tabla_, para enriquecer eventos con datos de referencia.
    - _Stream–stream_, limitados por ventanas temporales para evitar crecimiento infinito.

Esta aproximación facilita análisis complejos y en tiempo real sobre múltiples fuentes de datos.

## 5. Algoritmos en línea para datos en flujo

Se distinguen:

- **Algoritmos sin dependencia temporal**, que solo usan los datos de la ventana actual.
- **Algoritmos con memoria**, que dependen de resultados de ventanas anteriores (por ejemplo, medias móviles suavizadas).

La necesidad de memoria histórica introduce complejidad adicional en streaming.

## 6. Cálculo incremental de funciones matemáticas

Muchas estadísticas deben calcularse **de forma incremental**, evitando almacenar todos los datos:

- Estadísticos simples (mínimo, máximo, media) se mantienen con pocas variables.
- Estadísticos complejos (varianza, desviación estándar) requieren reformulación matemática.

Se presenta el **método de Welford** como ejemplo de algoritmo estable y eficiente para calcular la varianza en una sola pasada sobre el stream.

## 7. Técnicas de síntesis (aproximadas)

Cuando el volumen de datos impide cálculos exactos, se emplean técnicas probabilísticas:

- **Reservoir sampling**: mantiene una muestra aleatoria representativa de tamaño fijo del stream.
    
- **Conteo de elementos distintos**: estimación de cardinalidad mediante algoritmos como **HyperLogLog / HyperLogLog++**, basados en patrones de bits de funciones hash.
    
- **Frecuencia de elementos**: uso de **Count-Min Sketch** para estimar cuántas veces aparece un valor, proporcionando cotas superiores.
    
- **Membership**: comprobación aproximada de si un elemento ha aparecido antes (introduciendo estructuras como Bloom filters).

Estas técnicas sacrifican exactitud a cambio de **eficiencia, escalabilidad y uso acotado de memoria**.

---

# Procesado de datos en streaming con Spark Streaming, Structured Streaming y Storm

## Visión general del stream processing

El **procesamiento de datos en streaming** engloba las técnicas para capturar, procesar y almacenar datos continuos e ilimitados (_unbounded data_) en tiempo casi real. A diferencia del procesamiento por lotes (_batch_), los datos se analizan conforme llegan. El documento se centra en tres tecnologías principales: **Spark Streaming**, **Spark Structured Streaming** y **Apache Storm**, destacando que todas permiten ejecución distribuida en clústeres y procesamiento transparente para el usuario .

## Procesamiento de streaming con Spark

Apache Spark ofrece **dos enfoques distintos** para el streaming:

1. **Spark Streaming**
    - Basado en **RDDs** y el concepto de **micro-batching**.  
    - Usa la abstracción **DStream**, que representa un flujo como una secuencia de RDDs discretos.
        
2. **Spark Structured Streaming**
    - Basado en **DataFrames y Datasets (Spark SQL)**.  
    - Trata el stream como una **tabla ilimitada**, permitiendo consultas SQL continuas y garantías _exactly-once_ end-to-end.
        

Ambos enfoques permiten consumir datos desde múltiples fuentes (Kafka, sockets, ficheros, etc.), pero difieren en modelo conceptual, expresividad y semántica de consistencia .

## Spark Streaming

### Conceptos fundamentales

Spark Streaming procesa los datos en **intervalos temporales fijos** (micro-batches). Cada intervalo genera un RDD, y el conjunto forma un DStream. Este modelo convierte un flujo continuo en un flujo discreto procesable por el motor Spark.

### Fuentes de entrada

- **Básicas**: sockets, sistemas de archivos.
- **Avanzadas**: Kafka, Kinesis (requieren dependencias).
- **Configurables**: receptores personalizados (Java/Scala).

### Transformaciones y acciones

Ofrece operaciones similares a RDDs (`map`, `flatMap`, `filter`, `reduceByKey`, `join`, etc.) y funciones específicas:

- **Stateful**: `updateStateByKey` para mantener estado entre micro-batches.
- **transform**: permite aplicar operaciones RDD no nativas de DStreams.

### Ventanas

Soporta **ventanas temporales** (no por evento), configurando:

- Duración de la ventana.
- Desplazamiento (_slide interval_).  
    Permite ventanas deslizantes y fijas (_tumbling windows_).

### Persistencia y checkpointing

- **persist/cache**: optimizan rendimiento reutilizando datos.
- **checkpointing**: asegura tolerancia a fallos guardando metadatos y estado en almacenamiento confiable (como HDFS).

### Semántica de tolerancia a fallos

Spark Streaming garantiza:

- Transformaciones: _exactly once_.
- Entrada y salida: dependen de la fuente y del _sink_, pudiendo ser _at least once_ o _exactly once_ si se programa explícitamente.

### Ejemplo ilustrativo

Se presenta un **word count** clásico usando sockets y micro-batches de 1 segundo, mostrando versiones _stateless_ y _stateful_, y el uso de `netcat` como generador de datos .

## Spark Structured Streaming

### Modelo conceptual

Structured Streaming trata el stream como una **tabla infinita** a la que se añaden filas continuamente. Las consultas se definen igual que en Spark SQL y se ejecutan de forma incremental.

### Pipeline de procesamiento

1. Definir fuente de entrada.
2. Transformar los datos.
3. Definir salida (sink y modo).
4. Configurar triggers y checkpointing.
5. Iniciar la consulta.

### Fuentes de entrada

- **Ficheros** (JSON, CSV, Parquet, etc., con restricciones).
- **Kafka** (fuente principal en producción).
- **Socket** (solo para pruebas).
- **Rate** (generador sintético para testing y benchmarking).
### Modos de salida

- **Complete**: escribe la tabla completa.
- **Append**: solo nuevas filas.
- **Update**: solo filas modificadas.

### Transformaciones

- **Stateless**: `select`, `filter`, `map`.
- **Stateful**: `groupBy`, agregaciones y ventanas temporales.

### Ventanas con event time y watermarking

Structured Streaming permite:

- Procesar datos según **tiempo del evento (event time)**.
- Gestionar datos retrasados mediante **watermarks**, limitando cuánto tiempo se aceptan eventos tardíos para evitar consumo excesivo de recursos.

### Operaciones join

- **Stream-static**: joins con datasets estáticos.
- **Stream-stream**: joins entre flujos, usando condiciones temporales y watermarking para controlar el estado.

### Casos de uso

Aplicaciones típicas incluyen:

- Análisis de ventas en tiempo real.
- Comportamiento de clientes.
- Publicidad digital (impresiones y clics).
- Monitorización de sensores y temperaturas.
- Integración con MLlib para análisis predictivo en tiempo real.

Se muestra un caso completo con datos simulados usando `rate`, agregaciones y salida por consola .

## Apache Storm

### Características generales

Apache Storm es un sistema distribuido especializado en **streaming puro en tiempo real**, ampliamente usado en arquitecturas Lambda (capa _speed_).

### Arquitectura

- **Nimbus (master)**: distribuye código y gestiona fallos.
- **Supervisors (workers)**: ejecutan tareas.
- **ZooKeeper**: coordinación y estado del clúster.

### Modelo de datos

- **Spout**: fuente de datos.
- **Bolt**: unidad de procesamiento (filtrado, agregación, persistencia).
- **Topology**: DAG que conecta spouts y bolts.
- **Tuple**: unidad de datos inmutable.
- **Streams**: flujos de tuplas con distintos tipos de _grouping_ (shuffle, fields, global, all).

### Implementación

Storm permite varios lenguajes. El documento describe una implementación en **Python usando Flux**, donde la topología se define en YAML y los spouts/bolts se programan en Python. Se ejemplifica con un **contador de palabras** completo (spout → split bolt → count bolt) .

## Comparativa

- **Spark Streaming**: enfoque clásico basado en micro-batching; sencillo y consolidado, pero limitado a ventanas temporales.
    
- **Spark Structured Streaming**: modelo moderno, declarativo y unificado con Spark SQL, con soporte avanzado para _event time_, watermarking y _exactly-once_ end-to-end.
    
- **Apache Storm**: streaming nativo en tiempo real, altamente flexible y de baja latencia, pero con mayor complejidad de desarrollo


---

# La captura de datos en tiempo real con Kafka y Flume

El documento analiza en profundidad **Apache Flume** y **Apache Kafka**, dos herramientas fundamentales para la **captura, ingesta y preprocesamiento de datos en tiempo real**, ampliamente utilizadas en arquitecturas big data y sistemas distribuidos. Aunque ambas siguen el patrón de **publicación/suscripción**, están diseñadas con **objetivos distintos** y responden a **casos de uso diferentes**.

### Enfoque general y diferencias clave

- **Flume** forma parte del **ecosistema Hadoop** y está orientado a la **ingesta eficiente de grandes volúmenes de datos** (logs, redes sociales, ficheros, eventos) hacia sistemas como **HDFS, Hive o HBase**.
    
- **Kafka** es una **plataforma de streaming distribuida de propósito general**, muy robusta y escalable, utilizada tanto en entornos big data como en **arquitecturas de microservicios**, monitorización y procesamiento de eventos.

Las principales diferencias radican en:

- **Escalabilidad y concurrencia**: Kafka permite aumentar consumidores sin impacto significativo; en Flume esto requiere rediseñar la topología.
    
- **Tolerancia a fallos**: Kafka replica datos dentro del clúster; Flume no replica eventos internamente.
    
- **Uso principal**: Flume se emplea sobre todo para ingesta hacia Hadoop; Kafka para desacoplar sistemas y conectar múltiples productores y consumidores.


## Apache Flume

### Concepto y características

Flume surge inicialmente para la recogida de **logs**, pero evoluciona hacia una herramienta flexible para capturar datos en tiempo real desde múltiples fuentes. Sus principales ventajas son la **ingesta casi en tiempo real**, la **capacidad para grandes volúmenes de datos** y la **escalabilidad horizontal**. Como limitaciones, ofrece semántica **at-least-once** (puede haber duplicados) y su rendimiento depende en gran medida del tipo de canal utilizado.

### Arquitectura

Un **agente Flume** se compone de:

- **Sources (fuentes)**: reciben datos de sistemas externos (logs, Twitter, Kafka, comandos Unix).
    
- **Channels (canales)**: almacenamiento temporal que actúa como buffer.
    
- **Sinks (sumideros)**: escriben los datos en el sistema de destino (HDFS, Hive, HBase, Elasticsearch).
    

Un agente puede tener múltiples fuentes, canales y sinks, permitiendo flujos de datos complejos.

### Configuración

La definición de Flume se realiza mediante **ficheros de configuración** donde se especifican:

- El flujo (conexión entre fuentes, canales y sinks).
    
- Las propiedades individuales de cada componente.
    
- La posibilidad de múltiples flujos dentro de un mismo agente.
    

### Componentes principales

- **Sources**: destacan _Exec source_ (ejecución de comandos), _Kafka source_ (lectura desde topics Kafka) y _Twitter source_ (captura del 1 % del firehose).
    
- **Channels**:
    
    - _Memory channel_: mayor rendimiento, posible pérdida de datos.
        
    - _File channel_: persistencia en disco, mayor fiabilidad.
        
- **Sinks**: HDFS, Hive, HBase y Elasticsearch, cada uno adaptado a distintos sistemas de almacenamiento y análisis.

### Ejemplo práctico

El documento incluye un ejemplo completo donde Flume recibe mensajes por consola (netcat) y los almacena en HDFS, mostrando el flujo completo desde la fuente hasta el almacenamiento final.

## Apache Kafka

### Origen y objetivos

Kafka nace en LinkedIn para gestionar grandes volúmenes de eventos con **alta escalabilidad, baja latencia y tolerancia a fallos**. Evoluciona hasta convertirse en una plataforma central de streaming usada por grandes empresas.

### Definición

Kafka es una **plataforma distribuida de streaming** que permite **publicar, almacenar y consumir flujos de datos en tiempo real**, desacoplando productores y consumidores.

### Arquitectura y componentes

- **Brókeres**: servidores que almacenan y gestionan los datos.
- **Zookeeper**: coordina el clúster y mantiene la metainformación.
- **Producers**: generan y envían mensajes (clave–valor).
- **Topics**: categorías lógicas donde se almacenan los mensajes.
- **Partitions**: fragmentos de un topic que permiten paralelismo y escalabilidad
- **Consumers**: leen mensajes mediante un modelo pull.
- **Grupos de consumidores**: permiten escalar el consumo repartiendo particiones entre consumidores.

### Funcionamiento clave

- Los mensajes están **ordenados dentro de cada partición**, pero no a nivel global del topic.
- Kafka permite **múltiples consumidores** leyendo el mismo topic sin borrar mensajes.
- El control de lectura se basa en **offsets**, que indican hasta dónde ha leído cada consumidor

### Tolerancia a fallos

- **Producción**: configurada mediante ACKs (0, 1 o -1) para equilibrar fiabilidad y rendimiento.
- **Consumo**: uso de offsets y commits para evitar pérdidas, asumiendo posibles reprocesos.

### Configuración avanzada

Kafka ofrece opciones para:

- **Retención de datos** (por tiempo o tamaño).
- **Compactación de logs** (mantener el último valor por clave).
- **Cuotas** para limitar el uso de recursos.
- **Seguridad** mediante cifrado, autenticación, ACLs y auditoría.

### Ejemplos

Se muestran ejemplos de uso tanto por **línea de comandos** como mediante **APIs de programación**, ilustrando la creación de topics, productores y consumidores.

---
# **Capítulo 10 – Spark Streaming**

El capítulo introduce **Spark Streaming**, el módulo de Apache Spark diseñado para procesar datos que llegan continuamente en tiempo real. Permite escribir aplicaciones de streaming usando una API casi idéntica a la de los RDDs, lo que facilita reutilizar código y conocimientos de Spark batch.

## 1. **Concepto y Arquitectura de Spark Streaming**

Spark Streaming sigue una arquitectura de **micro-batch**, donde el flujo continuo se divide en pequeños lotes (batches) en intervalos regulares.
Cada lote se convierte en un **RDD**, procesado como cualquier otro trabajo de Spark.
El parámetro clave es el **batch interval**, normalmente entre 0.5 y varios segundos.

El tipo base de datos de Spark Streaming es el **DStream (Discretized Stream)**:

- Representa una secuencia de RDDs, uno por cada intervalo.
- Se crea desde fuentes externas (Kafka, Flume, sockets, HDFS).
- Soporta transformaciones y operaciones de salida.

La arquitectura interna incluye **receivers**, que ejecutan tareas de larga duración en los ejecutores para recibir y replicar datos de fuentes externas, manteniendo tolerancia a fallos mediante réplicas o checkpoints.

## 2. **Un Ejemplo Simple**

El capítulo inicia con un ejemplo donde:

1. Se crea un **StreamingContext** con batch interval de 1 segundo.
2. Se reciben líneas de texto desde un socket TCP en el puerto 7777.
3. Se filtran las líneas que contienen “error”.
4. Se imprimen las coincidencias.

El programa requiere llamar a:

- **start()** para iniciar el procesamiento
- **awaitTermination()** para mantener la aplicación activa

El ejemplo demuestra cómo Spark ejecuta pequeños trabajos Spark por cada intervalo.

## 3. **Transformaciones en DStreams**

Las transformaciones se clasifican en:

### ✔️ **3.1. Transformaciones estateless**

No dependen de batches previos.
Equivalen a transformaciones RDD clásicas:

- map
- filter
- flatMap
- reduceByKey
- join
- union

Ejemplo: contar visitas por IP en cada batch usando map + reduceByKey.

También existe **transform()**, que permite operar directamente sobre cada RDD del DStream, útil para reutilizar lógica batch.

### ✔️ **3.2. Transformaciones stateful**

Permiten incorporar historial entre batches. Requieren **checkpointing**.

#### **A) Ventanas (windowed operations)**

Procesan múltiples batches dentro de un periodo de tiempo definido:

- **window(windowDuration, slideDuration)**
- **reduceByWindow()**
- **reduceByKeyAndWindow()** (con opción de función inversa para eficiencia)
- **countByWindow()**, **countByValueAndWindow()**

Ejemplo: contar peticiones por IP en una ventana de 30 segundos y deslizamiento de 10 segundos.

#### **B) updateStateByKey()**

Mantiene estado arbitrario por clave entre batches.
Ejemplo: contar cuántas veces aparece cada código HTTP desde el inicio del programa.

## 4. **Output Operations**

Son las equivalentes a “acciones” en streaming y son necesarias para que Spark ejecute el flujo.

Incluyen:

- **print()** (debugging)
- **saveAsTextFiles()**, **saveAsHadoopFiles()**
- **foreachRDD()** (para escribir en bases de datos u otros sistemas externos)

foreachRDD es la operación más flexible, permitiendo abrir conexiones, escribir por partición, etc.

## 5. **Fuentes de Entrada (Input Sources)**

Spark Streaming soporta múltiples fuentes:

### ✔️ **Core sources**

- **Sockets (socketTextStream)**
- **Streams de archivos en HDFS o sistemas compatibles**

  - Deben llegar mediante operaciones atómicas (ej: mv)
  - Se pueden leer tanto texto como SequenceFiles

- **Akka actor streams**

### ✔️ **Fuentes adicionales (requieren artefactos extra)**

- **Apache Kafka**

  - Modo clásico: createStream(), basado en Zookeeper
  - Modo moderno: createDirectStream(), más eficiente y tolerante a fallos

- **Apache Flume**

  - Push-based receiver
  - Pull-based receiver (más seguro)

- Twitter
- Kinesis
- ZeroMQ
- Receivers personalizados

El capítulo detalla las configuraciones y diferencias de resiliencia entre cada receptor.

## 6. **Uso de Múltiples Receivers y Tamaño de Clúster**

Cada receiver ocupa un núcleo completo de un executor.
Reglas:

- Número de cores ≥ número de receivers + cores para el procesamiento.
- No usar _local[1]_, ya que no deja CPU libre para procesar datos.
- Puede combinar streams con union().

## 7. **Operación 24/7 y Tolerancia a Fallos**

Spark Streaming garantiza **exactly-once semantics** para los cálculos si los datos de entrada son fiables.

### ✔️ **7.1. Checkpointing**

Sirve para:

- Guardar estado periódicamente para reducir recomputación.
- Permitir reiniciar el driver sin perder progreso.

Debe configurarse con un directorio en HDFS/S3/NFS.

### ✔️ **7.2. Tolerancia a fallos del driver**

Se logra usando:

```scala
StreamingContext.getOrCreate(checkpointDir, createContext)
```

Combinado con supervisión externa o spark-submit con `--supervise`.

### ✔️ **7.3. Fallos de workers y receivers**

- Spark recompone RDDs mediante lineage.
- Los receivers se relanzan automáticamente.
- La robustez depende de la fuente (HDFS, Kafka directo = seguros).

Spark recomienda activar **write-ahead logs** para evitar pérdida de datos en fuentes no fiables.

## 8. **Garantías de Procesamiento**

Spark garantiza que cada dato influye exactamente una vez en los resultados computados.
Sin embargo, la escritura en sistemas externos puede duplicarse si se reintentan tareas → se recomienda usar operaciones **idempotentes** o transacciones.

## 9. **Streaming UI**

Spark provee un tab “Streaming” donde se puede ver:

- tasas de procesamiento
- tiempos por batch
- retraso de scheduling
- estado de los receivers

Es clave para identificar cuellos de botella.

## 10. **Consideraciones de Rendimiento**

### ✔️ Batch interval

- Mínimo recomendado: **500 ms**
- Comenzar con 10 s y reducir según carga

### ✔️ Paralelismo

Se puede aumentar mediante:

- más receivers
- repartition en DStreams
- más tareas en reduceByKey

### ✔️ Garbage collection

Se recomienda:

- Concurrent Mark-Sweep GC
- Kryo serialization
- Evicción con **spark.cleaner.ttl**

## 11. **Conclusión del Capítulo**

Spark Streaming permite procesar datos en tiempo real usando los mismos conceptos que Spark batch gracias a DStreams.
El manejo adecuado de ventanas, estado, fuentes de datos y checkpointing permite construir sistemas fiables y escalables. El siguiente capítulo se enfoca en Machine Learning con Spark.
