# Soluciones y Explicaciones: Actividad Streaming

Este documento contiene las soluciones y explicaciones detalladas para el notebook `Actividad_Streaming_ES.ipynb`.

## Parte I: Introducción a Apache Kafka

En esta sección se utilizan comandos de terminal para interactuar con Kafka. En un entorno Jupyter, estos comandos se pueden ejecutar anteponiendo `!` a la celda de código, o directamente en una terminal.

### Ejercicio 1: Crear un tema con Kafka

**Objetivo:** Crear un tema llamado `activity2<usuario>` con factor de replicación 1, 1 partición y retención de 2 horas.

**Código:**

```bash
# Sustituye <usuario> por tu nombre de usuario real
!kafka-topics.sh --create \
  --topic activity2<usuario> \
  --bootstrap-server eimtcld3node1:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --config retention.ms=7200000
```

**Explicación:**

- `--create`: Indica que queremos crear un nuevo tema.
- `--topic`: Nombre del tema.
- `--bootstrap-server`: Dirección del servidor Kafka (broker).
- `--replication-factor 1`: Solo una copia de los datos.
- `--partitions 1`: El tema se divide en una sola partición.
- `--config retention.ms=7200000`: Configura el tiempo de retención de mensajes a 7,200,000 milisegundos (2 horas).

### Ejercicio 2: Lista los temas de Kafka

**Objetivo:** Verificar que el tema se ha creado correctamente.

**Código:**

```bash
!kafka-topics.sh --list --bootstrap-server eimtcld3node1:9092 | grep activity2<usuario>
```

**Explicación:**

- `--list`: Lista todos los temas disponibles en el servidor.
- `grep`: Filtra la salida para mostrar solo nuestro tema.

### Ejercicio 3: Borra el tema de Kafka

**Objetivo:** Eliminar el tema creado.

**Código:**

```bash
!kafka-topics.sh --delete --topic activity2<usuario> --bootstrap-server eimtcld3node1:9092
```

**Explicación:**

- `--delete`: Marca el tema para ser borrado.

### Ejercicio 4: Describe el tema de Kafka

**Objetivo:** Volver a crear el tema y describir sus detalles.

**Código:**

```bash
# Primero lo recreamos (mismo comando que Ejercicio 1)
!kafka-topics.sh --create --topic activity2<usuario> --bootstrap-server eimtcld3node1:9092 --replication-factor 1 --partitions 1 --config retention.ms=7200000

# Luego lo describimos
!kafka-topics.sh --describe --topic activity2<usuario> --bootstrap-server eimtcld3node1:9092
```

**Explicación:**

- `--describe`: Muestra detalles como el número de particiones, el factor de replicación y la configuración de los líderes y réplicas.

### Ejercicio 5: Crea un productor en Kafka

**Objetivo:** Enviar mensajes al tema desde la consola.

**Código (Terminal):**

```bash
kafka-console-producer.sh --topic activity2<usuario> --broker-list eimtcld3node1:9092
```

**Explicación:**

- Abre una sesión interactiva donde cada línea introducida se envía como un mensaje al tema especificado.

### Ejercicio 6: Crea un consumidor en Kafka

**Objetivo:** Leer los mensajes del tema desde la consola.

**Código (Terminal):**

```bash
kafka-console-consumer.sh --topic activity2<usuario> --bootstrap-server eimtcld3node1:9092 --from-beginning
```

**Explicación:**

- `--from-beginning`: Indica que el consumidor debe leer todos los mensajes presentes en el tema desde el principio, no solo los nuevos que lleguen después de iniciar el consumidor.

---

## Parte II: Ingesta de datos con Apache Kafka (Python)

### Ejercicio 7: Escribe un tema en Kafka

**Objetivo:** Usar `KafkaProducer` en Python para enviar 300 números binarios cada 3 segundos.

**Código:**

```python
from kafka import KafkaProducer
import numpy as np
import time

# Configuración del productor
producer = KafkaProducer(bootstrap_servers='eimtcld3node1:9092')
topic = 'activity2<usuario>'

for i in range(1, 301):
    # Clave: número como bytes
    key = str(i).encode('utf-8')
    # Valor: representación binaria del número
    value = bin(i).encode('utf-8')

    # Enviar mensaje
    producer.send(topic, key=key, value=value)

    # Esperar 3 segundos
    time.sleep(3)

producer.flush()
```

**Explicación:**

- Se inicializa el `KafkaProducer` apuntando al servidor bootstrap.
- Se itera del 1 al 300.
- `producer.send` envía el mensaje de forma asíncrona.
- `time.sleep(3)` introduce el retardo requerido.
- `producer.flush()` asegura que todos los mensajes en el buffer se envíen antes de terminar.

### Ejercicio 8: Leer un tema de Kafka

**Objetivo:** Usar `KafkaConsumer` en Python para leer los mensajes enviados.

**Código:**

```python
from kafka import KafkaConsumer

# Configuración del consumidor
consumer = KafkaConsumer(
    'activity2<usuario>',
    bootstrap_servers='eimtcld3node1:9092',
    auto_offset_reset='earliest', # Leer desde el principio si no hay offset guardado
    enable_auto_commit=True,
    group_id='my-group-<usuario>'
)

for message in consumer:
    # Mostrar solo el valor decodificado
    print(message.value.decode('utf-8'))
```

**Explicación:**

- `KafkaConsumer` se suscribe al tema.
- El bucle `for` itera indefinidamente sobre los mensajes recibidos.
- `message.value` contiene el payload del mensaje (el número binario).

---

## Parte III: Procesamiento de datos en tiempo real con Apache Spark Streaming

### Ejercicio 10.1: Counting in Windows

**Objetivo:** Contar toots originales (no retweets) cada 5 segundos.

**Código:**

```python
import findspark
findspark.init()

from pyspark import SparkContext
from pyspark.streaming import StreamingContext
from pyspark.streaming.kafka import KafkaUtils
import json

app_name = "TootCountApp"

# Inicializar contextos
try:
    sc = SparkContext("local[*]", appName=app_name)
except ValueError:
    sc.stop()
    sc = SparkContext("local[*]", appName=app_name)
sc.setLogLevel("ERROR")

batch_interval = 5
ssc = StreamingContext(sc, batch_interval)
ssc.checkpoint("checkpoint")

kafka_server = "eimtcld3node1:9092"
kafka_topic = "mastodon"
kafka_group = "group_<usuario>"

kafkaParams = {
    "metadata.broker.list": kafka_server,
    "group.id": kafka_group
}

kafkaStream = KafkaUtils.createDirectStream(ssc, [kafka_topic], kafkaParams)

# Procesamiento
tootCounts = kafkaStream\
    .map(lambda x: json.loads(x[1]))\
    .filter(lambda x: x.get('content') is not None and x.get('reblog') is None)\
    .count()

tootCounts.pprint()

try:
    ssc.start()
    ssc.awaitTermination()
except KeyboardInterrupt:
    ssc.stop()
    sc.stop()
```

**Explicación:**

- `filter`: Selecciona toots que tienen contenido y donde `reblog` es `None` (o vacío), indicando que es un toot original.
- `count()`: Cuenta el número de elementos en el RDD del batch actual (cada 5 segundos).

### Ejercicio 10.2: Contando Toots por Idioma

**Objetivo:** Top 10 idiomas por número de toots originales cada 5 segundos.

**Código:**

```python
# ... (Configuración inicial igual al anterior) ...

tootLangCounts = kafkaStream\
    .map(lambda x: json.loads(x[1]))\
    .filter(lambda x: x.get('reblog') is None)\
    .map(lambda x: (x.get('language'), 1))\
    .reduceByKey(lambda a, b: a + b)\
    .transform(lambda rdd: rdd.sortBy(lambda x: x[1], ascending=False))

tootLangCounts.pprint(10)

# ... (Arranque del contexto) ...
```

**Explicación:**

- Se mapea cada toot a un par `(idioma, 1)`.
- `reduceByKey` suma los unos por idioma.
- `transform` con `sortBy` ordena los resultados de mayor a menor frecuencia.
- `pprint(10)` muestra los top 10.

### Ejercicio 10.3: Manteniendo el Conteo (Stateful)

**Objetivo:** Conteo acumulado histórico de toots por idioma.

**Código:**

```python
# ... (Configuración inicial) ...

def updateFunction(newValues, runningCount):
    if runningCount is None:
        runningCount = 0
    return sum(newValues, runningCount)

tootCounts = kafkaStream\
    .map(lambda x: json.loads(x[1]))\
    .filter(lambda x: x.get('reblog') is None)\
    .map(lambda x: (x.get('language'), 1))\
    .updateStateByKey(updateFunction)\
    .transform(lambda rdd: rdd.sortBy(lambda x: x[1], ascending=False))

tootCounts.pprint()

# ... (Arranque) ...
```

**Explicación:**

- `updateStateByKey`: Permite mantener un estado (el conteo acumulado) a través de los batches.
- La función `updateFunction` suma los nuevos valores del batch actual al `runningCount` histórico.

### Ejercicio 10.4: Windowed Counting

**Objetivo:** Conteo en ventana deslizante (últimos 60 segundos, actualizado cada 5 segundos).

**Código:**

```python
# ... (Configuración inicial) ...

# Window duration: 60 seconds, Slide duration: 5 seconds
tootCounts = kafkaStream\
    .map(lambda x: json.loads(x[1]))\
    .filter(lambda x: x.get('reblog') is None)\
    .map(lambda x: (x.get('language'), 1))\
    .reduceByKeyAndWindow(lambda a, b: a + b, lambda a, b: a - b, 60, 5)\
    .transform(lambda rdd: rdd.sortBy(lambda x: x[1], ascending=False))

tootCounts.pprint(10)

# ... (Arranque) ...
```

**Explicación:**

- `reduceByKeyAndWindow`: Aplica la reducción sobre una ventana de tiempo.
- Se proporcionan dos funciones: una para sumar (cuando entran nuevos datos a la ventana) y otra para restar (cuando salen datos viejos de la ventana), lo cual es más eficiente.

### Ejercicio 10.5: Powering Up

**Objetivo:** Tabla compleja con estadísticas por idioma (conteo, longitud promedio, usuario más popular).

**Código:**

```python
# ... (Configuración inicial) ...

def combine_stats(a, b):
    # a y b son tuplas: (count, total_len, max_user, max_followers)
    count = a[0] + b[0]
    total_len = a[1] + b[1]
    if a[3] >= b[3]:
        max_user = a[2]
        max_foll = a[3]
    else:
        max_user = b[2]
        max_foll = b[3]
    return (count, total_len, max_user, max_foll)

# Mapeamos a: (idioma, (1, longitud, usuario, seguidores))
kafkaStream\
    .map(lambda x: json.loads(x[1]))\
    .filter(lambda x: x.get('reblog') is None)\
    .map(lambda x: (x.get('language'), (1, len(x.get('content', '')), x['account']['username'], x['account']['followers_count'])))\
    .window(60, 5)\
    .reduceByKey(combine_stats)\
    .map(lambda x: (x[0], x[1][0], x[1][1]/x[1][0], x[1][2], x[1][3]))\
    .transform(lambda rdd: rdd.sortBy(lambda x: x[1], ascending=False))\
    .pprint(10)

# ... (Arranque) ...
```

**Explicación:**

- Se extrae toda la información necesaria en el primer `map`.
- Se aplica `window(60, 5)` para agrupar los RDDs en la ventana deseada.
- `reduceByKey` combina las estadísticas: suma conteos y longitudes, y se queda con el usuario con más seguidores.
- El último `map` calcula el promedio (`total_len / count`) y formatea la salida.

---

## Parte IV: Structured Streaming

### Ejercicio 11.1: Getting the Schema

**Objetivo:** Inferir esquema de un lote y aplicarlo al stream para mostrar toots seleccionados.

**Código:**

```python
# ... (Imports y SparkSession) ...

kafka_topic = "mastodon"
kafka_bootstrap_servers = "eimtcld3node1:9092"

# Leer lote para inferencia
batch_df = spark \
    .read \
    .format("kafka") \
    .option("kafka.bootstrap.servers", kafka_bootstrap_servers) \
    .option("subscribe", kafka_topic) \
    .option("startingOffsets", "earliest") \
    .option("endingOffsets", "{\"" + kafka_topic + "\":{\"0\":10}}") \
    .load()

schema = spark.read.json(batch_df.selectExpr("CAST(value AS STRING)").rdd.map(lambda x: x[0])).schema

# Leer stream
toots = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", kafka_bootstrap_servers) \
    .option("subscribe", kafka_topic) \
    .option("startingOffsets", "latest") \
    .load()

toots_df = toots\
    .select(from_json(col("value").cast("string"), schema).alias("parsed_value"))\
    .filter(col("parsed_value.reblog").isNull())\
    .select(
        col("parsed_value.id").alias("id"),
        col("parsed_value.created_at").alias("created_at"),
        col("parsed_value.content").alias("content"),
        col("parsed_value.language").alias("language"),
        col("parsed_value.account.username").alias("username"),
        col("parsed_value.account.followers_count").alias("followers_count")
    )

query = toots_df \
        .writeStream \
        .outputMode("append") \
        .format("console")\
        .start()

query.awaitTermination()
```

**Explicación:**

- Se infiere el esquema leyendo los primeros 10 mensajes.
- Se filtra `reblog` nulo para obtener originales.
- Se seleccionan los campos anidados usando la notación de punto.
- `outputMode("append")`: Adecuado para mostrar cada nuevo registro que llega.

### Ejercicio 11.2: Agregando Datos desde un Flujo

**Objetivo:** Conteo acumulado global por idioma.

**Código:**

```python
# ... (Setup) ...

toots_df = toots\
    .select(from_json(col("value").cast("string"), schema).alias("parsed_value"))\
    .filter(col("parsed_value.reblog").isNull())\
    .select(col("parsed_value.language").alias("language"))\
    .groupBy("language")\
    .count()\
    .orderBy("count", ascending=False)

query = toots_df \
        .writeStream \
        .outputMode("complete") \
        .format("console")\
        .trigger(processingTime='10 seconds')\
        .start()
```

**Explicación:**

- `groupBy("language").count()`: Agregación global.
- `outputMode("complete")`: Necesario cuando hay agregaciones sin marcas de agua (watermarks) o cuando queremos ver la tabla completa actualizada (incluyendo el ordenamiento).
- `trigger(processingTime='10 seconds')`: Controla el intervalo de actualización.

### Ejercicio 11.3: Windowed Counting (Structured)

**Objetivo:** Conteo por idioma en ventanas de 1 minuto, sliding cada 5s.

**Código:**

```python
# ... (Setup) ...

toots_df = toots\
    .select(from_json(col("value").cast("string"), schema).alias("parsed_value"))\
    .filter(col("parsed_value.reblog").isNull())\
    .select(
        col("parsed_value.language").alias("language"),
        to_timestamp(col("parsed_value.created_at"), "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'").alias("timestamp") # Ajustar formato fecha si es necesario
    )\
    .groupBy(
        window(col("timestamp"), "1 minute", "5 seconds"),
        col("language")
    )\
    .count()\
    .orderBy("window", "count", ascending=False)

query = toots_df \
        .writeStream \
        .outputMode("complete") \
        .format("console")\
        .option("truncate", "false")\
        .trigger(processingTime='5 seconds')\
        .start()
```

**Explicación:**

- Es crucial convertir el campo de tiempo a tipo `Timestamp` para usar `window`.
- `window(...)` crea la columna de ventana.
- Se agrupa por ventana e idioma.

### Ejercicio 11.4: Unir Flujos

**Objetivo:** Unir stream de originales con stream de retweets.

**Código:**

```python
# ... (Setup) ...

# Esquema manual para los topics pre-agregados
schema = StructType([
    StructField("window", StructType([
        StructField("start", StringType()),
        StructField("end", StringType())
    ])),
    StructField("mastodon_instance", StringType()),
    StructField("count", IntegerType())
])

toots_original_topic = "mastodon_toots_original_domain"
toots_retoot_topic = "mastodon_toots_retoot_domain"

# Leer Stream Originales
toots_original = spark.readStream.format("kafka")...load()
toots_original_df = toots_original\
    .select(from_json(col("value").cast("string"), schema).alias("data"))\
    .select(
        col("data.window.start").alias("window_start"),
        col("data.window.end").alias("window_end"),
        col("data.mastodon_instance"),
        col("data.count").alias("original_count")
    )

# Leer Stream Retweets
toots_retoot = spark.readStream.format("kafka")...load()
toots_retoot_df = toots_retoot\
    .select(from_json(col("value").cast("string"), schema).alias("data"))\
    .select(
        col("data.window.start").alias("window_start"),
        col("data.window.end").alias("window_end"),
        col("data.mastodon_instance"),
        col("data.count").alias("retweet_count")
    )

# Join
# Nota: Para stream-stream joins, idealmente se necesitan watermarks y condiciones de tiempo.
# Dado que los datos ya vienen "ventaneados" en el payload, hacemos un join por igualdad de campos.
toots_join_df = toots_original_df.join(
    toots_retoot_df,
    ["window_start", "window_end", "mastodon_instance"],
    "left_outer"
)

query = toots_join_df\
        .writeStream \
        .outputMode("append") \
        .format("console")\
        .option("numRows", 100)\
        .start()
```

**Explicación:**

- Se definen los esquemas manualmente.
- Se leen ambos streams y se extraen los campos planos.
- Se realiza un `left_outer` join usando los campos comunes (`window_start`, `window_end`, `mastodon_instance`) como claves.
- `outputMode("append")` es típico para resultados de joins, aunque puede requerir watermarks si se quiere limpiar el estado. En este ejercicio simplificado, asumimos que los datos llegan alineados o aceptamos la latencia del estado.
