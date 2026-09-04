# Fase 0 — Base Linux (laboratorio)

Proyecto general: levantar y mantener un servidor propio, por fases.
Fase 0 (VM local, terminal pura) -> Fase 1 (VPS real, SSH/firewall/TLS) ->
Fase 2 (Docker, servicios reales) -> Fase 3 (backups, monitoreo) ->
Fase 4 (cloud con Terraform, IAM) -> Fase 5 (atacar y endurecer lo propio).

Duración estimada de esta fase: 2 a 4 semanas.

---

## Objetivo

Una VM Debian 13 **sin entorno gráfico**, a la que entras **solo por SSH**,
y donde dominas la terminal antes de exponer nada a internet.

---

## Montaje

### 1. VirtualBox

Windows:
```
winget install --id Oracle.VirtualBox -e
```
Linux (Debian/Ubuntu como anfitrión):
```
sudo apt update && sudo apt install virtualbox
```

Requisito: virtualización habilitada en BIOS/UEFI (VT-x / AMD-V).
Si el anfitrión ya corre Hyper-V o WSL2, VirtualBox irá más lento;
en ese caso considera usar KVM/virt-manager si el anfitrión es Linux.

### 2. ISO de Debian

https://www.debian.org/CD/netinst/  ->  imagen `amd64` netinst (~700 MB).

### 3. Crear la VM

| Opción | Valor |
|---|---|
| Nombre | lab-debian |
| Tipo / Versión | Linux / Debian (64-bit) |
| Skip Unattended Installation | MARCAR (instalamos a mano) |
| RAM | 4096 MB (2048 si el equipo tiene 8 GB) |
| CPUs | 2 |
| Disco | 25 GB, dinámico |

Con la VM apagada: Configuración -> Red -> Adaptador 1 -> Avanzado ->
Reenvío de puertos, agregar:

| Nombre | Protocolo | Puerto anfitrión | IP invitada | Puerto invitado |
|---|---|---|---|---|
| ssh | TCP | 2222 | (vacío) | 22 |

### 4. Instalar Debian

Arrancar con el ISO, elegir **Install** (modo texto).

Dos pantallas críticas:

**a) Particionado**
"Guiado - utilizar todo el disco" -> "Todos los ficheros en una partición".
(Es el disco virtual, no el del anfitrión.)

**b) Selección de programas (tasksel)** — aquí se gana o se pierde la fase:
```
[ ] Debian desktop environment    <- DESMARCAR (barra espaciadora)
[ ] ... GNOME                     <- DESMARCAR
[*] SSH server                    <- MARCAR
[*] standard system utilities     <- dejar marcado
```
Contraseña de root: **dejarla vacía** -> Debian pone tu usuario en `sudo`.

### 5. Snapshot inmediato

Al terminar la instalación y entrar la primera vez:
```
sudo poweroff
```
VirtualBox -> Instantáneas -> Tomar -> nombre: `recien-instalado`

Esta es la red de seguridad de toda la fase. Se usa para romper cosas sin miedo.

### 6. Conectarse por SSH

Desde el anfitrión:
```
ssh TUUSUARIO@127.0.0.1 -p 2222
```
A partir de aquí, no se vuelve a usar la ventana de VirtualBox.
Se trabaja por SSH, como en un servidor real.

---

## Checklist de conocimientos de la Fase 0

Marca solo lo que puedas hacer **sin buscar en Google**.

### Archivos y navegación
- [ ] Moverse y explorar: `pwd`, `cd`, `ls -la`, rutas absolutas vs relativas
- [ ] Ver contenido: `cat`, `less`, `head`, `tail -f`
- [ ] Buscar archivos: `find / -name ... -type f`, `locate`
- [ ] Buscar dentro de archivos: `grep -r`, `grep -i`, `grep -v`
- [ ] Tuberías y redirección: `|`, `>`, `>>`, `2>`, `2>&1`, `/dev/null`
- [ ] Encadenar: `ps aux | grep ssh | wc -l`
- [ ] Comprimir: `tar -czf`, `tar -xzf`
- [ ] Entender el FHS: qué vive en `/etc`, `/var`, `/usr`, `/opt`, `/proc`

### Permisos
- [ ] Leer la salida de `ls -l` completa (tipo, permisos, dueño, grupo)
- [ ] `chmod` en octal y simbólico; explicar qué es 755 y qué es 644
- [ ] `chown`, `chgrp`, usuarios y grupos (`/etc/passwd`, `/etc/group`)
- [ ] `sudo` vs `su`; qué hace `/etc/sudoers` (editar solo con `visudo`)
- [ ] Saber qué es el bit SUID y por qué importa en seguridad

### Procesos y systemd
- [ ] `ps aux`, `top` / `htop`, `kill`, `kill -9`, señales
- [ ] Primer y segundo plano: `&`, `jobs`, `fg`, `Ctrl+Z`
- [ ] `systemctl status|start|stop|restart|enable|disable <servicio>`
- [ ] `journalctl -u <servicio>`, `journalctl -f`, `journalctl -p err`
- [ ] Escribir una unidad `.service` propia y activarla

### Red
- [ ] `ip a`, `ip r` — leer la IP, la máscara y la puerta de enlace
- [ ] `ss -tulpn` — qué proceso escucha en qué puerto
- [ ] `dig` / `host` — resolver un dominio, entender A / CNAME / MX
- [ ] `curl -I`, `curl -v` — leer cabeceras HTTP
- [ ] `ping`, `traceroute`; entender por qué a veces ping falla y HTTP funciona
- [ ] `/etc/hosts` y `/etc/resolv.conf`: para qué sirve cada uno

### Paquetes
- [ ] `apt update` vs `apt upgrade` (no son lo mismo)
- [ ] `apt install`, `apt remove` vs `apt purge`, `apt search`, `apt show`
- [ ] `/etc/apt/sources.list`: qué es un repositorio
- [ ] `dpkg -l`, `dpkg -L <paquete>` (qué archivos instaló)

### Edición y shell
- [ ] `vim`: abrir, editar, guardar, salir, buscar. Sin pánico.
- [ ] Variables de entorno: `echo $PATH`, `export`, `.bashrc`
- [ ] Historial: `history`, `Ctrl+R`
- [ ] Escribir un script bash con `#!/bin/bash`, argumentos, `if`, `for`
- [ ] `cron` / `systemd timers`: programar una tarea

---

## Práctica obligatoria

1. **OverTheWire Bandit** — https://overthewire.org/wargames/bandit/
   Los 25 niveles. Es gratis y fuerza a usar la shell de verdad.
   Conectarse desde la VM, no desde el anfitrión.

2. **Libro de cabecera** — *The Linux Command Line*, William Shotts
   Gratis en PDF: https://linuxcommand.org/tlcl.php

---

## Retos de cierre de fase

Si puedes resolver estos cinco, la Fase 0 está superada:

1. **Servicio propio**: crear un script que escriba la fecha en un log cada
   minuto, convertirlo en un servicio systemd que arranque solo al bootear,
   y demostrarlo reiniciando la VM.

2. **Auditoría de puertos**: listar todo lo que escucha en la VM, identificar
   qué proceso y qué paquete lo instaló, y apagar todo lo que no sea SSH.

3. **Caza de logs**: provocar 5 intentos fallidos de login por SSH y encontrar
   el registro exacto en el journal, filtrando por servicio y por prioridad.

4. **Romper y reparar**: dañar a propósito `/etc/ssh/sshd_config`, perder el
   acceso SSH, y repararlo desde la consola de VirtualBox. (Snapshot antes.)

5. **Disco lleno**: llenar `/` con `dd` hasta que el sistema falle, diagnosticar
   con `df` y `du`, encontrar el archivo culpable y recuperar el sistema.

Cuando los cinco estén hechos: nueva snapshot, `fase0-completa`, y a la Fase 1.

---

## Reglas de la casa

1. **Documenta todo en un repo git.** Un README con la arquitectura y runbooks
   ("cómo restauro", "cómo renuevo el certificado"). Ese repo es el portafolio
   y vale más que un certificado en una entrevista.
2. **Rompe cosas a propósito.** Diagnosticar es la habilidad real; los tutoriales
   solo enseñan el camino feliz.
3. **No pases de fase por aburrimiento.** La fase aburrida es la que te hace bueno.
4. **Nada de GUI.** Si aparece un escritorio, algo se hizo mal.
