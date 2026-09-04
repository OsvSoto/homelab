# 2026-09-03 — Instalación VM lab-debian

## Resumen 
Instalé Debian 13 en una VM con KVM/libvirt (`lab-debian`): agregué mi usuario
a los grupos `libvirt` y `kvm`, descargué la ISO netinst, y armé la VM con
`virt-install`. El procedimiento completo queda documentado en
`runbooks/crear-vm-lab-debian.md`.

## Qué falló
- El primer intento de `virt-install` falló: QEMU corre como usuario
  `libvirt-qemu`, que no tenía permiso para leer la ISO guardada en
  `~/Descargas` (error "Permission denied" al abrir el blockdev).
- El primer intento de instalación completa, usando `--graphics none` con
  consola serie (`--extra-args "console=ttyS0,115200n8 serial"`), se colgó
  después de instalar GRUB: la consola quedó en blanco, la VM no respondía
  a `ping` ni SSH, aunque el proceso de QEMU seguía "corriendo".
- `virsh domifaddr` mostró, en más de un intento, una IP que en realidad no
  correspondía a la VM (una IP vieja/residual de una lease de DHCP), lo que
  generó confusión al intentar conectarse por SSH ("No route to host").

## Cómo se arregló
- Moví la ISO a `/var/lib/libvirt/images/` (el pool por defecto de libvirt)
  para que el usuario `libvirt-qemu` pudiera leerla.
- Descarté la VM colgada (`virsh destroy` + `virsh undefine --remove-all-storage`)
  y la recreé usando `--graphics vnc` en vez de consola serie, con
  `virt-viewer` para verla. Con consola gráfica confirmé visualmente que la
  instalación avanzaba bien y llegaba al prompt de login sin problema.
- Para encontrar la IP real de la VM, en vez de confiar en `virsh domifaddr`,
  usé `ip neigh show dev virbr0` y busqué la dirección marcada `REACHABLE`
  junto con la MAC de la VM.

## Pendiente 
- Por qué `domifaddr` mostró una IP incorrecta/residual en más de un intento
  (posible relación con leases viejas de dnsmasq que no se limpian solas).
- Falta actualizar el sistema recién instalado
  (`sudo apt update && sudo apt upgrade -y`) y arrancar con el checklist de
  conocimientos de la Fase 0 (`fases/fase0laboratoriolinux.md`).
