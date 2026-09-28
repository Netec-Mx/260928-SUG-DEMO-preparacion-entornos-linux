# Confirmar el sistema operativo

## Metadatos

| Campo | Valor |
|---|---|
| **Duración** | 40 minutos |
| **Complejidad** | Fácil |
| **Nivel Bloom** | Aplicar |
| **Curso** | Curso piloto — Validación de la Setup Guide de Thor |
| **Sistema operativo** | Ubuntu Desktop 24.04.2 LTS (Noble Numbat) |
| **ISO verificada** | `ubuntu-24.04.2-desktop-amd64.iso` — SHA256 publicado en [https://releases.ubuntu.com/24.04/SHA256SUMS](https://releases.ubuntu.com/24.04/SHA256SUMS) |

## Descripción General

Este es el primer laboratorio del curso piloto. Su propósito es verificar y documentar el estado inicial del sistema operativo Ubuntu Desktop 24.04.2 LTS instalado en la máquina virtual, incluyendo la versión exacta de la distribución, la arquitectura del procesador, el kernel activo y el estado del catálogo de paquetes APT. Al finalizar, el estudiante habrá creado el archivo de referencia acumulativo `/home/alumno/lab-baseline.txt`, que será el punto de partida para todos los laboratorios posteriores y el entregable final del curso.

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Verificar y registrar la versión exacta de Ubuntu Desktop instalada en la VM (nombre de distribución, número de versión `24.04.2` y codename `noble`).
- [ ] Identificar y documentar la arquitectura del procesador (`x86_64` / `amd64`) y la versión del kernel activo mediante `uname`.
- [ ] Consultar el estado del catálogo APT y listar los paquetes con actualizaciones pendientes sin aplicarlas.
- [ ] Crear un registro de referencia (baseline) estructurado en `/home/alumno/lab-baseline.txt` que sirva como punto de partida para el laboratorio 01-00-02.

## Prerrequisitos

### Conocimientos previos

| Conocimiento | Nivel |
|---|---|
| Navegación básica en terminal Linux (`cd`, `ls`, `cat`, `echo`) | Básico |
| Concepto de distribución, kernel y arquitectura (Lección 1.1) | Básico |
| Uso de redirección de salida en Bash (`>`, `>>`, `tee`) | Básico |

### Acceso y recursos necesarios

| Recurso | Detalle |
|---|---|
| Máquina virtual | Ubuntu Desktop 24.04.2 LTS instalada y funcional en VirtualBox 7.0.18 |
| Usuario | `alumno` (contraseña: `Lab2024!`) con permisos `sudo` |
| Conexión a internet | Activa en la VM (adaptador NAT configurado) para ejecutar `apt update` |
| Terminal | Aplicación «Terminal» de GNOME o acceso mediante `Ctrl+Alt+T` |

## Entorno de Laboratorio

### Hardware asignado a la VM

| Componente | Valor mínimo |
|---|---|
| CPU | 2 núcleos virtuales |
| RAM | 4 GB (host con mínimo 8 GB) |
| Disco principal | 30 GB (VDI, asignación dinámica) |
| Adaptador de red 1 | NAT (acceso a internet) |
| Resolución de pantalla | 1280×768 mínimo en el host |

### Software relevante

| Software | Versión exacta | Fuente oficial |
|---|---|---|
| Ubuntu Desktop | 24.04.2 LTS (Noble Numbat) | [https://releases.ubuntu.com/24.04/](https://releases.ubuntu.com/24.04/) |
| VirtualBox | 7.0.18 | [https://www.virtualbox.org/wiki/Download_Old_Builds_7_0](https://www.virtualbox.org/wiki/Download_Old_Builds_7_0) |
| APT | 2.7.14 (incluido en Ubuntu 24.04.2 LTS) | Repositorio oficial `archive.ubuntu.com` |
| Bash | 5.2.21 (incluido en Ubuntu 24.04.2 LTS) | Repositorio oficial `archive.ubuntu.com` |
| lsb-release | 12.0-2 (incluido en Ubuntu 24.04.2 LTS) | Repositorio oficial `archive.ubuntu.com` |
| util-linux (uname) | 2.39.3 (incluido en Ubuntu 24.04.2 LTS) | Repositorio oficial `archive.ubuntu.com` |

### Constantes del laboratorio

```text
Usuario:                    alumno
Contraseña:                 Lab2024!
Directorio de trabajo:      /home/alumno/
Archivo baseline:           /home/alumno/lab-baseline.txt
```

### Preparación inicial

Antes de comenzar, abre una terminal en la VM (`Ctrl+Alt+T`) y confirma que tienes sesión con el usuario correcto:

```bash
whoami
```

**Salida esperada:**

```
alumno
```

Si la salida no es `alumno`, cierra sesión y vuelve a iniciar con las credenciales indicadas.

## Instrucciones Paso a Paso

### Paso 1: Verificar la versión de la distribución con `lsb_release`

**Objetivo:** Confirmar que la VM ejecuta Ubuntu Desktop 24.04.2 LTS con codename `noble` utilizando el comando estándar `lsb_release`.

**Instrucciones:**

1. En la terminal, ejecuta el siguiente comando para obtener la información completa de la distribución:

```bash
lsb_release -a
```

2. Observa la salida y verifica que cada campo coincida con los valores esperados.

**Salida esperada:**

```
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.2 LTS
Release:        24.04
Codename:       noble
```

**Verificación:**

- [ ] El campo `Distributor ID` muestra `Ubuntu`.
- [ ] El campo `Description` incluye `24.04.2 LTS` (no `24.04 LTS` ni `24.04.1 LTS`).
- [ ] El campo `Codename` muestra `noble`.

> **Nota importante:** Si el campo `Description` muestra una versión diferente a `24.04.2 LTS`, la ISO utilizada para la instalación no es la correcta. Consulta la sección de Solución de Problemas antes de continuar.

### Paso 2: Inspeccionar el archivo `/etc/os-release`

**Objetivo:** Obtener los metadatos estructurados de la distribución desde el archivo estándar `systemd`, que es el método recomendado para scripts de automatización.

**Instrucciones:**

1. Muestra el contenido completo del archivo de identificación del sistema:

```bash
cat /etc/os-release
```

2. Identifica los campos clave: `PRETTY_NAME`, `VERSION_ID`, `VERSION_CODENAME` y `ID`.

**Salida esperada (campos relevantes):**

```
PRETTY_NAME="Ubuntu 24.04.2 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.2 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```

3. Para extraer únicamente los campos que se almacenarán en el baseline, ejecuta:

```bash
grep -E "^(PRETTY_NAME|VERSION_ID|VERSION_CODENAME|ID=)" /etc/os-release
```

**Salida esperada:**

```
PRETTY_NAME="Ubuntu 24.04.2 LTS"
VERSION_ID="24.04"
ID=ubuntu
VERSION_CODENAME=noble
```

**Verificación:**

- [ ] `PRETTY_NAME` contiene `Ubuntu 24.04.2 LTS`.
- [ ] `VERSION_CODENAME` es `noble`.
- [ ] `ID` es `ubuntu` e `ID_LIKE` es `debian` (confirma la familia de la distribución).

### Paso 3: Identificar la arquitectura y el kernel activo

**Objetivo:** Documentar la versión del kernel en ejecución y la arquitectura del procesador virtual asignado a la VM.

**Instrucciones:**

1. Consulta la versión del kernel:

```bash
uname -r
```

**Salida esperada (el número de build puede variar según parches aplicados durante la instalación):**

```
6.8.0-55-generic
```

> **Nota:** La versión exacta del kernel depende de los parches incluidos en la ISO `24.04.2`. El formato esperado es `6.8.0-XX-generic` donde `XX` es el número de build. Si tu salida muestra un kernel de la serie `6.8.0`, es correcto.

2. Consulta la arquitectura del hardware:

```bash
uname -m
```

**Salida esperada:**

```
x86_64
```

3. Obtén toda la información del kernel en una sola línea:

```bash
uname -a
```

**Salida esperada (ejemplo):**

```
Linux alumno-VirtualBox 6.8.0-55-generic #57-Ubuntu SMP PREEMPT_DYNAMIC Wed Feb 12 14:12:42 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

4. Confirma la arquitectura desde la perspectiva del gestor de paquetes:

```bash
dpkg --print-architecture
```

**Salida esperada:**

```
amd64
```

**Verificación:**

- [ ] `uname -r` devuelve un kernel de la serie `6.8.0-XX-generic`.
- [ ] `uname -m` devuelve `x86_64`.
- [ ] `dpkg --print-architecture` devuelve `amd64` (equivalente a `x86_64` en la nomenclatura Debian/Ubuntu).

### Paso 4: Verificar las fuentes de repositorios APT

**Objetivo:** Confirmar que los repositorios APT configurados apuntan a los servidores oficiales de Ubuntu antes de sincronizar el catálogo de paquetes.

**Instrucciones:**

1. En Ubuntu 24.04, la configuración de repositorios utiliza el formato DEB822. Verifica el archivo de fuentes principal:

```bash
cat /etc/apt/sources.list.d/ubuntu.sources
```

**Salida esperada (fragmento representativo):**

```
Types: deb deb-src
URIs: http://archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

> **Nota:** Dependiendo de la región configurada durante la instalación, la URI puede mostrar un mirror regional como `http://XX.archive.ubuntu.com/ubuntu/` (donde `XX` es el código de país). Esto es válido siempre que sea un mirror oficial de Ubuntu.

2. Verifica también si existe el componente de seguridad:

```bash
grep -i "security" /etc/apt/sources.list.d/ubuntu.sources
```

**Salida esperada:**

```
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
```

3. Confirma que no existen repositorios de terceros no autorizados:

```bash
ls /etc/apt/sources.list.d/
```

**Salida esperada:**

```
ubuntu.sources
```

> Si aparecen archivos adicionales, documéntalos pero no los elimines en este laboratorio.

**Verificación:**

- [ ] Las URIs apuntan a `archive.ubuntu.com` (o un mirror regional oficial `XX.archive.ubuntu.com`).
- [ ] Los paquetes están firmados con `/usr/share/keyrings/ubuntu-archive-keyring.gpg`.
- [ ] Las suites incluyen `noble`, `noble-updates` y `noble-security`.

### Paso 5: Actualizar el índice de paquetes APT y listar actualizaciones pendientes

**Objetivo:** Sincronizar el catálogo de paquetes con los repositorios oficiales y documentar cuántas actualizaciones están disponibles, **sin instalar ninguna**.

**Instrucciones:**

1. Actualiza el índice de paquetes (requiere `sudo`):

```bash
sudo apt update
```

Ingresa la contraseña `Lab2024!` cuando se solicite.

**Salida esperada (ejemplo; los números variarán):**

```
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:3 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
45 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

2. Lista los paquetes con actualizaciones disponibles:

```bash
apt list --upgradable
```

**Salida esperada (ejemplo parcial):**

```
Listing... Done
base-files/noble-updates 13ubuntu10.2 amd64 [upgradable from: 13ubuntu10]
curl/noble-updates 8.5.0-2ubuntu10.6 amd64 [upgradable from: 8.5.0-2ubuntu10.4]
...
```

3. Cuenta el número de paquetes actualizables (excluyendo la línea de encabezado):

```bash
apt list --upgradable 2>/dev/null | grep -c "upgradable"
```

**Salida esperada (ejemplo):**

```
45
```

> **IMPORTANTE:** No ejecutes `apt upgrade` ni `apt dist-upgrade` en este laboratorio. La aplicación de actualizaciones se realizará en el laboratorio 01-00-02.

**Verificación:**

- [ ] `sudo apt update` se ejecuta sin errores de conexión ni advertencias de firmas GPG.
- [ ] `apt list --upgradable` muestra la lista de paquetes pendientes con sus versiones actuales y disponibles.
- [ ] El número de paquetes actualizables ha sido registrado.

### Paso 6: Crear el archivo de baseline inicial

**Objetivo:** Consolidar toda la información recopilada en los pasos anteriores en el archivo `/home/alumno/lab-baseline.txt`, que será el registro de referencia acumulativo del curso.

**Instrucciones:**

1. Crea el archivo de baseline ejecutando el siguiente bloque de comandos. Cada sección está claramente delimitada para facilitar su lectura y actualización en laboratorios posteriores:

```bash
cat << 'EOF' > /home/alumno/lab-baseline.txt
=============================================================
 BASELINE ACUMULATIVO — Setup Guide de Thor (Curso Piloto)
 Archivo: /home/alumno/lab-baseline.txt
=============================================================

[Lab 01-00-01] Fecha de ejecución: $(date)

-------------------------------------------------------------
 SECCIÓN 1: SISTEMA OPERATIVO — Estado Inicial
-------------------------------------------------------------
EOF
```

2. Ahora, añade la información del sistema de forma dinámica:

```bash
{
  echo "[Lab 01-00-01] Fecha de ejecución: $(date)"
  echo ""
  echo "-------------------------------------------------------------"
  echo " SECCIÓN 1: SISTEMA OPERATIVO — Estado Inicial"
  echo "-------------------------------------------------------------"
  echo ""
  echo "--- Distribución (lsb_release -a) ---"
  lsb_release -a 2>/dev/null
  echo ""
  echo "--- Distribución (/etc/os-release - campos clave) ---"
  grep -E "^(PRETTY_NAME|VERSION_ID|VERSION_CODENAME|ID=)" /etc/os-release
  echo ""
  echo "--- Kernel ---"
  echo "Versión del kernel: $(uname -r)"
  echo "Información completa: $(uname -a)"
  echo ""
  echo "--- Arquitectura ---"
  echo "Arquitectura hardware (uname -m): $(uname -m)"
  echo "Arquitectura paquetes (dpkg): $(dpkg --print-architecture)"
  echo ""
  echo "--- Fuentes APT ---"
  cat /etc/apt/sources.list.d/ubuntu.sources
  echo ""
  echo "--- Paquetes con actualizaciones pendientes ---"
  echo "Cantidad: $(apt list --upgradable 2>/dev/null | grep -c 'upgradable')"
  echo ""
  echo "Lista completa:"
  apt list --upgradable 2>/dev/null | grep "upgradable"
  echo ""
  echo "-------------------------------------------------------------"
  echo " FIN SECCIÓN 1 — Lab 01-00-01"
  echo "-------------------------------------------------------------"
} > /home/alumno/lab-baseline.txt
```

3. Verifica que el archivo se creó correctamente:

```bash
cat /home/alumno/lab-baseline.txt
```

4. Confirma el tamaño del archivo (debe ser mayor a 0 bytes):

```bash
ls -la /home/alumno/lab-baseline.txt
```

**Salida esperada (ejemplo):**

```
-rw-rw-r-- 1 alumno alumno 2847 jun 15 10:30 /home/alumno/lab-baseline.txt
```

5. Verifica que los campos críticos están presentes en el archivo:

```bash
echo "=== Verificación de campos críticos ==="
echo -n "PRETTY_NAME: "; grep "PRETTY_NAME" /home/alumno/lab-baseline.txt
echo -n "Kernel: "; grep "Versión del kernel" /home/alumno/lab-baseline.txt
echo -n "Arquitectura: "; grep "uname -m" /home/alumno/lab-baseline.txt
echo -n "Paquetes pendientes: "; grep "Cantidad:" /home/alumno/lab-baseline.txt
```

**Salida esperada (ejemplo):**

```
=== Verificación de campos críticos ===
PRETTY_NAME: PRETTY_NAME="Ubuntu 24.04.2 LTS"
Kernel: Versión del kernel: 6.8.0-55-generic
Arquitectura: Arquitectura hardware (uname -m): x86_64
Paquetes pendientes: Cantidad: 45
```

**Verificación:**

- [ ] El archivo `/home/alumno/lab-baseline.txt` existe y tiene contenido (tamaño > 0).
- [ ] Contiene la versión exacta de la distribución (`Ubuntu 24.04.2 LTS`).
- [ ] Contiene la versión del kernel (`6.8.0-XX-generic`).
- [ ] Contiene la arquitectura (`x86_64` y `amd64`).
- [ ] Contiene la cantidad y lista de paquetes actualizables.
- [ ] Contiene las fuentes APT configuradas.

## Validación y Pruebas

Ejecuta las siguientes comprobaciones finales para confirmar que el laboratorio se completó correctamente.

### Prueba 1: Consistencia de la versión del SO

```bash
## Verificar que lsb_release y /etc/os-release reportan la misma versión
LSB_VER=$(lsb_release -rs)
OS_VER=$(grep "^VERSION_ID" /etc/os-release | cut -d'"' -f2)
if [ "$LSB_VER" = "$OS_VER" ]; then
  echo "PASS: Versiones consistentes — lsb_release ($LSB_VER) = os-release ($OS_VER)"
else
  echo "FAIL: Inconsistencia — lsb_release ($LSB_VER) ≠ os-release ($OS_VER)"
fi
```

**Resultado esperado:**

```
PASS: Versiones consistentes — lsb_release (24.04) = os-release (24.04)
```

### Prueba 2: Kernel de la serie correcta

```bash
## Verificar que el kernel pertenece a la serie 6.8
KERNEL=$(uname -r)
if [[ "$KERNEL" == 6.8.* ]]; then
  echo "PASS: Kernel de la serie correcta — $KERNEL"
else
  echo "FAIL: Kernel inesperado — $KERNEL (se esperaba serie 6.8.x)"
fi
```

**Resultado esperado:**

```
PASS: Kernel de la serie correcta — 6.8.0-55-generic
```

### Prueba 3: Arquitectura correcta

```bash
## Verificar arquitectura x86_64 / amd64
ARCH_HW=$(uname -m)
ARCH_PKG=$(dpkg --print-architecture)
if [ "$ARCH_HW" = "x86_64" ] && [ "$ARCH_PKG" = "amd64" ]; then
  echo "PASS: Arquitectura correcta — hardware=$ARCH_HW, paquetes=$ARCH_PKG"
else
  echo "FAIL: Arquitectura inesperada — hardware=$ARCH_HW, paquetes=$ARCH_PKG"
fi
```

**Resultado esperado:**

```
PASS: Arquitectura correcta — hardware=x86_64, paquetes=amd64
```

### Prueba 4: Repositorios oficiales configurados

```bash
## Verificar que las fuentes APT apuntan a servidores oficiales de Ubuntu
if grep -qE "archive\.ubuntu\.com|security\.ubuntu\.com" /etc/apt/sources.list.d/ubuntu.sources; then
  echo "PASS: Repositorios oficiales de Ubuntu configurados"
else
  echo "FAIL: No se encontraron repositorios oficiales de Ubuntu"
fi
```

**Resultado esperado:**

```
PASS: Repositorios oficiales de Ubuntu configurados
```

### Prueba 5: Integridad del archivo baseline

```bash
## Verificar que el archivo baseline contiene todas las secciones requeridas
BASELINE="/home/alumno/lab-baseline.txt"
ERRORS=0
for CAMPO in "PRETTY_NAME" "Versión del kernel" "uname -m" "Cantidad:" "archive.ubuntu.com"; do
  if grep -q "$CAMPO" "$BASELINE"; then
    echo "PASS: Campo '$CAMPO' encontrado en baseline"
  else
    echo "FAIL: Campo '$CAMPO' NO encontrado en baseline"
    ERRORS=$((ERRORS + 1))
  fi
done
echo ""
if [ $ERRORS -eq 0 ]; then
  echo "RESULTADO FINAL: Baseline completo — todos los campos presentes"
else
  echo "RESULTADO FINAL: Baseline incompleto — $ERRORS campo(s) faltante(s)"
fi
```

**Resultado esperado:**

```
PASS: Campo 'PRETTY_NAME' encontrado en baseline
PASS: Campo 'Versión del kernel' encontrado en baseline
PASS: Campo 'uname -m' encontrado en baseline
PASS: Campo 'Cantidad:' encontrado en baseline
PASS: Campo 'archive.ubuntu.com' encontrado en baseline

RESULTADO FINAL: Baseline completo — todos los campos presentes
```

### Prueba 6: Caso adverso — Información inexistente

Esta prueba valida que las herramientas de inspección del sistema reportan correctamente cuando se consulta información que no existe, en lugar de devolver datos inventados o engañosos.

```bash
## Intentar consultar un campo inexistente en os-release
RESULTADO=$(grep "^FABRICANTE_INVENTADO=" /etc/os-release)
if [ -z "$RESULTADO" ]; then
  echo "PASS: El archivo os-release no contiene campos inventados (comportamiento correcto)"
else
  echo "FAIL: Se encontró un campo inesperado: $RESULTADO"
fi

## Intentar usar lsb_release con una opción inválida
lsb_release --opcion-falsa 2>/dev/null
if [ $? -ne 0 ]; then
  echo "PASS: lsb_release rechaza opciones inválidas correctamente"
else
  echo "FAIL: lsb_release aceptó una opción inválida (comportamiento inesperado)"
fi
```

**Resultado esperado:**

```
PASS: El archivo os-release no contiene campos inventados (comportamiento correcto)
PASS: lsb_release rechaza opciones inválidas correctamente
```

> **Reflexión pedagógica:** Esta prueba refuerza un principio fundamental de la validación de entornos: las herramientas del sistema deben fallar de forma predecible cuando se les pide información que no existe. En contextos profesionales, confiar en salidas no verificadas puede llevar a decisiones erróneas. Siempre valida que la ausencia de datos se reporte como tal.

## Solución de Problemas

### Problema 1: `lsb_release` muestra una versión diferente a 24.04.2

**Síntomas:**

El campo `Description` de `lsb_release -a` muestra `Ubuntu 24.04 LTS` o `Ubuntu 24.04.1 LTS` en lugar de `Ubuntu 24.04.2 LTS`.

**Causa:**

La máquina virtual fue instalada con una imagen ISO diferente a `ubuntu-24.04.2-desktop-amd64.iso`. Las versiones point release de Ubuntu (`.1`, `.2`, etc.) incluyen diferentes conjuntos de parches y versiones de kernel preinstaladas. El campo `Description` refleja la ISO utilizada durante la instalación.

**Solución:**

1. Verifica qué ISO se utilizó consultando el log de instalación (si está disponible):

```bash
cat /var/log/installer/media-info 2>/dev/null || echo "Archivo no disponible"
```

2. Si la versión es `24.04` o `24.04.1`, la VM debe ser reinstalada con la ISO correcta. Descarga la ISO oficial desde:

```
https://releases.ubuntu.com/24.04/ubuntu-24.04.2-desktop-amd64.iso
```

3. Antes de reinstalar, verifica el hash SHA256 de la ISO descargada:

```bash
sha256sum ubuntu-24.04.2-desktop-amd64.iso
```

Compara el resultado con el hash publicado en [https://releases.ubuntu.com/24.04/SHA256SUMS](https://releases.ubuntu.com/24.04/SHA256SUMS).

4. Reinstala la VM y repite el laboratorio desde el Paso 1.

### Problema 2: `sudo apt update` falla con errores de conexión

**Síntomas:**

Al ejecutar `sudo apt update`, aparecen mensajes como:

```
Err:1 http://archive.ubuntu.com/ubuntu noble InRelease
  Could not resolve 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble/InRelease
   Could not resolve 'archive.ubuntu.com'
```

**Causa:**

La VM no tiene conectividad a internet. Las causas más comunes son: el adaptador de red NAT no está habilitado en VirtualBox, el cable virtual no está conectado, o la resolución DNS no funciona dentro de la VM.

**Solución:**

1. Verifica que el adaptador de red NAT está activo en VirtualBox:
   - Apaga la VM si es necesario.
   - En VirtualBox → Configuración de la VM → Red → Adaptador 1: debe estar habilitado y configurado como **NAT**.
   - Marca la casilla **Cable conectado**.

2. Dentro de la VM, verifica la interfaz de red:

```bash
ip addr show enp0s3
```

Debe mostrar una dirección IP asignada por DHCP (típicamente `10.0.2.15/24`).

3. Si no hay IP asignada, solicita una:

```bash
sudo dhclient enp0s3
```

4. Prueba la resolución DNS:

```bash
ping -c 3 archive.ubuntu.com
```

5. Si el DNS no resuelve, configura un DNS temporal:

```bash
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

6. Vuelve a ejecutar `sudo apt update`.

## Limpieza

Este laboratorio no requiere limpieza, ya que no se instaló ni modificó software del sistema. El archivo `/home/alumno/lab-baseline.txt` creado en el Paso 6 es un artefacto acumulativo que **debe conservarse** para los laboratorios posteriores (01-00-02 y siguientes).

**No elimines el archivo baseline.** Si necesitas regenerarlo por un error, repite el Paso 6 completo.

Verifica que el archivo permanece intacto al cerrar la sesión:

```bash
wc -l /home/alumno/lab-baseline.txt
```

El archivo debe tener al menos 25 líneas de contenido.

## Resumen

En este laboratorio has completado la primera fase de validación del entorno de laboratorio para el curso piloto de la Setup Guide de Thor. Has aplicado los conceptos de la Lección 1.1 para:

1. **Verificar la distribución** usando dos métodos complementarios: `lsb_release -a` para inspección interactiva y `/etc/os-release` para consultas estructuradas compatibles con scripts.
2. **Documentar el kernel y la arquitectura** con `uname -r`, `uname -m` y `dpkg --print-architecture`, confirmando la equivalencia entre `x86_64` y `amd64`.
3. **Validar las fuentes APT** inspeccionando `/etc/apt/sources.list.d/ubuntu.sources` para confirmar que los repositorios apuntan a servidores oficiales de Ubuntu con firmas GPG válidas.
4. **Sincronizar el catálogo de paquetes** con `sudo apt update` y listar las actualizaciones pendientes con `apt list --upgradable`, sin aplicar ningún cambio al sistema.
5. **Crear el archivo de baseline acumulativo** `/home/alumno/lab-baseline.txt` con toda la información recopilada, estableciendo el punto de referencia para comparaciones en laboratorios futuros.

### Relación con el siguiente laboratorio

En el **Lab 01-00-02**, aplicarás las actualizaciones pendientes identificadas en este laboratorio y actualizarás el archivo baseline para registrar el estado post-actualización. La comparación entre el estado inicial (documentado aquí) y el estado posterior permitirá verificar que las actualizaciones se aplicaron correctamente.

### Recursos adicionales

| Recurso | URL |
|---|---|
| Documentación oficial de `uname` — GNU Coreutils | [https://www.gnu.org/software/coreutils/manual/html_node/uname-invocation.html](https://www.gnu.org/software/coreutils/manual/html_node/uname-invocation.html) |
| Especificación del archivo `os-release` — freedesktop.org | [https://www.freedesktop.org/software/systemd/man/os-release.html](https://www.freedesktop.org/software/systemd/man/os-release.html) |
| Ubuntu 24.04 Release Notes | [https://discourse.ubuntu.com/t/noble-numbat-release-notes/39890](https://discourse.ubuntu.com/t/noble-numbat-release-notes/39890) |
| Formato DEB822 para fuentes APT | [https://manpages.ubuntu.com/manpages/noble/man5/sources.list.5.html](https://manpages.ubuntu.com/manpages/noble/man5/sources.list.5.html) |
| SHA256SUMS de Ubuntu 24.04 | [https://releases.ubuntu.com/24.04/SHA256SUMS](https://releases.ubuntu.com/24.04/SHA256SUMS) |

---

# Lab: Actualizar el sistema

## Metadatos

| Campo | Valor |
|---|---|
| **Duración** | 40 minutos |
| **Complejidad** | Fácil |
| **Nivel de Bloom** | Aplicar |
| **Tecnologías** | Ubuntu Desktop 24.04.2 LTS, APT 2.7.14, bash 5.2.21, systemd 255.4, util-linux 2.39.3 |
| **Prerrequisito directo** | Lab 01-00-01 completado |

## Descripción General

En este laboratorio aplicarás la secuencia completa de actualización de paquetes en Ubuntu Desktop 24.04.2 LTS: refresco del índice de repositorios, aplicación de actualizaciones, gestión de dependencias cambiantes, limpieza de paquetes huérfanos y caché local. Compararás el estado del kernel y los paquetes antes y después de la actualización, utilizando como referencia el archivo `lab-baseline.txt` creado en el laboratorio 01-00-01. Si el kernel fue actualizado, ejecutarás un reinicio controlado y verificarás el arranque correcto. Finalmente, documentarás el estado post-actualización como nuevo baseline de referencia para los módulos siguientes del curso piloto de validación de la Setup Guide de Thor.

## Objetivos de Aprendizaje

- [ ] Aplicar todas las actualizaciones de paquetes pendientes usando `apt update`, `apt upgrade -y` y `apt full-upgrade -y`, verificando la ejecución exitosa de cada comando.
- [ ] Comparar la versión del kernel antes y después de la actualización usando `uname -r` y el baseline registrado en `lab-baseline.txt`.
- [ ] Reiniciar el sistema de forma controlada si el kernel fue actualizado, y confirmar que arranca correctamente con el nuevo kernel.
- [ ] Documentar el estado post-actualización del sistema (kernel, fecha, paquetes actualizados) como nuevo baseline de referencia en `lab-baseline.txt`.

## Prerrequisitos

### Conocimientos previos

- Comprensión de las tres capas del sistema Linux: distribución, arquitectura y kernel (lección 1.1).
- Capacidad de interpretar la salida de `uname -r`, `lsb_release -a` y `/etc/os-release`.
- Uso básico de la terminal bash y el editor de texto desde línea de comandos.
- Comprensión del concepto de gestor de paquetes APT y la diferencia entre `update`, `upgrade` y `full-upgrade`.

### Acceso y recursos

- Laboratorio 01-00-01 completado: archivo `/home/alumno/lab-baseline.txt` existente con el baseline inicial del sistema.
- Máquina virtual Ubuntu Desktop 24.04.2 LTS ejecutándose en VirtualBox 7.0.18 (o VMware Workstation Pro 17.5.2 como alternativa documentada).
- Conexión a internet activa en la VM (Adaptador 1 en modo NAT configurado en VirtualBox).
- Sesión iniciada como usuario `alumno` con permisos `sudo` configurados (contraseña: `Lab2024!`).

## Entorno de Laboratorio

### Especificaciones de la máquina virtual

| Componente | Valor requerido |
|---|---|
| **Sistema operativo** | Ubuntu Desktop 24.04.2 LTS (Noble Numbat) — ISO: `ubuntu-24.04.2-desktop-amd64.iso` |
| **Fuente oficial ISO** | [https://releases.ubuntu.com/24.04/](https://releases.ubuntu.com/24.04/) |
| **Hipervisor** | Oracle VirtualBox 7.0.18 — [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) |
| **CPU asignada** | Mínimo 2 núcleos |
| **RAM asignada** | Mínimo 4 GB (host con 8 GB mínimo, 16 GB recomendado) |
| **Disco principal** | 30 GB |
| **Red** | Adaptador 1: NAT (acceso a internet) |
| **Usuario** | `alumno` |
| **Contraseña** | `Lab2024!` |

### Software involucrado

| Herramienta | Versión exacta | Fuente |
|---|---|---|
| APT (Advanced Package Tool) | 2.7.14 (incluido en Ubuntu 24.04.2 LTS) | Paquete `apt` del repositorio oficial `archive.ubuntu.com` |
| bash | 5.2.21 (incluido en Ubuntu 24.04.2 LTS) | Paquete `bash` del repositorio oficial |
| systemd (reboot) | 255.4 (incluido en Ubuntu 24.04.2 LTS) | Paquete `systemd` del repositorio oficial |
| util-linux (uname) | 2.39.3 (incluido en Ubuntu 24.04.2 LTS) | Paquete `util-linux` del repositorio oficial |
| lsb-release | 12.0-2 (incluido en Ubuntu 24.04.2 LTS) | Paquete `lsb-release` del repositorio oficial |

### Verificación del entorno antes de comenzar

Antes de ejecutar cualquier paso, confirma que el baseline del laboratorio anterior existe y que tienes conectividad:

```bash
## Verificar que el archivo baseline del lab 01-00-01 existe
ls -la /home/alumno/lab-baseline.txt

## Verificar conectividad a internet (necesaria para apt update)
ping -c 3 archive.ubuntu.com

## Verificar que el usuario tiene permisos sudo
sudo whoami
## Salida esperada: root
```

Si el archivo `lab-baseline.txt` no existe, **no continúes**: debes completar primero el laboratorio 01-00-01.

## Instrucciones Paso a Paso

### Paso 1: Registrar el estado pre-actualización del sistema

**Objetivo:** Capturar la versión del kernel, la lista de paquetes actualizables y la fecha actual como punto de comparación antes de aplicar cambios.

1. Abre una terminal en Ubuntu Desktop (atajo: `Ctrl+Alt+T`).

2. Registra la versión actual del kernel y la fecha en una variable temporal y muéstrala en pantalla:

    ```bash
    echo "========================================="
    echo "ESTADO PRE-ACTUALIZACIÓN"
    echo "========================================="
    echo "Fecha y hora: $(date '+%Y-%m-%d %H:%M:%S %Z')"
    echo "Kernel actual: $(uname -r)"
    echo "Distribución: $(grep PRETTY_NAME /etc/os-release | cut -d'"' -f2)"
    echo "Arquitectura: $(dpkg --print-architecture)"
    ```

3. Guarda la versión del kernel actual en una variable de shell para comparación posterior:

    ```bash
    KERNEL_PRE=$(uname -r)
    echo "Variable KERNEL_PRE establecida: $KERNEL_PRE"
    ```

4. Consulta el contenido actual del baseline para confirmar que contiene los datos del lab 01-00-01:

    ```bash
    echo "----- Contenido actual de lab-baseline.txt -----"
    cat /home/alumno/lab-baseline.txt
    echo "----- Fin del contenido -----"
    ```

5. Refresca el índice de paquetes y lista los paquetes actualizables sin instalar nada todavía:

    ```bash
    sudo apt update
    ```

6. Genera la lista de paquetes actualizables y cuenta cuántos hay:

    ```bash
    apt list --upgradable 2>/dev/null | tail -n +2 | tee /tmp/paquetes-actualizables.txt
    TOTAL_ACTUALIZABLES=$(wc -l < /tmp/paquetes-actualizables.txt)
    echo ""
    echo "Total de paquetes actualizables: $TOTAL_ACTUALIZABLES"
    ```

**Salida esperada:**

La salida de `sudo apt update` mostrará líneas `Hit:` y/o `Get:` para cada repositorio, finalizando con un resumen como:

```
All packages are up to date.
```

O bien:

```
N packages can be upgraded. Run 'apt list --upgradable' to see them.
```

La lista de paquetes actualizables mostrará líneas con formato `nombre-paquete/noble-updates versión_nueva arch [upgradable from: versión_actual]` o estará vacía si el sistema ya está actualizado.

**Verificación:**

```bash
## La variable KERNEL_PRE debe contener una versión válida del kernel
echo "$KERNEL_PRE" | grep -qE '^[0-9]+\.[0-9]+\.[0-9]+-[0-9]+-' && echo "✓ KERNEL_PRE válido: $KERNEL_PRE" || echo "✗ ERROR: KERNEL_PRE no tiene formato válido"

## El archivo de paquetes actualizables debe existir
[ -f /tmp/paquetes-actualizables.txt ] && echo "✓ Lista de paquetes actualizables generada ($TOTAL_ACTUALIZABLES paquetes)" || echo "✗ ERROR: No se generó la lista"
```

---

### Paso 2: Aplicar actualizaciones de paquetes existentes

**Objetivo:** Ejecutar `apt upgrade -y` para actualizar todos los paquetes que no requieran cambios de dependencias (instalación o eliminación de paquetes adicionales).

1. Ejecuta la actualización estándar de paquetes. La opción `-y` acepta automáticamente la confirmación:

    ```bash
    sudo apt upgrade -y 2>&1 | tee /tmp/apt-upgrade-output.txt
    ```

    > **Nota:** El comando `tee` guarda simultáneamente la salida en un archivo para documentación posterior, mientras la muestra en pantalla.

2. Verifica el código de retorno del comando:

    ```bash
    echo "Código de retorno de apt upgrade: $?"
    ```

3. Revisa el resumen de la actualización al final de la salida. Busca la línea que indica cuántos paquetes fueron actualizados:

    ```bash
    grep -E "^[0-9]+ upgraded" /tmp/apt-upgrade-output.txt || echo "No se encontró línea de resumen estándar; revisar salida completa."
    ```

**Salida esperada:**

Al finalizar, APT mostrará un resumen similar a:

```
XX upgraded, X newly installed, X to remove and X not upgraded.
```

Si no había paquetes pendientes, verás:

```
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

**Verificación:**

```bash
## El código de retorno debe ser 0 (éxito)
## Si el código fue distinto de 0, revisar /tmp/apt-upgrade-output.txt para diagnóstico
tail -5 /tmp/apt-upgrade-output.txt
```

---

### Paso 3: Aplicar actualizaciones con cambios de dependencias

**Objetivo:** Ejecutar `apt full-upgrade -y` para gestionar actualizaciones que requieren instalar nuevos paquetes o eliminar paquetes existentes por cambios de dependencias.

1. Ejecuta `full-upgrade` para resolver cualquier cambio de dependencias pendiente:

    ```bash
    sudo apt full-upgrade -y 2>&1 | tee /tmp/apt-full-upgrade-output.txt
    ```

2. Verifica el código de retorno:

    ```bash
    echo "Código de retorno de apt full-upgrade: $?"
    ```

3. Revisa si se realizaron cambios adicionales:

    ```bash
    grep -E "^[0-9]+ upgraded" /tmp/apt-full-upgrade-output.txt
    ```

**Salida esperada:**

En la mayoría de los casos, si `apt upgrade` ya aplicó todas las actualizaciones, `full-upgrade` reportará:

```
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

Si había paquetes retenidos o con dependencias complejas, verás actualizaciones adicionales.

**Verificación:**

```bash
## Confirmar que no quedan paquetes pendientes de actualización
apt list --upgradable 2>/dev/null | tail -n +2
## La salida debe estar vacía (sin líneas de paquetes)
PENDIENTES=$(apt list --upgradable 2>/dev/null | tail -n +2 | wc -l)
[ "$PENDIENTES" -eq 0 ] && echo "✓ Todos los paquetes están actualizados" || echo "✗ ATENCIÓN: Quedan $PENDIENTES paquetes sin actualizar"
```

---

### Paso 4: Verificar si el kernel fue actualizado

**Objetivo:** Comparar la versión del kernel registrada antes de la actualización con la versión del kernel más reciente instalada, para determinar si es necesario reiniciar.

1. Consulta la versión del kernel actualmente en ejecución (aún no ha cambiado porque no se ha reiniciado):

    ```bash
    KERNEL_ACTUAL=$(uname -r)
    echo "Kernel en ejecución (aún sin reiniciar): $KERNEL_ACTUAL"
    ```

2. Busca la versión más reciente del paquete de kernel instalado:

    ```bash
    KERNEL_INSTALADO=$(dpkg -l | grep -E '^ii\s+linux-image-[0-9]' | awk '{print $3}' | sort -V | tail -1)
    echo "Versión más reciente del paquete linux-image instalado: $KERNEL_INSTALADO"
    ```

3. Verifica también usando el directorio `/boot` para confirmar qué imágenes de kernel están disponibles:

    ```bash
    echo "Imágenes de kernel disponibles en /boot:"
    ls -la /boot/vmlinuz-* | awk '{print $NF}'
    ```

4. Determina si es necesario reiniciar. Ubuntu crea el archivo `/var/run/reboot-required` cuando una actualización requiere reinicio:

    ```bash
    if [ -f /var/run/reboot-required ]; then
        echo "⚠ REINICIO REQUERIDO"
        echo "Motivo:"
        cat /var/run/reboot-required.pkgs 2>/dev/null || echo "(sin detalle de paquetes)"
        NECESITA_REINICIO="SI"
    else
        echo "✓ No se requiere reinicio"
        NECESITA_REINICIO="NO"
    fi
    echo "Variable NECESITA_REINICIO: $NECESITA_REINICIO"
    ```

**Salida esperada:**

Si el kernel fue actualizado, verás:

```
⚠ REINICIO REQUERIDO
Motivo:
linux-base
```

Si no fue actualizado:

```
✓ No se requiere reinicio
```

**Verificación:**

```bash
echo "Kernel pre-actualización (variable KERNEL_PRE): $KERNEL_PRE"
echo "Kernel en ejecución actual: $(uname -r)"
echo "Necesita reinicio: $NECESITA_REINICIO"
```

---

### Paso 5: Reiniciar el sistema (condicional) y verificar el nuevo kernel

**Objetivo:** Si el kernel fue actualizado, ejecutar un reinicio controlado y confirmar que el sistema arranca correctamente con el nuevo kernel. Si no fue actualizado, documentar que el reinicio no fue necesario.

> **Importante:** Antes de reiniciar, guarda temporalmente la versión pre-actualización del kernel en un archivo, ya que las variables de shell se pierden al reiniciar.

1. Guarda la información pre-reinicio en un archivo temporal:

    ```bash
    echo "$KERNEL_PRE" > /tmp/kernel-pre-update.txt
    echo "$TOTAL_ACTUALIZABLES" > /tmp/total-actualizables.txt
    echo "Datos pre-reinicio guardados en /tmp/"
    ```

2. **Si se requiere reinicio** (`NECESITA_REINICIO="SI"`), ejecuta el reinicio controlado:

    ```bash
    # SOLO ejecutar si NECESITA_REINICIO="SI"
    sudo reboot
    ```

    > Espera a que la VM se reinicie completamente y vuelve a iniciar sesión como `alumno`.

3. **Después del reinicio** (o si no fue necesario reiniciar), abre una terminal y verifica el kernel en ejecución:

    ```bash
    echo "========================================="
    echo "VERIFICACIÓN POST-REINICIO"
    echo "========================================="
    echo "Fecha y hora: $(date '+%Y-%m-%d %H:%M:%S %Z')"
    echo "Kernel en ejecución: $(uname -r)"
    echo "Distribución: $(grep PRETTY_NAME /etc/os-release | cut -d'"' -f2)"
    echo "Uptime del sistema: $(uptime -p)"
    ```

4. Si reiniciaste, compara con la versión guardada:

    ```bash
    if [ -f /tmp/kernel-pre-update.txt ]; then
        KERNEL_PRE=$(cat /tmp/kernel-pre-update.txt)
        KERNEL_POST=$(uname -r)
        echo ""
        echo "Kernel ANTES de actualizar: $KERNEL_PRE"
        echo "Kernel DESPUÉS de actualizar: $KERNEL_POST"
        if [ "$KERNEL_PRE" != "$KERNEL_POST" ]; then
            echo "✓ El kernel fue actualizado exitosamente"
        else
            echo "ℹ El kernel no cambió (no se instaló nueva versión de kernel)"
        fi
    else
        echo "ℹ Archivo /tmp/kernel-pre-update.txt no encontrado (posiblemente limpiado en reinicio)."
        echo "Kernel actual: $(uname -r)"
        echo "Consultar lab-baseline.txt para la versión pre-actualización."
    fi
    ```

    > **Nota:** En algunos sistemas, el directorio `/tmp` se limpia al reiniciar. Si el archivo no existe, consulta el valor de kernel registrado en `lab-baseline.txt` del lab 01-00-01 como referencia.

**Salida esperada:**

```
=========================================
VERIFICACIÓN POST-REINICIO
=========================================
Fecha y hora: 2025-01-15 10:35:22 UTC
Kernel en ejecución: 6.8.0-52-generic
Distribución: Ubuntu 24.04.2 LTS
Uptime del sistema: up 1 minute
```

Si el kernel cambió:

```
Kernel ANTES de actualizar: 6.8.0-51-generic
Kernel DESPUÉS de actualizar: 6.8.0-52-generic
✓ El kernel fue actualizado exitosamente
```

**Verificación:**

```bash
## Confirmar que el sistema arrancó correctamente verificando que systemd está activo
systemctl is-system-running
## Salida esperada: "running" o "degraded" (degraded es aceptable si hay servicios no críticos fallidos)

## Verificar que no hay errores críticos en el arranque
journalctl -b -p err --no-pager | head -20
```

---

### Paso 6: Limpiar paquetes huérfanos y caché local

**Objetivo:** Eliminar paquetes que ya no son necesarios (dependencias huérfanas) y limpiar la caché de paquetes descargados para liberar espacio en disco.

1. Elimina paquetes huérfanos con `--purge` para remover también sus archivos de configuración:

    ```bash
    sudo apt autoremove --purge -y 2>&1 | tee /tmp/apt-autoremove-output.txt
    echo "Código de retorno: $?"
    ```

2. Limpia la caché local de paquetes descargados:

    ```bash
    sudo apt clean
    echo "Caché de APT limpiada"
    ```

3. Verifica el espacio en disco antes y después conceptualmente revisando el estado actual:

    ```bash
    echo "Espacio en disco del sistema de archivos raíz:"
    df -h / | awk 'NR==2{print "  Usado: "$3"  Disponible: "$4"  Uso: "$5}'
    ```

4. Confirma que el sistema de paquetes está en estado limpio:

    ```bash
    sudo apt update 2>&1 | tail -1
    apt list --upgradable 2>/dev/null | tail -n +2 | wc -l
    # Debe mostrar 0
    ```

**Salida esperada:**

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

La cuenta de paquetes actualizables debe ser `0`.

**Verificación:**

```bash
PENDIENTES_FINAL=$(apt list --upgradable 2>/dev/null | tail -n +2 | wc -l)
[ "$PENDIENTES_FINAL" -eq 0 ] && echo "✓ Sistema completamente actualizado y limpio" || echo "✗ Quedan $PENDIENTES_FINAL paquetes pendientes"
```

---

### Paso 7: Documentar el estado post-actualización en lab-baseline.txt

**Objetivo:** Actualizar el archivo acumulativo `/home/alumno/lab-baseline.txt` con la información post-actualización, incluyendo nueva versión de kernel, fecha, número de paquetes actualizados y estado del sistema.

1. Recupera la versión del kernel pre-actualización desde el baseline existente (si `/tmp/kernel-pre-update.txt` no sobrevivió al reinicio):

    ```bash
    # Intentar recuperar del archivo temporal; si no existe, extraer del baseline
    if [ -f /tmp/kernel-pre-update.txt ]; then
        KERNEL_PRE=$(cat /tmp/kernel-pre-update.txt)
    else
        KERNEL_PRE=$(grep -m1 "Kernel" /home/alumno/lab-baseline.txt | awk '{print $NF}')
    fi
    echo "Kernel pre-actualización: $KERNEL_PRE"
    ```

2. Recupera el total de paquetes actualizables identificados:

    ```bash
    if [ -f /tmp/total-actualizables.txt ]; then
        TOTAL_ACTUALIZABLES=$(cat /tmp/total-actualizables.txt)
    else
        TOTAL_ACTUALIZABLES="(dato no disponible tras reinicio - consultar salida del Paso 1)"
    fi
    echo "Paquetes que estaban pendientes: $TOTAL_ACTUALIZABLES"
    ```

3. Añade la sección de actualización al archivo baseline:

    ```bash
    cat >> /home/alumno/lab-baseline.txt << EOF

=========================================
LAB 01-00-02: ACTUALIZACIÓN DEL SISTEMA
Fecha de ejecución: $(date '+%Y-%m-%d %H:%M:%S %Z')
=========================================

--- Estado pre-actualización ---
Kernel antes de actualizar: $KERNEL_PRE
Paquetes pendientes de actualización: $TOTAL_ACTUALIZABLES

--- Estado post-actualización ---
Kernel después de actualizar: $(uname -r)
Distribución: $(grep PRETTY_NAME /etc/os-release | cut -d'"' -f2)
Arquitectura: $(dpkg --print-architecture)
Estado de systemd: $(systemctl is-system-running)
Reinicio ejecutado: $(if [ "$KERNEL_PRE" != "$(uname -r)" ]; then echo "SÍ (kernel cambió)"; else echo "NO (kernel sin cambios)"; fi)
Paquetes pendientes restantes: $(apt list --upgradable 2>/dev/null | tail -n +2 | wc -l)

--- Versiones de herramientas clave post-actualización ---
APT: $(apt --version | head -1)
bash: $(bash --version | head -1)
systemd: $(systemctl --version | head -1)
uname (util-linux): $(uname --version 2>/dev/null || dpkg -l util-linux | grep '^ii' | awk '{print $3}')

--- Verificación de integridad ---
Comando de verificación: apt list --upgradable (debe retornar 0 paquetes)
Resultado: $(apt list --upgradable 2>/dev/null | tail -n +2 | wc -l) paquetes pendientes
Sistema limpio (autoremove ejecutado): SÍ
Caché APT limpiada (apt clean ejecutado): SÍ
=========================================
EOF

    echo "✓ Baseline actualizado exitosamente"
    ```

4. Verifica el contenido completo del archivo baseline actualizado:

    ```bash
    echo "===== CONTENIDO COMPLETO DE lab-baseline.txt ====="
    cat /home/alumno/lab-baseline.txt
    echo "===== FIN DEL ARCHIVO ====="
    ```

5. Verifica la integridad del archivo (que no esté corrupto y tenga contenido razonable):

    ```bash
    LINEAS=$(wc -l < /home/alumno/lab-baseline.txt)
    BYTES=$(wc -c < /home/alumno/lab-baseline.txt)
    echo "Archivo lab-baseline.txt: $LINEAS líneas, $BYTES bytes"
    [ "$LINEAS" -gt 10 ] && echo "✓ El archivo tiene contenido suficiente" || echo "✗ ADVERTENCIA: El archivo parece demasiado corto"
    ```

**Salida esperada:**

El archivo `lab-baseline.txt` debe contener ahora dos secciones: la del lab 01-00-01 (baseline inicial) y la del lab 01-00-02 (actualización). La sección nueva incluirá datos como:

```
=========================================
LAB 01-00-02: ACTUALIZACIÓN DEL SISTEMA
Fecha de ejecución: 2025-01-15 10:42:18 UTC
=========================================

--- Estado pre-actualización ---
Kernel antes de actualizar: 6.8.0-51-generic
Paquetes pendientes de actualización: 47

--- Estado post-actualización ---
Kernel después de actualizar: 6.8.0-52-generic
Distribución: Ubuntu 24.04.2 LTS
Arquitectura: amd64
Estado de systemd: running
Reinicio ejecutado: SÍ (kernel cambió)
Paquetes pendientes restantes: 0
...
```

**Verificación:**

```bash
## Verificar que la sección del lab 01-00-02 existe en el archivo
grep -c "LAB 01-00-02" /home/alumno/lab-baseline.txt
## Debe retornar 1

## Verificar que se registró el kernel post-actualización
grep "Kernel después de actualizar" /home/alumno/lab-baseline.txt
## Debe mostrar la versión actual del kernel

## Verificar que se registraron 0 paquetes pendientes
grep "Paquetes pendientes restantes: 0" /home/alumno/lab-baseline.txt && echo "✓ Registro de 0 pendientes confirmado" || echo "✗ Revisar registro de paquetes pendientes"
```

## Validación y Pruebas

Ejecuta el siguiente bloque de validación integral para confirmar que todos los objetivos del laboratorio se cumplieron. **Todos los criterios deben pasar** para considerar el laboratorio completado.

### Validación automatizada

```bash
#!/bin/bash
## Script de validación integral - Lab 01-00-02
## Ejecutar como usuario alumno

echo "============================================="
echo "VALIDACIÓN INTEGRAL - LAB 01-00-02"
echo "Fecha: $(date '+%Y-%m-%d %H:%M:%S %Z')"
echo "============================================="
echo ""

ERRORES=0

## Criterio 1: El sistema no tiene paquetes pendientes de actualización
echo "[1/6] Verificando que no hay paquetes pendientes..."
PENDIENTES=$(apt list --upgradable 2>/dev/null | tail -n +2 | wc -l)
if [ "$PENDIENTES" -eq 0 ]; then
    echo "  ✓ PASA: 0 paquetes pendientes de actualización"
else
    echo "  ✗ FALLA: $PENDIENTES paquetes aún pendientes"
    ERRORES=$((ERRORES + 1))
fi

## Criterio 2: El archivo lab-baseline.txt contiene la sección del lab 01-00-02
echo "[2/6] Verificando sección LAB 01-00-02 en baseline..."
if grep -q "LAB 01-00-02" /home/alumno/lab-baseline.txt 2>/dev/null; then
    echo "  ✓ PASA: Sección LAB 01-00-02 encontrada en lab-baseline.txt"
else
    echo "  ✗ FALLA: Sección LAB 01-00-02 no encontrada"
    ERRORES=$((ERRORES + 1))
fi

## Criterio 3: El kernel registrado en el baseline coincide con el kernel en ejecución
echo "[3/6] Verificando coherencia del kernel registrado..."
KERNEL_REGISTRADO=$(grep "Kernel después de actualizar" /home/alumno/lab-baseline.txt 2>/dev/null | awk '{print $NF}')
KERNEL_ACTUAL=$(uname -r)
if [ "$KERNEL_REGISTRADO" = "$KERNEL_ACTUAL" ]; then
    echo "  ✓ PASA: Kernel registrado ($KERNEL_REGISTRADO) coincide con kernel en ejecución ($KERNEL_ACTUAL)"
else
    echo "  ✗ FALLA: Kernel registrado ($KERNEL_REGISTRADO) NO coincide con kernel en ejecución ($KERNEL_ACTUAL)"
    ERRORES=$((ERRORES + 1))
fi

## Criterio 4: systemd reporta estado operativo
echo "[4/6] Verificando estado de systemd..."
ESTADO_SYSTEMD=$(systemctl is-system-running 2>/dev/null)
if [ "$ESTADO_SYSTEMD" = "running" ] || [ "$ESTADO_SYSTEMD" = "degraded" ]; then
    echo "  ✓ PASA: systemd reporta estado '$ESTADO_SYSTEMD'"
    if [ "$ESTADO_SYSTEMD" = "degraded" ]; then
        echo "    ℹ Nota: 'degraded' indica servicios no críticos fallidos. Aceptable para laboratorio."
        echo "    Servicios fallidos:"
        systemctl --failed --no-pager 2>/dev/null | head -10
    fi
else
    echo "  ✗ FALLA: systemd reporta estado '$ESTADO_SYSTEMD'"
    ERRORES=$((ERRORES + 1))
fi

## Criterio 5: El baseline contiene ambas secciones (lab 01-00-01 y 01-00-02)
echo "[5/6] Verificando integridad acumulativa del baseline..."
SECCIONES=$(grep -c "^===.*===$" /home/alumno/lab-baseline.txt 2>/dev/null)
if [ "$SECCIONES" -ge 4 ]; then
    echo "  ✓ PASA: Baseline tiene $SECCIONES líneas de separación (múltiples secciones)"
else
    echo "  ⚠ ADVERTENCIA: Baseline tiene pocas secciones ($SECCIONES). Verificar manualmente que contiene datos de lab 01-00-01 y 01-00-02."
fi

## Criterio 6 (adversarial): Verificar que el baseline no contiene datos inconsistentes
echo "[6/6] Verificación adversarial: coherencia de datos en baseline..."
## Verificar que la distribución registrada es Ubuntu 24.04 (no otra distro ni versión)
DISTRO_BASELINE=$(grep "Distribución:" /home/alumno/lab-baseline.txt | tail -1 | cut -d':' -f2 | xargs)
if echo "$DISTRO_BASELINE" | grep -q "Ubuntu 24.04"; then
    echo "  ✓ PASA: Distribución registrada es coherente ($DISTRO_BASELINE)"
else
    echo "  ✗ FALLA: Distribución registrada no es Ubuntu 24.04 LTS ($DISTRO_BASELINE)"
    echo "    Esto podría indicar que el baseline fue editado manualmente con datos incorrectos"
    echo "    o que la VM no es la correcta. Verificar con: cat /etc/os-release"
    ERRORES=$((ERRORES + 1))
fi

## Verificar que la fecha registrada no es futura ni de hace más de 7 días
FECHA_BASELINE=$(grep "Fecha de ejecución:" /home/alumno/lab-baseline.txt | tail -1 | cut -d':' -f2- | xargs)
if [ -n "$FECHA_BASELINE" ]; then
    echo "  ℹ Fecha registrada en baseline: $FECHA_BASELINE"
    echo "  ℹ Fecha actual del sistema: $(date '+%Y-%m-%d %H:%M:%S %Z')"
    echo "  (Verificar manualmente que son coherentes)"
else
    echo "  ⚠ No se encontró fecha de ejecución en el baseline"
fi

echo ""
echo "============================================="
if [ "$ERRORES" -eq 0 ]; then
    echo "RESULTADO FINAL: ✓ TODOS LOS CRITERIOS PASARON"
    echo "El laboratorio 01-00-02 se completó exitosamente."
else
    echo "RESULTADO FINAL: ✗ $ERRORES CRITERIO(S) FALLARON"
    echo "Revisa los puntos marcados con ✗ y corrige antes de continuar."
fi
echo "============================================="
```

### Criterios de aceptación medibles

| # | Criterio | Evidencia | Resultado esperado |
|---|---|---|---|
| 1 | No quedan paquetes pendientes | `apt list --upgradable 2>/dev/null \| tail -n +2 \| wc -l` | `0` |
| 2 | Sección del lab 01-00-02 existe en baseline | `grep -c "LAB 01-00-02" /home/alumno/lab-baseline.txt` | `1` |
| 3 | Kernel registrado coincide con kernel en ejecución | Comparación de `grep "Kernel después" lab-baseline.txt` con `uname -r` | Valores idénticos |
| 4 | Sistema operativo arrancó correctamente | `systemctl is-system-running` | `running` o `degraded` |
| 5 | Baseline acumulativo tiene datos de ambos labs | Inspección visual de `cat lab-baseline.txt` | Secciones 01-00-01 y 01-00-02 presentes |
| 6 | Distribución registrada es coherente | `grep "Distribución" lab-baseline.txt` | Contiene "Ubuntu 24.04" |

### Caso adversarial: verificación de datos no fabricados

Este criterio asegura que el estudiante no copió datos de otro compañero ni editó manualmente el baseline con valores inventados:

```bash
## Verificar que el kernel registrado realmente existe en /boot
KERNEL_REG=$(grep "Kernel después de actualizar" /home/alumno/lab-baseline.txt | awk '{print $NF}')
if [ -f "/boot/vmlinuz-$KERNEL_REG" ]; then
    echo "✓ El kernel registrado ($KERNEL_REG) existe en /boot"
else
    echo "✗ ALERTA: El kernel registrado ($KERNEL_REG) NO existe en /boot"
    echo "  Esto indica que el baseline contiene datos que no corresponden a esta VM."
    echo "  Kernels disponibles en /boot:"
    ls /boot/vmlinuz-*
fi
```

## Solución de Problemas

### Problema 1: `apt update` falla con errores de conexión o repositorios

**Síntomas:**

```
Err:1 http://archive.ubuntu.com/ubuntu noble InRelease
  Could not resolve 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble/InRelease  Could not resolve 'archive.ubuntu.com'
W: Some index files failed to download. They have been ignored, or old ones used instead.
```

O bien:

```
E: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/noble-updates/...
   Connection timed out [IP: 91.189.91.82 80]
```

**Causa:** La máquina virtual no tiene conectividad a internet. Esto ocurre típicamente porque el Adaptador 1 de VirtualBox no está configurado en modo NAT, el cable virtual está desconectado, o el servicio DNS del host no está funcionando.

**Solución:**

1. Verifica la configuración de red de la VM en VirtualBox:
   - Apaga la VM si es necesario.
   - En VirtualBox → Configuración de la VM → Red → Adaptador 1: debe estar habilitado, en modo **NAT**, con "Cable conectado" marcado.

2. Dentro de la VM, verifica la conectividad:

    ```bash
    # Verificar que la interfaz tiene IP asignada
    ip addr show enp0s3

    # Probar conectividad básica
    ping -c 3 8.8.8.8

    # Probar resolución DNS
    ping -c 3 archive.ubuntu.com

    # Si la interfaz no tiene IP, reiniciar el servicio de red
    sudo systemctl restart NetworkManager
    # Esperar 10 segundos y volver a verificar
    sleep 10
    ip addr show enp0s3
    ```

3. Si la resolución DNS falla pero el ping por IP funciona, configura DNS temporalmente:

    ```bash
    echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
    ```

4. Reintenta `sudo apt update` después de restaurar la conectividad.

---

### Problema 2: El sistema no arranca correctamente después del reinicio (kernel panic o pantalla negra)

**Síntomas:**

Después de ejecutar `sudo reboot` tras la actualización del kernel, la VM muestra una pantalla negra, se queda en el logo de arranque indefinidamente, o muestra un mensaje de "kernel panic".

**Causa:** La actualización del kernel instaló una versión que es incompatible con la configuración actual de la VM (raro pero posible), o la imagen initramfs del nuevo kernel no se generó correctamente durante la actualización (puede ocurrir si el disco estaba lleno o si se interrumpió `apt upgrade`).

**Solución:**

1. **Acceder al menú GRUB para arrancar con el kernel anterior:**
   - Reinicia la VM.
   - Mantén presionada la tecla `Shift` (izquierda) durante el arranque, o presiona `Esc` repetidamente inmediatamente después del logo de VirtualBox.
   - En el menú GRUB, selecciona **"Advanced options for Ubuntu"**.
   - Selecciona la entrada con el kernel anterior (la versión que tenías antes de actualizar, registrada en `lab-baseline.txt`).

2. Una vez dentro del sistema con el kernel anterior, regenera la imagen initramfs del nuevo kernel:

    ```bash
    # Listar kernels instalados
    dpkg -l | grep linux-image

    # Identificar la versión del nuevo kernel (la que falló)
    NUEVO_KERNEL=$(dpkg -l | grep -E '^ii\s+linux-image-[0-9]' | awk '{print $2}' | sort -V | tail -1 | sed 's/linux-image-//')

    # Regenerar initramfs para ese kernel
    sudo update-initramfs -u -k "$NUEVO_KERNEL"

    # Actualizar GRUB
    sudo update-grub

    # Reiniciar para probar
    sudo reboot
    ```

3. Si el problema persiste, verifica que hay suficiente espacio en `/boot`:

    ```bash
    df -h /boot
    # Si /boot está lleno, eliminar kernels antiguos:
    sudo apt autoremove --purge -y
    ```

4. Si ninguna solución funciona, continúa el curso con el kernel anterior (funcional) y documenta la incidencia en `lab-baseline.txt`. El kernel anterior sigue siendo válido para los laboratorios posteriores.

## Limpieza

Este laboratorio **no requiere revertir cambios**: las actualizaciones aplicadas son el estado deseado para los laboratorios siguientes. El sistema actualizado es el punto de partida obligatorio para el Módulo 2.

Limpia únicamente los archivos temporales de diagnóstico creados durante el laboratorio:

```bash
## Eliminar archivos temporales de este laboratorio
rm -f /tmp/paquetes-actualizables.txt
rm -f /tmp/apt-upgrade-output.txt
rm -f /tmp/apt-full-upgrade-output.txt
rm -f /tmp/apt-autoremove-output.txt
rm -f /tmp/kernel-pre-update.txt
rm -f /tmp/total-actualizables.txt

echo "✓ Archivos temporales eliminados"

## Verificar que el archivo baseline permanece intacto
ls -la /home/alumno/lab-baseline.txt
echo "✓ lab-baseline.txt preservado ($(wc -l < /home/alumno/lab-baseline.txt) líneas)"
```

> **No eliminar** el archivo `/home/alumno/lab-baseline.txt`. Este archivo es acumulativo y será actualizado en cada laboratorio posterior hasta completar el curso piloto.

## Resumen

### Lo que se logró en este laboratorio

1. **Refrescaste el índice de repositorios** con `apt update`, asegurando que APT conoce las versiones más recientes disponibles en los repositorios oficiales de Ubuntu.
2. **Aplicaste actualizaciones estándar** con `apt upgrade -y`, actualizando todos los paquetes que no requerían cambios de dependencias.
3. **Gestionaste cambios de dependencias** con `apt full-upgrade -y`, resolviendo cualquier paquete retenido o con dependencias complejas.
4. **Determinaste si el kernel fue actualizado** comparando la versión pre y post-actualización, y verificando la existencia de `/var/run/reboot-required`.
5. **Ejecutaste un reinicio controlado** (si fue necesario) y confirmaste que el sistema arrancó correctamente con el nuevo kernel.
6. **Limpiaste el sistema** eliminando paquetes huérfanos (`apt autoremove --purge`) y la caché de APT (`apt clean`).
7. **Documentaste el estado post-actualización** en el archivo acumulativo `lab-baseline.txt`, estableciendo el nuevo baseline de referencia para los módulos siguientes.

### Relación con el curso piloto

Este laboratorio establece el **estado actualizado y documentado** del sistema operativo que será la base para:

- **Lab 02-00-01:** Configuración de swap persistente (requiere sistema actualizado).
- **Lab 02-00-02:** Particionado y formateo de discos adicionales (requiere kernel actualizado para soporte de dispositivos).
- **Lab 03-00-01:** Configuración de red con Netplan (requiere paquetes de red actualizados).

El archivo `lab-baseline.txt` acumulará información de cada laboratorio y será el **entregable final del curso piloto** para validar la Setup Guide de Thor.

### Comandos clave utilizados

| Comando | Propósito |
|---|---|
| `sudo apt update` | Refrescar índice de repositorios |
| `sudo apt upgrade -y` | Actualizar paquetes sin cambios de dependencias |
| `sudo apt full-upgrade -y` | Actualizar paquetes con cambios de dependencias |
| `sudo apt autoremove --purge -y` | Eliminar paquetes huérfanos y sus configuraciones |
| `sudo apt clean` | Limpiar caché local de paquetes descargados |
| `uname -r` | Mostrar versión del kernel en ejecución |
| `apt list --upgradable` | Listar paquetes pendientes de actualización |
| `systemctl is-system-running` | Verificar estado general de systemd |
| `sudo reboot` | Reiniciar el sistema de forma controlada |

### Recursos adicionales

- Documentación oficial de APT: [https://manpages.ubuntu.com/manpages/noble/en/man8/apt.8.html](https://manpages.ubuntu.com/manpages/noble/en/man8/apt.8.html)
- Ubuntu 24.04 Release Notes: [https://discourse.ubuntu.com/t/noble-numbat-release-notes/39890](https://discourse.ubuntu.com/t/noble-numbat-release-notes/39890)
- Guía de gestión de paquetes en Ubuntu: [https://ubuntu.com/server/docs/package-management](https://ubuntu.com/server/docs/package-management)
- Documentación del kernel de Ubuntu: [https://wiki.ubuntu.com/Kernel](https://wiki.ubuntu.com/Kernel)
