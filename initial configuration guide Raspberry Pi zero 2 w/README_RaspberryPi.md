# Guía de Trabajo con Raspberry Pi

## Programacion de sistemas linux embebidos
#### Docente:  *Juan Bernardo Gomez Mendoza* -*jbgomezm@unal.edu.co*
#### Monitor. *Edward Fabian Goyeneche Velandia* - *egoyeneche@unal.edu.co*


### 2025-I
----

## Introducción
La presente guia presenta la configuracion inicial de comunicacion serial por UART  para configuracion de red y habilitacion de comunicacion `ssh`(*Secure Sheel*)  a la  `Raspberry Pi Zero 2 W`

## Requisitos Previos
- Raspberry Pi Zero 2 W
- microSD con **Raspberry Pi OS Lite** ya instalado
- Cable USB a UART (3.3V TTL, por ejemplo CP2102 o FTDI)
- PC con **Linux (Ubuntu/Debian/wsl)**





##  Paso 1: Conectar UART a la Raspberry Pi

![Conexión UART a Raspberry Pi]( zero2-hero.webp)

### Conexión de pines:

| Raspberry Pi Pin | Función | USB-UART Cable |
|------------------|---------|----------------|
| Pin 6            | GND     | GND            |
| Pin 8            | TXD     | RX             |
| Pin 10           | RXD     | TX             |



##  Paso 2: Conectarse por consola serial desde Linux

* Se coneta  el UART al pc , se abre la teminal y se hace la conexion serial:


   ```bash
   dmesg | grep tty
    ```

* Una vez abierta la consola serial se identica el puerto:
    
    `dev/ttyUSB0` o `/dev/ttyACM0`


* abrir la consola serial:
    ```bash
    sudo screen /dev/ttyUSB0 115200
    ```


### NOTA:  *lo realizao  hasta este punto se necesita  no haber conectado la tarjeta.*




*  Insertar la microSD en la RasspberriPI, conéctalo a energía.
*  en terminal se debe ver algo así :
   ```
   raspberrypi login:
   ```

---

## Paso 3: Iniciar sesión

Usuario por defecto:

```text
Login: pi
Password: raspberry
```

---

## Paso 4: Configurar red  Inalambrica:

Abrir y editar el archivo de configuracion

```bash
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

Se deb  realizar la siguiente confiracion  (reemplaza `ssid`: *nombre de la red* y `psk`: contraseña de  la red):

```conf
country=ES
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="Red Estudiantes UN "
    psk="RedEstudiantes"
}
```

Se guarda la configuracion realizada y se compila de nuevo el servicio de la red:

```bash
sudo wpa_cli -i wlan0 reconfigure
```

 Despues de esto se puede realizar la verificacion de la direccion IP asignada:

```bash
ifconfig wlan0
```

---

 Paso 5: Habilitar el servicio SSH

Activa SSH:

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

Tambien se puede usar lo siguiente:

```bash
sudo raspi-config
```

- navegar dentro del archvio y encontrar  a: `Interfacing Options` → `SSH` → Activar

---

Conectarte por SSH 

Por lo general cualquier red tiene soporte ys e puede usar   mDNS,:

```bash
ssh pi@raspberrypi.local
```

pero cómo se conecto a la wi-fi , con la IP obtenida:

```bash
ssh pi@192.168.0.1
```



### Listar redes WiFi disponibles:

```bash
sudo iwlist wlan0 scan | grep ESSID
```


```conf
network={
    ssid="UN"
    psk="*****"
    priority=1
}

network={
    ssid="PSLE"
    psk="psl22025"
    priority=2
}
```
---

##  Instalar y usar `rsync` para sincronización los  archivos pc y la Raspberrypi

`rsync`  sirve para  archivos entre tu Raspberry Pi y  el  PC de forma rápida y eficiente. Funciona perfectamente sobre SSH.
📦 Instalación en la Raspberry Pi

Conectado por UART o SSH, luego:


### Raspberry Pi:

```bash
sudo apt update
sudo apt install rsync
```

### PC:

```bash
sudo apt install rsync
```

### Ejecucion:

Subir un archivo desde tu PC al Pi:
```bash
rsync -avz ./archivo.txt pi@raspberrypi.local:/home/pi/
```
Descargar una carpeta desde el Pi a al PC:

```bash
rsync -avz pi@raspberrypi.local:/home/pi/proyecto/ ./backup-proyecto/
```

Sincronizar un proyecto entero (solo archivos nuevos o modificados):
```bash
rsync -avz ./proyecto-local/ pi@raspberrypi.local:/home/pi/proyecto/
```

### NOTA:

Se puede usar  la direccion  IP de las RasberryPi en vez de ```raspberrypi.local``` si mDNS no funciona, esto teniendo en cuenta que previamente se haya  hecho  la confiracion de red.