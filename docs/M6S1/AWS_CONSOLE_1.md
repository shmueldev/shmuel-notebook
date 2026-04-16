# Guía paso a paso: Configuración de Entorno en AWS

A continuación, se detallan los pasos para crear y configurar todo tu espacio de trabajo desde la consola de AWS, abarcando la red, seguridad y los nodos (Master/Worker).

## 1. Crear VPC (Virtual Private Cloud)
1. Inicia sesión en la consola de AWS y dirígete al servicio **VPC**.
2. En el panel lateral, haz clic en **Your VPCs** y luego en el botón **Create VPC**.
3. Selecciona **VPC only** (para tener control manual) en lugar de "VPC and more".
4. Asigna un nombre en **Name tag** (ej. `mi-vpc-entorno`).
5. En **IPv4 CIDR block**, selecciona *IPv4 CIDR manual input* y asigna un rango de direcciones, por ejemplo: `10.0.0.0/16`.
6. Haz clic en **Create VPC**.

## 2. Crear Internet Gateway
El Gateway permite que los recursos en tu VPC tengan acceso y salida a Internet.
1. En el mismo panel de navegación de VPC, dirígete a **Internet Gateways**.
2. Haz clic en **Create internet gateway**.
3. Asigna un nombre en **Name tag** (ej. `mi-igw-entorno`).
4. Haz clic en **Create internet gateway**.
5. Una vez creado, arriba a la derecha haz clic en **Actions** y selecciona **Attach to VPC**.
6. Selecciona la VPC creada en el Paso 1 (`mi-vpc-entorno`) y haz clic en **Attach internet gateway**.

## 3. Crear Subredes (Públicas y Privadas)
Las subredes dividen tu VPC para separar recursos públicos (accesibles por internet) de recursos privados.

### Crear Subred Pública
1. En el panel lateral navega a **Subnets** y haz clic en **Create subnet**.
2. En la lista, selecciona la VPC recién creada (`mi-vpc-entorno`).
3. En la configuración de la subred:
   - **Subnet name:** `subred-publica`
   - **Availability Zone:** Selecciona la que prefieras (ej. `us-east-1a`).
   - **IPv4 CIDR block:** Asigna un sub-rango (ej. `10.0.1.0/24`).
4. Haz clic en **Create subnet**.
5. **Paso Clave:** Selecciona esta subred la lista, ve a **Actions** > **Edit subnet settings**, marca **Enable auto-assign public IPv4 address** para que los nodos aquí tomen IP públicas automáticamente, y guarda.

**Vincular Subred Pública al Internet Gateway (Tabla de Rutas):**
1. Ve a **Route Tables**, selecciona la tabla principal asignada a tu VPC (puedes nombrarla `tabla-publica-rt`).
2. En la parte inferior ve a la lengüeta **Routes** y dale a **Edit routes**.
3. Haz clic en **Add route**: Destination `0.0.0.0/0` y Target selecciona **Internet Gateway** (escoge `mi-igw-entorno`). Guarda la ruta.
4. En **Subnet associations**, dale a **Edit subnet associations**, selecciona `subred-publica` y guarda.

### Crear Subred Privada
1. Vuelve a **Subnets** y haz clic en **Create subnet** nuevamente (sobre la VPC `mi-vpc-entorno`).
2. Configuración:
   - **Subnet name:** `subred-privada`
   - **Availability Zone:** Selecciona una de la lista.
   - **IPv4 CIDR block:** Asigna otro sub-rango diferente (ej. `10.0.2.0/24`).
3. Haz clic en **Create subnet**.
*Nota: Como no la asociaremos directamente al Internet Gateway, mantiene su carácter privado, pero necesitará un NAT Gateway para descargar paquetes de Internet.*

### Crear NAT Gateway (Para dar Internet a los Workers)
Para que los workers (en la subred privada) puedan descargar actualizaciones, instalar Docker o clonar desde GitHub, necesitan poder salir a Internet sin exponer sus servidores (sin IP pública directamente asignada a ellos). Para esto usamos un **NAT Gateway**.

1. En el panel de navegación de la VPC, ve a **NAT Gateways** y haz clic en **Create NAT gateway**.
2. **Name:** Asigna un nombre (ej. `mi-nat-gw`).
3. **Subnet:** Selecciona la **Subred Pública** (`subred-publica`). ¡Importante! El NAT debe vivir en la subred pública porque necesita salir por el Internet Gateway.
4. **Elastic IP allocation ID:** Haz clic en el botón secundario **Allocate Elastic IP** (esto le asignará una IP pública fija de forma automática suministrada por AWS).
5. Haz clic en **Create NAT gateway** y espera unos minutos a que el estado se refleje como *Available*.

**Vincular Subred Privada al NAT Gateway (Tabla de Rutas Privada):**
1. Ve a la sección **Route Tables** y haz clic en **Create route table**.
2. **Name tag:** Llámala `tabla-privada-rt`. **VPC:** Selecciona `mi-vpc-entorno` y haz clic en **Create route table**.
3. Selecciona tu nueva tabla `tabla-privada-rt`, ve a la parte inferior a la lengüeta **Routes** y dale a **Edit routes**.
4. Haz clic en **Add route**: Destination `0.0.0.0/0` y en Target selecciona **NAT Gateway** (escoge el que creaste `mi-nat-gw`). Guarda la ruta.
5. Ve a la lengüeta **Subnet associations**, haz clic en **Edit subnet associations**, selecciona la `subred-privada` y guarda los cambios.
*De este modo, todo el tráfico de la subred privada que vaya a internet, viajará mediante el NAT Gateway y la red se mantendrá segura.*

## 4. Crear Security Groups
Los Security Groups funcionan como un firewall a nivel de instancia, definiendo qué tráfico entra o sale, por lo general tenemos dos grupos de segurida, uno para master y otro para workers:

### Crear Security Group para Master
1. Dentro de EC2 o VPC, en el menú lateral busca la sección **Security Groups** y haz clic en **Create security group**.
2. Detalles básicos:
   - **Security group name:** `sg-master`
   - **Description:** Grupo de seguridad para habilitar conexiones requeridas.
   - **VPC:** Elimina la "Default" y selecciona `mi-vpc-entorno`.
3. **Inbound rules (Reglas de entrada):** Agrega las siguientes.
   - Tipo `SSH` (Puerto 22) -> Source `Anywhere-IPv4` (`0.0.0.0/0`) *Nota: para mayor seguridad puedes poner solo tu IP ("My IP").*
   - Tipo `Custom TCP` (Puerto 5672, 8080,  15672, 5555) *Nota: estos puertos son para rabbitmq, airflow, etc.*
4. **Outbound rules (Reglas de salida):** Normalmente se deja la que está por defecto (`All traffic` hacia `0.0.0.0/0`).
5. Haz clic en **Create security group**.

### Crear Security Group para Worker
1. Dentro de EC2 o VPC, en el menú lateral busca la sección **Security Groups** y haz clic en **Create security group**.
2. Detalles básicos:
   - **Security group name:** `sg-worker`
   - **Description:** Grupo de seguridad para habilitar conexiones requeridas.
   - **VPC:** Elimina la "Default" y selecciona `mi-vpc-entorno`.
3. **Inbound rules (Reglas de entrada):** Agrega las siguientes.
   - Tipo `SSH` (Puerto 22) -> Source `Custom` (`sg-master`) *Nota: para mayor seguridad puedes poner solo tu IP ("My IP").*
4. **Outbound rules (Reglas de salida):** Normalmente se deja la que está por defecto (`All traffic` hacia `0.0.0.0/0`).
5. Haz clic en **Create security group**.

## 5. Crear Nodos Master y Worker
Por último, lanzamos instancias EC2 de tipo "Master" y "Worker" dentro del esquema de red preparado.

### Instanciar Nodo(s) Master
1. Dirígete a la consola del servicio **EC2** y haz clic en **Launch instances**.
2. **Name:** `Master-Node`.
3. **AMI (OS):** Selecciona Ubuntu o la distribución de tu preferencia.
4. **Instance type:** Para un nodo master suele requerirse mínimo `t2.medium` (ej. en Kubernetes o clústeres complejos).
5. **Key pair:** Selecciona o crea un par de claves para permitir conexión SSH.
6. **Network settings:** Dale a **Edit**.
   - **VPC:** Selecciona `mi-vpc-entorno`.
   - **Subnet:** Selecciona `subred-publica` (para que le llegue una IP pública y puedas conectarte al master).
   - **Firewall (security groups):** Escoge *Select existing security group* y marca `sg-master`.
7. Ajusta el almacenamiento necesario y haz clic en **Launch instance**.

### Instanciar Nodo(s) Worker
1. Vuelve a hacer clic en **Launch instances**.
2. **Name:** `Worker-Node-1` (y 2, 3... dependiendo cuántos lances). Puedes cambiar la cantidad de instancias a lanzar en el panel derecho superior si necesitas varios a la vez.
3. **AMI (OS):** La misma versión usada en el Master.
4. **Instance type:** Puede ser algo menor como `t2.micro` o de la misma capacidad, según tus necesidades.
5. **Key pair:** Selecciona la misma llave (Key pair) utilizada antes.
6. **Network settings:** Dale a **Edit**.
   - **VPC:** Selecciona `mi-vpc-entorno`.
   - **Subnet:** Selecciona `subred-privada`. *(A menudo los workers van en la privada por seguridad, pero si necesitas acceso rápido a ellos, puedes ponerlos en la pública o usar un Bastion Host / de configuración interconectada)*.
   - **Firewall (security groups):** Selecciona `sg-workers`.
7. Haz clic en **Launch instance**.

¡Listo! Con esto tendrás una arquitectura sólida con la red segregada y los servidores configurados en tu cuenta de AWS.
