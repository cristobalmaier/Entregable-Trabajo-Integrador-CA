### 1. Blanqueo de Credenciales y Entorno Inicial

- **Acceso de Emergencia**: Se inició la máquina virtual con el parámetro `init=/bin/bash` para omitir la autenticación inicial.
    
- **Permisos de Escritura**: Se remontó la partición raíz para permitir cambios en el disco: `mount -o remount,rw /`.
    
- **Nueva Clave de Root**:
    
    - Comando: `passwd root`.
        
    - Clave establecida: **palermo**.
        
- **Reinicio e Inicio de Sesión**: Ingreso normal con usuario `root` y la nueva clave.
    
- **Cambio de Hostname**:
    
    - Comando: `hostnamectl set-hostname TPServer`.
        
    - Se editó `/etc/hosts` para vincular la dirección `127.0.1.1` con el nombre **TPServer**.
        

### 2. Actualización del Sistema Operativo

- **Verificación Inicial**: La versión original del sistema era **Debian 11.10**.
    
- **Cambio de Repositorios**:
    
    - Se editó el archivo `/etc/apt/sources.list` reemplazando todas las instancias de `bullseye` por `bookworm`.
        
    - Se incluyó la sección `non-free-firmware` requerida para la compatibilidad en Debian 12.
        
- **Proceso de Upgrade**: Se ejecutaron los comandos `apt update` y `apt full-upgrade -y` para realizar la transición completa.
    
- **Instalación de GRUB**: Durante la actualización, se seleccionó el disco principal (`/dev/sda`) para reinstalar el cargador de arranque y asegurar el inicio del sistema.
    
- **Confirmación Final**: Tras el reinicio, el comando `cat /etc/debian_version` confirmó la actualización a **Debian 12**.
    

### 3. Configuración de Servicios

- **Preparación de Archivos**:
    
    - Se identificó el archivo `Material_Adicional_TPVMCA.tar.gz` en el Home de root.
        
    - Se descomprimió el contenido en `/root/Material_Adicional_TPVMCA/`obteniendo las llaves SSH, archivos web y el script SQL.
    
- **SSH (Acceso por Llaves)**:
    
    - Se creó el directorio `/root/.ssh` con permisos `700`.
        
    - Se autorizó la llave pública mediante: `cat /root/Material_Adicional_TPVMCA/clave_publica.pub >> /root/.ssh/authorized_keys`.
        
    - Se aplicaron permisos `600` al archivo `authorized_keys`.
        
    - Se configuró `/etc/ssh/sshd_config` con la directiva `PermitRootLogin prohibit-password`.
	