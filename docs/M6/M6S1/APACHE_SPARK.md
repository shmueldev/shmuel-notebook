# Apache Spark en AWS EC2
En la indrustria se suele utilizar para procesar grandes volumenes de datos a una gran velocidad. A diferencia de otros sistemas de big data que guardaban registros de cada paso en el disco duro, Apache Spark hace casi todo desde la RAM, distribuyendo el trabajo en varios nodos. Por lo tanto, es un sistema distribuido para el procesamiento de grandes volúmenes de datos.

## 1. ¿Qué es y por qué se usa?
Es un motor de computación distribuida. Imagina que tienes un archivo de 1 TB; una sola computadora tardaría horas en leerlo. Spark divide ese archivo en 1,000 pedazos y los procesa en 10, 50 o 100 máquinas al mismo tiempo.
En vez de leer el archivo de 1 TB de golpe, Spark lo divide en trozos más pequeños (particiones) y envía cada trozo a una máquina diferente. Al procesar todo en paralelo, el resultado que antes tardaba horas, ahora sale en segundos.

* **¿Cuándo usarlo?** Cuando tus datos ya no caben en la RAM de una sola laptop o cuando procesos de Python (como Pandas) se quedan "congelados" por el volumen de información.

* **¿Cómo se utiliza?** Principalmente mediante scripts en Python (PySpark), Scala o SQL.

## 2. Conceptos Clave (Los Tipos)
1. **Estructuras de Datos**
* **RDD (Resilient Distributed Dataset)**: La base de Spark. Es de bajo nivel y difícil de usar, pero es lo que ocurre "bajo el capó".
    * **Ejemplo:** Imagina que tienes una lista de miles de frases y quieres contar cuántas veces aparece la palabra "error".
        * **RDD puro:** "Toma la frase 1, búscate la palabra. Toma la frase 2, búscate la palabra..." (Le dices paso a paso qué hacer). 
        * **Concepto:** Es como darle a un grupo de personas hojas sueltas con texto y pedirles: "Busquen la palabra, anoten un 1 si la ven, y luego súmenlos todos".
        * **Código (PySpark):**
            ```
            lineas = sc.textFile("log_servidor.txt")
            conteo = lineas.filter(lambda x: "error" in x).count()
            ```

* **DataFrames**: Es lo que usarás el 90% del tiempo. Es como una tabla de Excel o SQL distribuida. Es rápido porque Spark aplica una capa de optimización llamada Catalyst.
    * **Ejemplo:** Imagina la misma lista de errores, pero ahora es una tabla con columnas: fecha, servidor, mensaje.
        * **Concepto:** Es como una tabla de Excel. Puedes decir "Filtra la columna mensaje donde diga error". Es más inteligente porque Spark sabe qué hay en cada columna y optimiza la búsqueda.
        * **Código (PySpark):**
            ```
            df = spark.read.json("logs.json")
            df.filter(df.mensaje == "error").show()
            ```
* **Datasets**: Una mezcla de los dos anteriores (exclusivo de Scala/Java).

2. **Arquitectura (Cómo funciona)**
Spark funciona con un modelo de Maestro-Esclavo. Para entender cómo interactúan el Driver, los Executors y el Cluster Manager, imagina la cocina de un restaurante grande:
* **El Driver (El Chef Ejecutivo):** Él tiene la receta (tu código). No pica la cebolla, pero decide: "Tú (Obrero 1) pica 10 kilos de cebolla, tú (Obrero 2) cocina la carne". Si un obrero se corta, el Chef le da la tarea a otro.
* **El Cluster Manager (El Dueño del Restaurante):** Es quien decide cuántos cocineros hay en la cocina hoy. Si hay mucho trabajo, contrata más (Escalado). Si usas AWS, este sería YARN o el modo Standalone.
* **Los Executors (Los Cocineros/Obreros):** Son los que tienen el cuchillo y el fuego. Reciben órdenes del Chef, ejecutan la tarea en su estación de trabajo (RAM/CPU) y le devuelven el plato terminado.

## 3. Tipos de Procesamiento en Spark
Spark no es solo para tablas; tiene "módulos" especializados:
| Módulo           | Para qué sirve                                                                 |
|------------------|---------------------------------------------------------------------------------|
| Spark SQL        | Consultar datos usando lenguaje SQL.                                           |
| Spark Streaming  | Procesar datos en tiempo real (ej. tweets o transacciones bancarias al segundo).|
| MLlib            | Aprendizaje automático (Machine Learning) a gran escala.                       |
| GraphX           | Análisis de redes y grafos (ej. conexiones de amigos en Facebook).             |

### 1. Spark SQL (Análisis de Datos Estructurados)
Es el más común. Se usa para transformar datos crudos en información útil.
* **_Ejemplo:_** Una empresa de retail tiene millones de tickets de venta en archivos CSV.
* **_Uso:_** Usas Spark SQL para unir la tabla de "Ventas" con la de "Productos" y calcular: "¿Cuál fue el producto más vendido por ciudad en el último mes?". El resultado se guarda en tu base de datos final.

### 2. Spark Streaming (Datos en Tiempo Real)**
Ideal para procesos que no pueden esperar a mañana.
* **_Ejemplo:_** Detección de fraudes en tarjetas de crédito.
* **_Uso:_** Spark recibe un flujo constante de transacciones desde una herramienta como Kafka. Si detecta que una tarjeta se usó en dos países distintos en menos de 10 minutos, dispara una alerta en segundos para bloquear la cuenta.

### 3. MLlib (Machine Learning a Gran Escala)**
Cuando los datos son demasiados para las librerías normales de Python (como Scikit-Learn).
* **_Ejemplo:_** Sistema de recomendaciones de una plataforma de streaming (tipo Netflix).
* **_Uso:_** Spark analiza el historial de millones de usuarios simultáneamente para encontrar patrones y predecir: "Si al Usuario A le gustó la película X, hay un 90% de probabilidad de que le guste la película Y".

### 4. GraphX (Análisis de Redes y Grafos)
Se usa cuando el valor no está en el dato individual, sino en la conexión entre ellos.
* **_Ejemplo:_** Investigación de redes de lavado de dinero o propagación de virus.
* **_Uso:_** Analizas una red bancaria para identificar "comunidades" de cuentas que se transfieren dinero entre sí de forma circular. GraphX permite visualizar y calcular la importancia de cada "nodo" (cuenta) en esa red.

## Despliegue de spark en AWS EC2.