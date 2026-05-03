# DAGS APACHE-AIRFLOW
**_Es un plan maestro que le dice al sistema:_** _"Ejecuta estas tareas en este orden específico, asegúrate de que no se queden dando vueltas en círculo y muéstrame el mapa visual para saber si todo esta bien"_

## 1. ¿Qué es un Dag (Grafo Acíclico Dirigido)?
Es el concepto fundamental en `Apache-Airflow` sirve para definir y organizar flujos de trabajo. En palabras simples, es el plano o mapa que le indica a nuestro cluster que tareas debe ejecutar, en que orden y bajo que condiciones.

## 2. ¿Qué significa exactamente DAG?
**1. Dirigido (Directed): El sentido único** Esto significa que los datos en airflow siempre avanzan
* **¿Por qué importa?:** si tenemos un la Tarea A (extraer datos) y la Tarea B (limpiar datos) Airflow garantiza que B nuenca empiece antes que A. Hay una jerarquía clara de **padres he hijos.**
* **En tu cluster:** Esto nos permite saber exactamente en qué punto de la tubería se quedó un proceso si algo falla.

**2. Acíclico (Acyclic): El "No al infinito"**
Esta prohibido que una tarea vuelva a una tarea anterior:
* **El problema de los bucles:** Si la Tarea C enviara de vuelta a la Tarea A, crearías un bucle infinito que consumiría todos los recursos de tu cluster y nunca terminaría.
* **Cómo se soluciona:** Si necesitas repetir algo (como procesar 10 archivos), no haces un círculo, sino que creas 10 tareas en paralelo o una tras otra en línea recta.

**3. Grafo (Graph): El mapa de conexiones**
En matemáticas y computación, un grafo es simplemente un conjunto de puntos unidos por líneas.
* **Nodos (las tareas)** Son los puntos del mapa, Ejemplo: "Enviar email", "Ejecutar Query SQL"
* **Aristas (Las Flechas):** Son las líneas que unen a los nodos. No llevan datos por sí mismas, representan la lógica de dependencias.
* **Visualización:** La mayor ventaja de que sea un "Grafo" es que Airflow puede dibujarlo. Entras a la interfaz web y ves un diagrama de flujo real, lo que facilita explicarle el proceso a alguien que no sabe programar.

## 3. Elementos principales de un DAG en Airflow
**1. Tareas (Tasks):** Son la unidades básicas de ejecución (nodos). Pueden ser desde ejecutar archivos Python o subir archivos a la nube.

**2. Operadores (Operators)** Son las plantilla que indica que hace la tarea por ejemplo, `PythonOperator`, `BashOperator`

**3. Dependencias** Son las que definen el orden de ejecución. Se establecen mediante operadores de bits como `>>>` indicando que la tarea B solo inicia con la tarea A termina con éxito.
* **Flujo Lineal:** tarea_1 >> tarea_2 >> tarea_3 (1 después de 2, 2 después de 3).

* **Flujo en Abanico (Fan-out):** tarea_inicial >> [tarea_correo, tarea_base_datos] (Una tarea dispara dos procesos al mismo tiempo).

* **Por qué es vital:** Si la tarea_1 falla, Airflow marca automáticamente las siguientes como "Upstream Failed", deteniendo el proceso para no generar datos corruptos.

**4. Programación (Schedule)** Define cuando debe correr un DAG.

**5. Ejemplo sencillo**
```py
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator
from datetime import datetime

# 1. Definimos una función de Python que usará nuestro PythonOperator
def saludar():
    print("¡Hola! La tarea de Python se ejecutó correctamente en el cluster.")

# 2. Configuramos el DAG (el contenedor principal)
with DAG(
    dag_id='mi_primer_dag_basico',    # El nombre que verás en la interfaz de Airflow
    start_date=datetime(2023, 1, 1), # Cuándo empieza a ser válido (histórico)
    schedule_interval='@daily',      # Se ejecutará una vez al día
    catchup=False                    # No ejecutes los días pasados desde 2023
) as dag:

    # 3. Definimos la primera tarea usando BashOperator
    tarea_crear_carpeta = BashOperator(
        task_id='crear_directorio_temporal',
        bash_command='mkdir -p /tmp/mi_carpeta_airflow'
    )

    # 4. Definimos la segunda tarea usando PythonOperator
    tarea_saludar_python = PythonOperator(
        task_id='ejecutar_saludo_python',
        python_callable=saludar  # Aquí llamamos a la función definida arriba
    )

    # 5. Establecemos la dependencia (El Orden)
    # Primero se crea la carpeta, luego se ejecuta el saludo.
    tarea_crear_carpeta >> tarea_saludar_python
```
* **Conceptos clave que acabas de ver:** 
    1. **Importaciones**

        * **from airflow import DAG:** Importas el "contenedor". Sin esto, no puedes crear el objeto que agrupa tus tareas. Es el esqueleto del flujo.

        * **from airflow.operators.bash import BashOperator:** Traes la capacidad de hablar con la terminal (Linux). Si no la importas, Airflow no sabe cómo ejecutar comandos de consola.from airflow.operators.python import

        * **PythonOperator:** Traes la capacidad de ejecutar funciones de Python.

        * **from datetime import datetime:** Airflow vive del tiempo. Necesitas esto para decirle exactamente cuándo debe nacer tu flujo de trabajo.
    
    2. **El bloque with DAG(...) as dag:**
        En resumen, el with sirve como un "contenedor" inteligente que le dice a Airflow: "Todo lo que pongas aquí dentro pertenece a este flujo de trabajo y debe seguir estas reglas generales"

        * **dag_id:** Es el DNI de tu flujo. No puede haber dos iguales en el cluster.
        
        * **start_date:** Es la fecha de "nacimiento". Airflow no ejecutará nada que esté programado antes de esa fecha.
        
        * **schedule_interval:** Es el ritmo cardíaco. @daily (diario), @hourly (cada hora) o None (si solo quieres activarlo manualmente).

        * **catchup=False:** ¡Este es vital! Si pones una fecha de 2023 y catchup es True, Airflow intentará ejecutar inmediatamente cientos de veces el DAG para "ponerse al día" desde 2023 hasta hoy. Con False, ignoras el pasado y empiezas desde ho

    3. **Otros temas claves**

        * **dag_id:** Es el identificador único. Si tienes dos archivos con el mismo dag_id, Airflow se confundirá.

        * **task_id:** Cada tarea dentro del grafo debe tener su propio nombre. Así sabrás cuál falló en la gráfica.

        * **bash_command:** Es literalmente lo que escribirías en una terminal de Linux.python_callable: Es el nombre de la función de Python que quieres que Airflow ejecute.>>: Este símbolo es el que "dibuja" la flecha en el grafo. Dice: "espera a que termine la izquierda para empezar la derecha".

## 6. Propiedades estrellas (Las utilizaremos el 90% del tiempo)
1. PythonOperator (El más versátil)Además de python_callable, estas son sus claves:

    * **op_args:** Sirve para pasar una lista de argumentos a tu función.
        * **_Ejemplo:_** op_args=['dato1', 'dato2'].

    * **op_kwargs:** Pasa un diccionario de argumentos (muy usado para configurar parámetros).
        * **_Ejemplo:_** op_kwargs={'usuario': 'admin'}.

    * **templates_dict:** Permite pasar datos dinámicos (como fechas de ejecución) que Airflow calcula en el momento.

2. Propiedades comunes a TODOS los OperadoresCualquier operador (Bash, Python, SQL, etc.) hereda estas propiedades de la clase BaseOperator. Son las más importantes para la estabilidad:

    * **retries:** Cuántas veces debe reintentar la tarea si falla (por ejemplo, si se cae el internet un segundo).
        **Ejemplo:** retries=3.retry_delay: Cuánto tiempo esperar entre reintentos.Uso: retry_delay=timedelta(minutes=5).
        
    * **trigger_rule:** Define cuándo debe iniciar la tarea. Por defecto es "si todos los padres tuvieron éxito", pero puedes cambiarlo a "aunque el padre falle" (all_done).
    
    * **pool:** Limita cuántas tareas de este tipo pueden correr a la vez. Muy útil para no saturar tu base de datos o el cluster.

3. BashOperator (Para comandos de sistema)
    * **bash_command:** El comando real. Tip clave: Si termina en .sh, Airflow intentará ejecutar el archivo.
    * **env:** Un diccionario de variables de entorno que solo existirán mientras el comando corre.

4. EmailOperator (Para avisos)
    * **to:** Lista de correos.
    * **subject:** El asunto. Lo genial es que puedes usar Jinja Templates para que el asunto diga: "Fallo el DAG: {{ dag.dag_id }} el día {{ ds }}".
    
5. Sensores (Un tipo especial de operador) No ejecutan lógica, solo esperan.
    * **poke_interval:** Cada cuánto tiempo (en segundos) debe revisar si ya sucedió lo que espera (ej. si llegó un archivo).
    * **timeout:** Tiempo máximo de espera antes de marcar la tarea como fallida.

6. Ejemplo Practico
```bash
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

# 1. Nuestra función ahora recibe argumentos (nombre y rol)
def procesar_usuario(nombre, rol):
    print(f"Procesando al usuario: {nombre} con el rol de: {rol}")
    # Imaginemos que aquí hay una lógica que podría fallar
    # Airflow atrapará cualquier error y activará los 'retries'

with DAG(
    dag_id='dag_con_propiedades_clave',
    start_date=datetime(2023, 5, 1),
    schedule_interval='@daily',
    catchup=False,
    # Estas propiedades se aplican a TODAS las tareas del DAG por defecto
    default_args={
        'retries': 3,                          # Si falla, reintenta 3 veces
        'retry_delay': timedelta(minutes=5),   # Espera 5 min entre reintentos
    }
) as dag:

    # 2. Tarea usando 'op_args' (Pasa los datos como una lista)
    tarea_con_args = PythonOperator(
        task_id='tarea_usando_args',
        python_callable=procesar_usuario,
        op_args=['Juan Perez', 'Administrador'] # Se mapean en orden a la función
    )

    # 3. Tarea usando 'op_kwargs' (Pasa los datos como un diccionario)
    # Esto es más claro porque etiquetas cada dato
    tarea_con_kwargs = PythonOperator(
        task_id='tarea_usando_kwargs',
        python_callable=procesar_usuario,
        op_kwargs={'nombre': 'Maria Lopez', 'rol': 'Analista'},
        # Podemos sobrescribir los valores del DAG solo para esta tarea:
        retries=5, 
        retry_delay=timedelta(seconds=30)
    )

    # Orden de ejecución
    tarea_con_args >> tarea_con_kwargs
```

## 7. ¿Qué diferencias tiene con un archivo de pyhton?
Aunque un dag se escribe en en el lenguaje Python, su prorpósito y comportamiento son muy diferentes a los de un script de Python.

**Tabla de comparación**
## Diferencias clave: DAG vs. Script de Python común

| Característica       | Script de Python convencional                                      | DAG de Airflow (.py)                                                                 |
|---------------------|--------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| Propósito           | Ejecutar una lógica lineal o procesamiento de datos inmediato.     | Orquestar tareas: definir cuándo y en qué orden corren.                              |
| Ejecución           | Se ejecuta de principio a fin al invocar `python script.py`.       | El Scheduler lo lee constantemente solo para entender su estructura (parseo).        |
| Control de Fallos   | Si falla a la mitad, el proceso muere y debes reiniciarlo manualmente. | Tiene reintentos automáticos (retries) y alertas configurables.                      |
| Dependencias        | Las funciones se llaman una tras otra en el código.                | Las dependencias son explícitas (ej. `tarea_1 >> tarea_2`) y visibles en una UI.     |
| Contexto            | Corre en tu máquina o servidor actual.                             | Puede ejecutar tareas en diferentes workers o máquinas en paralelo.                  |

## 8. ¿Qué es exactamente un XCom?
**Xcom** Cross-Communication" (Comunicación Cruzada). Es un sistema que permite a las tareas enviarse pequeños mensajes entre si.

1. **¿Por qué existe? (El aislamiento)**
En nuestro cluster, la tarea A puede ejecutarse en una computadora (worker) y la tarea B en otra totalmente distinta, como no comparten memoria comparten variables, los XComs funcionan como un buzón de correos centralizado (normalmente en la base de datos de Airflow):
* **_Tarea A (Remitente):_** "Empuja" (push) un dato al buzón.
* **_Tarea B (Destinatario):_** "Jala" (pull) ese dato del buzón para usarlo.

2. **Ejemplo práctico:** Pasando un ID de una tarea a otraImagínate que la primera tarea crea un usuario y la segunda le envía un correo usando el ID generado:pythondef crear_usuario_en_db():

```bash
# ... lógica para crear usuario ...
id_generado = 12345
return id_generado  # Al hacer 'return', Airflow lo guarda automáticamente en XCom

def enviar_correo_bienvenida(ti):
    # 'ti' es el Task Instance, necesario para jalar datos
    id_para_el_correo = ti.xcom_pull(task_ids='crear_usuario')
    print(f"Enviando correo al usuario con ID: {id_para_el_correo}")

# En el DAG:
t1 = PythonOperator(task_id='crear_usuario', python_callable=crear_usuario_en_db)
t2 = PythonOperator(task_id='enviar_correo', python_callable=enviar_correo_bienvenida)

t1 >> t2
```

3. **Reglas de oro de los XComs:**

* **_Solo para datos pequeños:_** Úsalos para IDs, nombres de archivos, fechas o pequeñas configuraciones. Nunca los uses para pasar tablas de datos gigantes o DataFrames de Pandas, porque saturarás la base de datos del cluster.

* **_El "Return" automático:_** En un PythonOperator, cualquier valor que devuelvas con la palabra return se convierte automáticamente en un XCom.

**_Persistencia:_** Los datos se quedan guardados en la base de datos de Airflow hasta que limpies la ejecución, lo que te permite auditar qué datos se pasaron entre tareas.

## 9. ¿Cómo se crea?
En Airflow, los DAGS se escriben exclusivamente como **código python**. El scheduler lee nuestros archivos .py en una carpeta especifica y genera el grafo automáticamente en la interfaz web.

## 10. Ejemplo con ETL

1. **Ejemplo Básico de un ETL dentro de un DAG ¿Por qué esta es la manera profesional?**

* **Aislamiento de Memoria:** Si el proceso de EDA consume mucha RAM, solo afectará al worker que lo corre. Como el DataFrame no vive en Airflow, no pones en riesgo el cluster.

* **Uso de Parquet:** En clusters, usar .parquet en lugar de .csv es mejor porque conserva los tipos de datos (las fechas siguen siendo fechas) y el archivo pesa mucho menos.XCom Liviano: Solo pasas un "String" (la ruta /tmp/...). Esto mantiene la base de datos de Airflow rápida y limpia.

* **Tolerancia a fallos:** Si la tarea de EDA falla, el archivo limpio ya está guardado en el disco. Puedes reintentar solo el EDA sin tener que repetir todo el ETL.

```bash
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime
import pandas as pd
import os

# CONFIGURACIÓN DE RUTAS (Escalable: puedes cambiar esto por rutas de la nube como s3://...)
BASE_PATH = "/tmp/airflow_data"
RAW_FILE = f"{BASE_PATH}/raw_data.csv"
CLEAN_FILE = f"{BASE_PATH}/clean_data.parquet" # Parquet es mejor que CSV para clusters

# 1. Función de ETL (Extracción y Limpieza)
def ejecutar_etl():
    if not os.path.exists(BASE_PATH):
        os.makedirs(BASE_PATH)
    
    # Simulación de carga
    df = pd.read_csv(RAW_FILE)
    
    # Lógica de limpieza (ETL)
    df_clean = df.dropna().drop_duplicates()
    
    # GUARDADO INTERMEDIO: La clave de la escalabilidad
    df_clean.to_parquet(CLEAN_FILE)
    return CLEAN_FILE  # Pasamos la RUTA por XCom, no los datos

# 2. Función de EDA (Análisis)
def ejecutar_eda(ti):
    # Recuperamos la RUTA del archivo que guardó la tarea anterior
    ruta_archivo = ti.xcom_pull(task_ids='tarea_etl')
    
    df = pd.read_parquet(ruta_archivo)
    
    # Simulación de EDA
    print(f"Estadísticas del dataset: {df.describe()}")
    print(f"Columnas procesadas: {df.columns.tolist()}")

with DAG(
    dag_id='pipeline_eda_etl_profesional',
    start_date=datetime(2023, 1, 1),
    schedule_interval=None,
    catchup=False
) as dag:

    # Tarea 1: ETL
    tarea_etl = PythonOperator(
        task_id='tarea_etl',
        python_callable=ejecutar_etl
    )

    # Tarea 2: EDA
    tarea_eda = PythonOperator(
        task_id='tarea_eda',
        python_callable=ejecutar_eda
    )

    # Dependencia
    tarea_etl >> tarea_eda

```