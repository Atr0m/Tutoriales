# Arch Linux en Raspberry Pi

Hace algún tiempo me compré una **[Raspberry Pi](http://www.raspberrypi.org/)** y hace unas semanas decidí instalarle **[Arch Linux](http://archlinuxarm.org/)**, voy a relatar como lo hice.

Para poder realizar todo el proceso necesitaremos:

- Una Raspberry Pi.
- Una tarjeta SD de 2Gb mínimo.
- Un Pc con lector de tarjetas SD

## Instalación

Primero nos tenemos que descargar la imagen de **Arch Linux** **[aquí](http://archlinuxarm.org/platforms/armv6/raspberry-pi)**. Nos podemos descargar la imagen a través de descarga directa o torrent.

Una vez tengamos la imagen descargada hay que descomprimirla, para ello desde consola:

    unzip archlinux-hf-*.img.zip

Después de descomprimir la imagen hay que instalarla en la SD. Para ello hay que conectar la SD al PC, cuando el ordenador halla identificado la tarjeta hay que saber cual es el nombre del dispositivo. Lo podemos mirar con el comando **fdisk** o el comando **df**.

Para cargar la imagen en la SD:

    dd bs=1M if=/path/to/archlinux-hf-*.img of=/dev/sdX
    
Con este comando no aparece ninguna muestra de proceso, depende de la clase de vuestra tarjeta tardará más o menos. A mí en una tarjeta SD de 8Gb clase 4 me ha tardado entre **8 - 10 minutos**.

Cuando el comando termina ya tenemos nuestro **Arch Linux** instalado en la SD pero si tenéis una SD de gran tamaño hay que hacer un paso más. La instalación no aprovecha todo el tamaño disponible de la SD así que usando el **GParted** vamos a extender la última partición.

Con esto ya tenemos lista la SD para empezar a configurar la **Raspberry Pi**

## Configuración

Metemos la tarjeta SD en la **Raspberry Pi**, lo enchufamos a una toma eléctrica y al router por medio de un cable **RJ-45**.

El usuario y password por defecto:

- Usuario: **root**
- Password: **root**

La imagen que hemos instalado en la SD viene con el servicio **SSH** configurado y levantado. Como no tiene configurada ninguna IP estática el router le asignará una IP por medio del **DHCP**. Para ver que IP le ha asignado lo podemos mirar en el router o podemos rastrear nuestra red desde consola usando **Nmap**:

    nmap -sP 192.168.0.1/24

Una vez que sabemos la IP de nuestra **Raspberry Pi** (En mi caso 192.168.1.132), para poder acceder por **SSH** es tan fácil como:

    ssh root@192.168.1.132

Nos pedirá que aceptemos la clave pública del **SSH** y ya estamos dentro de nuestra **Raspberry Pi**. Lo primero que hacemos es actualizar todo el sistema con el comando:

    pacman -Syu
    
Cuando acabe de actualizar todo el sistema vamos a configurar mínimamente la **Raspberry Pi**.

Vamos a configurar el idioma del sistema:

1- Para configurar el idioma editamos el archivo **/etc/locale.gen** y descomentamos nuestro idioma borrando la **“#”** del inicio de línea:
    
    vi /etc/locale.gen

```
#es_EC ISO-8859-1 
es_ES.UTF-8 UTF-8
es_ES ISO-8859-1
es_ES@euro ISO-8859-15
#es_GT.UTF-8 UTF-8
```

2- Cargamos el idioma seleccionado:

    locale-gen

y una vez cargado hay que añadirlo (Aquí que cada uno lo cambie según el idioma elegido):

    localectl set-locale LANG="es_ES.UTF8", LC_TIME="es_ES.UTF8"

3- Ahora configuramos el huso horario, en mi caso:

    ln -s /usr/share/zoneinfo/Europe/Madrid /etc/localtime

4- Creamos un nuevo usuario

    useradd -m -g users -s /bin/bash tu_usuario

5- Cambiamos la password al usuario root y al usuario que acabamos de crear:

    passwd

y

    passwd tu_usuario

6- Le damos un nombre a nuestro host:

    echo "nombre_maquina" > /etc/hostname

7- Le configuramos una IP estática para ello creamos el archivo **/etc/conf.d/interface** y añadimos lo siguiente cambiando los **datos** en cada caso:

```
interface=eth0
address=192.168.1.200
netmask=24
broadcast=192.168.1.255
gateway=192.168.1.1
```
8- Ahora creamos el archivo /etc/systemd/system/network y añadimos lo siguiente:

```
[Unit]
 Description=Wireless Static IP Connectivity
 Wants=network.target
 Before=network.target

[Service]
 Type=oneshot
 RemainAfterExit=yes
 EnvironmentFile=/etc/conf.d/network
 ExecStart=/sbin/ip link set dev ${interface} up
 ExecStart=/sbin/ip addr add ${address}/${netmask} broadcast ${broadcast} dev ${interface}
 ExecStart=/sbin/ip route add default via ${gateway}
 
 ExecStop=/sbin/ip addr flush dev ${interface}
 ExecStop=/sbin/ip link set dev ${interface} down

[Install]
 WantedBy=multi-user.target
```
9- Paramos el servicio de **DHCP** e iniciamos el que acabamos de configurar:

    systemctl disable dhcpcd

    systemctl start network

    systemctl enable network

10- Reiniciamos la **Raspberry Pi** y nos conectamos por **SSH** con nuestro usuario:

    systemctl reboot
>*Este comando puede dejar a la Raspberry 1 minuto bloqueada*

    ssh tu_usuario@192.168.1.200

Hasta aquí la configuración de la **Raspberry Pi** con **Arch Linux** añadiré otros manuales con las cosas que vaya haciendo.

Fuentes:  
[Wiki Archlinux](https://wiki.archlinux.org/)  
[Archlinux ARM](http://archlinuxarm.org/)
