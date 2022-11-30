# Born2beRoot

## RESUMEN

Esta práctica es una introducción a la administración de sistemas. En esta prática se han interiorizado los principios de:

- La creación de **maquinas virtuales**
- La **instalación de SO** con unas reglas de particiones y encriptación estrictas
- **Seguridad**. Con la implementación de sudo, un firewall y una política de contraseñas fuerte
- **Acceso remoto**. Implementando un servidor SSH
- **Gestión de usuarios**. Trabajando con usuarios, grupos y sus privilegios
- **Monitorización**. Sabiendo como extraer información sobre la maquina (RAM disponible, número de conexiones activas, comandos ejecutados mediante sudo...)
    
    **Software utilizado**
    
    - VM: [VirtualBox](https://www.virtualbox.org/) 6.1
    - OS: [Debian bullseye](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/) 11.3.0 ([Debian vs CentOS](https://github.com/ampuEus/42School/blob/main/cursus/lvl1/Born2beroot/annex/1_Debian_VS_CentOS.es.md))

<aside>
💡 `ssh <user>@<ipaddress> -p 4242`

</aside>

<aside>
💡 Los servidores no deben reiniciarse para que no se interrumpan los procesos, es mejor hacer LOGOUT y LOGIN otra vez

</aside>

Comandos utiles

`su root`  —> cambiar a usuario root

`whoami` —> saber con que usuario estoy logueado

`dpkg -l`

`dpkg -l | grep ufw` —> me busca en la lista de paquetes el ufw. Para comprobar si se ha instalado bien.

`sudo apt list php`     // comprobar la versión de un programa antes de instalar o si lo tenemos instalado 

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled.png)

`env`  // ver detalles del sistema

Keep the system updated automatically:

`apt-get install unattended-upgrades`

## SYSTEMCTL

Se controla todo, veo los detalles de los procesos, el PID, si están activos…

`sudo systemctl **status** <process_name> or PID`   **********// comprobar info sobre un programa

START in the current session:

`sudo systemctl **start** <program_name>`   

STOP in the current session:

`sudo systemctl **stop** <program_name>`

RESTART a process:

`sudo systemctl **restart** <process_name>`  //reiniciar el proceso sin desloguearse o reiniciar todo el servidor

Enable/Disable at SYSTEM BOOT:

`sudo systemctl **enable** <program_name>`   // iniciar al iniciar el BOOT del sistema

`sudo systemctl **disable** <program_name>` // dejar de iniciar en BOOT del sistema

[systemctl Commands: Restart, Reload, and Stop Service](https://www.linode.com/docs/guides/introduction-to-systemctl/)

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%201.png)

## Instalación de DEBIAN

### Snapshots

Son fundamentales, es como ir tomando puntos de restauración / copias de seguridad

Menu / Machine / Take Snapshot (la maquina tiene que estar funcionando

### Set-up de VirtualBox

He seleccionado como red Bridged Adapter, para tener acceso desde el host a las direcciones IP y poder conectarme a la máquina virtual por SSH sin problema.

Discos siempre en VDI y flexible, para que se adapte.

# Disks, partitions and volumes

**Primero hay que crear una tabla de particiones. La tabla de particiones va a guardar la información sobre nuestras particiones (como una tabla o lista del resto de particiones.**

En el siguiente paso se te mostrará la nueva tabla de particiones creada donde hay que ir haciendo la segmentación que se quiere. La primera partición que se va ha crear va ha ser la de boot, que NO debe estar encriptada, ya que si no ni siquiera te arrancaría el OS. Selecciona el **espacio libre** que hay en el disco.

Se recomienda crear una partición para BOOT (unos 500mb suele ser suficiente), que es una partición FÍSICA (como si fuera un disco físico, la puede leer la BIOS). Se montará en /boot

Luego otra partición (LOGICAL, solo a nivel de software) para guardar cosas (en mi caso sda5, de 15gb), que puede ser ENCRYPTED. Hay dentro haremos otras particiones más, cada una en su sitio:

He creado 2 particiones en mi disco de 15GB. Una de 500MB que se monta automáticamente en /boot y es la partición PRIMARY. La de 15GB es una partición LOGICAL.

Dentro de sda5, se pueden crear más particiones separadas y enviar cada una a su carpeta.

<aside>
💡 En entornos de servidor se suele hacer una partición para cada carpeta, de manera que si se borra o estropea una, es como reemplazar ese disco únicamente, mientras el resto no se ve afectado y sigue funcionando.

</aside>

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%202.png)

Ahora tenemos que configurar el LOGICAL VOLUME MANAGER (LVM).

Configure Logical Volume Group

Va a ser para crear los grupos de discos dentro de sda5.

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%203.png)

Ahora, vamos a Create Logical Volume, vamos a ir creando los diferentes volumenes en sda5. Serán volumenes separados.

1. Tabla de particiones (guarda la info de las particiones)
2. Particiones (sda1, sda5)
3. Volumenes (sda5/home, sda5/root etc) son volumenes independientes.

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%204.png)

Luego, una vez creados los volumenes, hay que seleccionar sus formatos y donde queremos que se monten, a cada uno lo suyo.

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%205.png)

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%206.png)

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%207.png)

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%208.png)

GRUB

Es un boot loader / boot manager para Linux

Guia sobre instalación de Debian

[Born2beroot VB VM Installation (Bonus)](https://www.youtube.com/watch?v=2w-2MX5QrQw)

Guia sobre particiones

[Linux Partitioning Recommendations](https://averagelinuxuser.com/linux-partitioning-recommendations/#disk-selection)

# Starting VM & setup SSH

SSH: Secure Shell Conexion

Bien con el usuario root para seguir configurando o bien puedo cambiar de usuario con:

`su root`

Ahora que ya estoy en el usuario root.

Como root, ahora vamos a instalar sudo, que nos permite ejecutar comandos como superusuario root sin tener que estar cambiando cada vez.

`apt install sudo`

También vamos a dar permisos de sudo a alejarod:

`sudo addgroup nombre_usuario sudo`

Una vez añadido el usuario al grupo, hay que refrescar la información de login de dicho usuario. Esto se hace deslogueandote y volviéndote a loguear:

`logout` # Con este comando te deslogueas como usuario root

`exit` # Con este segundo exit te deslogueas como tú usuario normal

<aside>
💡 **Importante: Cada vez que nos logueamos con un usuario, estamos creando una capa nueva, por lo que es mejor hacer LOGOUT para no ir creando capas superpuestas**

</aside>

Ahora que alejarod ya está dentro del grupo sudo, ya podemos ejecutar comandos de superusuario (su o root) poniendo el prefijo sudo por delante del comando.

Primero voy a instalar SSH para hacer login remoto al CLIENTE desde un SERVIDOR:

`sudo apt install openssh-server`

Configurar SSH

> *etc is the folder where the configuration files are*
> 

vamos al archivo 

`/etc/ssh/sshd_config`

y lo editamos con vim

`vim sshd_config`

<aside>
💡 `sshd` is what listens for an incoming connection – i.e. when you attempt to connect via SSH from your local PC/Mac/Laptop/etc. So, at a server level, we need to config this file to be able to log in remotely from another device. *Some distros already come with sshd installed by default so we can directly access the server from another PC (Client) using the SSH command.*

</aside>

Como puedes ver, el servidor esta preconfigurado para que acepte las conexión procedentes del puerto 22, mira la línea `Server listening on :: port 22`. Para que el servidor solo acepte conexiones que vengan del puerto 4242 tienes que modificar la configuración por defecto del ssh. El archivo de configuración está en */etc/ssh/sshd_config*.

En este caso me quiero conectar por el puerto 4242 y no quiero habilitar usuario y contraseña remota para hacerlo más seguro.

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%209.png)

Comprobar si SSH está activo

`sudo systemctl ssh status`

Una vez guardado el servidor ssh devería cambiar el puerto de escucha automáticamente, si no es así prueba a reiniciar el servicio ssh.

`sudo systemctl restart ssh`

Falta abrir el puerto correspondiente en el Firewall.

Por último, hay que autorizar al firewall UFW que autorice las conexiones de entrada por el puerto 4242. Además de añadir esta regla, es posible que tengas que eliminar una nueva regla que permite las conexiones por el puerto 22 que se ha creado automáticamente al instalar OpenSSH.

`sudo ufw allow 4242
sudo ufw delete allow ssh` # To delete port 22 rule

# Firewall (UFW), puertos, AppArmor

UFW: Uncomplicated Firewall

### **INSTALACIÓN Y ACTIVACIÓN**

Para instalar [UFW](https://github.com/ampuEus/42School/blob/main/cursus/lvl1/Born2beroot/annex/7_Que_es_UFW.md) haz:

`sudo apt install ufw
sudo ufw enable`

Si todo ha ido bien tiene que aparecerte el mensaje *Firewall is active and enabled on system startup*. Además, para ver si efectivamente UFW se está corriendo en segundo plano puedes ejecutar el siguiente comando:

`sudo ufw status verbose # NOTA: La opción "verbose" es opcional, solo añade más información a la respuesta`

### **GESTIÓN DE LAS REGLAS**

Aunque de momento no es necesario aplicar ninguna regla en concreto, voy a enseñarte los primeros pasos para que sepas como crear, eliminar y monitorizar las reglas.

- Creación de reglas

`sudo ufw allow 4242` # Permite la entrada de tráfico por el puerto 4242
`sudo ufw deny 8080` # Deniega la entrada de tráfico por el puerto 8080

> Estas reglas no se aplican únicamente a los puertos de tú servidor, también se pueden aplicar a IPs por ejemplo.
> 
- Monitorización de reglas activas

`sudo ufw status numbered`

- Eliminación de reglas

`sudo ufw delete allow 4242
sudo ufw delete deny 8080`
# Otra forma sería usando el id que obtienes con el comando de status numbered
`sudo ufw delete 2`

> NOTA: Ten cuidado porque si intentas borrar más de una regla a la vez usando el id tienes que ordenarlos de mayor a menor, ya que si empiezas borrando los de menor valor los de mayor valor irán tomando esos valores.
> 

### **IMPLEMENTACIÓN DE AppArmor: Limitación de sudo**

Desde Debian 10 [AppArmor](https://github.com/ampuEus/42School/blob/main/cursus/lvl1/Born2beroot/annex/6_Que_es_AppArmor.md) viene preinstalado en Debian, por lo que yo usaré este mismo (además de ser más fácil de utilizar). Para ver AppArmor está instalado y corriendo correctamente hay que ejecutar lo siguiente:

`sudo aa-status`

Y tiene que aparecerta algo parecido a esto:

`apparmor module is loaded.
3 profiles are loaded.
[...]`

En este caso si todo está bien deja la configuración por defecto.

# SSH login

Un poco de teoría y crear llave pública y privada para servidores:

 [SSH - Secure Shell](https://www.notion.so/SSH-Secure-Shell-fafc5f4b88be41c98798a054526a25a6) 

**Obtener la ip adress.** SSH en su máquina virtual usando el puerto 4242. Para obtener el ip-address deberemos escribir en la máquina virtual:

`ip a s`

Nos aparecerá la ip. Debemos buscarla donde pone inet. Los números escritos detras de inet hasta el “/”.

[https://lh6.googleusercontent.com/2QT4NhUyQFSdQK5EqlP-IlejJkXs5d_WzAhEdqagNbL41-hBgEwstJSLgdPUPCKqWFHOm0KCGnoxQl8qdwndgQWGIPQxKl0IeNK0J4swvCeI1uQZAeBYnl-qa7FncF9_F1QWxYkIedJJe3hX7zY5Kp9fJVMdTpKRZI-6kKOWiFMb2jBtnf38Syn3](https://lh6.googleusercontent.com/2QT4NhUyQFSdQK5EqlP-IlejJkXs5d_WzAhEdqagNbL41-hBgEwstJSLgdPUPCKqWFHOm0KCGnoxQl8qdwndgQWGIPQxKl0IeNK0J4swvCeI1uQZAeBYnl-qa7FncF9_F1QWxYkIedJJe3hX7zY5Kp9fJVMdTpKRZI-6kKOWiFMb2jBtnf38Syn3)

En el apartado:

1: nos indica la ip para la red local (**lo = local**). Es decir, **dentro** del propio ordenador.

2: nos indica la ip para la red externa **(enp0s3 = adaptador creado por VirtualBox)**. Es decir, **fuera** del propio ordenador. Posibles usos:

- Otro ordenador se conecta en remoto a nuestro ordenador.
- También es la ip fija que puedes definir en tu router para conectarte a internet.
- La máquina virtual se puede dirigir desde otra terminal diferente a la suya, por ejemplo la máquina tiene la terminal de linux y con la ip puedes conectarte desde la terminal de mac. Esta es la función que vamos a utilizar en el siguiente punto.

<aside>
💡 **Trabajar desde la terminal de mac**. En la terminal del mac deberemos escribir:

</aside>

Para conectarte desde otro ordenador te hace falta tener un cliente ssh instalado. Suponiendo que ya lo tengas, lo único que tienes que hacer es abrir una terminal y

`ssh alejarod@localhost -p 4242
└┬┘ └──┬───┘ └───┬───┘  │ └┬─┘
 │     │         │      │  └────> Puerto por el que quieres conectarte
 │     │         │      └────> Opción de poder elegir otro puerto que no sea el de por defecto (por defecto es el 22)
 │     │         └────> IP a la que quieres conectarte
 │     └────> Usuario del sistema al que quieres conectarte
 └────> Comando para usar el cliente ssh`

<aside>
💡 `ssh <usuario>@<ip-address> -p 4242`

</aside>

Veremos que aparece nuestro usuario en color verde, eso significa que estaremos trabajando a distancia, en otra máquina.

- Finalice la sesión SSH en cualquier momento a través de logout.

`logout`

- Alternativamente, termine la sesión SSH.

`exit`

# SUDO pass & log setup

Por temas de seguridad, la práctica Born2beroot exige hacerle unas configuraciones extra a los ajustes de sudo que viene por defecto:

- El grupo sudo tiene un máximo de 3 intentos para introducir la contraseña correcta
- Cuando se utilice sudo y la contraseña sea incorrecta debe salir un mensaje personalizado
- Para cada comando ejecutado con sudo, tanto el input como el output deben quedar archivados en el directorio */var/log/sudo/*.
- El modo TTY debe estar activado por razones de seguridad.

Para añadir estos parámetros al programa sudo hay que modificar el fichero *sudoers.tmp*. Para abrir/modificar este fichero es recomendable utilizar el propio comando que te proporciona sudo, en vez de abrirlo como si fuera un fichero normal. Además de tener que abrirlo como usuario root.

`sudo visudo`

Ahora tienes que añadir las siguientes líneas al fichero:

```
Defaults passwd_tries=3 # Se expecifica un máximo de 3 intentos
Defaults badpass_message="Logging atempts with wrong password, try again." # Mesaje de error personalizado
Defaults logfile="/var/log/sudo/sudo.log" # Lugar donde se guardarán las interacciones con sudo
Defaults log_input # Guarda los input
Defaults log_output # Guarda los output
Defaults requiretty # Obliga a estar TTY abierto
```

> Si no existe el directorio donde quieres guardar los logs, tendrás que crearlo con mkdir.
> 

[https://lh4.googleusercontent.com/_7DGEOo45Is9He7Zyhej0sh84todiRTqcB4RS9ACgu7cJnQKNt903sMN974HDeEiDXD_qjCflrnWzaSIRS0OoTewZoTERUjqkvKQz9DCMuuZ_OYVcsuSALB_Q00Rw4-NRYtj4Kx4bYPVH-SmpSNrDYyKHd1UC2a_im4cwIwvzm7IZh6dAIKhOs_W](https://lh4.googleusercontent.com/_7DGEOo45Is9He7Zyhej0sh84todiRTqcB4RS9ACgu7cJnQKNt903sMN974HDeEiDXD_qjCflrnWzaSIRS0OoTewZoTERUjqkvKQz9DCMuuZ_OYVcsuSALB_Q00Rw4-NRYtj4Kx4bYPVH-SmpSNrDYyKHd1UC2a_im4cwIwvzm7IZh6dAIKhOs_W)

# Secure Password Policy

location: etc/login.defs —> login definitions

`sudo vim /etc/login.defs`

<aside>
💡 Hay que ejecutar vim con sudo para que no de problemas de readonly files

</aside>

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2010.png)

Pero cuidado, estos cambios no se aplican de forma automática a los usuarios preexistentes (el *root* y *alejarod* en mi caso). Para modificar estas reglas necesitas usa el comando  `chage`:

`sudo chage -M 30 root
sudo chage -M 30 alejarod
sudo chage -m 2 root
sudo chage -m 2 alejarod
sudo chage -W 7 root
sudo chage -W 7 alejarod`
# Para comprobar, el flag "-l" sirve para ver las reglas que tiene puestas
`sudo chage -l root
sudo chage -l <username>`

Para aplicar las demás características hace falta que te descargues el siguiente paquete:

`sudo apt install libpam-pwquality`

Y para imponer las características que se quieren en las contraseñas hay que configurar el archivo */etc/security/pwquality.conf*

> # Number of characters in the new password that must not be present in the
# old password.
`difok = 7`
# The minimum acceptable size for the new password (plus one if
# credits are not disabled which is the default)
> 

> `minlen = 10`
# The maximum credit for having digits in the new password. If less than 0
# it is the minimun number of digits in the new password.
> 

> `dcredit = -1`
# The maximum credit for having uppercase characters in the new password.
# If less than 0 it is the minimun number of uppercase characters in the new
> 

> `# password.
ucredit = -1`
> 

> # The maximum number of allowed consecutive same characters in the new password.
# The check is disabled if the value is 0.
`maxrepeat = 3`
# Whether to check it it contains the user name in some form.
# The check is disabled if the value is 0.
`usercheck = 1`
# Prompt user at most N times before returning with error. The default is 1.
retry = 3
# Enforces pwquality checks on the root user password.
# Enabled if the option is present.
> 

> `enforce_for_root`
> 

> Por último, cambia las contraseñas del root y tú usuario para que cumplan con la nueva política de contraseñas:
> 

> `sudo passwd <user/root>`
> 

# Users, Groups, Hostname

### Current Users Logged

`users`    // I can see the users that are logged in (layers and terminals)

`who` // details of the users that are logged in (layers and terminals)

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2011.png)

In this example alejarod was logged twice in the original machine and through ssh at the same time (2 sessions or layers)

### USERS

`getent passwd`     ver la database de usuarios

getent group.          ver la database de grupos

Los puedo cortar con este comando

`getent passwd | awk **-F:** '{ print $1}'`.  Por defecto, separa las columnas cuando hay espacio. -F: le dice que el separador son dos puntos (como un CSV en excel)

`su user_name`     //cambiar a usuario

`sudo **useradd** user_name`

`sudo **userdel** user_name`. // borrar usuario

`sudo **passwd** user_name`.  //Cambiar password

Verifique la información de caducidad de la contraseña del usuario recién creado.

`sudo chage -l nombre_usuario`

### **GROUPS**

`getent group`      // ver los grupos

`getent group group_name` // ver los usuarios de un grupo

Añadir a un grupo:

`addgroup <user_name> <group_name>`

Borrar grupo

`groupdel <group_name>`

### **2.7.1. MODIFICAR EL HOSTNAME**

Para modificar el hostname tienes 2 opciones:

1. Usar el comando `sudo hostnamectl set-hostname <new_hotsname>`
2. Editarlo directamente el archivo */etc/hostname*
3. NOTA: Si te sale el error `sudo: unable to resolve host <nuevo nombre del host>: Name or service not known` también deberás añadirlo al archivo */etc/hosts*

Para comprobar que efectivamente el nombre ha cambiado puedes ejecutar:

`hostnamectl status`

> Si ves que todavía no ha cambiado el nombre, prueba a reiniciar el equipo.
> 

# Script monitor

[script de Dampuru](https://www.notion.so/script-de-Dampuru-76ba54578c1a429792834cda869e3434)

| #!/bin/bash | Se usa al comienzo del script para indicar al Sistema operativo que use bash como intérprete. |
| --- | --- |
| $(comando) | Permite usar el resultado del comando entre paréntesis como una variable más. |
| awk | Herramienta/lenguaje que permite manipular información en Linux
Por ejemplo:
• awk ‘NR==5’ {print$2} (Muestra la línea 5 del segundo elemento)
• awk ‘$NF==”/”’ (Muestra las líneas cuyo último campo = “/”. (NF = Nb of fields) |
| grep | Global Regular Expression Print. Busca el patrón indicado a continuación. |
| if [... -gt0] | Comprueba si el resultado del comando es mayor que 0 |
| wc | Cuenta el número de elementos.
-l (Para líneas)
-w (Para palabras)
-c (Para bytes) |

| uname -a | Arquitectura del Sistema operativo (“-a” All info) |
| --- | --- |
| lscpu | Muestra información de la arquitectura de la CPU |
| free - - mega | Muestra en MB la memoria libre, usada total, etc. |
| df -h | Informa del espacio usado. (“h” human readable, en MB, GB, etc) |
| top -bn1 | • n: Indica el número de iteraciones a mostrar ( En este caso , 1)
• b: Bach mode. Permite enviar la salida del comando a archivos o programas. |
| who | Información sobre quién está logueado
-b: hora de ultimo boot
-a: all |
| hostname | -a: Alias
-d: Domain
-A: All
-i: IP
-l: All IP |
| ip -a | Muestra/edita parámetros de red. -a (address) |
| pvdisplay | Muestra propiedades físicas de cada volumen físico (Tamaño, extensiones, grupo, …) |
| lsblk | Lista de dispositivos de bloque, que son archivos especiales que hacen referencia o representan cualquier dispositivo del PC. |
| crontab | Permiten automatizar tareas
-e (Para visualizar el archivo crontab)
-l (Para visualizar) |

Lo primero que tenemos que hacer es crear un archivo en la carpeta /root , esto es debido a que dentro de nuestro script tendremos que utilizar comandos que necesitan permisos de sudo y si lo hacemos en /home tendremos problemas a la hora de mostrar *LVM use* y *Sudo.* Por lo tanto, tendremos que acceder al sistema como **root***.*

`touch /root/monitoring.sh`              //crearlo

`chmod 755 /root/monitoring.sh`     // darle permisos de ejecución

`vim /root/monitoring.sh`   // editamos el script 

<aside>
💡 Los archivos deben comenzar por `#!/bin/bash` y tener permisos de ejecución

</aside>

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2012.png)

#!/bin/bash

#Arquitecture

echo “#Architecture:" $(uname -a)

#CPU Physical

echo "#CPU physical:" $(lscpu | awk ' NR==5 {print $2}')

#Virtual CPUrm

echo "#vCPU :" $(lscpu | grep Socket\(s\) | awk '{print $2}')

#Memory Usage

free --mega | awk 'NR==2{printf "#Memory Usage: %s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }'

#Disk Usage

df -h | awk '$NF=="/"{printf "#Disk Usage: %d/%dGB (%s)\n", $3,$2,$5}'

#CPU Load

top -bn1 | grep load | awk '{printf "#CPU Load: %.2f%s\n", $(NF-2), "%"}'

#Last Boot

echo "#Last boot:" $(who -b | awk '{print($3 " " $4)}')

#LVM

echo "#LVM use:" $(if [ $(lsblk | grep lvm | wc -l) -eq 0 ]; then echo no; else echo yes; fi)

#Connections TCP

echo "#Connections TCP:" $(ss -s | grep TCP | awk 'NR==2 {printf "%d ESTABLISHED\n", $3}')

#User log

echo "#User log:" $(who | wc -l)

#Network IP

echo "#Network: IP" $(hostname -I) $(ip a | grep link/ether | awk '{printf " (%s)\n", $2}')

#Sudo

echo "#Sudo : " $(cat /var/log/sudo/sudo.log | grep USER | wc | awk '{printf "%s cmd\n", $1}')

**OUTPUT de este SCRIPT**

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2013.png)

#Architecture

`echo “#Architecture:" $(uname -a)`

`uname -a` nos da los detalles del sistema

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2014.png)

#CPU Physical

`lscpu`   // ver datos de la CPU

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2015.png)

`echo "#CPU physical:" $(lscpu | awk ' NR==5 {print $2}')`

NR == number of record

imprime “string” variable() ejecuta lscpu y muestra por pantalla la línea 5 y columna 2

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2016.png)

#Virtual CPUrm

`echo "#vCPU :" $(lscpu | grep Socket\(s\) | awk '{print $2}')`

aqui como antes, pero ahora, buscamos Socket con grep y luego sacamos esa info, pero bajo el nombre #vCPU

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2017.png)

#Memory Usage

`free --mega`

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2018.png)

`free --mega | awk 'NR==2{printf "#Memory Usage: %s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }'`

ver memoria libre en megabytes. Puedo meter un printf dentro de awk: 

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2019.png)

#Disk Usage

`df -h`

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2020.png)

`df -h | awk '$NF=="/"{printf "#Disk Usage: %d/%dGB (%s)\n", $3,$2,$5}'`

NF == number of fields

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2021.png)

#CPU Load

`top` (ver los procesos del sistema). man top para mas detalles

man top

`top -n | grep load.`    me busca la palabra load (aunque no se vea en pantalla)

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2022.png)

`top -bn1 | grep load | awk '{printf "#CPU Load: %.2f%s\n", $(NF-2), "%"}'`

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2023.png)

#Last boot

`who`      ver el usuario actual

`who -b`   ver cuando se booteo el sistema

`echo "#Last boot:" $(who -b | awk '{print($3 " " $4)}')`

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2024.png)

#LVM

`echo "#LVM use:" $(if [ $(lsblk | grep lvm | wc -l) -eq 0 ]; then echo no; else echo yes; fi)`

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2025.png)

#Connections TCP

`echo "#Connections TCP:" $(ss -s | grep TCP | awk 'NR==2 {printf "%d ESTABLISHED\n", $3}')`

`ss`               //muestra info de IP y puertos

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2026.png)

#User log

`echo "#User log:" $(who | wc -l)`. //count the number of users logged

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2027.png)

#Network: IP

`hostname -I (i mayus)`. // ver IP del host

ip a     // ver detalles de IP

echo "#Network: IP" $(hostname -I) $(ip a | grep link/ether | awk '{printf " (%s)\n", $2}')

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2028.png)

#Sudo

echo "#Sudo : " $(cat /var/log/sudo/sudo.log | grep USER | wc | awk '{printf "%s cmd\n", $1}')

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2029.png)

abre el log de los comandos sudo, busca GREP la pallabra user, cuenta WC, e imprime ese numero de veces 

# Cron / Crontab

Cron:

Herramienta predeterminada de Linux que automatiza tareas/procesos periódicamente.

Crontab:

Archivo de texto que establece reglas para Cron.

`man crontab`

`crontab -u ********username********`

`crontab -l`                          // display the current crontab

`crontab -e`                        // edit the current crontab

`crontab -u root -e`.          // abrir crontab como root y editarlo (recomendado)

<aside>
💡 Cada usuario tiene su propio crontab, si abrimos un crontab con otro usuario aparecerá vacio

</aside>

Si queremos parar el Script

`crontab -u root -e`    //comentamos la línea de la tarea

### **2.5.2. ENVIAR EL MENSAJE CADA 10 MINUTOS**

Para este cometido existe el servicio `crontab` (**c**h**ron**o **tab**le) que te permite la ejecución de scripts o software de forma automática cuando ocurre un evento en concreto que tú le hayas asignado, como puede ser una fecha y hora determinada o en un intervalo de tiempo específico.

Este es un programa que viene preinstalado con el propio Debian y para que se ejecute en segundo plano siempre que arranques el equipo usa el siguiente comando:

`systemctl enable cron`

Para programas los trabajos *cron* usa archivos llamados *crontab*. Logueado como root tienes que crear uno de estos archivos.

`crontab -e * * * * * <command to execute>`
         │ │ │ │ │ └────> Selecciona el día de la semana (0 - 6, 0 = Domingo)
         │ │ │ │ └────> Selecciona el mes del año (1 - 12)
         │ │ │ └────> Selecciona el día de la semana (1 - 31)
         │ │ └────> Seleciona la hora (0 - 23)
         │ └────> Selecciona el minuto (0 - 59)
         └────> Edita los crontab del usuario

> NOTA: '*' significa cualquier valor
> 

Y para que la salida del comando se muestre en todas las terminales hay que añadirle el comando `wall`, otra manera de hacerlo es introducir el comando `wall` directamente en el script.

`crontab -e */10 * * * * ./root/monitoring.sh | wall`

Aun así, este script se ejecutará cada 10 minutos de cada hora (00:10, 00:20, 00:30 ...) no cada 10 minutos desde que se **arranca el sistema**. 

El subject pide ejecutar cada 10 minutos desde el arranque.

Para conseguir esto tendrás un nuevo script que combine el comando `sleep` y que sepa a qué hora se ha iniciado en sistema.

### **2.5.3. CALCULAR CUANDO SE HA INICIADO EL SISTEMA**

Crea un nuevo script llamado *sleep.sh*. Este script, calculará la cantidad de segundos entre el tiempo de arranque preciso y el décimo minuto. Para hacer los cálculos puedes usar la calculadora basada en terminal `bc`, por lo que primero tienes que instalarla en el sistema

`sudo apt install bc`

Por otro lado, el código fuente de sleep.sh es:

#!bin/bash

# Get boot time minutes and seconds
BOOT_MIN=$(uptime -s | cut -d ":" -f 2)
BOOT_SEC=$(uptime -s | cut -d ":" -f 3)# Calculate number of seconds between the nearest 10th minute of the hour and boot time:
# Ex: if boot time was 11:43:36
# 43 % 10 = 3 minutes since 40th minute of the hour
# 3 * 60 = 180 seconds since 40th minute of the hour
# 180 + 36 = 216 seconds between nearest 10th minute of the hour and boot
DELAY=$(bc <<< $BOOT_MIN%10*60+$BOOT_SEC)# Wait that number of seconds
sleep $DELAY

Por último, añade este archivo al comando de crontab, de este modo cron llamará a `sleep.sh` y este pasado *X* tiempo llamará a `monitoring.sh`:

- `/10 * * * * bash /root/sleep.sh && bash /root/monitoring.sh`

# BONUS: Setting up a website

[Introducción a páginas web](https://www.notion.so/Introducci-n-a-p-ginas-web-d2c09e469c09461892f132d24212c68e) 

En la parte bonus aparte de crear las particiones extra que se pedían a la hora de la instalación del SO, hay que crear una página web con **WordPress**. Y como backend te dicen que debes implementar lo siguiente:

- El servidor HTTP a implementar tiene que ser **Lighttpd**
- Para programar el servidor para la parte del backend hay que usar **PHP**
- En cuanto a la implementación de la base de datos se va ha usar **MariaDB**

### 3.1. PHP

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2030.png)

En el momento en que estoy escribiendo esta guía, A la hora de instalar PHP si ejecutas `sudo apt list php` puedes ver como en Debian 11 la versión estable de PHP es la 7.4, aunque ya este la versión 8 disponible. En un principio con la versión 7.4 no tendrías que tener ningún problema. Además de PHP 7.4 también hacen falta los paquetes* php-common*, *php-cgi*, *php-cli* y *php-mysql* ya que son necesarios para WordPress.

`sudo apt update
sudo apt install php php-common php-cgi php-cli php-mysql
php -v # Para comprobar que versión se ha instalado`

### 3.2. LIGHTTPD

[Servidores WEB](https://www.notion.so/Servidores-WEB-b2ed4e106d554757a237c7e0c05080da)

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2031.png)

Antes de empezar con la instalación de lighttpd, si has instalado primero PHP es posible se te haya instalado apache2 como dependencia de php (para comprobar si está o no en el sistema puedes ejecutar `sudo apt list apache2` y si al final del paquete pone *[installed]* es que está instalado), por lo que para evitar posibles conflictos si está instalado ejecuta `sudo apt purge apache2` para eliminarlo del sistema.

Para instalar lighttpd simplemente tienes que ejecutar:

`sudo apt install lighttpd
lighttpd -v # Para comprobar que efectivamente se ha instalado`

Una vez instalado, toca activarlo y hacer que el servicio empiece a funcionan en segundo plano al arrancar el ordenador.

`sudo systemctl start lighttpd
sudo systemctl enable lighttpd
sudo systemctl status lighttpd`

Una vez comprobado con el comando `status` que el servicio esta ejecutándose, tienes que decirle al firewall que permita pasar tráfico con el protocolo *http* (o lo que es lo mismo por el puerto 80 y protocolo TCP/IP)

`sudo ufw allow http
sudo ufw status`

<aside>
💡 Si hemos conectado la máquina con BRIDGED ADAPTER, directamente poniendo la IP en el navegador ya nos sale la web. (ip a para encontra la IP)

</aside>

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2032.png)

> **En caso de que estemos conectados por NAT, tenemos que habilitar los puertos:**
> 
> 
> Y como ya has hecho con el puerto 4242 a la hora de permitir conexiones con SSH, tienes que volver a enrutar los puertos de la VM con los de su host. *Settings >> Network >> Adapter 1 >> Advanced >> Port Forwarding*  y para que el host pueda seguir usando el puerto 80 vas a usar su puerto 8080, así no tendrás interferencias con el tráfico de datos:
> 

![https://github.com/ampuEus/42School/raw/main/cursus/lvl1/Born2beroot/img/81_PortForward_Lighttpd.png](https://github.com/ampuEus/42School/raw/main/cursus/lvl1/Born2beroot/img/81_PortForward_Lighttpd.png)

Para testar que efectivamente el servidor web de Born2beroot está funcionando correctamente intenta conectarte a él. Vete al navegador del host de la VM y como la VM esta alojada en tú propio ordenador como dirección escribe la *localhost* (o 127.0.0.1) y como puerto el 8080 (que en realidad es el 80 de la VM).

![https://github.com/ampuEus/42School/raw/main/cursus/lvl1/Born2beroot/img/82_Lighttpd_server_working.png](https://github.com/ampuEus/42School/raw/main/cursus/lvl1/Born2beroot/img/82_Lighttpd_server_working.png)

¡PERFECTOOO! Ya tienes tu servidor web corriendo correctamente en la VM.

### 3.3. FastCGI (**Fast** **C**ommon **G**ateway **I**nterface)

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2033.png)

Pero todavía queda hacer que se vean las páginas dinámicas que se crean con PHP. A la hora de instalar PHP se ha creado un directorio donde dejar las páginas web creadas con PHP */var/www/html/*, para crear una página simple que muestre la información de php ejecuta:

`sudo vim /var/www/html/info.php`

Dentro del script escribe:

`<?php
phpinfo();
?>`

Si buscas la dirección *[http://localhost/info-php](http://localhost/info-php)* en tú navegador te responderá con el error *403 Forbidden*.

Para arreglar esto tienes que entender el paradigma de las páginas web dinámicas y estáticas. En las **webs estáticas**, el servidor web enviaba inmediatamente páginas HTML pre-escritas para cada solicitud HTTP que recibía. Por otro lado, en las **webs dinámicas**,
 el servidor web no responde de inmediato, sino que transfiere los datos de la solicitud HTTP a una aplicación externa (en este caso PHP) que los interpreta y le devuelve el código HTML generado al servidor web, y el este responde a la solicitud.

**En la comunicación entre la aplicación externa y el servidor web** es donde entra FastCGI, el cual es un protocolo binario que permite dicha comunicación. Necesitas configurar este protocolo entre lighttpd y PHP para poder acceder a la página *info.php* desde un navegador web.

`sudo lighty-enable-mod fastcgi` # Habilita el modo CGI

`sudo ligthty-enable-mod fastcgi-php` # Habilita la comunicación CGI con PHP

`sudo service lighttpd force-reload` # Reinicia el servicio para efectuar los cambios

Ahora si intentas leer la página *info.php* en tú ordenador veras algo parecido a esto:

![https://github.com/ampuEus/42School/raw/main/cursus/lvl1/Born2beroot/img/83_Lighttpd_and_PHP_working.png](https://github.com/ampuEus/42School/raw/main/cursus/lvl1/Born2beroot/img/83_Lighttpd_and_PHP_working.png)

### 3.4. MariaDB

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2034.png)

Para que el Wordpress almacene los datos de la web hay que usar MariaDB. Para instalarlo haz:

`sudo apt install mariadb-server`

Ahora iniciarlo y haz que se ejecute siempre en segundo plano al arrancar el equipo.

`sudo systemctl start mariadb
sudo systemctl enable mariadb
systemctl status mariadb`

Para hacer una [instalación más segura](https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html) ejecuta:

`sudo mysql_secure_installation`

Después toca rellenar los campos que te piden al ejecutar el comando anterior

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2035.png)

*Enter current password for root (enter for none): <Enter>
Switch to unix_socket authentication [Y/n]: Y
Set root password? [Y/n]: Y
New password: Andreider42$
Re-enter new password: Andreider42$
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]:  Y
Reload privilege tables now? [Y/n]:  Y*

Por último, reinicia MariaDB para que se implemente la nueva configuración.

`sudo systemctl restart mariadb`

Una vez instalado, vas a loguearte a MariaDB:

`sudo mariadb`

> Antes de empezar a usar la base de datos, tienes que saber que hay que usar su propio lenguaje (SQL) para poder gestionar las bases de datos y sus tablas. Yo te recomiendo que hagas una pausa y aprendas como hacer CRUD (Create, Read, Update y Delete) en SQL (aquí te dejo un cheatsheet [SQL](https://www.notion.so/SQL-6af72c0c7aeb4368a364e255fdec84a2) y aquí un compilador online para que pruebes). Esto es algo básico en todas las bases de datos que quieras usar.
> 

Para crear la base de datos de WordPress tienes que: **(no poner los < y >).**

`MariaDB [(none)]> CREATE DATABASE <database_name>;` --Crea la base de datos llamada "wordpress_db”

`MariaDB [(none)]> CREATE USER <username>@localhost IDENTIFIED BY 'check42_MVP';` --Crea un nuevo usuario con contraseña

`MariaDB [(none)]> GRANT ALL ON <database_name>.* TO <username>@localhost IDENTIFIED BY '<password>' WITH GRANT OPTION;` --Da privilegios de super usuario a "admin" en el contexto de esa base de datos

`MariaDB [(none)]> FLUSH PRIVILEGES;`

`MariaDB [(none)]> EXIT;`

> !!! ES IMPORTANTE PONER LOS ; EN LOS COMANDOS
> 

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2036.png)

- Verifique si el usuario de la base de datos se creó correctamente iniciando sesión en la consola de MariaDB.
    
    `mariadb -u nombre_usuario -p`
    

Para comprobar que has creado la base de datos puedes ejecutar estando logueado como root en MariaDB:

`MariaDB [(none)]> show databases;`

La salida debe ser parecida a:

`+--------------------+
| Database           |
+--------------------+
| <database_name>    |`

`information_schema   |
+--------------------+`

### 3.5. Wordpress

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2037.png)

Por último, toca instalar y configurar Wordpress

**Descarga y configuración de WordPress**

Nos bajamos el archivo, lo extraemos, lo copiamos a la carpeta del servidor (lighttpd, que por defecto es var/www/html/ y luego borramos el archivo y la otra carpeta.

- Instale wget.
    
    `sudo apt install wget`
    
- Descarga WordPress en /var/www/html.
    
    `sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html`
    
- Extraiga el contenido descargado.
    
    `sudo tar -xzvf /var/www/html/latest.tar.gz`
    
- Eliminar el tar
    
    `sudo rm /var/www/html/latest.tar.gz`
    
- Copie el contenido de /var/www/html/wordpress a /var/www/html.
    
    `sudo cp -r /var/www/html/wordpress/* /var/www/html`
    
- Quitar /var/www/html/wordpress o la ruta donde tengamos wordpress
    
    `sudo rm -rf /var/www/html/wordpress`
    
- Cree un archivo de configuración de WordPress a partir de su muestra.
    
    `sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php`
    
- Configure WordPress para hacer referencia a la base de datos MariaDB y al usuario creados previamente.
    
    `sudo vim /var/www/html/wp-config.php`
    
    Reemplace lo siguiente
    
    - 23 define( 'DB_NAME', 'database_name_here' );
    - 26 define( 'DB_USER', 'username_here' );
    - 29 define( 'DB_PASSWORD', 'password_here' );

con los datos que hemos puesto al instalar MariaDB

Para entrar en el navegador:

`http://<ip_address>`

`http://10.13.248.199/wp-admin/install.php`

Aqui accedemos a la carpeta por defecto que está online del servido:

/var/www/html/

que tiene todos los archivos del wordpress que hemos descargado:

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2038.png)

Aqui en el navegador es ir siguiendo las rutas de los archivos a los que queramos acceder.

Nos lleva por defecto a la carpeta wp-admin y dentro a install.php para que configuremos los archivos:

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2039.png)

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2040.png)

**Configuramos Wordpress y reiniciamos o logout y login**

### 3.6. Servicio Extra: FTP

**Instalación y configuración de FTP**

- Instalar FTP.
    
    `sudo apt install vsftpd`
    
- Verificar si vsftpd se instaló correctamente.
    
    `sudo dpkg -l | grep vsftpd`
    
- Permitir conexiones entrantes usando el puerto 21 (es el puerto
    
    `sudo ufw allow 21`
    
- Configurar vsftpd.
    
    `sudo vim /etc/vsftpd.conf`
    
- Para habilitar cualquier forma de comando de escritura FTP, elimine el comentario debajo de la línea:
    
    `31 #write_enable=YES`
    
- Para configurar la carpeta raíz para el usuario conectado a FTP /home/*usuario*/ftp, ejecute las siguientes líneas:
    
    `sudo mkdir /home/”*usuario”*/ftp`
    
    `sudo mkdir /home/”*usuario”*/ftp/files`
    
    `sudo chown nobody:nogroup /home/”*usuario”*/ftp`
    
    `sudo chmod a-w /home/”*usuario”*/ftp`
    
- Entramos en:
    
    `sudo nano /etc/vsftpd.conf`
    

Copiamos y pegamos el siguiente mensaje debajo del todo

<~~~>

`user_sub_token=$USER`

`local_root=/home/$USER/ftp`

<~~~>

Para evitar que el usuario acceda a los archivos o use comandos fuera del árbol de directorios, elimine el comentario debajo de la línea en

- `sudo nano /etc/vsftpd.conf`
- `114 #chroot_local_user=YES`

lo dejamos así, descomentado:

- `114 chroot_local_user=YES`
- Para incluir FTP en la lista blanca, agregue las siguientes líneas:
    - creamos el archivo vacío, lo guardamos y cerramos
    - `$ sudo nano /etc/vsftpd.userlist`
    - Con el siguiente comando escribimos el contenido como si estuviéramos en nano. Es decir, escribe el <username> en el vsftpd.userlist
    - `$ echo <username> | sudo tee -a /etc/vsftpd.userlist`
- Entramos en:
    
    `sudo nano /etc/vsftpd.conf`
    

Copiamos y pegamos el siguiente mensaje debajo del todo

`<~~~>`

`userlist_enable=YES`

`userlist_file=/etc/vsftpd.userlist`

`userlist_deny=NO`

`<~~~>`

<aside>
💡 **tee command**
Reads the standard input and writes it to both the standard output and one or more files. The command is named after the T-splitter used in plumbing. It basically breaks the output of a program so that it can be both displayed and saved in a file. It does both the tasks simultaneously, copies the result into the specified files or variables and also display the result.

</aside>

**Conexión al servidor a través de FTP**

FTP en su máquina virtual a través de ftp <ip-address>.

Escribir en el navegador (que no sea Safari porque da error):

`ftp://<ip-address>`

Nos va a salir esta ventana y le damos a open finder.

[https://lh5.googleusercontent.com/8TZ-XKjxrWAkyG80Ihfo9tkxfVoqtZJYP-URrtSdz6odzpRnBXHNWgenoEM25DgwsfMGLYswodCzpKIjukKttS1QNjQFr5uH9i-JfPP4LeMJZvD_4U2yyTasMQG5vCWXqQrEwUZzQYGZiLw3KL-chCiZ0n6ihz1lS6_CraLkt44FLNVbyEaOM4YQ](https://lh5.googleusercontent.com/8TZ-XKjxrWAkyG80Ihfo9tkxfVoqtZJYP-URrtSdz6odzpRnBXHNWgenoEM25DgwsfMGLYswodCzpKIjukKttS1QNjQFr5uH9i-JfPP4LeMJZvD_4U2yyTasMQG5vCWXqQrEwUZzQYGZiLw3KL-chCiZ0n6ihz1lS6_CraLkt44FLNVbyEaOM4YQ)

Nos va a pedir un usuario y contraseña (estos son los que hemos puesto en la instalación de FTP).

[https://lh4.googleusercontent.com/RGWTECFbOTkPYJalTS8gRnwa6z_YjG8t-q0FlC165Rmu1R2fZbaeMJoroMa3_MCtPC92k8dKNuqlbA6Er_ojsn6wrUWZ0NjuSuTmLmo9XiD4fK_Xx7cH9Hw0nPZtZ2yAr3hiSUoF77WZJjJZxdi9YVvRS_Quf-L9U4J4NamvQ3KrDT44BhxgBNn2](https://lh4.googleusercontent.com/RGWTECFbOTkPYJalTS8gRnwa6z_YjG8t-q0FlC165Rmu1R2fZbaeMJoroMa3_MCtPC92k8dKNuqlbA6Er_ojsn6wrUWZ0NjuSuTmLmo9XiD4fK_Xx7cH9Hw0nPZtZ2yAr3hiSUoF77WZJjJZxdi9YVvRS_Quf-L9U4J4NamvQ3KrDT44BhxgBNn2)

Finalice la sesión de FTP en cualquier momento a través de CTRL + D.

Para comprobar que la carpeta ftp funciona, ponemos `ftp://<ip-address>:21`, se abre una ventana donde nos van a pedir usuario y contraseña.

Ahora ya podemos subir archivos a través de finder

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2041.png)

### 3.7. Servicio Extra: Fail2Ban

Como servicio extra, uno de los más útiles a mi parecer para cualquier servidor (aún más si tienes un servidor SSH activado como es tú caso) es `Fail2ban`, se trata de un programa para la prevención de intrusos en un sistema, que actúa penalizando o bloqueando las conexiones remotas que intentan accesos por fuerza bruta. Si encuentra múltiples intentos de inicio de sesión fallidos o ataques automáticos desde una dirección IP, puede bloquearla con el firewall, ya sea de manera temporal o permanente.

Para la instalación:

`sudo apt install fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
sudo systemctl status fail2ban`

Y para la configuración hay que modificar el fichero */etc/fail2ban/jail.local*:

`sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`

Nota: lo copio porque lo dice el man: Provide customizations in a  jail.d/customisation.local.

`sudo vim /etc/fail2ban/jail.local`

Vete hasta el apartado sobre SSH (sobre la línea 280) y añade lo siguiente:

`# [...]

## SSH servers
#

[sshd]`

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
# mode   = normal

`enabled  = true
maxretry = 3
findtime = 10m
bantime  = 1d
port     = 4242
logpath  = %(sshd_log)s
backend  = %(sshd_backend)s

# [...]`

Una vez tengas todo funcionando, para ver los logs de los intentos fallidos solo tienes que hace lo siguiente:

`sudo fail2ban-client status`

`sudo fail2ban-client status sshd`

![Untitled](Born2beRoot%20bd24254de74d46baa3aa0e8702ea67ad/Untitled%2042.png)

`sudo tail -f /var/log/fail2ban.log`

Para probar que Fail2ban realmente está prohibiendo las direcciones IP, puedes cambiar el tiempo de prohibición de SSH a un valor más bajo, como 15 minutos, en el archivo de configuración /etc/fail2ban/jail.local. Luego intenta conectarse varias veces desde la
 máquina host a través de SSH con la contraseña incorrecta. Después de algunos intentos, debería rechazar la conexión y el comando fail2ban-client status sshd debería mostrar la dirección IP prohibida.
