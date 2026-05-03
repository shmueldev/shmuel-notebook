## Fase 3: Configuración y Despliegue (Docker)

11. **Archivo de conexion ssh**
    - archivo de configuración, comando de creación:
    
    ```
    # crear archivo de llaves
    ~/.ssh/key.pem

    # crear archivo de config
    ~/.ssh/config

    # permisos
    chmod 400 ~/.ssh/key.pem

    # contenido dentro de config
    Host airflow-master
        HostName [IP_ADDRESS]
        User ubuntu
        IdentityFile ~/.ssh/id_rsa
    ```

11. **Instalar Docker y Docker Compose**
    - En todas las instancias EC2 aprovisionadas.
   ```bash
   # crear archivo dentro de proxy
   nano docker_todos.sh

   # archivo de configuracion dentro de proxy
   #!/bin/bash

   NODOS=("master" "worker-1" "worker-2" "worker-3" "proxy" "rabbitmq")

   COMANDOS='
   set -e

   echo "🧹 Eliminando repositorios incorrectos de Docker..."
   sudo rm -f /etc/apt/sources.list.d/docker.list
   sudo rm -f /etc/apt/sources.list.d/*docker*

   echo "📦 Limpiando locks de apt..."
   sudo rm -f /var/lib/dpkg/lock-frontend /var/lib/apt/lists/lock

   echo "🔄 Actualizando paquetes..."
   sudo apt-get update -y

   echo "📥 Instalando dependencias..."
   sudo apt-get install -y ca-certificates curl gnupg lsb-release

   echo "🔑 Configurando keyring..."
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc

   echo "📦 Agregando repo correcto..."
   CODENAME=$(lsb_release -cs)

   echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $CODENAME stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

   echo "🔄 Actualizando repositorios..."
   sudo apt-get update -y

   echo "🐳 Instalando Docker..."
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

   echo "🚀 Habilitando Docker..."
   sudo systemctl enable --now docker

   echo "👤 Agregando usuario al grupo docker..."
   sudo usermod -aG docker $USER

   echo "✅ DOCKER INSTALADO EN: $(hostname)"
   '

   for nodo in "${NODOS[@]}"; do
   echo "------------------------------------------"
   echo "🚀 PROCESANDO: $nodo"
   echo "------------------------------------------"

   ssh -n -o StrictHostKeyChecking=no "$nodo" "$COMANDOS"
   done

   # Confirmar instalacion del docker
   for nodo in master worker-1 worker-2 worker-3 rabbitmq proxy; do 
     echo -n "$nodo: "
     ssh $nodo "docker --version"
   done
   ```
12. **Clonar repositorio de git en cada nodo**
   ```bash
   # archivo de configuracion
   nano clonar_repo.sh

   # contenido de ese archivo
   #!/bin/bash

   NODOS=("master" "worker-1" "worker-2" "worker-3" "proxy" "rabbitmq")

   REPO_URL="git@github.com:usuario/tu-repo.git"

   COMANDOS='
   set -e

   echo "📦 Instalando git..."
   sudo apt-get update -y
   sudo apt-get install -y git

   cd $HOME

   REPO_NAME=$(basename -s .git '"$REPO_URL"')

   if [ -d "$REPO_NAME" ]; then
   echo "🔄 Repo ya existe, haciendo pull..."
   cd "$REPO_NAME"
   git pull
   else
   echo "⬇️ Clonando repo en HOME..."
   git clone '"$REPO_URL"'
   fi

   echo "✅ REPO LISTO EN: $HOME/$REPO_NAME ($(hostname))"
   '

   for nodo in "${NODOS[@]}"; do
   echo "------------------------------------------"
   echo "🚀 PROCESANDO: $nodo"
   echo "------------------------------------------"

   ssh -n -o StrictHostKeyChecking=no "$nodo" "$COMANDOS"
   done

   # permisos
   chmod +x clonar_repo.sh

   # ejecutar
   ./clonar_repo.sh

   # verificar
   ls -la /opt/apps/repo
   ```

13. **Subir archivos de configuración**
    - Levantamos en la instancia proxy.
    ```bash
    cd proxy
    ./docker_compose.sh up -d
    ```
    - Luego creamos usuario y contraseña desde el panel de nginx (puerto 81).
    - Creamos los hosts y apuntamos a las ips privadas de los nodos con sus respectivos puertos
        - localhost:8080 -> master
        - localhost:5555 -> flower
        - localhost:15672 -> rabbitmq
    - encrypt DNS
    - crear certificado SSL para cada host desde nginx. 
    - crear bucket en s3 para guardar los archivos de airflow, los logs.
      - dentro de esta crear una folder: logs.  


14. levanatar servicios del master
   - Generar la fertnet key: `ttps://8gwifi.org/CipherFunctionality`
   - Generar secret key de airflow: `https://randomkeygen.com/` 
   - crear base de datos  
14. **Configurar Variables de Entorno (`.env`)**
    - Modificar los `.env` asegurando de colocar los **Endpoints** correctos de AWS para Postgres (`POSTGRES_HOST`) y RabbitMQ (`RABBITMQ_HOST`).
    - Unificar cuidadosamente la variable `AIRFLOW__CORE__FERNET_KEY` entre todos los nodos.

15. **Levantar Servicios Iniciales (Proxy y Master)**
    - En la máquina Proxy: `docker compose up -d`.
    - En la máquina Master, inicializar con cuidado la DB (este paso hace el migration de la DB configurada en el param `POSTGRES_HOST`): `docker compose up -d`.

16. **Levantar Recursos de Trabajo (Workers)**
    - En las instancias Worker: `docker compose up -d --scale airflow-worker=N` (ajustar N según capacidad local de esa máquina).

---

## Fase 5: Dominio y Proxy Inverso

17. **Configuración DNS**
    - En **Route 53** (o el proveedor actual), crear un registro tipo `A` que apunte al dominio del proyecto (ej. `airflow.mi-empresa.com`) apuntando a la **Elastic IP** asignada a la instancia de Nginx Proxy Manager.

18. **Configurar Proxy Inverso (NPM)**
    - Ingresar a la interfaz web de NPM (puerto 81).
    - Configurar los "Proxy Hosts" dirigiendo el tráfico web hacia las IP Privadas de las instancias EC2 (ej. Proxy a Master en el puerto 8080 para la UI de Airflow).
    - Generar certificados SSL gratuitos mediante Let's Encrypt desde la misma interfaz de NPM.