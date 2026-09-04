# Runbook: crear VM lab-debian (KVM/libvirt)

## Prerrequisitos
- Estar en los grupos `libvirt` y `kvm` (verificar con `groups`; si falta,
  `sudo usermod -aG libvirt,kvm $USER` y reiniciar sesión para que tome efecto).
- `libvirtd` activo: `sudo systemctl status libvirtd --no-pager` (debe decir
  `Active: active (running)`).
- Red `default` activa: `virsh net-list --all` (debe decir `default ... activo ... si`).
- `virt-viewer` instalado: `sudo apt install -y virt-viewer`.

## Descargar la ISO
```
ISO=$(curl -s https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/ | grep -oP 'debian-[0-9.]+-amd64-netinst\.iso' | head -1)
wget -P /var/lib/libvirt/images/ "https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/$ISO"
```
Guardarla directo en `/var/lib/libvirt/images/`, no en `$HOME` — el usuario
`libvirt-qemu` (con el que corre QEMU) no tiene permiso para leer dentro de
directorios de usuarios.

## Crear la VM
```
virt-install \
  --name lab-debian \
  --memory 4096 \
  --vcpus 2 \
  --disk size=25 \
  --location /var/lib/libvirt/images/debian-13.6.0-amd64-netinst.iso \
  --network network=default \
  --os-variant debian13 \
  --graphics vnc
```
Se abre una ventana de `virt-viewer` con el instalador de Debian en modo
texto. Usar `--graphics vnc` (no `--graphics none`): con consola serie tuvimos
un intento que se colgó sin poder diagnosticarlo por falta de visibilidad.

## Durante la instalación
- Idioma del instalador: elegir **Spanish** desde el arranque si el país
  (Argentina/Chile) no aparece en la lista de un instalador en inglés (evita
  el error "no hay locale definido para esa combinación").
- Contraseña de root: **vacía** (deja al usuario normal en el grupo `sudo`).
- Particionado: guiado, usar todo el disco, todo en una partición.
- Selección de software (`tasksel`): desmarcar todos los entornos de
  escritorio (GNOME, Xfce, KDE, Cinnamon, MATE, LXDE, LXQt, GNOME Flashback),
  "web server" y "Debian blends"; dejar marcados solo **SSH server** y
  **standard system utilities**.
- Instalar GRUB en el disco principal (`/dev/vda`).

## Snapshot de referencia
Antes de loguearse por primera vez, apagar y tomar una snapshot limpia:
```
virsh shutdown lab-debian
virsh list --all   # esperar "apagado"
virsh snapshot-create-as lab-debian recien-instalado --description "Instalacion limpia, antes de cualquier cambio"
virsh start lab-debian
```

## Cómo conseguir la IP real
`virsh domifaddr lab-debian` puede mostrar una IP vieja/residual (pasó dos
veces). El método confiable:
```
ip neigh show dev virbr0
```
Buscar la línea con la MAC de la VM (columna `lladdr`) marcada como
`REACHABLE` — esa es la IP real, no necesariamente la que muestra `domifaddr`.

## Conectarse
```
ping -c 3 <ip-real>
ssh <usuario>@<ip-real>
```
Aceptar el fingerprint de SSH la primera vez (`yes`) — es normal, SSH lo
guarda en `~/.ssh/known_hosts` para verificarlo en conexiones futuras.

## Primer paso después de conectarse
```
sudo apt update && sudo apt upgrade -y
```
