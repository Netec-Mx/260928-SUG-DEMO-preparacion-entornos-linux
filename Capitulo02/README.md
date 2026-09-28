# Configurar memoria swap

## Metadatos

| Campo | Valor |
|---|---|
| **Duración** | 60 minutos |
| **Complejidad** | Media |
| **Nivel Bloom** | Aplicar (*Apply*) |
| **Módulo** | 02 — Persistencia de la Configuración del Sistema |
| **Dependencia** | Lab 01-00-02 completado (sistema actualizado, baseline existente) |

## Descripción General

En este laboratorio crearás y configurarás memoria swap persistente en el disco principal de la máquina virtual Ubuntu Desktop 24.04.2 LTS. Partirás de un sistema sin swap activo, crearás un archivo dedicado de 2 GB en `/swapfile`, lo formatearás y activarás, y garantizarás su montaje automático en cada reinicio mediante `/etc/fstab`. Adicionalmente, ajustarás el parámetro de kernel `vm.swappiness` a un valor conservador (10) y lo harás persistente mediante `/etc/sysctl.d/`. Este laboratorio aplica directamente los conceptos de persistencia de configuración aprendidos en la lección 2.1 y prepara el terreno para el trabajo con `/etc/fstab` que se profundizará en el laboratorio 02-00-02.

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] Crear un archivo de swap de 2 GB usando `fallocate`, establecer permisos restrictivos y formatearlo con `mkswap`.
- [ ] Activar el archivo de swap con `swapon` y verificar su estado con `swapon --show` y `free -h`.
- [ ] Configurar la persistencia del swap en `/etc/fstab` para garantizar su activación automática en cada reinicio.
- [ ] Ajustar `vm.swappiness=10` de forma persistente mediante `/etc/sysctl.d/99-swappiness.conf` y verificar su aplicación tras reinicio.
- [ ] Actualizar el archivo de baseline acumulativo `/home/alumno/lab-baseline.txt` con la configuración de swap documentada.

## Prerrequisitos

### Conocimientos previos

| Requisito | Detalle |
|---|---|
| Conceptos de persistencia en Linux | Diferencia entre estado en tiempo de ejecución (RAM) y estado persistente (disco `/etc`), cubierto en la lección 2.1 |
| Uso básico de terminal | Navegación de directorios, ejecución de comandos con `sudo`, redirección de salida |
| Estructura de `/etc/fstab` | Comprensión de los seis campos: dispositivo, punto de montaje, tipo, opciones, dump, pass |
| Parámetros de kernel vía `sysctl` | Concepto de `/proc/sys/` como interfaz al kernel y uso de archivos en `/etc/sysctl.d/` |

### Acceso y estado del sistema

| Requisito | Verificación |
|---|---|
| Lab 01-00-02 completado | El sistema Ubuntu 24.04.2 LTS está actualizado y funcional |
| Archivo baseline existente | `ls -l /home/alumno/lab-baseline.txt` debe mostrar el archivo con contenido del lab anterior |
| Usuario `alumno` con `sudo` | `sudo whoami` debe devolver `root` |
| Espacio libre mínimo 3 GB en disco principal | `df -h /` debe mostrar al menos 3 GB disponibles |
| No existe swap activo previamente | `swapon --show` no debe devolver líneas de datos (solo encabezado o vacío) |

## Entorno de Laboratorio

### Software utilizado

| Componente | Versión exacta | Fuente oficial | Método de verificación |
|---|---|---|---|
| Ubuntu Desktop | 24.04.2 LTS (Noble Numbat) | [releases.ubuntu.com/24.04/](https://releases.ubuntu.com/24.04/) | `lsb_release -a` |
| util-linux (fallocate, swapon, mkswap) | 2.39.3 | Incluido en Ubuntu 24.04.2 LTS | `fallocate --version && swapon --version` |
| systemd (systemctl) | 255.4 | Incluido en Ubuntu 24.04.2 LTS | `systemctl --version` |
| bash | 5.2.21 | Incluido en Ubuntu 24.04.2 LTS | `bash --version` |
| e2fsprogs | 1.47.0 | Incluido en Ubuntu 24.04.2 LTS | `mkfs.ext4 -V 2>&1 | head -1` |

### Constantes del laboratorio

| Variable | Valor |
|---|---|
| Usuario del sistema | `alumno` |
| Contraseña | `Lab2024!` |
| Archivo de swap | `/swapfile` |
| Tamaño de swap | 2 GB |
| Valor de `vm.swappiness` | 10 |
| Archivo sysctl personalizado | `/etc/sysctl.d/99-swappiness.conf` |
| Archivo de baseline acumulativo | `/home/alumno/lab-baseline.txt` |

### Verificación inicial del entorno

Antes de comenzar, ejecuta los siguientes comandos para confirmar que el entorno cumple los prerrequisitos:

```bash
## Verificar versión del sistema operativo
lsb_release -d

## Verificar versión de util-linux (incluye fallocate, swapon, mkswap)
fallocate --version

## Verificar espacio disponible en disco raíz
df -h / | awk 'NR==2 {print "Espacio disponible:", $4}'

## Verificar que no existe swap activo
echo "=== Estado actual de swap ==="
swapon --show
free -h | grep -i swap

## Verificar que el archivo baseline existe
ls -l /home/alumno/lab-baseline.txt

## Verificar permisos sudo
sudo whoami
```

**Salida esperada (ejemplo):**

```
Description:    Ubuntu 24.04.2 LTS
fallocate from util-linux 2.39.3
Espacio disponible: 12G
=== Estado actual de swap ===
               total        used        free
Swap:             0B          0B          0B
-rw-rw-r-- 1 alumno alumno 1847 jun  5 10:30 /home/alumno/lab-baseline.txt
root
```

> **Nota:** Si `swapon --show` muestra una línea de swap activa o `free -h` muestra swap mayor a 0B, consulta la sección **Solución de Problemas** antes de continuar.

## Instrucciones Paso a Paso

### Paso 1: Documentar el estado inicial de la memoria swap

**Objetivo:** Registrar evidencia del estado del sistema antes de cualquier modificación, estableciendo una línea base verificable para comparación posterior.

**Instrucciones:**

1. Abre una terminal en Ubuntu Desktop (`Ctrl+Alt+T`).

2. Verifica que no existe swap activa en el sistema:

```bash
swapon --show
```

3. Consulta el estado de la memoria incluyendo swap:

```bash
free -h
```

4. Verifica que no existe un archivo `/swapfile` previo:

```bash
ls -la /swapfile 2>&1
```

5. Consulta el valor actual de `vm.swappiness` en el kernel:

```bash
cat /proc/sys/vm/swappiness
```

6. Verifica si existe configuración de swappiness persistente:

```bash
grep -r "swappiness" /etc/sysctl.conf /etc/sysctl.d/ 2>/dev/null
echo "Código de retorno: $?"
```

7. Verifica las entradas actuales de swap en `/etc/fstab`:

```bash
grep -i swap /etc/fstab
```

8. Registra el estado inicial en el archivo de baseline:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "========================================" >> /home/alumno/lab-baseline.txt
echo "LAB 02-00-01: Configuración de Swap" >> /home/alumno/lab-baseline.txt
echo "Fecha: $(date '+%Y-%m-%d %H:%M:%S')" >> /home/alumno/lab-baseline.txt
echo "========================================" >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Estado INICIAL de swap ---" >> /home/alumno/lab-baseline.txt
echo "swapon --show:" >> /home/alumno/lab-baseline.txt
swapon --show >> /home/alumno/lab-baseline.txt 2>&1
echo "free -h (línea Swap):" >> /home/alumno/lab-baseline.txt
free -h | grep -i swap >> /home/alumno/lab-baseline.txt
echo "vm.swappiness actual: $(cat /proc/sys/vm/swappiness)" >> /home/alumno/lab-baseline.txt
echo "Entradas swap en fstab:" >> /home/alumno/lab-baseline.txt
grep -i swap /etc/fstab >> /home/alumno/lab-baseline.txt 2>&1 || echo "(ninguna)" >> /home/alumno/lab-baseline.txt
```

**Salida esperada:**

- `swapon --show` no devuelve líneas de datos (vacío o solo encabezado).
- `free -h` muestra `Swap: 0B 0B 0B`.
- `ls -la /swapfile` devuelve `No such file or directory`.
- `vm.swappiness` muestra el valor por defecto del kernel: `60`.
- `grep -r "swappiness"` no devuelve resultados (código de retorno `1`).
- `grep -i swap /etc/fstab` puede no devolver resultados o mostrar una línea comentada.

**Verificación:**

```bash
## Confirmar que el baseline se actualizó correctamente
tail -15 /home/alumno/lab-baseline.txt
```

Debes ver la sección "LAB 02-00-01" con la fecha actual y el estado inicial documentado.

---

### Paso 2: Crear el archivo de swap de 2 GB

**Objetivo:** Crear un archivo dedicado de 2 GB en `/swapfile` usando `fallocate` y establecer los permisos de seguridad restrictivos requeridos por el kernel.

**Instrucciones:**

1. Crea el archivo de swap de 2 GB con `fallocate`:

```bash
sudo fallocate -l 2G /swapfile
```

2. Verifica que el archivo se creó con el tamaño correcto:

```bash
ls -lh /swapfile
```

3. Establece permisos restrictivos. Solo el propietario `root` debe poder leer y escribir el archivo. Esto es un requisito de seguridad del kernel: si los permisos son demasiado abiertos, `swapon` rechazará activar el archivo:

```bash
sudo chmod 600 /swapfile
```

4. Verifica los permisos aplicados:

```bash
ls -la /swapfile
```

5. Confirma que el propietario es `root`:

```bash
stat /swapfile | grep -E "Access:|Uid:|Gid:"
```

**Salida esperada:**

```
-rw------- 1 root root 2.0G jun  5 11:00 /swapfile
```

El campo de permisos debe ser exactamente `-rw-------` (600 en octal), propietario `root`, grupo `root`, tamaño `2.0G`.

**Verificación:**

```bash
## Verificación compacta: tamaño y permisos en una sola línea
stat --format="Permisos: %a | Propietario: %U:%G | Tamaño: %s bytes" /swapfile
```

**Salida esperada:**

```
Permisos: 600 | Propietario: root:root | Tamaño: 2147483648 bytes
```

> **Concepto clave:** 2 GB = 2 × 1024 × 1024 × 1024 = 2,147,483,648 bytes. Si el tamaño no coincide exactamente, el archivo no se creó correctamente.

---

### Paso 3: Formatear el archivo como área de swap

**Objetivo:** Inicializar el archivo `/swapfile` como espacio de intercambio válido usando `mkswap`, generando la estructura interna y el UUID que el kernel necesita para utilizarlo.

**Instrucciones:**

1. Formatea el archivo como swap:

```bash
sudo mkswap /swapfile
```

2. Observa la salida del comando. `mkswap` reportará el tamaño y generará un UUID único para el archivo de swap. **Anota el UUID** que aparece en la salida, ya que lo necesitarás para referencia.

3. Verifica el UUID asignado al archivo de swap:

```bash
sudo blkid /swapfile
```

4. Guarda el UUID en una variable de entorno para uso posterior en este laboratorio:

```bash
SWAP_UUID=$(sudo blkid -s UUID -o value /swapfile)
echo "UUID del swapfile: $SWAP_UUID"
```

**Salida esperada:**

La salida de `mkswap` será similar a:

```
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

La salida de `blkid` será similar a:

```
/swapfile: UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890" TYPE="swap"
```

> **Nota:** El UUID será diferente en cada sistema. El valor mostrado arriba es un ejemplo ilustrativo.

**Verificación:**

```bash
## Confirmar que el archivo tiene tipo swap reconocido
file /swapfile
```

**Salida esperada:**

```
/swapfile: Linux swap file, 4k page size, little endian, ...
```

El resultado debe indicar `Linux swap file`. Si muestra `data` o cualquier otro tipo, el formateo no fue exitoso.

---

### Paso 4: Activar el swap y verificar su funcionamiento

**Objetivo:** Activar el archivo de swap en tiempo de ejecución con `swapon` y verificar que el sistema lo reconoce correctamente como memoria de intercambio disponible.

**Instrucciones:**

1. Activa el archivo de swap:

```bash
sudo swapon /swapfile
```

2. Verifica la activación con `swapon --show`:

```bash
swapon --show
```

3. Verifica el estado completo de la memoria incluyendo swap:

```bash
free -h
```

4. Consulta información detallada del swap activo:

```bash
cat /proc/swaps
```

5. Compara el estado actual con el estado inicial que documentaste en el Paso 1:

```bash
echo "=== Comparación antes/después ==="
echo "ANTES: Swap total = 0B"
echo "AHORA:"
free -h | grep -i swap
```

**Salida esperada:**

Salida de `swapon --show`:

```
NAME      TYPE SIZE USED PRIO
/swapfile file   2G   0B   -2
```

Salida de `free -h` (línea Swap):

```
               total        used        free
Swap:          2.0Gi          0B       2.0Gi
```

Salida de `cat /proc/swaps`:

```
Filename                Type        Size        Used        Priority
/swapfile               file        2097148     0           -2
```

**Verificación:**

```bash
## Verificación numérica: el swap total debe ser exactamente 2 GiB
SWAP_TOTAL=$(free -b | awk '/Swap/ {print $2}')
EXPECTED=$((2 * 1024 * 1024 * 1024))
## Tolerancia de 8 KiB por overhead de metadatos de mkswap
if [ "$SWAP_TOTAL" -ge $((EXPECTED - 8192)) ] && [ "$SWAP_TOTAL" -le "$EXPECTED" ]; then
    echo "VERIFICACIÓN EXITOSA: Swap activo = $(free -h | awk '/Swap/ {print $2}')"
else
    echo "ERROR: Swap total ($SWAP_TOTAL bytes) no coincide con lo esperado (~$EXPECTED bytes)"
fi
```

> **Importante:** En este punto el swap está activo pero **no es persistente**. Si reinicias la VM ahora, el swap no se activará automáticamente. Los siguientes pasos resuelven esto.

---

### Paso 5: Configurar persistencia del swap en /etc/fstab

**Objetivo:** Agregar la entrada correspondiente en `/etc/fstab` para que el archivo de swap se active automáticamente en cada arranque del sistema, aplicando el concepto de persistencia cubierto en la lección 2.1.

**Instrucciones:**

1. Crea una copia de seguridad del archivo `/etc/fstab` actual antes de modificarlo:

```bash
sudo cp /etc/fstab /etc/fstab.bak.$(date +%Y%m%d%H%M%S)
```

2. Verifica que la copia se creó correctamente:

```bash
ls -la /etc/fstab.bak.*
```

3. Verifica que no existe ya una entrada para `/swapfile` en fstab (evitar duplicados):

```bash
grep "/swapfile" /etc/fstab
echo "Código de retorno: $? (1 = no encontrado, correcto)"
```

4. Agrega la entrada de swap al final de `/etc/fstab`. Usamos el formato estándar con la ruta del archivo (en este caso `/swapfile` en lugar de UUID, ya que los archivos de swap en sistemas de archivos no siempre exponen UUID de forma consistente vía `blkid` tras reinicios):

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

5. Verifica que la entrada se agregó correctamente:

```bash
grep "/swapfile" /etc/fstab
```

6. Valida la sintaxis completa de `/etc/fstab` para detectar errores antes de reiniciar:

```bash
sudo findmnt --verify --tab-file /etc/fstab
```

7. Prueba que `mount -a` no produce errores (esto simula parcialmente lo que ocurre durante el arranque):

```bash
sudo mount -a
echo "Código de retorno de mount -a: $?"
```

**Salida esperada:**

Salida de `grep "/swapfile" /etc/fstab`:

```
/swapfile none swap sw 0 0
```

Salida de `findmnt --verify` debe completarse sin errores fatales. Puede mostrar advertencias informativas que no afectan la funcionalidad.

Salida de `mount -a`: código de retorno `0` (sin errores).

**Verificación:**

```bash
## Mostrar las últimas 5 líneas de fstab para contexto
echo "=== Últimas 5 líneas de /etc/fstab ==="
tail -5 /etc/fstab
```

> **Concepto clave de la lección 2.1:** Sin esta entrada en `/etc/fstab`, el swap que activamos en el Paso 4 sería un cambio exclusivamente en tiempo de ejecución (volátil). La línea en `/etc/fstab` convierte el cambio en persistente: `systemd` la leerá durante el arranque y activará el swap automáticamente. Esto ilustra la diferencia entre estado *runtime* y estado *persistent* descrita en la lección.

---

### Paso 6: Configurar vm.swappiness de forma persistente

**Objetivo:** Establecer el parámetro de kernel `vm.swappiness` al valor 10 tanto en tiempo de ejecución como de forma persistente, creando el archivo `/etc/sysctl.d/99-swappiness.conf`.

**Instrucciones:**

1. Consulta el valor actual de `vm.swappiness`:

```bash
cat /proc/sys/vm/swappiness
```

   El valor por defecto es `60`. Para un entorno de laboratorio con RAM limitada, un valor de `10` indica al kernel que prefiera mantener datos en RAM y use swap solo cuando sea necesario.

2. Aplica el cambio en tiempo de ejecución (efecto inmediato, no persistente):

```bash
sudo sysctl -w vm.swappiness=10
```

3. Verifica que el cambio se aplicó en memoria:

```bash
cat /proc/sys/vm/swappiness
```

4. Ahora haz el cambio persistente creando el archivo de configuración dedicado en `/etc/sysctl.d/`:

```bash
echo "vm.swappiness=10" | sudo tee /etc/sysctl.d/99-swappiness.conf
```

> **¿Por qué `/etc/sysctl.d/99-swappiness.conf` y no `/etc/sysctl.conf`?** El directorio `/etc/sysctl.d/` permite organizar parámetros en archivos separados por función. El prefijo `99-` garantiza que se carga después de otros archivos, aplicándose como última palabra. Esta es la práctica recomendada en distribuciones modernas basadas en systemd.

5. Verifica el contenido del archivo creado:

```bash
cat /etc/sysctl.d/99-swappiness.conf
```

6. Verifica que el sistema puede cargar el archivo sin errores:

```bash
sudo sysctl -p /etc/sysctl.d/99-swappiness.conf
```

7. Confirma la coherencia entre el estado en memoria y el estado en disco (el flujo de verificación de la lección 2.1):

```bash
echo "=== Verificación de coherencia ==="
echo "Valor en memoria (/proc): $(cat /proc/sys/vm/swappiness)"
echo "Valor en disco (/etc/sysctl.d/): $(grep 'swappiness' /etc/sysctl.d/99-swappiness.conf)"
```

**Salida esperada:**

Salida de `sysctl -w`:

```
vm.swappiness = 10
```

Salida de `cat /proc/sys/vm/swappiness`:

```
10
```

Salida de la verificación de coherencia:

```
=== Verificación de coherencia ===
Valor en memoria (/proc): 10
Valor en disco (/etc/sysctl.d/): vm.swappiness=10
```

**Verificación:**

```bash
## Ambos valores deben coincidir en "10"
RUNTIME=$(cat /proc/sys/vm/swappiness)
PERSISTENT=$(grep -oP '\d+' /etc/sysctl.d/99-swappiness.conf)
if [ "$RUNTIME" = "$PERSISTENT" ] && [ "$RUNTIME" = "10" ]; then
    echo "VERIFICACIÓN EXITOSA: swappiness=$RUNTIME (coherente runtime/disco)"
else
    echo "ERROR: Discrepancia - runtime=$RUNTIME, disco=$PERSISTENT"
fi
```

---

### Paso 7: Reiniciar y verificar persistencia completa

**Objetivo:** Reiniciar la máquina virtual y confirmar que tanto el swap como el valor de `vm.swappiness` se aplican automáticamente durante el arranque, validando la persistencia de todas las configuraciones realizadas.

**Instrucciones:**

1. Antes de reiniciar, documenta el estado actual como referencia:

```bash
echo "=== Estado PRE-REINICIO ===" | tee /tmp/pre-reboot-check.txt
echo "Swap activo:" | tee -a /tmp/pre-reboot-check.txt
swapon --show | tee -a /tmp/pre-reboot-check.txt
echo "Swappiness: $(cat /proc/sys/vm/swappiness)" | tee -a /tmp/pre-reboot-check.txt
echo "fstab swap entry:" | tee -a /tmp/pre-reboot-check.txt
grep swapfile /etc/fstab | tee -a /tmp/pre-reboot-check.txt
```

2. Reinicia la máquina virtual:

```bash
sudo reboot
```

3. Después del reinicio, inicia sesión como `alumno` (contraseña: `Lab2024!`) y abre una terminal (`Ctrl+Alt+T`).

4. Verifica que el swap está activo automáticamente:

```bash
swapon --show
```

5. Verifica el estado de la memoria:

```bash
free -h
```

6. Verifica que `vm.swappiness` tiene el valor persistente:

```bash
cat /proc/sys/vm/swappiness
```

7. Verifica que no hay errores relacionados con swap en los logs del arranque:

```bash
journalctl -b 0 | grep -i swap
```

8. Compara con el estado pre-reinicio:

```bash
echo "=== Estado POST-REINICIO ==="
echo "Swap activo:"
swapon --show
echo "Swappiness: $(cat /proc/sys/vm/swappiness)"
echo ""
echo "=== Estado PRE-REINICIO (referencia) ==="
cat /tmp/pre-reboot-check.txt 2>/dev/null || echo "(archivo temporal eliminado por reinicio — esto es normal)"
```

**Salida esperada:**

Salida de `swapon --show` (post-reinicio):

```
NAME      TYPE SIZE USED PRIO
/swapfile file   2G   0B   -2
```

Salida de `free -h` (línea Swap):

```
Swap:          2.0Gi          0B       2.0Gi
```

Salida de `cat /proc/sys/vm/swappiness`:

```
10
```

> **Nota:** El archivo `/tmp/pre-reboot-check.txt` se habrá eliminado durante el reinicio porque `/tmp` es volátil. Esto es comportamiento esperado y sirve como ejemplo adicional de la diferencia entre almacenamiento persistente (`/etc`, `/home`) y volátil (`/tmp`, `/proc`, `/sys`).

**Verificación:**

```bash
## Verificación integral post-reinicio
ERRORS=0

## Verificar swap activo
if swapon --show | grep -q "/swapfile"; then
    echo "[OK] Swap /swapfile está activo"
else
    echo "[FALLO] Swap /swapfile NO está activo"
    ERRORS=$((ERRORS + 1))
fi

## Verificar tamaño de swap
SWAP_SIZE=$(free -m | awk '/Swap/ {print $2}')
if [ "$SWAP_SIZE" -ge 2000 ] && [ "$SWAP_SIZE" -le 2048 ]; then
    echo "[OK] Tamaño de swap correcto: ${SWAP_SIZE} MiB"
else
    echo "[FALLO] Tamaño de swap inesperado: ${SWAP_SIZE} MiB (esperado ~2048)"
    ERRORS=$((ERRORS + 1))
fi

## Verificar swappiness
SWAPPINESS=$(cat /proc/sys/vm/swappiness)
if [ "$SWAPPINESS" = "10" ]; then
    echo "[OK] vm.swappiness = $SWAPPINESS"
else
    echo "[FALLO] vm.swappiness = $SWAPPINESS (esperado: 10)"
    ERRORS=$((ERRORS + 1))
fi

## Verificar entrada en fstab
if grep -q "/swapfile.*swap" /etc/fstab; then
    echo "[OK] Entrada de swap presente en /etc/fstab"
else
    echo "[FALLO] Entrada de swap NO encontrada en /etc/fstab"
    ERRORS=$((ERRORS + 1))
fi

## Verificar archivo sysctl
if [ -f /etc/sysctl.d/99-swappiness.conf ] && grep -q "vm.swappiness=10" /etc/sysctl.d/99-swappiness.conf; then
    echo "[OK] Archivo /etc/sysctl.d/99-swappiness.conf correcto"
else
    echo "[FALLO] Archivo sysctl no existe o tiene contenido incorrecto"
    ERRORS=$((ERRORS + 1))
fi

echo ""
if [ "$ERRORS" -eq 0 ]; then
    echo "=== TODAS LAS VERIFICACIONES PASARON ==="
else
    echo "=== $ERRORS VERIFICACIÓN(ES) FALLARON — revisar antes de continuar ==="
fi
```

---

### Paso 8: Actualizar el archivo de baseline acumulativo

**Objetivo:** Documentar la configuración final de swap en el archivo `/home/alumno/lab-baseline.txt`, manteniendo el registro acumulativo que se construye a lo largo de todos los laboratorios del curso.

**Instrucciones:**

1. Agrega la configuración final de swap al baseline:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Estado FINAL de swap (post-reinicio) ---" >> /home/alumno/lab-baseline.txt
echo "Fecha verificación: $(date '+%Y-%m-%d %H:%M:%S')" >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "swapon --show:" >> /home/alumno/lab-baseline.txt
swapon --show >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "free -h:" >> /home/alumno/lab-baseline.txt
free -h >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "vm.swappiness (runtime): $(cat /proc/sys/vm/swappiness)" >> /home/alumno/lab-baseline.txt
echo "vm.swappiness (persistente): $(cat /etc/sysctl.d/99-swappiness.conf)" >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "Entrada en /etc/fstab:" >> /home/alumno/lab-baseline.txt
grep "/swapfile" /etc/fstab >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "UUID del swapfile:" >> /home/alumno/lab-baseline.txt
sudo blkid /swapfile >> /home/alumno/lab-baseline.txt 2>&1
echo "" >> /home/alumno/lab-baseline.txt
echo "Permisos del swapfile:" >> /home/alumno/lab-baseline.txt
ls -la /swapfile >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "Archivo sysctl:" >> /home/alumno/lab-baseline.txt
cat /etc/sysctl.d/99-swappiness.conf >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Fin Lab 02-00-01 ---" >> /home/alumno/lab-baseline.txt
```

2. Revisa la sección del lab 02-00-01 en el baseline:

```bash
sed -n '/LAB 02-00-01/,/Fin Lab 02-00-01/p' /home/alumno/lab-baseline.txt
```

3. Verifica la integridad general del archivo baseline:

```bash
echo "Tamaño del baseline: $(wc -l < /home/alumno/lab-baseline.txt) líneas"
echo "Secciones de laboratorio registradas:"
grep -c "^===" /home/alumno/lab-baseline.txt
```

**Salida esperada:**

La sección del lab 02-00-01 debe contener:
- Estado inicial (swap = 0B, swappiness = 60)
- Estado final (swap = 2.0Gi, swappiness = 10)
- Entrada de fstab
- UUID del swapfile
- Permisos del archivo
- Contenido del archivo sysctl

**Verificación:**

```bash
## Verificar que las secciones clave están presentes en el baseline
echo "=== Verificación de contenido del baseline ==="
for PATRON in "Estado INICIAL" "Estado FINAL" "vm.swappiness" "/swapfile" "99-swappiness.conf" "Fin Lab 02-00-01"; do
    if grep -q "$PATRON" /home/alumno/lab-baseline.txt; then
        echo "[OK] Sección encontrada: $PATRON"
    else
        echo "[FALLO] Sección faltante: $PATRON"
    fi
done
```

## Validación y Pruebas

### Prueba 1: Validación funcional completa post-reinicio

Esta prueba verifica que todos los componentes configurados funcionan correctamente tras el reinicio:

```bash
echo "============================================"
echo "VALIDACIÓN INTEGRAL — Lab 02-00-01"
echo "============================================"
echo ""

TOTAL=0
PASS=0

## Test 1: Archivo swapfile existe con tamaño correcto
TOTAL=$((TOTAL + 1))
SIZE=$(stat -c%s /swapfile 2>/dev/null)
if [ "$SIZE" = "2147483648" ]; then
    echo "[PASS] T1: /swapfile existe con tamaño 2 GiB ($SIZE bytes)"
    PASS=$((PASS + 1))
else
    echo "[FAIL] T1: /swapfile tamaño incorrecto o no existe (obtenido: $SIZE)"
fi

## Test 2: Permisos del swapfile
TOTAL=$((TOTAL + 1))
PERMS=$(stat -c%a /swapfile 2>/dev/null)
if [ "$PERMS" = "600" ]; then
    echo "[PASS] T2: Permisos de /swapfile = 600"
    PASS=$((PASS + 1))
else
    echo "[FAIL] T2: Permisos de /swapfile = $PERMS (esperado: 600)"
fi

## Test 3: Swap activo en /swapfile
TOTAL=$((TOTAL + 1))
if swapon --show | grep -q "/swapfile"; then
    echo "[PASS] T3: Swap activo en /swapfile"
    PASS=$((PASS + 1))
else
    echo "[FAIL] T3: Swap NO activo en /swapfile"
fi

## Test 4: Entrada en /etc/fstab
TOTAL=$((TOTAL + 1))
if grep -qE "^/swapfile\s+none\s+swap\s+sw\s+0\s+0" /etc/fstab; then
    echo "[PASS] T4: Entrada correcta en /etc/fstab"
    PASS=$((PASS + 1))
else
    echo "[FAIL] T4: Entrada en /etc/fstab ausente o con formato incorrecto"
    echo "       Contenido actual:"
    grep swapfile /etc/fstab || echo "       (no encontrado)"
fi

## Test 5: vm.swappiness en runtime = 10
TOTAL=$((TOTAL + 1))
RUNTIME_VAL=$(cat /proc/sys/vm/swappiness)
if [ "$RUNTIME_VAL" = "10" ]; then
    echo "[PASS] T5: vm.swappiness runtime = 10"
    PASS=$((PASS + 1))
else
    echo "[FAIL] T5: vm.swappiness runtime = $RUNTIME_VAL (esperado: 10)"
fi

## Test 6: Archivo sysctl persistente existe y es correcto
TOTAL=$((TOTAL + 1))
if [ -f /etc/sysctl.d/99-swappiness.conf ] && grep -q "vm.swappiness=10" /etc/sysctl.d/99-swappiness.conf; then
    echo "[PASS] T6: /etc/sysctl.d/99-swappiness.conf correcto"
    PASS=$((PASS + 1))
else
    echo "[FAIL] T6: Archivo sysctl ausente o con contenido incorrecto"
fi

## Test 7: Coherencia runtime vs. disco
TOTAL=$((TOTAL + 1))
DISK_VAL=$(grep -oP '\d+' /etc/sysctl.d/99-swappiness.conf 2>/dev/null)
if [ "$RUNTIME_VAL" = "$DISK_VAL" ]; then
    echo "[PASS] T7: Coherencia runtime ($RUNTIME_VAL) = disco ($DISK_VAL)"
    PASS=$((PASS + 1))
else
    echo "[FAIL] T7: Incoherencia runtime ($RUNTIME_VAL) ≠ disco ($DISK_VAL)"
fi

## Test 8: Baseline actualizado con sección del lab
TOTAL=$((TOTAL + 1))
if grep -q "Fin Lab 02-00-01" /home/alumno/lab-baseline.txt 2>/dev/null; then
    echo "[PASS] T8: Baseline contiene sección Lab 02-00-01"
    PASS=$((PASS + 1))
else
    echo "[FAIL] T8: Baseline no contiene sección Lab 02-00-01"
fi

echo ""
echo "============================================"
echo "RESULTADO: $PASS/$TOTAL pruebas exitosas"
if [ "$PASS" -eq "$TOTAL" ]; then
    echo "ESTADO: LABORATORIO COMPLETADO EXITOSAMENTE"
else
    echo "ESTADO: LABORATORIO INCOMPLETO — revisar pruebas fallidas"
fi
echo "============================================"
```

### Prueba 2: Caso adverso — verificar que el sistema rechaza configuraciones inseguras

Esta prueba valida que el kernel rechaza un archivo de swap con permisos inadecuados, reforzando la importancia de los permisos 600:

```bash
echo "=== Prueba adversa: permisos inseguros ==="

## Crear un archivo temporal de swap con permisos incorrectos
sudo fallocate -l 64M /tmp/test-swap-inseguro
sudo chmod 644 /tmp/test-swap-inseguro
sudo mkswap /tmp/test-swap-inseguro > /dev/null 2>&1

## Intentar activar swap con permisos 644 (inseguros)
echo "Intentando activar swap con permisos 644..."
sudo swapon /tmp/test-swap-inseguro 2>&1
RESULT=$?

if [ "$RESULT" -ne 0 ]; then
    echo "[PASS] El kernel rechazó correctamente el swap con permisos inseguros (644)"
    echo "       Esto confirma que los permisos 600 en /swapfile son un requisito de seguridad"
else
    echo "[ADVERTENCIA] El kernel aceptó swap con permisos 644"
    echo "       Desactivando swap de prueba..."
    sudo swapoff /tmp/test-swap-inseguro 2>/dev/null
fi

## Limpieza del archivo de prueba
sudo rm -f /tmp/test-swap-inseguro
echo "Archivo de prueba eliminado."
```

> **Nota sobre limitaciones:** En algunas versiones del kernel, `swapon` puede emitir una advertencia pero aún activar el swap con permisos 644. El comportamiento exacto depende de la versión del kernel (6.8.x en Ubuntu 24.04.2 LTS). En cualquier caso, los permisos 600 son la práctica obligatoria de seguridad.

### Prueba 3: Verificar que un cambio solo en runtime NO es persistente (prueba conceptual)

Esta prueba refuerza el concepto central de la lección 2.1 — la diferencia entre configuración volátil y persistente:

```bash
echo "=== Prueba conceptual: volatilidad de /proc ==="

## Cambiar swappiness temporalmente a 50 (solo en runtime)
sudo sysctl -w vm.swappiness=50 > /dev/null

echo "Valor en runtime tras cambio temporal: $(cat /proc/sys/vm/swappiness)"
echo "Valor en disco (persistente): $(cat /etc/sysctl.d/99-swappiness.conf)"

## Verificar que difieren
RUNTIME=$(cat /proc/sys/vm/swappiness)
DISK=$(grep -oP '\d+' /etc/sysctl.d/99-swappiness.conf)

if [ "$RUNTIME" != "$DISK" ]; then
    echo "[DEMOSTRADO] El cambio en runtime ($RUNTIME) NO modifica el disco ($DISK)"
    echo "             Tras reiniciar, el valor volverá a ser $DISK"
else
    echo "[ERROR] Los valores no deberían coincidir en esta prueba"
fi

## Restaurar el valor correcto en runtime
sudo sysctl -w vm.swappiness=10 > /dev/null
echo "Valor restaurado en runtime: $(cat /proc/sys/vm/swappiness)"
```

## Solución de Problemas

### Problema 1: `swapon` falla con "swapon: /swapfile: swapon failed: Invalid argument"

**Síntomas:** Al ejecutar `sudo swapon /swapfile`, el sistema devuelve el error `swapon failed: Invalid argument`. El swap no se activa.

**Causa:** Este error ocurre típicamente cuando el archivo fue creado con un método que genera un archivo disperso (*sparse file*) en lugar de un archivo con bloques completamente asignados. En sistemas de archivos como `ext4` con ciertas configuraciones, `fallocate` puede no funcionar como se espera, o el archivo fue creado con `truncate` en lugar de `fallocate`. También puede ocurrir si el sistema de archivos raíz usa `btrfs`, que no soporta swap files creados con `fallocate` directamente.

**Solución:**

```bash
## Paso 1: Desactivar el swap si está parcialmente activo
sudo swapoff /swapfile 2>/dev/null

## Paso 2: Eliminar el archivo existente
sudo rm -f /swapfile

## Paso 3: Verificar el sistema de archivos raíz
df -T / | awk 'NR==2 {print "Sistema de archivos:", $2}'

## Paso 4: Recrear el archivo usando dd (método universal, funciona en todos los FS)
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress

## Paso 5: Restablecer permisos
sudo chmod 600 /swapfile

## Paso 6: Formatear como swap
sudo mkswap /swapfile

## Paso 7: Activar
sudo swapon /swapfile

## Paso 8: Verificar
swapon --show
```

> **Nota:** `dd` es más lento que `fallocate` pero escribe ceros reales en cada bloque, garantizando que el archivo no es disperso. En sistemas con `ext4` (el caso estándar de Ubuntu 24.04.2 LTS), `fallocate` debería funcionar correctamente; este problema es más frecuente en `btrfs` o `xfs` con ciertas opciones de montaje.

### Problema 2: Tras reiniciar, el swap no se activa automáticamente (swap = 0B)

**Síntomas:** Después de reiniciar la VM, `swapon --show` no muestra ninguna entrada y `free -h` reporta `Swap: 0B 0B 0B`, a pesar de haber configurado `/etc/fstab` en el Paso 5.

**Causa:** La entrada en `/etc/fstab` tiene un error de sintaxis (espacios extra, tabulaciones incorrectas, ruta mal escrita) o fue agregada dentro de un comentario. Otra causa frecuente es que el archivo `/swapfile` fue eliminado accidentalmente o sus permisos fueron modificados antes del reinicio.

**Solución:**

```bash
## Paso 1: Verificar que el archivo swapfile existe y tiene el tamaño correcto
ls -la /swapfile
## Debe mostrar: -rw------- 1 root root 2147483648 ...

## Paso 2: Si no existe, recrearlo (volver al Paso 2 del laboratorio)
## Si existe, verificar permisos
stat -c "%a %U:%G" /swapfile
## Debe mostrar: 600 root:root

## Paso 3: Verificar la entrada en fstab (buscar errores de formato)
cat -A /etc/fstab | grep swapfile
## El flag -A muestra caracteres ocultos. La línea debe ser:
## /swapfile none swap sw 0 0$
## Sin caracteres ^I (tabulaciones mezcladas) ni espacios trailing inusuales

## Paso 4: Si la entrada tiene errores, corregirla
## Primero eliminar la línea incorrecta
sudo sed -i '/swapfile/d' /etc/fstab
## Luego agregar la línea correcta
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

## Paso 5: Verificar manualmente que se puede activar
sudo swapon /swapfile
swapon --show

## Paso 6: Verificar logs del arranque para errores específicos
journalctl -b 0 | grep -iE "swap|fstab|mount" | head -20

## Paso 7: Reiniciar nuevamente para confirmar la corrección
sudo reboot
```

## Limpieza

Este laboratorio **no requiere limpieza**. Todos los cambios realizados son intencionales y deben permanecer para los laboratorios posteriores:

- El archivo `/swapfile` de 2 GB debe permanecer activo.
- La entrada en `/etc/fstab` debe permanecer para garantizar persistencia.
- El archivo `/etc/sysctl.d/99-swappiness.conf` debe permanecer para mantener `vm.swappiness=10`.
- El archivo `/home/alumno/lab-baseline.txt` es acumulativo y se seguirá actualizando en el laboratorio 02-00-02.

**Archivos de respaldo creados durante el laboratorio** (pueden limpiarse opcionalmente):

```bash
## Opcional: listar copias de seguridad de fstab creadas
ls -la /etc/fstab.bak.* 2>/dev/null

## Estas copias pueden conservarse como referencia o eliminarse:
## sudo rm -f /etc/fstab.bak.*
```

> **Importante:** No desactives el swap ni elimines las configuraciones. El laboratorio 02-00-02 (discos adicionales) asume que el swap está configurado y operativo como se dejó en este laboratorio.

## Resumen

### Lo que se logró

En este laboratorio aplicaste los conceptos de persistencia de configuración de la lección 2.1 para configurar memoria swap completa y duradera en Ubuntu Desktop 24.04.2 LTS:

1. **Documentaste el estado inicial** del sistema (sin swap, swappiness=60) como línea base de comparación.
2. **Creaste un archivo de swap** de 2 GB con `fallocate`, estableciste permisos de seguridad (600) y lo formateaste con `mkswap`.
3. **Activaste el swap** con `swapon` y verificaste su funcionamiento con `swapon --show` y `free -h`.
4. **Configuraste la persistencia** en `/etc/fstab` para que el swap se active automáticamente en cada arranque.
5. **Ajustaste `vm.swappiness=10`** tanto en tiempo de ejecución (`sysctl -w`) como de forma persistente (`/etc/sysctl.d/99-swappiness.conf`).
6. **Reiniciaste y verificaste** que todas las configuraciones sobrevivieron al reinicio, confirmando la persistencia.
7. **Actualizaste el baseline acumulativo** con toda la información de configuración de swap.

### Conceptos clave reforzados

| Concepto | Aplicación en este laboratorio |
|---|---|
| Estado runtime vs. persistente | Swap activado con `swapon` (runtime) + entrada en `/etc/fstab` (persistente) |
| Archivos de configuración en `/etc` | `/etc/fstab` para swap, `/etc/sysctl.d/` para parámetros del kernel |
| Verificación de coherencia | Comparación de `/proc/sys/vm/swappiness` con `/etc/sysctl.d/99-swappiness.conf` |
| Seguridad de permisos | Permisos 600 obligatorios en `/swapfile` |

### Próximo laboratorio

En el **Lab 02-00-02** trabajarás con los discos adicionales de la VM (sdb y sdc de 5 GB cada uno), aplicando particionado, formateo y montaje persistente con UUID en `/etc/fstab`. Los conceptos de persistencia y el flujo de verificación runtime/disco que practicaste aquí se aplicarán directamente al montaje de sistemas de archivos.

### Recursos adicionales

| Recurso | URL |
|---|---|
| Página de manual de `swapon` (man7.org) | [https://man7.org/linux/man-pages/man8/swapon.8.html](https://man7.org/linux/man-pages/man8/swapon.8.html) |
| Página de manual de `fstab` (man7.org) | [https://man7.org/linux/man-pages/man5/fstab.5.html](https://man7.org/linux/man-pages/man5/fstab.5.html) |
| Documentación de Ubuntu: SwapFaq | [https://help.ubuntu.com/community/SwapFaq](https://help.ubuntu.com/community/SwapFaq) |
| Arch Wiki: Swap (referencia técnica detallada) | [https://wiki.archlinux.org/title/Swap](https://wiki.archlinux.org/title/Swap) |
| Documentación del kernel: sysctl/vm | [https://www.kernel.org/doc/html/latest/admin-guide/sysctl/vm.html](https://www.kernel.org/doc/html/latest/admin-guide/sysctl/vm.html) |

---

# Lab: Preparar los discos adicionales

## Metadatos

| Campo | Detalle |
|---|---|
| **Duración** | 60 minutos |
| **Complejidad** | Media |
| **Nivel Bloom** | Aplicar (*Apply*) |
| **Módulo** | 02 — Preparación de almacenamiento |
| **Dependencia directa** | Lab 02-00-01 (swap configurado y persistente) |

---

## Descripción General

En este laboratorio el estudiante identificará dos discos virtuales adicionales (5 GB cada uno) previamente conectados a la VM por el instructor, creará tablas de particiones GPT, formateará las particiones con el sistema de archivos ext4 y configurará su montaje persistente en `/etc/fstab` utilizando UUID. Al finalizar, ambos discos quedarán montados automáticamente en `/mnt/datos1` y `/mnt/datos2` tras cada reinicio, y la configuración se registrará en el archivo de baseline acumulativo del curso.

---

## Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Identificar de forma segura los discos adicionales conectados a la VM usando `lsblk` y `blkid`, distinguiéndolos del disco del sistema operativo.
- [ ] Crear tablas de particiones GPT y particiones primarias en discos vírgenes utilizando `fdisk`.
- [ ] Formatear particiones con el sistema de archivos ext4 usando `mkfs.ext4` y obtener sus UUID.
- [ ] Configurar montaje persistente en `/etc/fstab` basado en UUID, validar la sintaxis con `mount -a` y confirmar la persistencia tras un reinicio completo.

---

## Prerrequisitos

### Conocimientos previos

| Conocimiento | Referencia |
|---|---|
| Concepto de persistencia de configuración en Linux (estado runtime vs. estado en disco) | Lección 2.1 del curso |
| Uso de UUID frente a nombres de dispositivo en `/etc/fstab` | Lección 2.1 del curso |
| Navegación básica en terminal Linux y uso de `sudo` | Módulo 01 del curso |
| Concepto de tabla de particiones GPT | Conocimiento general de administración de sistemas |

### Acceso y recursos requeridos

| Requisito | Detalle |
|---|---|
| VM operativa | Ubuntu Desktop 24.04.2 LTS sobre VirtualBox 7.0.18 (o VMware Workstation Pro 17.5.2) |
| Lab 02-00-01 completado | Swap de 2 GB configurado y persistente; `vm.swappiness=10` |
| Discos adicionales | Dos discos VDI de 5 GB cada uno (`sdb` y `sdc`) conectados por el instructor, sin particionar ni formatear |
| Archivo baseline existente | `/home/alumno/lab-baseline.txt` creado en labs anteriores |
| Usuario | `alumno` con permisos `sudo` (contraseña: `Lab2024!`) |

> **Nota para el instructor:** antes de iniciar este laboratorio, verifique que cada VM tiene los dos discos adicionales conectados. Puede confirmarlo ejecutando `lsblk` dentro de la VM y comprobando que aparecen `sdb` y `sdc` (o `vdb`/`vdc` en entornos virtio, o `nvme0n2`/`nvme0n3` en VMware). Si los nombres difieren, comuníquelo a los estudiantes antes de comenzar.

---

## Entorno de Laboratorio

### Software utilizado

| Componente | Versión exacta | Fuente oficial | Método de verificación |
|---|---|---|---|
| Ubuntu Desktop | 24.04.2 LTS (Noble Numbat) | [releases.ubuntu.com/24.04](https://releases.ubuntu.com/24.04/) | `lsb_release -a` |
| util-linux (`lsblk`, `blkid`, `fdisk`) | 2.39.3 | Incluido en Ubuntu 24.04.2 LTS | `fdisk --version` |
| e2fsprogs (`mkfs.ext4`, `tune2fs`) | 1.47.0 | Incluido en Ubuntu 24.04.2 LTS | `mkfs.ext4 -V 2>&1 | head -1` |
| systemd (`systemctl`, `mount`) | 255.4 | Incluido en Ubuntu 24.04.2 LTS | `systemctl --version | head -1` |
| bash | 5.2.21 | Incluido en Ubuntu 24.04.2 LTS | `bash --version | head -1` |

### Arquitectura del laboratorio

```
┌──────────────────────────────────────────────────┐
│                   VM Ubuntu 24.04.2 LTS          │
│                                                  │
│  /dev/sda ─── Disco del sistema (30 GB)          │
│    ├── sda1 → /boot/efi                          │
│    ├── sda2 → /                                  │
│    └── (swap: /swapfile — lab 02-00-01)          │
│                                                  │
│  /dev/sdb ─── Disco adicional 1 (5 GB) → NUEVO  │
│    └── sdb1 → /mnt/datos1  (a configurar)       │
│                                                  │
│  /dev/sdc ─── Disco adicional 2 (5 GB) → NUEVO  │
│    └── sdc1 → /mnt/datos2  (a configurar)       │
│                                                  │
└──────────────────────────────────────────────────┘
```

### Verificación inicial del entorno

Antes de comenzar, abra una terminal (`Ctrl+Alt+T`) y ejecute los siguientes comandos para confirmar que el entorno está listo:

```bash
## Verificar versiones de herramientas
fdisk --version
mkfs.ext4 -V 2>&1 | head -1

## Confirmar que el archivo baseline existe
ls -la /home/alumno/lab-baseline.txt

## Confirmar que el swap del lab anterior está activo
swapon --show
```

**Salida esperada (ejemplo):**

```
fdisk from util-linux 2.39.3
mke2fs 1.47.0 (5-Feb-2023)
-rw-rw-r-- 1 alumno alumno ... /home/alumno/lab-baseline.txt
NAME      TYPE SIZE USED PRIO
/swapfile file   2G   0B   -2
```

Si `swapon --show` no muestra el swapfile, revise el laboratorio 02-00-01 antes de continuar.

---

## Instrucciones Paso a Paso

### Paso 1: Identificar los discos adicionales conectados a la VM

**Objetivo:** Distinguir de forma segura los discos del sistema operativo de los discos adicionales sin particionar, utilizando `lsblk` y `blkid`.

**Instrucciones:**

1. Abra una terminal (`Ctrl+Alt+T`) si no la tiene abierta.

2. Liste todos los dispositivos de bloque con información detallada:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,UUID
```

3. Observe la salida e identifique:
   - **`sda`**: disco del sistema con particiones montadas (p. ej., `/`, `/boot/efi`). **No lo toque.**
   - **`sdb`**: disco adicional 1, 5 GB, sin particiones ni sistema de archivos.
   - **`sdc`**: disco adicional 2, 5 GB, sin particiones ni sistema de archivos.

4. Confirme que `sdb` y `sdc` no tienen ningún sistema de archivos ni UUID asignado:

```bash
sudo blkid /dev/sdb
sudo blkid /dev/sdc
```

5. Registre los nombres de dispositivo detectados para referencia durante el laboratorio:

```bash
echo "=== Lab 02-00-02: Discos detectados (antes de particionar) ===" | tee -a /home/alumno/lab-baseline.txt
lsblk -o NAME,SIZE,TYPE,FSTYPE | tee -a /home/alumno/lab-baseline.txt
```

**Salida esperada:**

```
NAME   SIZE TYPE FSTYPE MOUNTPOINT                UUID
sda     30G disk
├─sda1 512M part vfat   /boot/efi                 XXXX-XXXX
└─sda2 29.5G part ext4  /                         xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
sdb      5G disk
sdc      5G disk
```

Los comandos `blkid` para `sdb` y `sdc` no deben devolver ninguna salida (discos vírgenes).

**Verificación:**

```bash
## Confirmar que sdb y sdc existen y tienen 5 GB
lsblk /dev/sdb /dev/sdc -o NAME,SIZE,TYPE --nodeps
```

Salida esperada:

```
NAME SIZE TYPE
sdb    5G disk
sdc    5G disk
```

> **⚠️ ADVERTENCIA CRÍTICA:** Si los discos adicionales aparecen con nombres diferentes (por ejemplo, `vdb`/`vdc` en entornos virtio o `nvme0n2`/`nvme0n3` en VMware), sustituya `sdb`/`sdc` por los nombres correctos en **todos** los comandos siguientes. Confirme con el instructor antes de continuar.

> **⚠️ PROTECCIÓN DEL DISCO DEL SISTEMA:** Nunca ejecute `fdisk`, `mkfs` ni ningún comando destructivo sobre `sda` (o el disco que contenga `/`). Un error aquí destruiría el sistema operativo. Siempre verifique dos veces el nombre del dispositivo antes de ejecutar comandos de particionado o formateo.

---

### Paso 2: Crear tabla de particiones GPT y partición en el disco sdb

**Objetivo:** Crear una tabla de particiones GPT en `/dev/sdb` y una partición primaria que ocupe todo el disco.

**Instrucciones:**

1. Inicie `fdisk` en modo interactivo sobre `/dev/sdb`:

```bash
sudo fdisk /dev/sdb
```

2. Dentro de `fdisk`, ejecute los siguientes comandos en secuencia (cada línea es una entrada seguida de `Enter`):

```
g        ← Crear nueva tabla de particiones GPT
n        ← Crear nueva partición
1        ← Número de partición: 1
[Enter]  ← Primer sector: aceptar valor por defecto
[Enter]  ← Último sector: aceptar valor por defecto (usa todo el disco)
w        ← Escribir cambios y salir
```

3. Verifique que la partición se creó correctamente:

```bash
lsblk /dev/sdb -o NAME,SIZE,TYPE,PARTTYPE
```

4. Confirme la tabla de particiones GPT:

```bash
sudo fdisk -l /dev/sdb
```

**Salida esperada (extracto de `fdisk -l`):**

```
Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
...
Disklabel type: gpt
...

Device     Start      End  Sectors  Size Type
/dev/sdb1   2048 10485726 10483679    5G Linux filesystem
```

**Verificación:**

```bash
## Confirmar que sdb1 existe y es de tipo 'part'
lsblk /dev/sdb -o NAME,SIZE,TYPE | grep sdb1
```

Salida esperada:

```
└─sdb1    5G part
```

---

### Paso 3: Crear tabla de particiones GPT y partición en el disco sdc

**Objetivo:** Repetir el proceso de particionado GPT en `/dev/sdc`.

**Instrucciones:**

1. Inicie `fdisk` sobre `/dev/sdc`:

```bash
sudo fdisk /dev/sdc
```

2. Ejecute la misma secuencia de comandos:

```
g        ← Crear nueva tabla de particiones GPT
n        ← Crear nueva partición
1        ← Número de partición: 1
[Enter]  ← Primer sector: aceptar valor por defecto
[Enter]  ← Último sector: aceptar valor por defecto
w        ← Escribir cambios y salir
```

3. Verifique la partición:

```bash
sudo fdisk -l /dev/sdc
```

**Salida esperada (extracto):**

```
Disklabel type: gpt
...
Device     Start      End  Sectors  Size Type
/dev/sdc1   2048 10485726 10483679    5G Linux filesystem
```

**Verificación:**

```bash
lsblk /dev/sdb /dev/sdc -o NAME,SIZE,TYPE
```

Salida esperada:

```
NAME   SIZE TYPE
sdb      5G disk
└─sdb1   5G part
sdc      5G disk
└─sdc1   5G part
```

---

### Paso 4: Formatear ambas particiones con ext4

**Objetivo:** Crear sistemas de archivos ext4 en `/dev/sdb1` y `/dev/sdc1` y obtener sus UUID.

**Instrucciones:**

1. Formatee `/dev/sdb1` con ext4:

```bash
sudo mkfs.ext4 -L datos1 /dev/sdb1
```

El parámetro `-L datos1` asigna la etiqueta `datos1` al sistema de archivos (útil como identificador secundario).

2. Formatee `/dev/sdc1` con ext4:

```bash
sudo mkfs.ext4 -L datos2 /dev/sdc1
```

3. Obtenga los UUID de ambas particiones recién formateadas:

```bash
sudo blkid /dev/sdb1
sudo blkid /dev/sdc1
```

4. **Anote los UUID** que aparecen en la salida. Los necesitará en el Paso 6 para configurar `/etc/fstab`. Ejemplo de salida:

```
/dev/sdb1: LABEL="datos1" UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="Linux filesystem" PARTUUID="..."
/dev/sdc1: LABEL="datos2" UUID="f9e8d7c6-b5a4-3210-fedc-ba0987654321" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="Linux filesystem" PARTUUID="..."
```

5. Guarde los UUID en variables de entorno para facilitar los pasos siguientes (sustituya los valores de ejemplo por los UUID reales de su sistema):

```bash
UUID_SDB1=$(sudo blkid -s UUID -o value /dev/sdb1)
UUID_SDC1=$(sudo blkid -s UUID -o value /dev/sdc1)
echo "UUID sdb1: $UUID_SDB1"
echo "UUID sdc1: $UUID_SDC1"
```

**Salida esperada de `mkfs.ext4` (extracto):**

```
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 1310459 4k blocks and 327680 inodes
Filesystem UUID: a1b2c3d4-e5f6-7890-abcd-ef1234567890
...
Writing superblocks and filesystem accounting information: done
```

**Verificación:**

```bash
## Confirmar que ambas particiones tienen sistema de archivos ext4 y UUID
lsblk /dev/sdb1 /dev/sdc1 -o NAME,SIZE,FSTYPE,UUID,LABEL
```

Salida esperada (los UUID serán diferentes en cada instalación):

```
NAME SIZE FSTYPE UUID                                 LABEL
sdb1   5G ext4   a1b2c3d4-e5f6-7890-abcd-ef1234567890 datos1
sdc1   5G ext4   f9e8d7c6-b5a4-3210-fedc-ba0987654321 datos2
```

---

### Paso 5: Crear los puntos de montaje y montar las particiones

**Objetivo:** Crear los directorios `/mnt/datos1` y `/mnt/datos2`, montar las particiones y verificar el montaje.

**Instrucciones:**

1. Cree los directorios de punto de montaje:

```bash
sudo mkdir -p /mnt/datos1
sudo mkdir -p /mnt/datos2
```

2. Monte las particiones manualmente (montaje temporal para verificar que funcionan):

```bash
sudo mount /dev/sdb1 /mnt/datos1
sudo mount /dev/sdc1 /mnt/datos2
```

3. Verifique el montaje con `df -h`:

```bash
df -h /mnt/datos1 /mnt/datos2
```

4. Verifique también con `mount | grep datos`:

```bash
mount | grep datos
```

5. Cree un archivo de prueba en cada punto de montaje para confirmar que la escritura funciona:

```bash
echo "Prueba de escritura en datos1 - $(date)" | sudo tee /mnt/datos1/test.txt
echo "Prueba de escritura en datos2 - $(date)" | sudo tee /mnt/datos2/test.txt
cat /mnt/datos1/test.txt
cat /mnt/datos2/test.txt
```

**Salida esperada de `df -h`:**

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       4.9G   24K  4.6G   1% /mnt/datos1
/dev/sdc1       4.9G   24K  4.6G   1% /mnt/datos2
```

> **Nota:** el tamaño reportado (~4.9G) es menor que 5 GB porque ext4 reserva espacio para metadatos y para el usuario root (típicamente un 5%).

**Verificación:**

```bash
## Confirmar que los archivos de prueba existen y tienen contenido
ls -la /mnt/datos1/test.txt /mnt/datos2/test.txt
```

---

### Paso 6: Configurar montaje persistente en /etc/fstab usando UUID

**Objetivo:** Agregar entradas en `/etc/fstab` basadas en UUID para que ambas particiones se monten automáticamente en cada arranque del sistema.

**Instrucciones:**

1. **Antes de modificar `/etc/fstab`, cree una copia de respaldo:**

```bash
sudo cp /etc/fstab /etc/fstab.bak.$(date +%Y%m%d-%H%M%S)
ls -la /etc/fstab.bak.*
```

2. Recupere los UUID (si no los tiene en variables de entorno):

```bash
UUID_SDB1=$(sudo blkid -s UUID -o value /dev/sdb1)
UUID_SDC1=$(sudo blkid -s UUID -o value /dev/sdc1)
echo "UUID sdb1: $UUID_SDB1"
echo "UUID sdc1: $UUID_SDC1"
```

3. Agregue las dos líneas al final de `/etc/fstab`:

```bash
echo "# Discos adicionales de laboratorio - Lab 02-00-02" | sudo tee -a /etc/fstab
echo "UUID=${UUID_SDB1}  /mnt/datos1  ext4  defaults  0  2" | sudo tee -a /etc/fstab
echo "UUID=${UUID_SDC1}  /mnt/datos2  ext4  defaults  0  2" | sudo tee -a /etc/fstab
```

4. Verifique que las líneas se agregaron correctamente:

```bash
cat /etc/fstab
```

5. **Comprenda cada campo de las líneas agregadas:**

| Campo | Valor | Significado |
|---|---|---|
| Dispositivo | `UUID=a1b2c3d4-...` | Identificador único del sistema de archivos (estable entre reinicios) |
| Punto de montaje | `/mnt/datos1` | Directorio donde se monta la partición |
| Tipo FS | `ext4` | Sistema de archivos |
| Opciones | `defaults` | rw, suid, dev, exec, auto, nouser, async |
| dump | `0` | No incluir en respaldos con `dump` |
| pass | `2` | Orden de verificación con `fsck` al arranque (2 = después del root) |

> **¿Por qué UUID y no `/dev/sdb1`?** Como se explicó en la lección 2.1, los nombres de dispositivo como `/dev/sdb` pueden cambiar si se agregan, eliminan o reordenan discos. El UUID es un identificador generado al crear el sistema de archivos y permanece constante independientemente del orden de detección del hardware. Usar UUID en `/etc/fstab` es la práctica recomendada para evitar que el sistema monte el dispositivo equivocado o falle al arrancar.

**Salida esperada (últimas líneas de `/etc/fstab`):**

```
## Discos adicionales de laboratorio - Lab 02-00-02
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /mnt/datos1  ext4  defaults  0  2
UUID=f9e8d7c6-b5a4-3210-fedc-ba0987654321  /mnt/datos2  ext4  defaults  0  2
```

**Verificación:**

```bash
## Confirmar que hay exactamente dos líneas con /mnt/datos en fstab
grep "/mnt/datos" /etc/fstab
```

Debe mostrar exactamente dos líneas, ambas con UUID.

---

### Paso 7: Validar la sintaxis de fstab con mount -a

**Objetivo:** Verificar que las entradas en `/etc/fstab` son sintácticamente correctas y funcionales sin necesidad de reiniciar.

**Instrucciones:**

1. Primero, desmonte ambas particiones (actualmente montadas manualmente desde el Paso 5):

```bash
sudo umount /mnt/datos1
sudo umount /mnt/datos2
```

2. Confirme que están desmontadas:

```bash
df -h | grep datos
```

Este comando no debe devolver ninguna salida.

3. Ejecute `mount -a` para montar todo lo definido en `/etc/fstab`:

```bash
sudo mount -a
```

Si el comando no produce ningún error, la sintaxis es correcta.

4. Verifique que ambas particiones se montaron correctamente:

```bash
df -h /mnt/datos1 /mnt/datos2
```

5. Verifique que los archivos de prueba creados en el Paso 5 siguen accesibles:

```bash
cat /mnt/datos1/test.txt
cat /mnt/datos2/test.txt
```

**Salida esperada de `df -h`:**

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       4.9G   24K  4.6G   1% /mnt/datos1
/dev/sdc1       4.9G   24K  4.6G   1% /mnt/datos2
```

**Verificación:**

```bash
## Verificación integral: lsblk muestra puntos de montaje
lsblk /dev/sdb /dev/sdc -o NAME,SIZE,FSTYPE,MOUNTPOINT,UUID
```

Salida esperada:

```
NAME   SIZE FSTYPE MOUNTPOINT  UUID
sdb      5G
└─sdb1   5G ext4   /mnt/datos1 a1b2c3d4-e5f6-7890-abcd-ef1234567890
sdc      5G
└─sdc1   5G ext4   /mnt/datos2 f9e8d7c6-b5a4-3210-fedc-ba0987654321
```

> **⚠️ Si `mount -a` produce un error:** No reinicie la VM. Un error en `/etc/fstab` puede impedir el arranque normal. Revise la sintaxis, compare con la copia de respaldo (`/etc/fstab.bak.*`) y corrija antes de continuar. Consulte la sección Solución de Problemas.

---

### Paso 8: Reiniciar y confirmar persistencia del montaje

**Objetivo:** Demostrar que la configuración de `/etc/fstab` es verdaderamente persistente verificando que ambos discos se montan automáticamente tras un reinicio completo.

**Instrucciones:**

1. Reinicie la VM:

```bash
sudo reboot
```

2. Después del reinicio, inicie sesión como `alumno` y abra una terminal.

3. Verifique que ambas particiones están montadas automáticamente:

```bash
df -h /mnt/datos1 /mnt/datos2
```

4. Verifique que los archivos de prueba persisten:

```bash
cat /mnt/datos1/test.txt
cat /mnt/datos2/test.txt
```

5. Verifique que el swap del laboratorio anterior también sigue activo (integridad del entorno):

```bash
swapon --show
```

6. Ejecute una verificación completa del estado de almacenamiento:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,UUID
```

**Salida esperada de `df -h`:**

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       4.9G   24K  4.6G   1% /mnt/datos1
/dev/sdc1       4.9G   24K  4.6G   1% /mnt/datos2
```

**Verificación:**

```bash
## Triple verificación: mount, df y lsblk coinciden
mount | grep "/mnt/datos"
```

Salida esperada:

```
/dev/sdb1 on /mnt/datos1 type ext4 (rw,relatime)
/dev/sdc1 on /mnt/datos2 type ext4 (rw,relatime)
```

---

### Paso 9: Actualizar el archivo de baseline acumulativo

**Objetivo:** Documentar la configuración de discos adicionales en el archivo de baseline del curso, incluyendo UUID, puntos de montaje y sistemas de archivos.

**Instrucciones:**

1. Agregue un separador y encabezado para este laboratorio:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "=============================================" >> /home/alumno/lab-baseline.txt
echo "Lab 02-00-02: Discos adicionales - $(date '+%Y-%m-%d %H:%M:%S')" >> /home/alumno/lab-baseline.txt
echo "=============================================" >> /home/alumno/lab-baseline.txt
```

2. Registre la información completa de los discos:

```bash
echo "--- Dispositivos de bloque ---" >> /home/alumno/lab-baseline.txt
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,UUID >> /home/alumno/lab-baseline.txt
```

3. Registre los UUID específicos de las particiones de datos:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "--- UUID de particiones de datos ---" >> /home/alumno/lab-baseline.txt
echo "sdb1 UUID: $(sudo blkid -s UUID -o value /dev/sdb1)" >> /home/alumno/lab-baseline.txt
echo "sdb1 LABEL: $(sudo blkid -s LABEL -o value /dev/sdb1)" >> /home/alumno/lab-baseline.txt
echo "sdb1 Montaje: /mnt/datos1" >> /home/alumno/lab-baseline.txt
echo "sdb1 FS: ext4" >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "sdc1 UUID: $(sudo blkid -s UUID -o value /dev/sdc1)" >> /home/alumno/lab-baseline.txt
echo "sdc1 LABEL: $(sudo blkid -s LABEL -o value /dev/sdc1)" >> /home/alumno/lab-baseline.txt
echo "sdc1 Montaje: /mnt/datos2" >> /home/alumno/lab-baseline.txt
echo "sdc1 FS: ext4" >> /home/alumno/lab-baseline.txt
```

4. Registre las líneas relevantes de `/etc/fstab`:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Entradas en /etc/fstab (discos adicionales) ---" >> /home/alumno/lab-baseline.txt
grep "/mnt/datos" /etc/fstab >> /home/alumno/lab-baseline.txt
```

5. Registre las versiones de herramientas utilizadas:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Versiones de herramientas ---" >> /home/alumno/lab-baseline.txt
echo "fdisk: $(fdisk --version)" >> /home/alumno/lab-baseline.txt
echo "mkfs.ext4: $(mkfs.ext4 -V 2>&1 | head -1)" >> /home/alumno/lab-baseline.txt
```

6. Revise el contenido final del baseline:

```bash
cat /home/alumno/lab-baseline.txt
```

**Verificación:**

```bash
## Confirmar que el baseline contiene la sección de este laboratorio
grep -c "Lab 02-00-02" /home/alumno/lab-baseline.txt
```

Debe devolver `1` o más.

```bash
## Confirmar que contiene los UUID
grep -c "UUID:" /home/alumno/lab-baseline.txt
```

Debe devolver al menos `2` (uno por cada disco de datos).

---

## Validación y Pruebas

Ejecute la siguiente secuencia completa de validaciones para confirmar que el laboratorio se completó correctamente. **Todos los comandos deben producir las salidas indicadas.**

### Prueba 1: Particiones GPT creadas correctamente

```bash
sudo fdisk -l /dev/sdb | grep "Disklabel type"
sudo fdisk -l /dev/sdc | grep "Disklabel type"
```

**Resultado esperado:** Ambos deben mostrar `Disklabel type: gpt`.

### Prueba 2: Sistemas de archivos ext4 con etiquetas correctas

```bash
sudo blkid /dev/sdb1 | grep 'TYPE="ext4"' | grep 'LABEL="datos1"'
sudo blkid /dev/sdc1 | grep 'TYPE="ext4"' | grep 'LABEL="datos2"'
```

**Resultado esperado:** Cada comando devuelve exactamente una línea con el UUID, LABEL y TYPE correspondientes.

### Prueba 3: Montaje activo y funcional

```bash
mountpoint -q /mnt/datos1 && echo "datos1: MONTADO" || echo "datos1: NO MONTADO"
mountpoint -q /mnt/datos2 && echo "datos2: MONTADO" || echo "datos2: NO MONTADO"
```

**Resultado esperado:**

```
datos1: MONTADO
datos2: MONTADO
```

### Prueba 4: Persistencia en /etc/fstab con UUID (no con /dev/sdX)

```bash
## Verificar que las entradas usan UUID
grep "/mnt/datos1" /etc/fstab | grep -q "UUID=" && echo "datos1: USA UUID ✓" || echo "datos1: NO USA UUID ✗"
grep "/mnt/datos2" /etc/fstab | grep -q "UUID=" && echo "datos2: USA UUID ✓" || echo "datos2: NO USA UUID ✗"

## Verificar que NO se usaron nombres de dispositivo
grep "/mnt/datos" /etc/fstab | grep -q "/dev/sd" && echo "ERROR: Se usó /dev/sdX en fstab ✗" || echo "OK: No se usó /dev/sdX ✓"
```

**Resultado esperado:**

```
datos1: USA UUID ✓
datos2: USA UUID ✓
OK: No se usó /dev/sdX ✓
```

### Prueba 5: Escritura y lectura funcional en ambos discos

```bash
echo "validacion-final-$(date +%s)" | sudo tee /mnt/datos1/validacion.txt > /dev/null
echo "validacion-final-$(date +%s)" | sudo tee /mnt/datos2/validacion.txt > /dev/null
cat /mnt/datos1/validacion.txt
cat /mnt/datos2/validacion.txt
```

**Resultado esperado:** Ambos archivos se crean y se leen sin errores.

### Prueba 6: Integridad del entorno previo (swap del lab 02-00-01)

```bash
swapon --show | grep -q "/swapfile" && echo "Swap: ACTIVO ✓" || echo "Swap: INACTIVO ✗"
cat /proc/sys/vm/swappiness
```

**Resultado esperado:**

```
Swap: ACTIVO ✓
10
```

### Prueba 7: Caso adverso — verificar que fstab no contiene entradas inválidas

Esta prueba valida que no se introdujeron errores accidentales en `/etc/fstab` que pudieran comprometer el arranque del sistema:

```bash
## Verificar que findmnt puede parsear todo fstab sin errores
sudo findmnt --verify --tab-file /etc/fstab 2>&1
```

**Resultado esperado:** El comando no debe reportar errores de tipo `CRITICAL` ni `ERROR`. Advertencias menores (`WARNING`) sobre el campo `pass` son aceptables.

```bash
## Verificar que no hay UUIDs duplicados o vacíos en las líneas de datos
grep "/mnt/datos" /etc/fstab | awk '{print $1}' | sort | uniq -d
```

**Resultado esperado:** Sin salida (no hay duplicados).

### Prueba 8: Baseline acumulativo completo

```bash
## Verificar que el baseline contiene las secciones esperadas de este lab
grep -q "Lab 02-00-02" /home/alumno/lab-baseline.txt && echo "Sección lab: ✓" || echo "Sección lab: ✗"
grep -q "UUID de particiones de datos" /home/alumno/lab-baseline.txt && echo "UUIDs registrados: ✓" || echo "UUIDs registrados: ✗"
grep -q "/etc/fstab" /home/alumno/lab-baseline.txt && echo "fstab documentado: ✓" || echo "fstab documentado: ✗"
```

**Resultado esperado:** Las tres líneas muestran `✓`.

---

## Solución de Problemas

### Problema 1: `mount -a` falla con "wrong fs type, bad option, bad superblock"

**Síntomas:** Al ejecutar `sudo mount -a` en el Paso 7, se obtiene un error similar a:

```
mount: /mnt/datos1: wrong fs type, bad option, bad superblock on /dev/sdb1,
       missing codepage or helper program, or other error.
```

**Causa:** El UUID registrado en `/etc/fstab` no coincide con el UUID real de la partición. Esto puede ocurrir si se reformateó la partición después de copiar el UUID, o si se introdujo un error tipográfico al escribir la entrada manualmente en lugar de usar las variables de entorno.

**Solución:**

1. Compare el UUID en fstab con el UUID real:

```bash
grep "/mnt/datos1" /etc/fstab
sudo blkid /dev/sdb1
```

2. Si difieren, edite `/etc/fstab` con el UUID correcto:

```bash
## Obtener el UUID correcto
UUID_CORRECTO=$(sudo blkid -s UUID -o value /dev/sdb1)
echo "UUID correcto: $UUID_CORRECTO"

## Restaurar desde backup y rehacer
sudo cp /etc/fstab.bak.* /etc/fstab
## Repetir el Paso 6 con los UUID correctos
```

3. Vuelva a ejecutar `sudo mount -a` para validar.

---

### Problema 2: Tras reiniciar, los discos no se montan y la VM arranca en modo de emergencia

**Síntomas:** Después del reinicio en el Paso 8, la VM no arranca normalmente. En su lugar, muestra un mensaje de `systemd` indicando que entró en `emergency mode` porque no pudo montar un sistema de archivos definido en `/etc/fstab`.

**Causa:** Hay un error sintáctico en `/etc/fstab` (por ejemplo, un campo faltante, un UUID mal formado, o un punto de montaje que no existe como directorio). El sistema no puede completar el montaje y entra en modo de emergencia para evitar daños.

**Solución:**

1. En el prompt de emergencia, introduzca la contraseña de root (o de `alumno` si tiene permisos sudo) cuando se le solicite.

2. Remonte el sistema de archivos raíz en modo lectura-escritura:

```bash
mount -o remount,rw /
```

3. Restaure la copia de respaldo de fstab:

```bash
## Listar las copias de respaldo disponibles
ls /etc/fstab.bak.*

## Restaurar la más reciente
cp /etc/fstab.bak.* /etc/fstab
```

4. Si no tiene copia de respaldo, edite `/etc/fstab` manualmente y comente (con `#`) las líneas problemáticas:

```bash
nano /etc/fstab
## Comente las líneas de /mnt/datos1 y /mnt/datos2 agregando # al inicio
```

5. Reinicie normalmente:

```bash
reboot
```

6. Una vez que la VM arranque correctamente, repita los Pasos 6 y 7 con cuidado, verificando cada UUID antes de escribirlo en `/etc/fstab`.

> **Lección aprendida:** Siempre cree una copia de respaldo de `/etc/fstab` antes de editarlo, y siempre ejecute `sudo mount -a` para validar la sintaxis antes de reiniciar. Estas dos prácticas previenen la mayoría de los problemas de arranque relacionados con fstab.

---

## Limpieza

Este laboratorio **no requiere limpieza**. Los discos adicionales, particiones, sistemas de archivos y entradas en `/etc/fstab` configurados aquí son parte de la infraestructura acumulativa del curso y serán referenciados como contexto en el Módulo 3.

**Elementos que deben permanecer configurados:**

| Elemento | Estado requerido |
|---|---|
| `/dev/sdb1` | Partición GPT, ext4, etiqueta `datos1` |
| `/dev/sdc1` | Partición GPT, ext4, etiqueta `datos2` |
| `/mnt/datos1` | Directorio existente, montado automáticamente |
| `/mnt/datos2` | Directorio existente, montado automáticamente |
| `/etc/fstab` | Contiene entradas con UUID para ambos discos |
| `/etc/fstab.bak.*` | Copia de respaldo conservada |
| `/home/alumno/lab-baseline.txt` | Actualizado con configuración de discos |

Si en algún momento futuro necesita revertir la configuración de este laboratorio (por ejemplo, para reasignar los discos), los pasos serían:

```bash
## Solo si se necesita revertir (NO ejecutar ahora)
sudo umount /mnt/datos1 /mnt/datos2
sudo sed -i '/\/mnt\/datos/d' /etc/fstab
sudo rm -rf /mnt/datos1 /mnt/datos2
```

---

## Resumen

En este laboratorio se aplicaron los conceptos de persistencia de configuración aprendidos en la lección 2.1 al escenario concreto de preparación de discos adicionales:

| Concepto de la lección 2.1 | Aplicación en este laboratorio |
|---|---|
| Estado runtime vs. estado persistente | Montaje manual (temporal) vs. entrada en `/etc/fstab` (persistente) |
| Uso de UUID en `/etc/fstab` | Todas las entradas usan `UUID=` en lugar de `/dev/sdX1` |
| Verificación de persistencia comparando runtime con disco | `mount \| grep datos` comparado con `grep datos /etc/fstab` |
| Copia de respaldo antes de modificar archivos en `/etc` | `cp /etc/fstab /etc/fstab.bak.*` |
| Validación sin reinicio + confirmación con reinicio | `mount -a` seguido de `sudo reboot` |

**Puntos clave:**

- `lsblk` y `blkid` son las herramientas fundamentales para identificar discos de forma segura antes de cualquier operación destructiva.
- `fdisk` con tabla GPT es el estándar moderno para particionado de discos.
- `mkfs.ext4` crea el sistema de archivos y genera automáticamente un UUID único.
- `/etc/fstab` con UUID garantiza montaje persistente e independiente del orden de detección de hardware.
- `mount -a` permite validar la sintaxis de fstab sin reiniciar, evitando problemas de arranque.
- Siempre crear respaldo de `/etc/fstab` antes de editarlo.

**Estado del entorno al finalizar este laboratorio:**

El archivo `/home/alumno/lab-baseline.txt` ahora contiene: versión del SO y kernel (lab 01-00-01), lista de paquetes actualizados (lab 01-00-02), configuración de swap con UUID y swappiness (lab 02-00-01), y configuración de discos adicionales con UUID, puntos de montaje y sistemas de archivos (este laboratorio). En el Módulo 3 se agregará la configuración de red.

### Recursos adicionales

| Recurso | Enlace |
|---|---|
| Página de manual de `fstab(5)` | [man7.org/linux/man-pages/man5/fstab.5.html](https://man7.org/linux/man-pages/man5/fstab.5.html) |
| Página de manual de `fdisk(8)` | [man7.org/linux/man-pages/man8/fdisk.8.html](https://man7.org/linux/man-pages/man8/fdisk.8.html) |
| Arch Wiki: fstab (referencia completa) | [wiki.archlinux.org/title/Fstab](https://wiki.archlinux.org/title/Fstab) |
| Ubuntu: gestión de almacenamiento | [help.ubuntu.com/community/InstallingANewHardDrive](https://help.ubuntu.com/community/InstallingANewHardDrive) |
| Documentación de util-linux 2.39.3 | [github.com/util-linux/util-linux/releases/tag/v2.39.3](https://github.com/util-linux/util-linux/releases/tag/v2.39.3) |
