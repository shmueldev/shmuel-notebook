# Gestión de Organizaciones en GitHub y Deploy Keys

En grandes infraestructuras o equipos de trabajo, usar cuentas personales o contraseñas simples no es escalable ni seguro. GitHub ofrece **Organizaciones** para agrupar proyectos y **Deploy Keys** para autorizar servidores (como tus nodos Master y Worker en AWS) a leer o modificar código de un repositorio específico de forma fácil y automática.

---

## 1. Crear una Organización en GitHub

Una Organización te permite tener repositorios compartidos, gestionar permisos granulares por equipos y mantener el código seguro y centralizado de los individuos.

1. Ve a [GitHub](https://github.com/) e inicia sesión con tu cuenta personal.
2. En la esquina superior derecha, haz clic en el ícono de tu perfil (tu foto) y selecciona **Your organizations**.
3. Haz clic en el botón **New organization**.
4. Selecciona el plan gratuito (Free) o de pago dependiendo de qué funciones o tiempo en *actions* requieras.
5. Rellena los datos básicos:
   - **Organization account name:** Un nombre único, como por ejemplo `hipocondriacos-data`.
   - **Contact email:** Tu correo para notificaciones de facturación o seguridad.
6. Acepta los términos y haz clic en **Next** o **Create Organization**.
7. *(Opcional)* Invita a otros colaboradores de tu startup buscando por su usuario de GitHub, o simplemente haz clic en **Complete setup** para saltar este paso por ahora.
8. Una vez en el panel de tu organización, haz clic en **Create a new repository** para empezar a crear tu infraestructura (ej. `Data-project1`).

---

## 2. ¿Qué son las Deploy Keys?

Una **Deploy Key** es una llave criptográfica SSH generada directamente dentro de un servidor (como tus EC2 Master/Workers) que, en lugar de ligarse a tu usuario de GitHub humano, se vincula **exclusivamente a un repositorio en específico** dentro de la Organización.

**Beneficios:**
- **Máxima Seguridad:** Si un servidor se ve comprometido y roban la llave, el atacante solo tendrá acceso a ese repositorio aislado particular, no al resto de tus repos ni a tu cuenta global.
- **Automatización (Sin Contraseñas):** Las llaves gestionan la autenticación sin preguntar usuario o clave, lo que facilita los comandos automáticos, el uso en CI/CD y despliegues por Docker.

---

## 3. Generar y Configurar Deploy Keys en AWS

### Paso A: Generar llave SSH en tu Servidor (Ej. el nodo Master)
1. Conéctate a tu servidor mediante SSH.
2. Genera una nueva llave SSH utilizando el algoritmo criptográfico `ed25519` (el más rápido y seguro en la actualidad):
   ```bash
   ssh-keygen -t ed25519 -N ""
   ```
3. Cuando pregunte *Enter file in which to save the key*, presiona **Enter** para utilizar la ruta por defecto (`~/.ssh/id_ed25519`).
4. Cuando pida *passphrase* (contraseña opcional en local para encriptar la llave), déjalo en blanco presionando **Enter dos veces** (se requiere dejar en blanco en servidores si vas a automatizar `git pulls` por cron o pipelines, de lo contrario la ejecución se quedará pausada esperando que alguien la digite).
5. Muestra en terminal tu llave **Pública** e imprímela:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   *(Copia TODO el texto que aparecerá en pantalla, que incluye el prefijo `ssh-ed25519 AAA...`)*.

### Paso B: Agregar la Deploy Key a la Organización (Permisos de Escritura)
1. Entra al repositorio creado dentro de tu Organización en GitHub.
2. Ve a la pestaña de **Settings** (Configuraciones) del propio repositorio (no las configuraciones de tu usuario personal).
3. En el menú lateral izquierdo, bajo la sección de "Security", haz clic en **Deploy keys**.
4. Haz clic en el botón superior derecho **Add deploy key**.
5. **Title:** Asigna un nombre claro de qué servidor es (Ej. `AWS Production Master Node`).
6. **Key:** Pega todo el contenido de la llave pública que copiaste en el Paso A.
7. **Dar Permiso de Escritura (Crucial):** Si necesitas que este servidor haga modificaciones en el código y suba los cambios con `git push`, **marca la casilla [x] Allow write access**. (En contraparte, para los Workers suele desmarcarse para que tengan acceso exclusivo de lectura).
8. Haz clic en **Add key** (GitHub te pedirá confirmar tu contraseña personal por medidas de seguridad).

---

## 4. Trabajar de Manera Ágil usando SSH

Como ya hemos agregado la llave, el nodo está empadronado a tu repositorio de forma transparente.

1. Añade a GitHub a la lista de "Known Hosts" de tu servidor conectándote falsamente una vez:
   ```bash
   ssh -T git@github.com
   ```
   *(Si el sistema arroja una alerta de fingerprint the ECDSA, escribe `yes` y da Enter. Enseguida te confirmará: "Hi hipocondriacos-data/Data-project1! You've successfully authenticated...")*
2. Ve al repositorio en GitHub de tu organización, dale al botón verde **Code** y selecciona en el submenú la solapa **SSH**. Copia la variante de la URL. Debería lucir parecida a: `git@github.com:hipocondriacos-data/Data-project1.git`.
3. Descarga (clona) el repositorio:
   ```bash
   git clone git@github.com:hipocondriacos-data/Data-project1.git
   cd Data-project1
   ```

A partir de este momento, estás libre. Los comandos `git pull`, `git add .`, `git commit -m "update"` y `git push` funcionarán instantánea y silenciosamente sin requerir contraseñas engorrosas o Personal Access Tokens.
