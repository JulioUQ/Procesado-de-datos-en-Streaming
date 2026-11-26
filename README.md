<div align="center">

![Ingeniería De Datos](https://img.shields.io/badge/INGENIER%C3%8DA_DE_DATOS-EF6C00?style=for-the-badge)

# Procesamiento de datos en streaming

Captura y análisis de datos en tiempo real con Apache Kafka, Spark Streaming y Structured Streaming, con la teoría de las arquitecturas Lambda y Kappa.

![Apache Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=flat-square&logo=apachekafka&logoColor=white) ![Apache Spark](https://img.shields.io/badge/Apache_Spark-E25A1C?style=flat-square&logo=apachespark&logoColor=white) ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) ![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=flat-square&logo=jupyter&logoColor=white)

<img src="Visualizaciones/Toots%20acumulado%20por%20idioma%20por%20ventanas%20de%2060%20segs.png" alt="Gráfico de publicaciones acumuladas por idioma en ventanas de 60 segundos, calculado con Spark Streaming" width="100%" />

<sub>Publicaciones acumuladas por idioma en ventanas de 60 segundos.</sub>

</div>

---

## Descripción general

Proyecto práctico de procesamiento en flujo: se crean temas de Kafka, se producen y consumen mensajes y se procesan datos en tiempo real con Spark Streaming y Structured Streaming, agregando por ventanas de tiempo. Incluye la teoría de captura de datos en flujo, las arquitecturas Lambda y Kappa y los algoritmos para datos en streaming.

## Contenido

| Parte | Contenido |
| --- | --- |
| Kafka | Creación, listado, descripción y borrado de temas; productores y consumidores; ingesta y lectura de temas |
| Spark Streaming | Procesamiento por micro-lotes y agregaciones por ventanas |
| Structured Streaming | Consultas continuas sobre flujos |
| [Visualizaciones](Visualizaciones) | Resultados de los ejercicios: mensajes por consola, lotes y agregados por idioma |
| [Teoría](Teoria) | Kafka y Flume, patrones de captura de datos, algoritmos para streaming, Spark Streaming y Storm |

## Tecnologías y herramientas

| Ámbito | Herramientas |
| --- | --- |
| Mensajería | Apache Kafka, Apache Flume |
| Procesamiento | Spark Streaming, Structured Streaming |
| Entorno | Jupyter sobre un clúster remoto |

## Instalación y uso

El cuaderno está en `PEC/` y se ejecutó en un servidor con Kafka y Spark. Para revisarlo basta con Jupyter; para ejecutarlo hace falta un entorno con Kafka y PySpark.

## Aspectos técnicos destacados

- Del productor al resultado: ingesta en Kafka, procesamiento en Spark y agregados por ventanas de tiempo.
- Comparación entre Spark Streaming y Structured Streaming.

## Autor

**Julio Úbeda Quesada** · Data Scientist

[![Portfolio](https://img.shields.io/badge/Portfolio-0B2545?style=flat-square&logo=astro&logoColor=white)](https://juliouq.github.io/Portfolio-Web---JulioUQ/es/) [![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square)](https://www.linkedin.com/in/juliouq/) [![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/JulioUQ)

<sub>Material elaborado durante el Máster Universitario en Ciencia de Datos.</sub>
