# Guía de Instalación: Docker y Docker Compose en Nodos Master y Worker

Esta guía consolida los pasos necesarios de los archivos `readme`, ordenados de forma lógica. El objetivo es conectarnos a nuestras máquinas e instalar Docker y Docker Compose ("docker compass") en toda nuestra infraestructura.

---

## 1. Conexión a las Máquinas (SSH)

Antes de instalar nada, necesitamos tener acceso a la terminal de nuestros servidores.

### Conexión al Nodo Master
1. Creamos el archivo para almacenar la llave privada:
   ```bash
   # Hacemos esto ya que todos los workers tienen la misma key
   nano worker_key.pem
2. Creamos un config para ssh para no tener que escribir la llave y la ip cada vez
   ```bash
   nano ~/.ssh/config
   ```
   ```
   Host worker-1
       HostName <IP_DEL_WORKER_1>
       User ubuntu
       IdentityFile ~/.ssh/worker_key.pem
   Host worker-2
       HostName <IP_DEL_WORKER_2>
       User ubuntu
       IdentityFile ~/.ssh/worker_key.pem
   Host worker-3
       HostName <IP_DEL_WORKER_3>
       User ubuntu
       IdentityFile ~/.ssh/worker_key.pem
   ```

### Conexión a los Nodos Worker
Dependiendo de cómo tengas centralizados los workers (ej. mediante Termius), usa el comando correspondiente para cada uno:
```bash
ssh worker-1
ssh worker-2
ssh worker-3
```

---

## 2. Descargar e Instalar Docker y Docker Compose
> **IMPORTANTE:** Debes realizar esta instalación en **todos los equipos** (En la terminal del Master y también entrando a la terminal de cada Worker).

Los pasos oficiales para Ubuntu extraídos de la [documentación oficial](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) son:

1. **Desintalar paquetes de Docker**
```bash
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)
```

2. **Install using the apt repository**
```bash
# Add Docker's official GPG key:
    sudo apt update
    sudo apt install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    # Add the repository to Apt sources:
    sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
    Types: deb
    URIs: https://download.docker.com/linux/ubuntu
    Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
    Components: stable
    Architectures: $(dpkg --print-architecture)
    Signed-By: /etc/apt/keyrings/docker.asc
    EOF

    sudo apt update
```

3. **Install Docker packages**
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

4. **Verify Docker is running**
```bash
sudo systemctl status docker
```

5. **Iniciar Docker**
```bash
sudo systemctl start docker
```

6. **Verificar Intalación**
```bash
sudo docker run hello-world
```
7. Búcle para ser mucho más rapido
```
for host in worker-2 worker-3; do
    echo "------------------------------------------"
    echo "Instalando Docker en: $host"
    echo "------------------------------------------"
    
    ssh $host << 'EOF'
        # 2. Install using the apt repository
        sudo apt update
        sudo apt install -y ca-certificates curl
        sudo install -m 0755 -d /etc/apt/keyrings
        sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
        sudo chmod a+r /etc/apt/keyrings/docker.asc

        # Add the repository to Apt sources
        sudo tee /etc/apt/sources.list.d/docker.sources <<EOT
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOT

        sudo apt update

        # 3. Install Docker packages
        sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

        # 5. Iniciar Docker
        sudo systemctl start docker
        sudo systemctl enable docker

        # 6. Verificar Instalación
        sudo docker run hello-world
EOF
done
```

## Crear la estructura del proyecto
Hacemos el siguiente comando este nos permite crear la carpeta raiz y entrar en esta
```bash
mkdir -p Data-Project-Doc && cd Data-Project-Doc
```
Creamos las carpetas de la estructura de nuestro proyecto
```
data-platform/
│
├── .env
├── .gitignore
├── README.md
│
├── master/
│   └── docker-compose.yml
│
├── worker/
│   └── docker-compose.yml
│
└── airflow/
    ├── dags/
    │   └── example_dag.py
    ├── logs/
    └── plugins/
```
Comando para crear cada carpeta y comando para ver nuestros archivos de manera bonita
```bash
# Crear la carpeta
mkdir name_carpeta

# ver nuestros archivos 1
ls -R

# ver nuestros archivos 2
ls -R | grep -v "^$"

# ver nuestros archivos 3
find . -maxdepth 2 -not -path '*/.*'
```
## Subir la estructura de nuestro proyecto
```bash
# Crear el repositorio en GitHub
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/shmueldev/Data-Project-Doc.git
git push -u origin main

# Como tenemos carpetas que estan vacías hacemos entonces los pasos de la siguiente manera
find . -type d -not -path '*/.*' -empty -exec touch {}/.gitkeep \;
git add .
git commit -m "Añadir estructura de carpetas mediante archivos .gitkeep"
git push -u origin main
```
## Clonar el repositorio


1. **Configurar el repositorio de APT para Docker:**
   ```bash
   # Agregar llaves oficiales de Docker
   sudo apt-get update
   sudo apt-get install ca-certificates curl
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc

   # Agregar el repositorio
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
     $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update
   ```

2. **Instalar Docker Engine y Docker Compose:**
   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
   ```
   *(Nota: `docker-compose-plugin` es el que instala **Docker Compose**, lo que necesitas para ejecutar `docker compose up`)*.

3. **Permisos de Docker (Opcional):**
   Para no tener que escribir `sudo` antes de cada comando docker, agrega tu usuario al grupo:
   ```bash
   sudo usermod -aG docker $USER
   ```
   *(Deberás cerrar sesión y volver a entrar por SSH para que aplique)*.

---

## 3. Descargar el Código del Proyecto (Principalmente Master)
Una vez con Docker listo, prepararemos el entorno para levantar los servicios.

### Creación de Llave de Despliegue (Deploy Key)
Generamos una llave que usaremos para darle permisos de lectura al repositorio en GitHub.
```bash
ssh-keygen -t ed25519 -C "deploy-key"
cat ~/.ssh/id_ed25519.pub
```
*(Copia y registra la llave resultante `ssh-ed25519 AAA... deploy-key` en la configuración de GitHub).*

### Clonar el Repositorio
```bash
ssh -T git@github.com
git clone git@github.com:hipocondriacos-data/Data-project1.git
cd Data-project1
```

---

## 4. Ejecución del Clúster y Variables de Entorno

Antes de invocar el "docker compose", necesitamos asegurar nuestras credenciales.

1. **Archivo `.env`:** 
   Debes crear un archivo `.env` en la misma ruta donde tengas el `docker-compose.master.yml`.
   Este archivo aloja las contraseñas reales (Como `DB_PASSWORD`) y **nunca debes subirlo a GitHub**. El Docker Compose lee este archivo privado para levantar los contenedores sin exponer claves en código fuente.

2. **Levantar Servicios:**
   Una vez configurado y validado mediante un `ls -l docker-compose.master.yml`, ejecuta los servicios para Master usando el comando Compose que acabas de instalar:
   ```bash
   docker compose -f docker-compose.master.yml up -d
   ```
   *Esto pondrá a correr Postgres, RabbitMQ, y distintos contenedores de Airflow en el fondo.*
