### 1. Blanqueo de Credenciales y Entorno Inicial

1. **Acceso de Emergencia**: Se inició la máquina virtual con el parámetro `init=/bin/bash` para omitir la autenticación inicial.
  
2. **Permisos de Escritura**: Se remontó la partición raíz para permitir cambios en el disco: `mount -o remount,rw /`.
  
3. **Nueva Clave de Root**:

```shell
passwd root
```
  
4. **Colocamos clave**

```shell
palermo
```

  
6. **Reiniciamos la maquina** para poder ingresar como **root** con la contraseña **palermo**
 
7. **Cambio de Hostname**:

```shelll
hostnamectl set-hostname TPServer
```


### 2. Actualización del Sistema Operativo

1. **Verificación Inicial**: La versión original del sistema era **Debian 11.10**.
  
2. **Cambio de Repositorios**:
    
    Se editó el archivo `/etc/apt/sources.list` reemplazando todas las instancias de `bullseye` por `bookworm`.
      
    Se añadio la sección `non-free-firmware` requerida para la compatibilidad en Debian 12.

1. **Actualizacion**: Se ejecutaron los siguientes comandos para actualizar el sistema.

```shell
apt update 
apt full-upgrade -y
```
 
4. **Instalación de GRUB**: Durante la actualización, se seleccionó el disco principal (`/dev/sda`) para reinstalar el cargador de arranque y asegurar el inicio del sistema. SE SELECCIONA CON "ESPACIO", luego con tab nos vemos hasta la opcion de "ACEPTAR" y presionamos "ENTER"

5. **Confirmación Final**: Despues de reinciar, comprobamos en que version de debian estamos, si todo sale bien deberia salir **12.14**

```shell
cat /etc/debian_version
```

### 3. Servicios

#### SSH

1. Instalación de ssh

```shell
apt install openssh-server -y
```

2. Entramos a /etc/ssh/sshd_config y vemos si tiene el "#" de PermitRootLogin, si lo tiene, lo eliminamos y guardamos archivo

```shell
vi /etc/ssh/sshd_config
```

3. Reiniciamos ssh

```shell
systemctl restart ssh
```

4. Verificamos si el status activo

```shell
systemctl status ssh
```

5. Creamos el directorio de seguridad de SSH para root

```shell
mkdir -p /root/.ssh
chmod 700 /root/.ssh
```

6. Autorizamos la llave pública del TP que esta en el archivo "Material_Adicional_TPVMCA"

```shell
cat /root/Material_Adicional_TPVMCA/clave_publica.pub >> /root/.ssh/authorized_keys
```

7. Asignamos Permisos a la llave

```shell
chmod 600 /root/.ssh/authorized_keys
```

8. Aplicamos y Reiniciamos

```shell
systemctl restart ssh
```

9. Verificamos si tenemos el archivo con la llave

```shell
cat /root/.ssh/authorized_keys
```

#### Web

1. Instalamos los paquetes necesarios

```shell
apt install apache2 php libapache2-mod-php -y
```

2. Verificamos que este funcionado de manera correcta

```shell
systemctl status apache2
```

3. De manera determida apache busca los sitios web en "/var/www/html/", por ende tenemos que mover el "index.php" y el "logo.png" de la carpeta proporcionada

4. Limpiamos "/var/www/html"

```shell
rm /var/www/html/index.html
```

5. Copiamos el archivo "index.php" de la carpeta del trabajo practico a la ruta de apache dicha anteriormente

```shell
cp /root/Material_Adicional_TPVMCA/index.php /var/www/html/
```

6. copiamos el archivo "logo.png" de la carpeta del trabjo practico ala ruta de apache dicha anteriormente

```shell
cp /root/Material_Adicional_TPVMCA/logo.png /var/www/html/
```

7. Otorgamos Permisos a los Archivos Web para que pueda leer y mostrar los archivos sin problemas

```shell
chmod 644 /var/www/html/index.php
chmod 644 /var/www/html/logo.png
```

#### Base de datos

1. Instalamos el paquete de mariaDB

```shell
apt install mariadb-server -y
```

ME OLVIDE DE INSTALAR LA EXTENSION DE MYSQL, NO OLVIDARSE PORQUE SINO TE TIRA ERROR HTTP INTERNAL ERROR 500

```shell
apt install php-mysql -y
```

```shell
systemctl restart apache2
```


2. Verificamos si esta funcionando de manera correcta

```shell
systemctl status mariadb
```

3. Cargamos el script de sql que nos proporciona la carpeta del trabajo practico en nuestra base de datos

```shell
mysql < /root/Material_Adicional_TPVMCA/db.sql
```

4. No nos pide contraseña porque somos usuario "Root"

5. Vemos si se cargo la base de datos por el script

```shell
mysql -e "SHOW DATABASES;"
```

6. Hacemos una consulta al servidor web para ver si funciona

```shell
curl -I http://localhost/index.php
```

7. Entramos a MARIADB

```shell
mysql -u root -p
```

Nos va a pedir contraseña, Presionamos "ENTER" ya que no hay contraseña porque como ya somos ROOT.

podemos ver la base de datos que hay

```sql
SHOW DATABASES;
```

#### Verificamos si todo lo que hicimos anda en nuestro PC Real

1. Asegurarse que la maquina virtual tenga en "Configuracion" -> "Red" -> "Adaptador Puente" y Aceptamos

2. Iniciamos Debian

3. Es necesario ejecutar este comando ya que despierta la placa de red de Debian, borra la configuración vieja que ya no sirve, y la sincroniza con el router real de tu casa.

```shell
dhclient
```

5. Luego para ver la IP que dio el Router

```shell
hostname -I
```

6. Con esto ya tenemos todo, comprobar en nuestro PC la url que nos dio el "hostname -I", en mi caso

```shell
http://192.168.0.42/index.php
```

7. Nos abre la pagina web

![[Pasted image 20260518105432.png |650]]

### 4. Configuracion de Red

1. Vemos como se llama la placa de red de wifi

```shell
ip a
```

2. Aparecen 2, la primera "lo", despues "enp0s3" ESTA ES LA QUE NOS INTERESA
3. Editamos el archivo de network

```shell
vi /etc/network/interfaces
```

4. Eliminamos "dhcp" de la linea "iface enp0s3 inet dhcp" y colocamos "static", agregamos los datos que son necesarios para configurar la red, deberia quedar algo asi:

```shell
iface enp0s3 inet static
    address 192.168.0.xxx -> la que vos quieras 
    netmask 255.255.255.0
    gateway 192.168.0.1
```

5. Reiniciamos la red

```shell
systemctl restart networking
```

NOTA: ME PASO QUE REINICIANDO EL SERVICIO, NO ME TOMABA LA NUEVA RED, SI PASA ESO,SIGNIFICA QUE VAMOS A FORZAR A APAGAR EL SERVICIO Y LUEGO LO VOLVEMOS A PRENDER, HAY QUE HACER LO SIGUIENTE

```shell
systemctl restart networking
ifdown enp0s3 --force
ifup enp0s3
```

6. Verificamos que quedo Estática

```shell
hostname -I
```

### 5. Almacenamiento

1. Agregamos un nuevo disco de 10 GB a la maquina virtual
2. Verificamos como se llama con "lsblk"

```shell
lsblk
```

3. Creamos las particiones del disco

```shell
fdisk /dev/sdc
```

4. Primera particion

```shell
n
p
1
Presionamos ENTER
+3G
```

5. Segunda particion

```shell
n
p
2
Presionamos ENTER
+6G
```

6. Presionamos W para salir

```shell
w
```

7. Con esto ya tenemos las 2 particiones creadas en el nuevo disco
8. Formateamos las particiones

```shell
mkfs.ext4 /dev/sdcx
```

9. Creamos los directorios

```shell
mkdir /www_dir
mkdir /backup_dir
```

10. Montamos los discos con los nuevos directorios

```shell
mount /dev/sdc1 /www_dir
mount /dev/sdc2 /backup_dir
```

11. Para verificar que lo hicimos bien

```shell
df -h
```

12. Movemos los archivos y le damos permisos por las dudas del TP a las nuevas particiones

```shell
cp /var/www/html/index.php /www_dir/
cp /var/www/html/logo.png /www_dir/
```

```shell
chmod 644 /www_dir/index.php
chmod 644 /www_dir/logo.png
```

13. Cambiamos la ruta de apache para que busque a /www_dir en vez de /var/www/html

```shell
vi /etc/apache2/sites-available/000-default.conf
```

Modificamos la linea "DocumentRoot /var/www/html" por "DocumentRoot /www_dir"

Guardamos cambios

14. Tambien hay que darle permiso al archivo "apache2"

```shell
vi /etc/apache2/apache2.conf
```

Agregamos esto abajo del original para que apunte a la nueva ruta

```shell
<Directory /www_dir/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>

```

Reiniciamos apache

```shell
systemctl restart apache2
```

15. Configurar el montaje automático al iniciar el sistema

```shell
vi /etc/fstab
```

Agregamos estas 2 lineas al final del archivo

```shell
/dev/sdc1 /www_dir ext4 defaults 0 2
/dev/sdc2 /backup_dir ext4 defaults 0 2
```

Esto sirve para que cada vez que arranque el sistema, las particiones se monten automáticamente en sus respectivas rutas.

16. Creamos el archivo `/opt/particion` para almacenar el contenido de `/proc/partitions`. 
 
```shell
cat /proc/partitions > /opt/particion
```

### 6. Backup

1. Creamos el direcotiro y el scritp

```bash
mkdir -p /opt/scripts
vi /opt/scripts/backup_full.sh
```

2. Creamos el script

```bash
#!/bin/bash
if [ "$1" = "-help" ] || [ -z "$1" ] || [ -z "$2" ]; then
	echo "Uso: $0 [origen] [destino]"
	exit 0
fi

FECHA=$(date +"%Y%m%d_%H%M%S")
B=$(basename "$1")

if [ ! -d "$1" ] || [ ! -d "$2" ]; then
	echo "ERROR: Carpetas no disponibles"
	exit 1
fi

tar -czf "$2/${B}_bkp_${FECHA}.tar.gz" -C "$(dirname "$1")" "$B"
echo "Backup Realizado"
```

3. Le damos permisos

```bash
chmod +x /opt/scripts/backup_full.sh
```

4. Probamos que funcione el "-help"

```bash
/opt/scripts/backup_full.sh -help
```

5. Probamos que funcione realmente el script

```bash
/opt/scripts/backup_full.sh /var/log /backup_dir
```

6. Si sale todo bien deberia salir "Backup Realizado"
7. Lo verificamos

```bash
ls -l /backup_dir
```

8. Deberia aparecer un archivo parecido a este "log_bkp_20260601.tar.gz"
9. Para hacerlo automaticamente el backup usamos crontab

```bash
crontab -e
```

10. Agregas estas 2 Lineas

```bash
0 0 * * * /opt/scripts/backup_full.sh /var/log /backup_dir
0 23 * * 1,3,5 /opt/scripts/backup_full.sh /www_dir /backup_dir
```

11. Verificamos que se guardo el archivo

```bash
crontab -l
```

12. Tuve problemas a la hora de hacer el backup, porque se sobreescribia el archivo, para arreglarlo encontra una menra que era la de poner los segundas en la varibale de la fecha **FECHA=$(date +"%Y%m%d_%H%M%S")**

### 7. Transferencia de archivos y GitHub

1. Verificamos que el servicio SSH estuviera funcionando correctamente en Debian.

```
systemctl status ssh
```

2. Habilitamos el acceso remoto del usuario root mediante contraseña editando la configuración de SSH.

```
vi /etc/ssh/sshd_config
```

Para realizar la transferencia mediante SCP se habilitó temporalmente la autenticación por contraseña para el usuario root.

Modificamos:

```
PermitRootLogin yes
PasswordAuthentication yes
```

3. Reiniciamos el servicio SSH para aplicar los cambios.

```
systemctl restart ssh
```

4. Desde la computadora anfitriona (host), comprobamos la conectividad con la máquina virtual.

```
ping 192.168.0.150
```

5. Probamos el acceso remoto por SSH.

```
ssh root@192.168.0.150
```

6. Creamos un directorio destinado a almacenar los archivos de entrega.

```
mkdir -p /root/entrega
```

7. Comprimimos individualmente los directorios solicitados por la consigna.

```
tar -czf /root/entrega/root.tar.gz /root
tar -czf /root/entrega/etc.tar.gz /etc
tar -czf /root/entrega/opt.tar.gz /opt
tar -czf /root/entrega/www_dir.tar.gz /www_dir
tar -czf /root/entrega/backup_dir.tar.gz /backup_dir
```

8. Comprimimos el directorio `/var`.

```
tar -czf /root/entrega/var.tar.gz /var
```

9. Debido al tamaño del archivo, dividimos el comprimido de `/var` en partes más pequeñas para facilitar su subida a GitHub.

```
split -b 100M /root/entrega/var.tar.gz /root/entrega/var.tar.gz.part_
```

10. Verificamos la correcta generación de todos los archivos.

```
ls -lh /root/entrega
```

11. Salimos de la sesión SSH para volver a la terminal de la computadora anfitriona.

```
exit
```

12. Copiamos el directorio de entrega desde la máquina virtual hacia la computadora anfitriona utilizando SCP.

```
scp -r root@192.168.0.150:/root/entrega .
```

Se utilizó SCP porque permite transferir archivos de forma segura entre computadoras mediante SSH.

13. Ingresamos la contraseña del usuario root cuando fue solicitada.

14. Verificamos que los archivos se descargaron correctamente.

```
ls -lh entrega
```

15. Confirmamos la presencia de los archivos requeridos para la entrega.

```
root.tar.gz
etc.tar.gz
opt.tar.gz
www_dir.tar.gz
backup_dir.tar.gz
var.tar.gz
var.tar.gz.part_aa
```

16. Se creó un repositorio en GitHub y se confeccionó el archivo `README.md` con los integrantes del grupo.

17. Se copiaron los archivos descargados al repositorio local y se agregaron al control de versiones.

```
git add .
```

18. Se realizó el commit de la entrega.

```
git commit -m "Entrega TP VMCA"
```

19. Finalmente, se publicó el contenido en GitHub.

```
git push origin main
```

20. Se verificó en GitHub la presencia del archivo `README.md` y de todos los archivos comprimidos solicitados por la consigna.
