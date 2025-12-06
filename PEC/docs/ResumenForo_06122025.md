# 📌 **Resumen Organizado por Temáticas del Foro de Streaming (M5-ES)**

---

# 1️⃣ **Problemas con KafkaUtils y versión de PySpark**

## **Problema**

- Al ejecutar el ejercicio 10.1 aparece:
  **ModuleNotFoundError: No module named 'pyspark.streaming.kafka'**
- La librería KafkaUtils no existe en la versión instalada.
- No se reciben toots ni mensajes desde Kafka aunque el código parezca correcto.

## **Causa**

- El entorno usa Spark/PySpark en una versión donde **KafkaUtils ya no está soportado**.
- El notebook incluía imports y configuración incompatibles.
- El servicio que enviaba toots tenía interrupciones ocasionales.

## **Solución aplicada**

1. **Eliminar completamente:**

   ```python
   from pyspark.streaming.kafka import KafkaUtils
   ```

2. **Reemplazar el bloque completo de Kafka por un socket:**

   ```python
   socket_host = "localhost"
   socket_port = 9999
   kafkaStream = ssc.socketTextStream(socket_host, socket_port)
   ```

3. **Modificar la lectura de JSON:**

   ```python
   .map(lambda x: json.loads(x.strip()))
   ```

4. **Cambiar el kernel a:**
   **python 3.8 (Pyspark driver)**
5. **Ejecutar el script usando `python3.8` en lugar de `python3`**.

---

# 2️⃣ **Falta de importación de json**

## **Problema**

- Al ejecutar:

  ```python
  .map(lambda x: json.loads(x.strip()))
  ```

  se obtiene:
  **NameError: name 'json' is not defined**

## **Causa**

- Falta el import correspondiente.

## **Solución**

```python
import json
```

---

# 3️⃣ **Fallo al recibir toots desde el socket**

## **Problema**

- No se imprimen mensajes aunque el código esté bien.
- Alumnos ven outputs vacíos al hacer `kafkaStream.pprint()`.

## **Causa**

- Interrupciones temporales del servidor que emite toots.
- En ocasiones el flujo se detuvo o tardó en reanudarse.

## **Solución**

- Se reinició el servidor y posteriormente los toots comenzaron a llegar.
- Se recomendó volver a ejecutar el código tras unos minutos.

---

# 4️⃣ **Parada repentina de toots**

## **Problema**

- Los toots dejan de llegar de forma aleatoria, a pesar de que el código no cambia.

## **Causa**

- El servidor de emisión no siempre manda mensajes continuamente.
- Puede haber pausas normales en la actividad del stream.

## **Solución**

- Confirmación de que es comportamiento esperado.
- Se indicó volver a ejecutar más tarde y no asumir fallo del código.

---

# 5️⃣ **Error de conexión con el socket**

## **Problema**

Error:

```
Error connecting to localhost:9999 - java.net.ConnectException: Connection refused
```

## **Causa**

- El servidor de toots no estaba activo.
- El puerto 9999 no tenía un proceso escuchando.

## **Solución**

- Se reactivó el servicio en el backend.
- Tras ello, el error dejó de reproducirse.

---

# 6️⃣ **Problemas ejecutando comandos de Kafka en terminal**

## **Problema**

- Al ejecutar:

  ```
  !kafka-topics --create ...
  ```

  aparece:

  ```
  /bin/bash: kafka-topics: command not found
  ```

## **Causa**

- `kafka-topics` no está en el PATH del entorno.

## **Solución**

- Buscar la ruta real del script:

  ```bash
  !find / -name "kafka-topics*" 2>/dev/null
  ```

- Ejecutarlo con ruta completa:

  ```bash
  !/usr/bin/kafka-topics.sh --create --bootstrap-server ...
  ```

---

# 7️⃣ **Confirmación sobre cambios globales en el ejercicio 10**

## **Duda surgida**

- Si los cambios propuestos (socket en lugar de Kafka) debían aplicarse en todos los subejercicios del capítulo 10.

## **Respuesta**

- Sí, los cambios aplican **a todos los apartados** donde se usaba KafkaUtils.
- También es obligatorio usar el **nuevo kernel**.

---

# 8️⃣ **Consulta sobre la fecha de entrega**

## **Problema**

- En el anuncio la entrega aparecía cambiada al día 14.
- En la plataforma seguía mostrando fecha 10.

## **Solución**

- Se solicitó la actualización de la fecha de entrega (pendiente de confirmación).

---

# 9️⃣ **Problemas intermitentes del sistema (servicios caídos)**

## **Problema**

- Varios usuarios reportaron que el entorno dejó de funcionar durante un periodo.

## **Causa**

- Problema temporal del servidor de streaming.

## **Solución**

- Se reinició el sistema y volvió a funcionar.
- Se pidió volver a probar una vez resuelto.

---

# 📚 **Resumen Final**

El foro giró principalmente alrededor de **tres grandes problemas**:

1. **Incompatibilidad de KafkaUtils con la versión de Spark instalada**
   → Solucionado reemplazándolo por un socket y cambiando kernel.

2. **Errores de importación, conexión o lectura JSON**
   → Solucionado con ajustes simples en el código.

3. **Interrupciones temporales del servidor de toots o de los comandos Kafka**
   → Solucionado reiniciando servicios y usando rutas completas.

Además, hubo consultas sobre configuración, ejecución en terminal y la fecha de entrega.

---

Si quieres, puedo también:
📌 convertir esto en un documento PDF,
📌 preparar un resumen aún más breve,
📌 o generar una **FAQ oficial** para compartir con todos.
