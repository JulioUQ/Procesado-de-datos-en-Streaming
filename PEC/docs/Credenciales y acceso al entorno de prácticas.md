# 🔐 **Credenciales y acceso al entorno de prácticas**

Esta asignatura usa **un servidor de la UOC ya configurado con Spark, Hadoop, Kafka y otros componentes Big Data**.
No tienes que instalar nada en tu ordenador: **todo se hace en el servidor mediante JupyterLab**.

A continuación tienes cada parámetro explicado:

## 👤 **Usuario**

```
jubedaq
```

Tu nombre de usuario para acceder a los recursos del servidor.

## 🔑 **Contraseña**

```
oIZyBeS9
```

Tu contraseña personal para iniciar sesión.
**No la compartas con nadie**, ya que da acceso completo a tu entorno de trabajo.

# 🌐 **Acceso vía navegador (JupyterLab)**

### **URL para entrar a JupyterLab**

```
https://eimtcld3.uoclabs.uoc.es
```

Este es el portal web donde ejecutarás las prácticas.
Una vez dentro del navegador:

1. Accede a la URL.
2. Introduce tu **usuario** y **contraseña**.
3. Entrarás directamente en **JupyterLab**, donde están los notebooks de la asignatura.

### 👉 Este es el método principal de trabajo.

Aquí harás ejercicios de Spark, Streaming, análisis de datos, etc.

# 🖥️ **Acceso al servidor por SSH (opcional, solo si lo necesitas)**

En caso de que algún ejercicio o necesidad avanzada requiera conectarte al servidor por terminal:

## 🔗 **Servidor**

```
eimtcld.uoc.edu
```

## 🔌 **Puerto SSH**

```
55000
```

Esto permite conectarte mediante una terminal SSH:

```bash
ssh -p 55000 jubedaq@eimtcld.uoc.edu
```

## 🧱 **Puerto interno asignado**

```
12245
```

Este puerto se usa en algunos ejercicios (como Kafka o sockets locales).
Es **tu puerto exclusivo**, para que no interfieras con otros estudiantes.

**Ejemplo típico:**
Configurar un servicio escuchando en tu puerto:

```bash
nc -lk 12245
```

O conectarte desde Spark Streaming a:

```python
socket_host = "localhost"
socket_port = 12245
```

# 📘 **Guía rápida: cómo usar JupyterLab en esta asignatura**

JupyterLab será tu herramienta principal. Aquí tienes lo esencial:

## 🚀 1. **Entrar en JupyterLab**

1. Ve a la URL:

   ```
   https://eimtcld3.uoclabs.uoc.es
   ```

2. Inicia sesión con tu usuario y contraseña.
3. En la parte izquierda verás las carpetas de la asignatura.

## 📂 2. **Descargar / abrir la actividad**

Cada actividad está en un archivo `.ipynb`.

Para trabajar:

- Haz doble clic en el notebook → se abrirá como pestaña.
- Algunas actividades requieren descargar materiales ZIP (hay un botón en la interfaz).

## ⚡ 3. **Elegir el kernel correcto**

Para Spark / Streaming es importante seleccionar:

```
python 3.8 (Pyspark driver)
```

Lo encontrarás arriba a la derecha → botón **Kernel**.

## ▶️ 4. **Ejecutar celdas**

Para ejecutar:

- Selecciona una celda y pulsa:
  **Shift + Enter**
- O usa el botón “Run” ▶️ en la barra superior.

El output aparece justo debajo.

## 📦 5. **Acceso a la terminal dentro de JupyterLab**

Puedes abrir una terminal:

- Menú **File → New → Terminal**

Ahí puedes ejecutar:

- comandos de Linux
- comandos de Hadoop
- comandos de Kafka (con su ruta completa)
- scripts con Python 3.8
