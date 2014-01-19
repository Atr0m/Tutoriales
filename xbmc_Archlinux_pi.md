# Instalación de XBMC en Raspberry Pi con Arch Linux

Voy a explicar como instalar el servidor **XBMC** en la **Raspberry Pi** con **Arch Linux**. Para saber como instalar **Arch Linux** en una **Raspberry Pi** podéis mirar mi anterior tutorial [**aquí**](https://github.com/Atr0m/Tutoriales/blob/master/Arch_Linux_%2B_Raspberry_Pi.md).

##Overclocking

Primero aumentamos la frecuencia de reloj de nuestra **Raspberry Pi** para ello modificamos el archivo **/boot/config.txt** y descomentamos la sección **Turbo** al final del archivo quedando así:

```
##Turbo
arm_freq=1000
core_freq=500
sdram_freq=500
over_voltage=6
```
Mientras que no subamos la frecuencia de reloj de **1GHz**, no se pierde garantía como podemos leer el la web de **Raspberry Pi** [**aquí**](http://www.raspberrypi.org/archives/2008). Aumentar la frecuencia de reloj del procesador no es obligatorio pero en mi caso el **XBMC** sin el **overclocking** me resulta lento.

## Instalación

Instalamos los paquetes necesarios para el **XBMC**:

    pacman -S xbmc xorg-server
    
Ejecutamos el siguiente comando como **root** para añadir permisos al **XBMC**:

    echo 'SUBSYSTEM=="vchiq",GROUP="video",MODE="0660"' > /etc/udev/rules.d/10-vchiq-permissions.rules
    
Ahora vamos a añadir al usuario **XBMC** a los grupos necesarios:

    usermod -a -G audio,storage,power,video xbmc
    
Solo falta habilitar el servicio **XBMC**:

    systemctl enable xbmc
    
Con esta ya tenemos montado nuestro **XBMC** en nuestra **Raspberry Pi**, cuando la conectemos a un televisor por **HDMI** el demonio lanzara **XBMC**.

### Extra

En algunas pantallas de televisión como por ejemplo en la mía no se ve nada por **HDMI** y hay que forzar a la **Raspberry Pi** a conectarse, para ello modificamos el archivo **/boot/config.txt** y descomentamos la siguiente línea:

    hdmi_force_hotplug=1
    
Por último añadimos un **HDD** externo conectado por **USB** donde meteremos los archivos a reproducir. Ejecutamos **fdisk** para saber el dispositivo correcto y creamos una carpeta donde montar el **HDD**:

    mkdir /mnt/Datos
    
Ahora añadimos el **HDD** editando nuestro **/etc/fstab** quedando en mi caso así:

    /dev/sda1    /mnt/Datos	ext4	defaults,user	0	0
    
El paso anterior necesita permisos root por lo que necesitamos cambiar permisos:

    chown -R tu_usuario:users /mnt/Datos

Y ya tenemos nuestro servidor **XBMC** y nuestro HDD listos.

Espero que les sea de utilidad.

Fuentes:

[**Wiki Arch Linux**](https://wiki.archlinux.org/)  
[**Foros Raspberry Pi**](http://www.raspberrypi.org/forum/)
