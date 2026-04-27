1. Blanquerar la ISO
	init = /bin/bash
2. Remonta la partición raíz para poder escribir cambios
	`mount -o remount,rw /`
3. Cambiar la contraseña de Root
	1. `passwd root`
	2. `palermo`
	3. `palermo`
4. Reiniciamos la maquina virtual
5. Iniciamos sesión
	1. login: `root`
	2. password: `palermo`
6. Cambiamos nombre del host
	1. `hostnamectl set-hostname TPServer`
7. Actualizamos Debian
	1. vemos la version actual de Debian 
		1. `cd etc` `cat debian_version` = 11.10
	2. 