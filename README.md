<p align="center">
  <img src="https://raw.githubusercontent.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux/main/assets/LogoNetec.png" alt="NETEC" width="180" />
</p>

# Preparación de entornos Linux

Curso demo para validar la Setup Guide consolidada de Thor. Preparar, configurar y validar una máquina virtual Linux con memoria swap persistente, discos adicionales e interfaces de red, siguiendo únicamente la guía del curso.

## Accesos rápidos

- [**Setup Guide del curso**](https://github.com/Netec-Mx/260928-SUG-DEMO-preparacion-entornos-linux/blob/main/SETUP_GUIDE.md)
- [Laboratorios por capítulo](#lista-de-laboratorios)

## Estructura

- `SETUP_GUIDE.md`: guía de instalación y preparación del entorno.
- `CapituloXX/README.md`: guía de laboratorio por capítulo.

## Lista de laboratorios

### Capítulo 1

- [Lab: Confirmar el sistema operativo](Capitulo01/README.md#lab-confirmar-el-sistema-operativo)
  - Descripción: Comprobar con cat /etc/os-release, uname -m y uname -r que la salida coincide con la versión, edición, arquitectura y kernel aprobados, y registrar la evidencia.
  - Duración estimada: 40 min
- [Lab: Actualizar el sistema](Capitulo01/README.md#lab-actualizar-el-sistema)
  - Descripción: Ejecutar apt update y apt upgrade, registrar la salida sin errores críticos y confirmar que las actualizaciones se completaron.
  - Duración estimada: 40 min
  - [Ver capítulo completo](Capitulo01/README.md)

### Capítulo 2

- [Lab: Configurar memoria swap](Capitulo02/README.md#lab-configurar-memoria-swap)
  - Descripción: Crear /swapfile con fallocate, protegerlo con chmod 600, activarlo con mkswap y swapon, respaldar /etc/fstab antes de editar y confirmar la persistencia con swapon --show y free -h.
  - Duración estimada: 60 min
- [Lab: Preparar los discos adicionales](Capitulo02/README.md#lab-preparar-los-discos-adicionales)
  - Descripción: Usar lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS para confirmar los dispositivos y su contenido antes de cualquier operación destructiva.
  - Duración estimada: 60 min
  - [Ver capítulo completo](Capitulo02/README.md)

### Capítulo 3

- [Lab: Configurar las interfaces de red](Capitulo03/README.md#lab-configurar-las-interfaces-de-red)
  - Descripción: Comprobar ip -br link, ip -br address y nmcli device status; confirmar que la interfaz principal tiene conectividad y la secundaria conserva el estado inicial definido.
  - Duración estimada: 75 min
- [Demo: Validación integral del entorno](Capitulo03/README.md#demo-validación-integral-del-entorno)
  - Descripción: El instructor ejecuta la verificación completa del entorno (sistema, swap, discos y red) y muestra cómo diagnosticar los fallos más frecuentes.
  - Duración estimada: 75 min
  - [Ver capítulo completo](Capitulo03/README.md)

## Flujo de colaboración

- Trabajar en `changes_course`.
- Crear Pull Request hacia `main`.
- Merge por `Squash and merge`.

---

*Material didáctico preparado por Global K, S.A. de C.V.*
