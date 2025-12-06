# Notas Resumidas sobre la Actividad 2: Streaming

## 1. Inicio de la Actividad 2

- La actividad 2 (Streaming) ya está disponible en **JupyterLab**.
- Se descarga igual que la actividad anterior.
- **Nueva fecha límite:** **14 de diciembre**.

## 2. Correcciones obligatorias en el Notebook

Debido a problemas de versiones y librerías, hay que modificar el código original.

### Cambios fundamentales

1. **Eliminar**:

   ```python
   from pyspark.streaming.kafka import KafkaUtils
   ```

2. **Sustituir bloque Kafka por socketTextStream**:

   ❌ Eliminar:

   ```python
   # Parámetros Kafka (...)
   kafkaStream = KafkaUtils.createDirectStream(...)
   ```

   ✅ Añadir:

   ```python
   socket_host = "localhost"
   socket_port = 9999
   kafkaStream = ssc.socketTextStream(socket_host, socket_port)
   ```

3. **Corregir parseo JSON**:
   ❌ Antes:

   ```python
   .map(lambda x: json.loads(x[1]))
   ```

   ✅ Ahora:

   ```python
   .map(lambda x: json.loads(x.strip()))
   ```

### 🔹 **Nuevo kernel**

- Usar el kernel:
  **python 3.8 (Pyspark driver)**
  para que todo funcione correctamente.

---

## **3. Correcciones adicionales para ejercicios 11.1, 11.2 y 11.3**

- **Eliminar** la línea:

  ```python
  .option("endingOffsets", "{\"" + kafka_topic + "\":{\"0\":10}}")
  ```
