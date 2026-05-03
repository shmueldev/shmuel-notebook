# Data Platform 🚀

Plataforma distribuida basada en **Apache Airflow + Celery** para desplegar rápidamente un cluster de orquestación de pipelines de datos.

Este proyecto permite levantar una arquitectura desacoplada con:

* Nodo **Master** (Scheduler + Webserver + Init)
* Múltiples **Workers** (ejecución distribuida)
* Integración con:
* PostgreSQL (metadata DB)
* RabbitMQ (broker)

---

## 🧱 Estructura del proyecto

<pre class="overflow-visible! px-0!" data-start="642" data-end="857"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>data-platform</span><br/><span>├── .gitignore</span><br/><span>├── README.md</span><br/><span>├── master</span><br/><span>│   ├── .env.template</span><br/><span>│   ├── docker-compose.yml</span><br/><span>├── pipelines</span><br/><span>│   ├── dags</span><br/><span>│   └── tasks</span><br/><span>└── worker</span><br/><span>    ├── .env.template</span><br/><span>    ├── docker-compose.yml</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

---

## 🧠 Arquitectura

El sistema sigue el patrón distribuido de Airflow con  **CeleryExecutor** :

<pre class="overflow-visible! px-0!" data-start="959" data-end="2039"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>                +-------------------+</span><br/><span>                |   Airflow Master  |</span><br/><span>                |-------------------|</span><br/><span>                | Scheduler         |</span><br/><span>                | Webserver         |</span><br/><span>                | DB Init           |</span><br/><span>                +---------+---------+</span><br/><span>                          |</span><br/><span>                          v</span><br/><span>                +-------------------+</span><br/><span>                |    RabbitMQ       |</span><br/><span>                +-------------------+</span><br/><span>                          |</span><br/><span>          +---------------+---------------+</span><br/><span>          |                               |</span><br/><span>          v                               v</span><br/><span>+-------------------+          +-------------------+</span><br/><span>| Airflow Worker 1  |          | Airflow Worker N  |</span><br/><span>|-------------------|          |-------------------|</span><br/><span>| Ejecuta tareas    |          | Ejecuta tareas    |</span><br/><span>+-------------------+          +-------------------+</span><br/><span>                          |</span><br/><span>                          v</span><br/><span>                +-------------------+</span><br/><span>                |   PostgreSQL      |</span><br/><span>                | (Metadata DB)     |</span><br/><span>                +-------------------+</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

---

## ⚙️ Requisitos

* Docker
* Docker Compose
* Acceso a:
* PostgreSQL
* RabbitMQ

---

## 🚀 Levantar el cluster

### 1. Configurar variables de entorno

#### Master

<pre class="overflow-visible! px-0!" data-start="2216" data-end="2259"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class="relative"><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼs">cd</span><span> master</span><br/><span class="ͼs">cp</span><span> .env.template .env</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

Editar `.env`:

<pre class="overflow-visible! px-0!" data-start="2277" data-end="2565"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class="relative"><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>AIRFLOW__CORE__EXECUTOR=CeleryExecutor</span><br/><br/><span># PostgreSQL</span><br/><span>POSTGRES_HOST=...</span><br/><span>POSTGRES_PORT=5432</span><br/><span>POSTGRES_DB=...</span><br/><span>POSTGRES_USER=...</span><br/><span>POSTGRES_PASSWORD=...</span><br/><br/><span># RabbitMQ</span><br/><span>RABBITMQ_HOST=...</span><br/><span>RABBITMQ_PORT=5672</span><br/><span>RABBITMQ_USER=...</span><br/><span>RABBITMQ_PASSWORD=...</span><br/><br/><span># Seguridad</span><br/><span>AIRFLOW__CORE__FERNET_KEY=...</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

---

#### Worker

<pre class="overflow-visible! px-0!" data-start="2585" data-end="2628"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class="relative"><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼs">cd</span><span> worker</span><br/><span class="ͼs">cp</span><span> .env.template .env</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

⚠️ Debe usar **exactamente las mismas credenciales** que el master.

---

## ▶️ 2. Iniciar Master

<pre class="overflow-visible! px-0!" data-start="2729" data-end="2771"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class="relative"><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼs">cd</span><span> master</span><br/><span>docker compose up </span><span class="ͼu">-d</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

Esto levanta:

* DB migration (init)
* Scheduler
* Webserver

---

## 👷 3. Iniciar Workers

En cada nodo worker:

<pre class="overflow-visible! px-0!" data-start="2888" data-end="2930"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class="relative"><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼs">cd</span><span> worker</span><br/><span>docker compose up </span><span class="ͼu">-d</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

Puedes escalar horizontalmente:

<pre class="overflow-visible! px-0!" data-start="2965" data-end="3022"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class="relative"><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>docker compose up </span><span class="ͼu">-d</span><span></span><span class="ͼu">--scale</span><span></span><span class="ͼt">airflow-worker</span><span class="ͼn">=</span><span class="ͼq">3</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

---

## 📂 Pipelines

Ubicación:

<pre class="overflow-visible! px-0!" data-start="3058" data-end="3101"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>pipelines/</span><br/><span>├── dags/</span><br/><span>└── tasks/</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

* `dags/`: definición de workflows
* `tasks/`: lógica reutilizable

👉 Estos deben montarse en los contenedores de Airflow.

---

## ⚠️ Consideraciones importantes

### 🔴 1. Migraciones de base de datos

Solo el **master** ejecuta:

<pre class="overflow-visible! px-0!" data-start="3337" data-end="3367"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class="relative"><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>airflow db upgrade</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

👉 Los workers **NO deben ejecutar migraciones**

---

### 🔴 2. Consistencia de configuración

Todos los nodos deben compartir:

* `FERNET_KEY`
* Conexión a DB
* Broker (RabbitMQ)

---

### 🔴 3. Recursos de Workers

Ajustar concurrencia según RAM:

<pre class="overflow-visible! px-0!" data-start="3620" data-end="3669"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class="relative"><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>airflow celery worker </span><span class="ͼu">--autoscale</span><span></span><span class="ͼq">4</span><span>,1</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

---

## 🧪 Validación

### Worker correcto

<pre class="overflow-visible! px-0!" data-start="3715" data-end="3746"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼk ͼy"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>celery@worker ready</span></div></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

### Errores comunes

* `pg_advisory_lock` → múltiples migraciones
* `exit code 137` → falta de memoria
* tareas en cola → problema con RabbitMQ

---

## 🔐 Seguridad

* Nunca subir `.env`
* Rotar credenciales si fueron expuestas
* Usar redes privadas entre servicios

---

## 📈 Escalabilidad

Este diseño permite:

* Agregar workers dinámicamente
* Separar cargas por colas
* Distribuir ejecución en múltiples servidores

---

## 🎯 Objetivo del proyecto

Proveer una base simple pero robusta para:

* Orquestación de pipelines de datos
* Procesamiento distribuido
* Escalabilidad horizontal

---

## 🧠 Notas finales

Este setup está diseñado para:

* entornos productivos ligeros
* pruebas de arquitectura distribuida
* bootstrap rápido de plataformas de datos
