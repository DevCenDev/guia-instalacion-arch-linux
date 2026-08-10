# Guía de Instalación de Arch Linux
![Arch linux](https://archlinux.org/static/logos/archlinux-logo-dark-90dpi.ebdee92a15b3.png)

<p>
Arch Linux es un sistema operativo basado en Linux que tiene características ideales para usuarios medios y avanzados que buscan principalmente dos cualidades: mantener el sistema operativo ligero y seguir el principio del acrónimo KISS (Keep It Simple, Stupid). Su primer lanzamiento fue en marzo de 2002 y posee una comunidad de usuarios y contribuidores bastante activa.

Antes de empezar, aclaramos que no es una distribución basada en otra distribución, sino la raíz de una familia de distribuciones de Linux que cubren esa misma filosofía. 
</p>

## Requisitos Previos
- USB con capacidad mínima de 8GB
- Acceso a internet durante la instalación
- Conocimientos básicos de línea de comandos en linux

## Pasos de Instalación

### 1. Descargar la Imagen ISO

```bash
# Descarga desde el sitio oficial
https://archlinux.org/download/
```

### 2. Crear USB Booteable

##### En Linux:
```bash
sudo dd if=archlinux-x86_64.iso of=/dev/sdX bs=4M status=progress
sudo sync
```

##### En Windows:

- Usa herramientas como Rufus o ventoy
- Selecciona el archivo ISO y la unidad USB
- Haz clic en "Grabar/Guardar"

### 3. Iniciar desde USB

- Reinicia tu equipo
- Accede a la BIOS (usualmente SUPR, DEL, ESC o F2)
- Selecciona tu USB como dispositivo de inicio

#### 3.1 Configuracion del teclado
```bash
# Teclado español - latam
loadkeys la-latin1
```

### 4. Modo de arranque
```bash
# Verificar el modo de arranque (BIOS, UEFI)
ls /sys/firmware/uefi 
```
> Nota: Si aparece directorios dentro de UEFI es porque se arranco de ese modo caso contrario esta en modo BIOS.

<details>
<summary>Modo BIOS</summary>
<img src="./assets/BIOS-Arch-Linux.png" width="400">
</details>

<details>
<summary>Modo UEFI</summary>
<img src="./assets/UEFI-Arch-Linux.png" width="400">
</details>

### 5. Conexion a internet

```bash
# Comprueba si la interfaz de red esta habilitada
 ip link
```
#### 5.1 Ethernet
conectar un cable de red 
- Cat 5
- Cat 5e
- Cat 6
- Cat 6e, etc.

#### 5.2 WiFi
```bash
# Asegurate que la tarjeta de red no este bloqueada con rfkill
rfkill list
```
```bash
# Si el kernel bloquea la tarjeta desactivelo
rfkill unblock wifi
```

```bash
# Enviar una peticion a iwd para autentificarse a una red inalambrica
iwctl
```
``` bash
# Lista de dispositivos
[iwd] device list
```
```bash
# Buscar redes
station [device] scan
```
```bash
# Lista de redes escaneadas
station [device] get-networks
```
```bash
# Conectarse a la red 
station [device] connect [SSID]
```
```bash
# Verificar conexion
[iwd] station [device] show
```
```bash
# Ctrl + c
exit
```

### 6. Particionar el Disco
Utilizando una herramienta de particionamiento más adecuada para tu sistema (gdisk, fdisk, cfdisk, etc.), crea una nueva tabla de particiones **GPT** o **MBR**, si no existe. Se requiere una tabla **GPT** en modo UEFI; se requiere una tabla **MBR** si el sistema arranca en modo BIOS.

```bash
# Listar discos disponibles
lsblk
# Crear particiones
cfdisk 
```
**UEFI con GPT**
| montaje | Particion | Tipo de archivo   | Tamaño   |
|:-------:|-----------|-------------------|:---------|
| /boot   | /dev/uefi |efi_System         |1GB       |
| /       | /dev/root |linux_file         |+23GiB    |
| [SWAP]  | /dev/swap |linux_swap         |4GiB      |

**BIOS con MBR**
| montaje | Particion | Tipo de archivo   | Tamaño   |
|:-------:|-----------|-------------------|:---------|
| /       | /dev/root |linux_file         |+23GiB    |
| [SWAP]  | /dev/swap |linux_swap         |4GiB      |


### 7. Formatear Particiones

```bash
# Formatear EFI 
mkfs.fat -F 32 /dev/<efi_particion>

# Formatear root 
mkfs.ext4 /dev/<root_partition>
```
#### 7.1 Particion Swap
```bash
mkswap /dev/<swap_partition>

# activar la particion swap
swapon /dev/<swap_partition>
```

### 8. Montar Particiones

```bash
# Montar root
mount /dev/<root_partition> /mnt

# Crear directorio boot
mkdir -p /mnt/boot
mount /dev/<efi_partition> /mnt/boot

```

### 9. Instalar Paquetes Base

```bash
pacstrap -K /mnt base linux linux-firmware [cpu]-ucode nano
```

### 10. Generar fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

### 11. Ingreso al Sistema

```bash
arch-chroot /mnt
```