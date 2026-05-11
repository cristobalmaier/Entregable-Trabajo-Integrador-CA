1. Blanqueo de Credenciales y Entorno Inicial

	- Acceso de Emergencia: Se inició con el parámetro init=/bin/bash.

	- Permisos de Escritura: Se remontó la partición raíz: mount -o remount,rw /.

Nueva Clave de Root:

- Comando: passwd root.

- Clave establecida: palermo.

- Reinicio e Inicio de Sesión: Ingreso normal con usuario root y clave palermo.

Cambio de Hostname:

Comando: hostnamectl set-hostname TPServer.

Se editó /etc/hosts para vincular 127.0.1.1 con TPServer.

2. Actualización del Sistema Operativo
Verificación Inicial: La versión original era Debian 11.10.

Cambio de Repositorios:

Se editó /etc/apt/sources.list reemplazando bullseye por bookworm.

Se incluyó la sección non-free-firmware requerida en Debian 12.

Proceso de Upgrade:

apt update

apt full-upgrade -y.

Instalación de GRUB: Se seleccionó el disco principal (/dev/sda) para el cargador de arranque.  

Confirmación Final: Tras el reinicio, cat /etc/debian_version arroja versión 12.