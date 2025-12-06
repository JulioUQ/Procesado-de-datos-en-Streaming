# **Capítulo 10 – Spark Streaming**

El capítulo introduce **Spark Streaming**, el módulo de Apache Spark diseñado para procesar datos que llegan continuamente en tiempo real. Permite escribir aplicaciones de streaming usando una API casi idéntica a la de los RDDs, lo que facilita reutilizar código y conocimientos de Spark batch.

---

## 1. **Concepto y Arquitectura de Spark Streaming**

Spark Streaming sigue una arquitectura de **micro-batch**, donde el flujo continuo se divide en pequeños lotes (batches) en intervalos regulares.
Cada lote se convierte en un **RDD**, procesado como cualquier otro trabajo de Spark.
El parámetro clave es el **batch interval**, normalmente entre 0.5 y varios segundos.

El tipo base de datos de Spark Streaming es el **DStream (Discretized Stream)**:

- Representa una secuencia de RDDs, uno por cada intervalo.
- Se crea desde fuentes externas (Kafka, Flume, sockets, HDFS).
- Soporta transformaciones y operaciones de salida.

La arquitectura interna incluye **receivers**, que ejecutan tareas de larga duración en los ejecutores para recibir y replicar datos de fuentes externas, manteniendo tolerancia a fallos mediante réplicas o checkpoints.

---

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

---

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

---

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

---

## 4. **Output Operations**

Son las equivalentes a “acciones” en streaming y son necesarias para que Spark ejecute el flujo.

Incluyen:

- **print()** (debugging)
- **saveAsTextFiles()**, **saveAsHadoopFiles()**
- **foreachRDD()** (para escribir en bases de datos u otros sistemas externos)

foreachRDD es la operación más flexible, permitiendo abrir conexiones, escribir por partición, etc.

---

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

---

## 6. **Uso de Múltiples Receivers y Tamaño de Clúster**

Cada receiver ocupa un núcleo completo de un executor.
Reglas:

- Número de cores ≥ número de receivers + cores para el procesamiento.
- No usar _local[1]_, ya que no deja CPU libre para procesar datos.
- Puede combinar streams con union().

---

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

---

## 8. **Garantías de Procesamiento**

Spark garantiza que cada dato influye exactamente una vez en los resultados computados.
Sin embargo, la escritura en sistemas externos puede duplicarse si se reintentan tareas → se recomienda usar operaciones **idempotentes** o transacciones.

---

## 9. **Streaming UI**

Spark provee un tab “Streaming” donde se puede ver:

- tasas de procesamiento
- tiempos por batch
- retraso de scheduling
- estado de los receivers

Es clave para identificar cuellos de botella.

---

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

---

## 11. **Conclusión del Capítulo**

Spark Streaming permite procesar datos en tiempo real usando los mismos conceptos que Spark batch gracias a DStreams.
El manejo adecuado de ventanas, estado, fuentes de datos y checkpointing permite construir sistemas fiables y escalables. El siguiente capítulo se enfoca en Machine Learning con Spark.
