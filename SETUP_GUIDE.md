# Preparación de entornos Linux — Setup Guide

<p align="center"><img src="assets/LogoNetec.png" alt="NETEC" width="180" /></p>

Guía consolidada de instalación y preparación del entorno.

| Curso | Clave | Versión | Estado | Última validación |
|---|---|---|---|---|
| Preparación de entornos Linux | [CLAVE COMPLETA] | [V1.0] | BORRADOR | [FECHA DE VALIDACIÓN] |

## Accesos

- **Repositorio:** [https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux](https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux)
- **Repositorio de laboratorios:** [https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux](https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux)
- **README principal:** [INSERTAR ENLACE]
- **Archivos de instalación:** [INSERTAR ENLACE]
- **Documentación oficial:** [INSERTAR ENLACE]

## Software y versiones

| Componente | Versión exacta | Fuente oficial | Momento | Validación |
|---|---|---|---|---|
| Ubuntu Desktop | 24.04.2 LTS (Noble Numbat) | [https://releases.ubuntu.com/24.04/](https://releases.ubuntu.com/24.04/) | Antes del curso | [COMANDO DE PRUEBA] |
| VirtualBox | 7.0.18 | [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) | Antes del curso | [COMANDO DE PRUEBA] |
| APT (Advanced Package Tool) | 2.7.14 (incluido en Ubuntu 24.04.2 LTS) | [https://salsa.debian.org/apt-team/apt](https://salsa.debian.org/apt-team/apt) | Antes del curso | [COMANDO DE PRUEBA] |
| Netplan | 1.0.1 (incluido en Ubuntu 24.04.2 LTS) | [https://netplan.io/](https://netplan.io/) | Antes del curso | [COMANDO DE PRUEBA] |
| util-linux (fdisk, lsblk, blkid) | 2.39.3 (incluido en Ubuntu 24.04.2 LTS) | [https://github.com/util-linux/util-linux](https://github.com/util-linux/util-linux) | Antes del curso | [COMANDO DE PRUEBA] |
| e2fsprogs (mkfs.ext4, tune2fs) | 1.47.0 (incluido en Ubuntu 24.04.2 LTS) | [https://e2fsprogs.sourceforge.net/](https://e2fsprogs.sourceforge.net/) | Antes del curso | [COMANDO DE PRUEBA] |
| systemd (systemctl, journalctl) | 255.4 (incluido en Ubuntu 24.04.2 LTS) | [https://systemd.io/](https://systemd.io/) | Antes del curso | [COMANDO DE PRUEBA] |
| bash | 5.2.21 (incluido en Ubuntu 24.04 LTS) | [https://www.gnu.org/software/bash/](https://www.gnu.org/software/bash/) | Antes del curso | [COMANDO DE PRUEBA] |
| curl | 8.5.0 (incluido en Ubuntu 24.04 LTS) | [https://curl.se/](https://curl.se/) | Antes del curso | [COMANDO DE PRUEBA] |
| net-tools | 2.10-0.1ubuntu4 (paquete APT en Ubuntu 24.04 LTS) | [https://packages.ubuntu.com/noble/net-tools](https://packages.ubuntu.com/noble/net-tools) | Antes del curso | [COMANDO DE PRUEBA] |
| iproute2 | 6.1.0-1ubuntu6 (incluido en Ubuntu 24.04 LTS) | [https://packages.ubuntu.com/noble/iproute2](https://packages.ubuntu.com/noble/iproute2) | Antes del curso | [COMANDO DE PRUEBA] |
| lsb-release | 12.0-2 (incluido en Ubuntu 24.04 LTS) | [https://packages.ubuntu.com/noble/lsb-release](https://packages.ubuntu.com/noble/lsb-release) | Antes del curso | [COMANDO DE PRUEBA] |

## Matriz de prácticas

| Lab | Nombre | Duración | Resultado esperado | Acceso |
|---|---|---|---|---|
| 01-00-01 | Lab: Confirmar el sistema operativo | 40 min | Archivo /home/alumno/lab-baseline.txt creado con versión exacta del SO, arquitectura, versión de kernel y lista de paquetes con actualizaciones pendientes | [Capítulo 01](https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux/blob/main/Capitulo01/README.md#lab-confirmar-el-sistema-operativo) |
| 01-00-02 | Lab: Actualizar el sistema | 40 min | Sistema Ubuntu 24.04.2 LTS completamente actualizado con todos los parches de seguridad y actualizaciones de paquetes disponibles a la fecha del laboratorio | [Capítulo 01](https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux/blob/main/Capitulo01/README.md#lab-actualizar-el-sistema) |
| 02-00-01 | Lab: Configurar memoria swap | 60 min | Archivo /swapfile de 2 GB creado, formateado y activo como espacio de intercambio | [Capítulo 02](https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux/blob/main/Capitulo02/README.md#lab-configurar-memoria-swap) |
| 02-00-02 | Lab: Preparar los discos adicionales | 60 min | Dos discos adicionales (sdb, sdc) correctamente particionados con tabla GPT y una partición ext4 cada uno | [Capítulo 02](https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux/blob/main/Capitulo02/README.md#lab-preparar-los-discos-adicionales) |
| 03-00-01 | Lab: Configurar las interfaces de red | 75 min | Interfaz de red de laboratorio configurada con IP estática 192.168.100.10/24 mediante archivo /etc/netplan/99-lab-static.yaml | [Capítulo 03](https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux/blob/main/Capitulo03/README.md#lab-configurar-las-interfaces-de-red) |
| 03-00-02 | Demo: Validación integral del entorno | 75 min | Los estudiantes comprenden el proceso sistemático de validación de un entorno Linux de laboratorio siguiendo una Setup Guide formal | [Capítulo 03](https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux/blob/main/Capitulo03/README.md#demo-validación-integral-del-entorno) |
---

*Material didáctico preparado por Global K, S.A. de C.V.*
