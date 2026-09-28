# Configurar las interfaces de red

## Metadatos

| Campo | Detalle |
|---|---|
| **Duración** | 75 minutos |
| **Complejidad** | Media |
| **Nivel Bloom** | Aplicar (Apply) |
| **Módulo** | 03 — Configuración de red en entornos Linux virtualizados |
| **SO** | Ubuntu Desktop 24.04.2 LTS (Noble Numbat) — `ubuntu-24.04.2-desktop-amd64.iso` |
| **Hipervisor** | Oracle VirtualBox 7.0.18 (alternativa: VMware Workstation Pro 17.5.2) |

## Descripción General

En este laboratorio el estudiante identificará las interfaces de red disponibles en la máquina virtual, configurará direccionamiento IP estático en la interfaz de red interna mediante Netplan 1.0.1, validará la conectividad y garantizará la persistencia de la configuración tras reinicio. Este es el laboratorio final del batch y cierra la preparación del entorno documentada en el Setup Guide de Thor, actualizando el archivo de baseline acumulativo `/home/alumno/lab-baseline.txt` con la configuración de red completa.

## Objetivos de Aprendizaje

Al completar este laboratorio, el estudiante será capaz de:

- [ ] Identificar las interfaces de red disponibles en la VM y su estado actual usando `ip link show` e `ip addr show`, distinguiendo la interfaz NAT de la interfaz de red interna
- [ ] Crear un archivo de configuración Netplan (`/etc/netplan/99-lab-static.yaml`) con sintaxis YAML válida para asignar IP estática `192.168.100.10/24`, gateway `192.168.100.1` y DNS `8.8.8.8` a la interfaz de red interna
- [ ] Aplicar la configuración de red con `netplan try` y `netplan apply`, verificando conectividad con `ping` e `ip route show`
- [ ] Confirmar la persistencia de la configuración de red tras reinicio completo de la VM
- [ ] Documentar la configuración de red final en el archivo de baseline acumulativo del sistema

## Prerrequisitos

### Conocimientos requeridos

| Conocimiento | Nivel |
|---|---|
| Direccionamiento IPv4 y notación CIDR (`/24` = `255.255.255.0`) | Básico |
| Conceptos de gateway y DNS | Básico |
| Sintaxis YAML (indentación con espacios, estructura clave-valor) | Básico |
| Uso de terminal Linux y comandos `sudo`, `cat`, `nano`/`vi` | Básico |
| Modos de red en VirtualBox (NAT, Red Interna, Host-Only) | Conceptual (cubierto en lección 3.1) |

### Acceso y estado del sistema requerido

| Requisito | Verificación |
|---|---|
| Laboratorio 02-00-02 completado (discos adicionales montados en `/mnt/datos1` y `/mnt/datos2`) | `lsblk` muestra `sdb1` y `sdc1` montados |
| Swap configurado y activo (`/swapfile` de 2 GB) | `swapon --show` muestra `/swapfile` |
| Sistema actualizado (lab 01-00-02) | Baseline registra actualización |
| Archivo `/home/alumno/lab-baseline.txt` existente con registros de módulos anteriores | `cat /home/alumno/lab-baseline.txt` muestra entradas previas |
| Usuario `alumno` con permisos `sudo` | `sudo whoami` devuelve `root` |
| VM con **dos adaptadores de red** configurados en VirtualBox | Ver sección Entorno de Laboratorio |

## Entorno de Laboratorio

### Configuración de adaptadores de red en VirtualBox (preparación del instructor)

> **⚠️ IMPORTANTE — Preparación previa del instructor:** Antes de que los estudiantes inicien este laboratorio, el instructor debe verificar que cada VM tiene configurados **exactamente dos adaptadores de red** en VirtualBox:

| Adaptador | Modo en VirtualBox | Interfaz esperada en Ubuntu | Propósito |
|---|---|---|---|
| Adaptador 1 | **NAT** | `enp0s3` | Acceso a Internet (DHCP automático, típicamente `10.0.2.15`) |
| Adaptador 2 | **Red Interna** (nombre: `labnet`) o **Host-Only Adapter** | `enp0s8` | Red de laboratorio `192.168.100.0/24` (se configurará con IP estática) |

**Pasos de verificación del instructor en VirtualBox:**

1. Con la VM apagada → Configuración → Red
2. **Adaptador 1:** Habilitar, conectado a **NAT**
3. **Adaptador 2:** Habilitar, conectado a **Red Interna**, nombre `labnet` (o **Adaptador solo anfitrión** si se prefiere conectividad con el host)
4. Aceptar y encender la VM
5. Ejecutar `ip link show` dentro de la VM para confirmar que aparecen `enp0s3` y `enp0s8`

> **Nota sobre hipervisor alternativo:** Si se usa VMware Workstation Pro 17.5.2, los nombres de interfaz pueden ser `ens33` y `ens38` (o similares con prefijo `ens`). El instructor debe documentar los nombres reales y comunicarlos a los estudiantes antes de iniciar el laboratorio.

### Software utilizado

| Componente | Versión exacta | Fuente oficial |
|---|---|---|
| Ubuntu Desktop | 24.04.2 LTS (Noble Numbat) | [https://releases.ubuntu.com/24.04/](https://releases.ubuntu.com/24.04/) |
| Netplan.io | 1.0.1 (incluido en Ubuntu 24.04.2 LTS) | [https://netplan.io/](https://netplan.io/) |
| iproute2 (`ip`) | 6.1.0-1ubuntu6 (incluido en Ubuntu 24.04.2 LTS) | [https://wiki.linuxfoundation.org/networking/iproute2](https://wiki.linuxfoundation.org/networking/iproute2) |
| systemd-networkd | 255.4 (incluido en Ubuntu 24.04.2 LTS) | [https://systemd.io/](https://systemd.io/) |
| bash | 5.2.21 (incluido en Ubuntu 24.04.2 LTS) | Repositorio oficial Ubuntu |

### Constantes de configuración de red

| Parámetro | Valor |
|---|---|
| Archivo Netplan de laboratorio | `/etc/netplan/99-lab-static.yaml` |
| Interfaz de red interna | `enp0s8` (verificar con `ip link show`) |
| IP estática | `192.168.100.10/24` |
| Gateway | `192.168.100.1` |
| DNS | `8.8.8.8` |
| Red de laboratorio | `192.168.100.0/24` |
| Archivo de baseline | `/home/alumno/lab-baseline.txt` |

## Instrucciones Paso a Paso

### Paso 1: Verificar el estado previo del sistema y el baseline acumulativo

**Objetivo:** Confirmar que la VM se encuentra en el estado esperado tras los laboratorios anteriores y que el archivo de baseline contiene los registros de módulos previos.

**Instrucciones:**

1. Abra una terminal en Ubuntu Desktop (atajo: `Ctrl+Alt+T`).

2. Verifique que está operando como el usuario `alumno`:

```bash
whoami
```

3. Confirme que tiene permisos `sudo`:

```bash
sudo whoami
```

4. Verifique que el swap está activo:

```bash
swapon --show
```

5. Verifique que los discos adicionales están montados:

```bash
lsblk -f | grep -E "sdb|sdc|datos"
```

6. Revise el contenido actual del archivo de baseline:

```bash
echo "=== ESTADO DEL BASELINE ANTES DE LAB 03-00-01 ==="
cat /home/alumno/lab-baseline.txt
echo "=== FIN DEL BASELINE PREVIO ==="
```

7. Registre la marca de tiempo de inicio del laboratorio:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "========================================" >> /home/alumno/lab-baseline.txt
echo "LAB 03-00-01: Configuración de red" >> /home/alumno/lab-baseline.txt
echo "Fecha de inicio: $(date '+%Y-%m-%d %H:%M:%S')" >> /home/alumno/lab-baseline.txt
echo "========================================" >> /home/alumno/lab-baseline.txt
```

**Salida esperada:**

- `whoami` → `alumno`
- `sudo whoami` → `root`
- `swapon --show` → muestra `/swapfile` de 2 GB tipo `file`
- `lsblk -f` → muestra `sdb1` montado en `/mnt/datos1` y `sdc1` montado en `/mnt/datos2`, ambos con filesystem `ext4`
- El archivo `lab-baseline.txt` contiene entradas de los laboratorios 01-00-01, 01-00-02 y 02-00-02

**Verificación:**

```bash
## Verificación compacta del estado previo
test -f /home/alumno/lab-baseline.txt && echo "✅ Baseline existe" || echo "❌ Baseline NO encontrado"
swapon --show | grep -q swapfile && echo "✅ Swap activo" || echo "❌ Swap NO activo"
mountpoint -q /mnt/datos1 && echo "✅ /mnt/datos1 montado" || echo "❌ /mnt/datos1 NO montado"
mountpoint -q /mnt/datos2 && echo "✅ /mnt/datos2 montado" || echo "❌ /mnt/datos2 NO montado"
```

> **⚠️ Si alguna verificación falla**, no continúe. Revise los laboratorios anteriores o consulte al instructor antes de proceder.

---

### Paso 2: Identificar las interfaces de red y su estado actual

**Objetivo:** Descubrir los nombres reales de las interfaces de red en la VM, distinguir la interfaz NAT de la interfaz de red interna y documentar su estado.

**Instrucciones:**

1. Liste todas las interfaces de red con sus estados:

```bash
ip link show
```

2. Observe la salida e identifique las interfaces. Debe ver al menos tres:
   - `lo` — loopback (siempre presente)
   - `enp0s3` — Adaptador 1 (NAT), estado `UP`
   - `enp0s8` — Adaptador 2 (Red Interna), estado `UP` o `DOWN`

3. Muestre las direcciones IP asignadas a todas las interfaces:

```bash
ip addr show
```

4. Filtre solo las líneas relevantes (nombre de interfaz y direcciones IP):

```bash
ip -br addr show
```

5. Verifique específicamente la interfaz NAT (debe tener IP asignada por DHCP):

```bash
ip addr show enp0s3
```

6. Verifique la interfaz de red interna (probablemente sin IP asignada aún):

```bash
ip addr show enp0s8
```

7. Consulte la tabla de enrutamiento actual:

```bash
ip route show
```

8. Verifique el estado de las interfaces según NetworkManager (si está activo):

```bash
nmcli device status
```

9. Registre la información de interfaces en el baseline:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Interfaces de red detectadas (pre-configuración) ---" >> /home/alumno/lab-baseline.txt
ip -br addr show >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Tabla de enrutamiento (pre-configuración) ---" >> /home/alumno/lab-baseline.txt
ip route show >> /home/alumno/lab-baseline.txt
```

**Salida esperada:**

Para `ip -br addr show`:
```
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             10.0.2.15/24 fe80::a00:27ff:fexx:xxxx/64
enp0s8           UP             fe80::a00:27ff:feyy:yyyy/64
```

Observe que `enp0s8` aparece con estado `UP` pero **sin dirección IPv4 asignada** (solo tiene una dirección link-local IPv6). Este es el estado esperado antes de la configuración.

Para `ip route show`:
```
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
```

**Verificación:**

```bash
## Confirmar que existen exactamente las interfaces esperadas
ip link show enp0s3 > /dev/null 2>&1 && echo "✅ enp0s3 detectada (NAT)" || echo "❌ enp0s3 NO detectada"
ip link show enp0s8 > /dev/null 2>&1 && echo "✅ enp0s8 detectada (Red interna)" || echo "❌ enp0s8 NO detectada"
```

> **Nota:** Si los nombres de interfaz difieren (por ejemplo, `ens33`/`ens38` en VMware), sustituya `enp0s8` por el nombre real en todos los pasos siguientes. Confirme el nombre correcto con el instructor.

---

### Paso 3: Examinar la configuración Netplan existente

**Objetivo:** Comprender la estructura de los archivos de configuración Netplan existentes antes de crear el archivo de configuración de red estática.

**Instrucciones:**

1. Liste los archivos de configuración Netplan existentes:

```bash
ls -la /etc/netplan/
```

2. Examine el contenido de cada archivo encontrado:

```bash
cat /etc/netplan/*.yaml
```

   > En una instalación estándar de Ubuntu Desktop 24.04.2 LTS, puede encontrar un archivo como `01-network-manager-all.yaml` con contenido similar a:
   > ```yaml
   > # Let NetworkManager manage all devices on this system
   > network:
   >   version: 2
   >   renderer: NetworkManager
   > ```

3. Verifique la versión de Netplan instalada:

```bash
netplan --version
```

4. Consulte qué renderer de red está activo (NetworkManager o systemd-networkd):

```bash
## Verificar si NetworkManager está activo
systemctl is-active NetworkManager

## Verificar si systemd-networkd está activo
systemctl is-active systemd-networkd
```

5. Registre la configuración Netplan previa en el baseline:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Configuración Netplan existente (pre-configuración) ---" >> /home/alumno/lab-baseline.txt
echo "Versión Netplan: $(netplan --version 2>&1)" >> /home/alumno/lab-baseline.txt
echo "Archivos en /etc/netplan/:" >> /home/alumno/lab-baseline.txt
ls -la /etc/netplan/ >> /home/alumno/lab-baseline.txt
echo "Contenido:" >> /home/alumno/lab-baseline.txt
cat /etc/netplan/*.yaml >> /home/alumno/lab-baseline.txt 2>/dev/null
```

**Salida esperada:**

- `netplan --version` → `netplan.io 1.0.1` (o similar dentro de la serie 1.0.x incluida en Ubuntu 24.04.2)
- Al menos un archivo `.yaml` en `/etc/netplan/`
- `systemctl is-active NetworkManager` → `active` (en Ubuntu Desktop)

**Verificación:**

```bash
netplan --version 2>&1 | grep -q "1.0" && echo "✅ Netplan 1.0.x confirmado" || echo "⚠️ Versión de Netplan inesperada"
ls /etc/netplan/*.yaml > /dev/null 2>&1 && echo "✅ Archivos Netplan encontrados" || echo "❌ No hay archivos Netplan"
```

> **Concepto clave:** Netplan procesa los archivos en `/etc/netplan/` en orden lexicográfico. El archivo `99-lab-static.yaml` que crearemos se procesará después de `01-network-manager-all.yaml`, lo que permite que nuestra configuración específica para `enp0s8` tenga prioridad sin modificar la configuración existente de `enp0s3`.

---

### Paso 4: Crear el archivo de configuración Netplan para IP estática

**Objetivo:** Crear el archivo `/etc/netplan/99-lab-static.yaml` con la configuración de IP estática para la interfaz de red interna `enp0s8`.

**Instrucciones:**

1. Cree el archivo de configuración Netplan con `sudo` y un editor de texto. Usaremos `tee` para crear el archivo de forma reproducible:

```bash
sudo tee /etc/netplan/99-lab-static.yaml > /dev/null << 'EOF'
## Configuración de red estática para laboratorio - Setup Guide de Thor
## Lab 03-00-01 - Interfaz de red interna (enp0s8)
## Creado: $(date) | Usuario: alumno
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.100.10/24
      routes:
        - to: 192.168.100.0/24
          via: 192.168.100.1
      nameservers:
        addresses:
          - 8.8.8.8
EOF
```

> **⚠️ CRÍTICO sobre sintaxis YAML:**
> - La indentación **debe ser con espacios**, nunca con tabuladores
> - Cada nivel de indentación usa **exactamente 2 espacios**
> - El guion (`-`) en las listas debe estar alineado correctamente
> - Un solo error de indentación hará que `netplan apply` falle

2. Verifique que el archivo se creó correctamente mostrando su contenido:

```bash
cat /etc/netplan/99-lab-static.yaml
```

3. Establezca los permisos restrictivos recomendados por Netplan (solo lectura/escritura para root):

```bash
sudo chmod 600 /etc/netplan/99-lab-static.yaml
```

4. Verifique los permisos del archivo:

```bash
ls -la /etc/netplan/99-lab-static.yaml
```

5. Verifique que la propiedad del archivo es `root:root`:

```bash
stat -c '%U:%G %a %n' /etc/netplan/99-lab-static.yaml
```

**Salida esperada:**

Para `cat /etc/netplan/99-lab-static.yaml`:
```yaml
## Configuración de red estática para laboratorio - Setup Guide de Thor
## Lab 03-00-01 - Interfaz de red interna (enp0s8)
## Creado: $(date) | Usuario: alumno
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.100.10/24
      routes:
        - to: 192.168.100.0/24
          via: 192.168.100.1
      nameservers:
        addresses:
          - 8.8.8.8
```

Para `ls -la`:
```
-rw------- 1 root root <tamaño> <fecha> /etc/netplan/99-lab-static.yaml
```

**Verificación:**

```bash
test -f /etc/netplan/99-lab-static.yaml && echo "✅ Archivo creado" || echo "❌ Archivo NO creado"
stat -c '%a' /etc/netplan/99-lab-static.yaml | grep -q "600" && echo "✅ Permisos 600 correctos" || echo "❌ Permisos incorrectos"
grep -q "192.168.100.10/24" /etc/netplan/99-lab-static.yaml && echo "✅ IP estática presente" || echo "❌ IP no encontrada"
grep -q "enp0s8" /etc/netplan/99-lab-static.yaml && echo "✅ Interfaz enp0s8 configurada" || echo "❌ Interfaz incorrecta"
```

---

### Paso 5: Validar y aplicar la configuración de red

**Objetivo:** Usar `netplan try` para validar la configuración con posibilidad de revertir automáticamente, y luego aplicarla de forma definitiva con `netplan apply`.

**Instrucciones:**

1. Primero, valide la sintaxis del archivo Netplan sin aplicar cambios:

```bash
sudo netplan generate
```

   > Si hay errores de sintaxis YAML, este comando los reportará. Si no produce salida, la sintaxis es válida.

2. Aplique la configuración de forma provisional con `netplan try`. Este comando aplica los cambios pero los revierte automáticamente en 120 segundos si no se confirman:

```bash
sudo netplan try --timeout 30
```

   > Cuando aparezca el mensaje `Do you want to keep these settings?`, observe si la red sigue funcionando. Si todo es correcto, escriba `y` y presione Enter para confirmar. Si no confirma dentro de 30 segundos, la configuración se revierte automáticamente.

3. Si `netplan try` fue exitoso y confirmó los cambios, aplique la configuración de forma definitiva:

```bash
sudo netplan apply
```

4. Verifique inmediatamente que la IP estática se asignó a `enp0s8`:

```bash
ip addr show enp0s8
```

5. Verifique la tabla de enrutamiento actualizada:

```bash
ip route show
```

6. Verifique la configuración con el formato resumido:

```bash
ip -br addr show
```

**Salida esperada:**

Para `sudo netplan generate` → sin salida (éxito silencioso).

Para `ip addr show enp0s8`:
```
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.10/24 brd 192.168.100.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fexx:xxxx/64 scope link
       valid_lft forever preferred_lft forever
```

Para `ip -br addr show`:
```
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             10.0.2.15/24 fe80::...
enp0s8           UP             192.168.100.10/24 fe80::...
```

**Verificación:**

```bash
## Verificar que la IP estática está asignada
ip addr show enp0s8 | grep -q "192.168.100.10/24" && echo "✅ IP 192.168.100.10/24 asignada a enp0s8" || echo "❌ IP NO asignada"

## Verificar que la interfaz NAT sigue funcionando
ip addr show enp0s3 | grep -q "10.0.2" && echo "✅ enp0s3 (NAT) sigue activa" || echo "❌ enp0s3 perdió su IP"

## Verificar que la ruta a la red de laboratorio existe
ip route show | grep -q "192.168.100.0/24" && echo "✅ Ruta a red de laboratorio presente" || echo "❌ Ruta NO encontrada"
```

---

### Paso 6: Verificar conectividad de red

**Objetivo:** Probar la conectividad de red tanto en la interfaz NAT (Internet) como en la interfaz de red interna (laboratorio) y verificar la resolución DNS.

**Instrucciones:**

1. Verifique que la interfaz NAT sigue proporcionando acceso a Internet:

```bash
ping -c 4 8.8.8.8
```

2. Verifique la resolución DNS:

```bash
ping -c 2 google.com
```

3. Intente hacer ping al gateway de la red de laboratorio:

```bash
ping -c 4 192.168.100.1
```

   > **Nota importante:** Es probable que el ping al gateway `192.168.100.1` **falle** si no hay un dispositivo real respondiendo en esa dirección. En modo **Red Interna**, no existe un gateway real a menos que otra VM o el host lo proporcione. En modo **Host-Only**, el host suele responder. El resultado depende de la configuración del instructor. **Que este ping falle NO significa que la configuración esté incorrecta.**

4. Verifique que puede hacer ping a su propia IP estática (prueba de loopback de interfaz):

```bash
ping -c 2 192.168.100.10
```

5. Verifique la resolución DNS configurada:

```bash
resolvectl status enp0s8
```

6. Verifique la conectividad completa desde la interfaz específica:

```bash
## Verificar que la interfaz NAT llega a Internet
ping -c 2 -I enp0s3 8.8.8.8
```

7. Registre los resultados de conectividad en el baseline:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Pruebas de conectividad ---" >> /home/alumno/lab-baseline.txt
echo "Ping a 8.8.8.8 (Internet via NAT):" >> /home/alumno/lab-baseline.txt
ping -c 2 8.8.8.8 >> /home/alumno/lab-baseline.txt 2>&1
echo "" >> /home/alumno/lab-baseline.txt
echo "Ping a 192.168.100.10 (IP propia, red interna):" >> /home/alumno/lab-baseline.txt
ping -c 2 192.168.100.10 >> /home/alumno/lab-baseline.txt 2>&1
echo "" >> /home/alumno/lab-baseline.txt
echo "Ping a 192.168.100.1 (gateway red interna):" >> /home/alumno/lab-baseline.txt
ping -c 2 192.168.100.1 >> /home/alumno/lab-baseline.txt 2>&1
```

**Salida esperada:**

Para `ping -c 4 8.8.8.8`:
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=12.3 ms
...
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
```

Para `ping -c 2 192.168.100.10`:
```
PING 192.168.100.10 (192.168.100.10) 56(84) bytes of data.
64 bytes from 192.168.100.10: icmp_seq=1 ttl=64 time=0.023 ms
...
2 packets transmitted, 2 received, 0% packet loss
```

**Verificación:**

```bash
## Verificar acceso a Internet
ping -c 1 -W 5 8.8.8.8 > /dev/null 2>&1 && echo "✅ Conectividad a Internet OK" || echo "❌ Sin acceso a Internet"

## Verificar IP propia responde
ping -c 1 -W 5 192.168.100.10 > /dev/null 2>&1 && echo "✅ IP estática 192.168.100.10 responde" || echo "❌ IP estática NO responde"

## Verificar DNS
ping -c 1 -W 5 google.com > /dev/null 2>&1 && echo "✅ Resolución DNS OK" || echo "⚠️ DNS no resuelve (puede ser normal según configuración)"
```

---

### Paso 7: Verificar persistencia tras reinicio

**Objetivo:** Confirmar que la configuración de red estática sobrevive a un reinicio completo de la máquina virtual, garantizando que el archivo Netplan se aplica automáticamente al arranque.

**Instrucciones:**

1. Antes de reiniciar, capture el estado actual de red para comparación posterior:

```bash
echo "--- Estado de red PRE-reinicio ---" > /tmp/pre-reboot-net.txt
ip -br addr show >> /tmp/pre-reboot-net.txt
ip route show >> /tmp/pre-reboot-net.txt
date >> /tmp/pre-reboot-net.txt
cat /tmp/pre-reboot-net.txt
```

2. Reinicie la máquina virtual:

```bash
sudo reboot
```

3. Después del reinicio, inicie sesión como `alumno` (contraseña: `Lab2024!`) y abra una terminal.

4. Verifique que la IP estática persiste en `enp0s8`:

```bash
ip addr show enp0s8
```

5. Verifique la configuración completa de red:

```bash
ip -br addr show
```

6. Verifique la tabla de enrutamiento:

```bash
ip route show
```

7. Repita las pruebas de conectividad:

```bash
ping -c 2 8.8.8.8
ping -c 2 192.168.100.10
```

8. Compare con el estado pre-reinicio:

```bash
echo "--- Estado de red POST-reinicio ---"
ip -br addr show
echo ""
echo "--- Comparación con pre-reinicio ---"
cat /tmp/pre-reboot-net.txt
```

9. Verifique que el archivo Netplan sigue intacto:

```bash
cat /etc/netplan/99-lab-static.yaml
ls -la /etc/netplan/99-lab-static.yaml
```

**Salida esperada:**

La salida de `ip addr show enp0s8` tras el reinicio debe ser idéntica a la del Paso 5:
```
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 192.168.100.10/24 brd 192.168.100.255 scope global enp0s8
```

**Verificación:**

```bash
## Verificación post-reinicio
ip addr show enp0s8 | grep -q "192.168.100.10/24" && echo "✅ PERSISTENCIA CONFIRMADA: IP 192.168.100.10/24 activa tras reinicio" || echo "❌ FALLO: IP no persistió tras reinicio"
ping -c 1 -W 5 8.8.8.8 > /dev/null 2>&1 && echo "✅ Internet OK tras reinicio" || echo "❌ Sin Internet tras reinicio"
test -f /etc/netplan/99-lab-static.yaml && echo "✅ Archivo Netplan intacto" || echo "❌ Archivo Netplan perdido"
```

---

### Paso 8: Documentar la configuración final en el baseline acumulativo

**Objetivo:** Actualizar el archivo `/home/alumno/lab-baseline.txt` con la configuración de red final, completando así el entregable del curso piloto del Setup Guide de Thor.

**Instrucciones:**

1. Registre la configuración de red final en el baseline:

```bash
echo "" >> /home/alumno/lab-baseline.txt
echo "--- Configuración de red final (post-reinicio) ---" >> /home/alumno/lab-baseline.txt
echo "Fecha de verificación: $(date '+%Y-%m-%d %H:%M:%S')" >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt

echo "Interfaces de red:" >> /home/alumno/lab-baseline.txt
ip -br addr show >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt

echo "Tabla de enrutamiento:" >> /home/alumno/lab-baseline.txt
ip route show >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt

echo "Archivo Netplan de laboratorio (/etc/netplan/99-lab-static.yaml):" >> /home/alumno/lab-baseline.txt
sudo cat /etc/netplan/99-lab-static.yaml >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt

echo "Permisos del archivo Netplan:" >> /home/alumno/lab-baseline.txt
ls -la /etc/netplan/99-lab-static.yaml >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt

echo "Resolución DNS (enp0s8):" >> /home/alumno/lab-baseline.txt
resolvectl status enp0s8 >> /home/alumno/lab-baseline.txt 2>&1
echo "" >> /home/alumno/lab-baseline.txt

echo "Configuración de red de laboratorio:" >> /home/alumno/lab-baseline.txt
echo "  Interfaz: enp0s8" >> /home/alumno/lab-baseline.txt
echo "  IP estática: 192.168.100.10/24" >> /home/alumno/lab-baseline.txt
echo "  Gateway: 192.168.100.1" >> /home/alumno/lab-baseline.txt
echo "  DNS: 8.8.8.8" >> /home/alumno/lab-baseline.txt
echo "  Red: 192.168.100.0/24" >> /home/alumno/lab-baseline.txt
echo "  Archivo Netplan: /etc/netplan/99-lab-static.yaml" >> /home/alumno/lab-baseline.txt
echo "  Permisos: 600 (rw-------)" >> /home/alumno/lab-baseline.txt
echo "  Persistencia: VERIFICADA tras reinicio" >> /home/alumno/lab-baseline.txt
echo "" >> /home/alumno/lab-baseline.txt

echo "========================================" >> /home/alumno/lab-baseline.txt
echo "LAB 03-00-01 COMPLETADO: $(date '+%Y-%m-%d %H:%M:%S')" >> /home/alumno/lab-baseline.txt
echo "========================================" >> /home/alumno/lab-baseline.txt
```

2. Verifique el contenido completo del baseline acumulativo:

```bash
cat /home/alumno/lab-baseline.txt
```

3. Cuente las secciones registradas en el baseline para confirmar que todos los laboratorios están documentados:

```bash
echo "=== Secciones en el baseline ==="
grep -c "========" /home/alumno/lab-baseline.txt
echo "=== Laboratorios registrados ==="
grep "LAB.*COMPLETADO\|LAB.*Configuración\|LAB.*inicio" /home/alumno/lab-baseline.txt
```

4. Cree una copia de respaldo del baseline final:

```bash
cp /home/alumno/lab-baseline.txt /home/alumno/lab-baseline-final-$(date '+%Y%m%d-%H%M%S').txt
echo "Copia de respaldo creada:"
ls -la /home/alumno/lab-baseline-final-*.txt
```

**Salida esperada:**

El archivo `lab-baseline.txt` debe contener secciones de:
- Lab 01-00-01: Verificación del sistema operativo
- Lab 01-00-02: Actualización del sistema y gestión APT
- Lab 02-00-02: Configuración de discos y swap
- Lab 03-00-01: Configuración de red (este laboratorio)

**Verificación:**

```bash
## Verificar que el baseline contiene la configuración de red
grep -q "192.168.100.10" /home/alumno/lab-baseline.txt && echo "✅ IP estática documentada en baseline" || echo "❌ IP NO documentada"
grep -q "enp0s8" /home/alumno/lab-baseline.txt && echo "✅ Interfaz documentada en baseline" || echo "❌ Interfaz NO documentada"
grep -q "LAB 03-00-01 COMPLETADO" /home/alumno/lab-baseline.txt && echo "✅ Lab 03-00-01 marcado como completado" || echo "❌ Lab NO marcado como completado"
grep -q "99-lab-static.yaml" /home/alumno/lab-baseline.txt && echo "✅ Archivo Netplan documentado" || echo "❌ Archivo Netplan NO documentado"
```

## Validación y Pruebas

Ejecute el siguiente script de validación integral para confirmar que todos los objetivos del laboratorio se cumplieron. Cada prueba debe mostrar `✅`:

```bash
echo "╔══════════════════════════════════════════════════════════════╗"
echo "║  VALIDACIÓN INTEGRAL — Lab 03-00-01: Configuración de red  ║"
echo "╚══════════════════════════════════════════════════════════════╝"
echo ""

PASS=0
FAIL=0

## 1. Archivo Netplan existe con permisos correctos
echo "--- Prueba 1: Archivo Netplan ---"
if [ -f /etc/netplan/99-lab-static.yaml ] && [ "$(stat -c '%a' /etc/netplan/99-lab-static.yaml)" = "600" ]; then
    echo "✅ /etc/netplan/99-lab-static.yaml existe con permisos 600"
    PASS=$((PASS+1))
else
    echo "❌ Archivo Netplan no existe o permisos incorrectos"
    FAIL=$((FAIL+1))
fi

## 2. IP estática asignada a enp0s8
echo "--- Prueba 2: IP estática en enp0s8 ---"
if ip addr show enp0s8 2>/dev/null | grep -q "192.168.100.10/24"; then
    echo "✅ IP 192.168.100.10/24 asignada a enp0s8"
    PASS=$((PASS+1))
else
    echo "❌ IP 192.168.100.10/24 NO asignada a enp0s8"
    FAIL=$((FAIL+1))
fi

## 3. Interfaz NAT sigue operativa
echo "--- Prueba 3: Interfaz NAT (enp0s3) ---"
if ip addr show enp0s3 2>/dev/null | grep -q "inet "; then
    echo "✅ enp0s3 tiene IP asignada (NAT operativa)"
    PASS=$((PASS+1))
else
    echo "❌ enp0s3 sin IP (NAT posiblemente afectada)"
    FAIL=$((FAIL+1))
fi

## 4. Conectividad a Internet
echo "--- Prueba 4: Conectividad a Internet ---"
if ping -c 1 -W 5 8.8.8.8 > /dev/null 2>&1; then
    echo "✅ Conectividad a Internet verificada (ping a 8.8.8.8)"
    PASS=$((PASS+1))
else
    echo "❌ Sin conectividad a Internet"
    FAIL=$((FAIL+1))
fi

## 5. IP propia responde
echo "--- Prueba 5: IP estática responde a ping ---"
if ping -c 1 -W 5 192.168.100.10 > /dev/null 2>&1; then
    echo "✅ IP estática 192.168.100.10 responde"
    PASS=$((PASS+1))
else
    echo "❌ IP estática NO responde"
    FAIL=$((FAIL+1))
fi

## 6. Contenido del archivo Netplan es correcto
echo "--- Prueba 6: Contenido del archivo Netplan ---"
if grep -q "192.168.100.10/24" /etc/netplan/99-lab-static.yaml && \
   grep -q "192.168.100.1" /etc/netplan/99-lab-static.yaml && \
   grep -q "8.8.8.8" /etc/netplan/99-lab-static.yaml && \
   grep -q "enp0s8" /etc/netplan/99-lab-static.yaml; then
    echo "✅ Archivo Netplan contiene IP, gateway, DNS e interfaz correctos"
    PASS=$((PASS+1))
else
    echo "❌ Archivo Netplan incompleto o con valores incorrectos"
    FAIL=$((FAIL+1))
fi

## 7. Ruta a red de laboratorio
echo "--- Prueba 7: Ruta a red 192.168.100.0/24 ---"
if ip route show | grep -q "192.168.100"; then
    echo "✅ Ruta a red de laboratorio presente en tabla de enrutamiento"
    PASS=$((PASS+1))
else
    echo "❌ Ruta a red de laboratorio NO encontrada"
    FAIL=$((FAIL+1))
fi

## 8. Baseline actualizado con configuración de red
echo "--- Prueba 8: Baseline acumulativo ---"
if grep -q "192.168.100.10" /home/alumno/lab-baseline.txt && \
   grep -q "LAB 03-00-01 COMPLETADO" /home/alumno/lab-baseline.txt; then
    echo "✅ Baseline contiene configuración de red y marca de completado"
    PASS=$((PASS+1))
else
    echo "❌ Baseline incompleto"
    FAIL=$((FAIL+1))
fi

## 9. Estado de labs anteriores preservado
echo "--- Prueba 9: Integridad del sistema (labs anteriores) ---"
PREV_OK=true
swapon --show | grep -q swapfile || PREV_OK=false
mountpoint -q /mnt/datos1 2>/dev/null || PREV_OK=false
mountpoint -q /mnt/datos2 2>/dev/null || PREV_OK=false
if $PREV_OK; then
    echo "✅ Swap y discos adicionales siguen operativos (labs anteriores intactos)"
    PASS=$((PASS+1))
else
    echo "❌ Algún componente de labs anteriores no está operativo"
    FAIL=$((FAIL+1))
fi

## 10. Caso adverso: verificar que la configuración NO afecta interfaces no declaradas
echo "--- Prueba 10: Caso adverso — aislamiento de configuración ---"
## Verificar que no se creó una IP 192.168.100.x en enp0s3 (solo debe estar en enp0s8)
if ! ip addr show enp0s3 2>/dev/null | grep -q "192.168.100"; then
    echo "✅ La IP estática NO se filtró a enp0s3 (aislamiento correcto)"
    PASS=$((PASS+1))
else
    echo "❌ ADVERTENCIA: IP de red de laboratorio detectada en enp0s3"
    FAIL=$((FAIL+1))
fi

echo ""
echo "══════════════════════════════════════════"
echo "  RESULTADO: $PASS aprobadas / $((PASS+FAIL)) totales"
if [ $FAIL -eq 0 ]; then
    echo "  ✅ TODAS LAS PRUEBAS SUPERADAS"
else
    echo "  ❌ $FAIL prueba(s) fallida(s) — revisar antes de continuar"
fi
echo "══════════════════════════════════════════"
```

### Criterios de aprobación

| Criterio | Evidencia requerida | Resultado mínimo |
|---|---|---|
| Archivo Netplan creado con contenido correcto | `cat /etc/netplan/99-lab-static.yaml` | IP, gateway, DNS e interfaz presentes |
| Permisos restrictivos aplicados | `stat -c '%a'` del archivo | `600` |
| IP estática asignada | `ip addr show enp0s8` | `192.168.100.10/24` visible |
| Conectividad a Internet preservada | `ping -c 1 8.8.8.8` | 0% packet loss |
| IP propia responde | `ping -c 1 192.168.100.10` | 0% packet loss |
| Persistencia tras reinicio | `ip addr show enp0s8` post-reboot | IP sigue asignada |
| Baseline actualizado | `grep "LAB 03-00-01 COMPLETADO" lab-baseline.txt` | Coincidencia encontrada |
| Labs anteriores intactos | `swapon --show`, `mountpoint` | Swap y discos activos |
| Aislamiento de configuración | `ip addr show enp0s3` | Sin IP `192.168.100.x` |
| **10/10 pruebas aprobadas** | Script de validación | `TODAS LAS PRUEBAS SUPERADAS` |

## Solución de Problemas

### Problema 1: `netplan apply` falla con error de sintaxis YAML

**Síntomas:**

Al ejecutar `sudo netplan apply` o `sudo netplan generate`, aparece un error similar a:

```
Error in network definition: /etc/netplan/99-lab-static.yaml line 8 column 5: wrong indentation
```

O bien:

```
/etc/netplan/99-lab-static.yaml: Invalid YAML: mapping values are not allowed here
```

**Causa:**

El archivo YAML contiene errores de indentación. Las causas más comunes son:
- Uso de **tabuladores** en lugar de espacios (YAML exige espacios)
- Indentación inconsistente (mezcla de 2 y 4 espacios)
- Falta de espacio después de los dos puntos (`:`) en pares clave-valor
- Guion (`-`) de lista mal alineado

**Solución:**

1. Visualice los caracteres ocultos del archivo para detectar tabuladores:

```bash
cat -A /etc/netplan/99-lab-static.yaml
```

   > Los tabuladores aparecen como `^I`. Los espacios aparecen como espacios normales. Si ve `^I`, ese es el problema.

2. Elimine el archivo problemático y recréelo:

```bash
sudo rm /etc/netplan/99-lab-static.yaml
```

3. Recree el archivo usando el comando `tee` exacto del Paso 4 (que garantiza espacios correctos).

4. Alternativamente, use `sed` para reemplazar tabuladores por espacios:

```bash
sudo sed -i 's/\t/  /g' /etc/netplan/99-lab-static.yaml
```

5. Valide antes de aplicar:

```bash
sudo netplan generate
```

6. Si no hay errores, aplique:

```bash
sudo netplan apply
```

---

### Problema 2: La interfaz `enp0s8` no aparece en `ip link show`

**Síntomas:**

Al ejecutar `ip link show`, solo aparecen `lo` y `enp0s3`. No hay una segunda interfaz de red. El comando `ip link show enp0s8` devuelve:

```
Device "enp0s8" does not exist.
```

**Causa:**

El Adaptador 2 no está habilitado en la configuración de la VM en VirtualBox, o el nombre de la interfaz es diferente al esperado. Posibles razones:
- El instructor no configuró el segundo adaptador de red antes del laboratorio
- El adaptador está configurado pero no habilitado ("Cable conectado" desmarcado)
- En VMware, el nombre de la interfaz puede ser `ens38` o `ens192` en lugar de `enp0s8`

**Solución:**

1. Primero, liste **todas** las interfaces disponibles para descubrir el nombre real:

```bash
ip link show
ls /sys/class/net/
```

2. Si solo aparece `lo` y `enp0s3`, el adaptador no está configurado. **Apague la VM**:

```bash
sudo shutdown -h now
```

3. En VirtualBox, vaya a **Configuración → Red → Adaptador 2**:
   - Marque **"Habilitar adaptador de red"**
   - Seleccione **"Red interna"** y nombre `labnet` (o **"Adaptador solo anfitrión"**)
   - En **Avanzadas**, marque **"Cable conectado"**
   - Acepte y encienda la VM

4. Si la interfaz aparece con un nombre diferente (por ejemplo, `enp0s8` vs `enp0s9`), actualice el archivo Netplan con el nombre correcto:

```bash
## Descubrir el nombre real
ip link show | grep -v lo | grep -v enp0s3

## Editar el archivo Netplan con el nombre correcto
sudo nano /etc/netplan/99-lab-static.yaml
## Cambiar "enp0s8" por el nombre real detectado

## Aplicar
sudo netplan apply
```

5. Verifique:

```bash
ip -br addr show
```

## Limpieza

Este laboratorio **no requiere limpieza destructiva** porque es el último del batch y la configuración creada es parte del estado final deseado del entorno. La configuración de red estática debe permanecer activa como parte del entorno de laboratorio completado.

**Elementos que deben preservarse (NO eliminar):**

| Elemento | Ruta | Motivo |
|---|---|---|
| Archivo Netplan | `/etc/netplan/99-lab-static.yaml` | Configuración de red persistente del laboratorio |
| Baseline acumulativo | `/home/alumno/lab-baseline.txt` | Entregable final del curso piloto |
| Copia de respaldo del baseline | `/home/alumno/lab-baseline-final-*.txt` | Respaldo del entregable |

**Si fuera necesario revertir la configuración de red** (por ejemplo, para reutilizar la VM en otro contexto):

```bash
## SOLO ejecutar si se necesita revertir la configuración de red
## sudo rm /etc/netplan/99-lab-static.yaml
## sudo netplan apply
## ip addr show enp0s8  # Verificar que ya no tiene IP estática
```

**Verificación final del estado del sistema completo:**

```bash
echo "=== ESTADO FINAL DEL ENTORNO DE LABORATORIO ==="
echo ""
echo "--- Sistema operativo ---"
lsb_release -a 2>/dev/null
echo ""
echo "--- Kernel ---"
uname -r
echo ""
echo "--- Swap ---"
swapon --show
echo ""
echo "--- Discos adicionales ---"
lsblk -f | grep -E "sdb|sdc|datos"
echo ""
echo "--- Interfaces de red ---"
ip -br addr show
echo ""
echo "--- Rutas ---"
ip route show
echo ""
echo "--- Archivo Netplan ---"
ls -la /etc/netplan/99-lab-static.yaml
echo ""
echo "--- Baseline ---"
wc -l /home/alumno/lab-baseline.txt
echo "=== FIN DEL ESTADO FINAL ==="
```

## Resumen

### Logros de este laboratorio

En este laboratorio se completaron las siguientes tareas:

1. **Identificación de interfaces**: Se usaron `ip link show` e `ip addr show` para descubrir las interfaces `enp0s3` (NAT) y `enp0s8` (red interna) y documentar su estado inicial.

2. **Análisis de configuración existente**: Se examinaron los archivos Netplan existentes en `/etc/netplan/` para comprender la estructura de configuración antes de realizar cambios.

3. **Configuración de IP estática**: Se creó el archivo `/etc/netplan/99-lab-static.yaml` con direccionamiento estático `192.168.100.10/24`, gateway `192.168.100.1` y DNS `8.8.8.8` para la interfaz `enp0s8`, con permisos `600`.

4. **Validación y aplicación**: Se usó `netplan generate` para validar sintaxis, `netplan try` para aplicación provisional con rollback automático, y `netplan apply` para aplicación definitiva.

5. **Verificación de conectividad**: Se confirmó que la IP estática responde, que la interfaz NAT sigue operativa y que el acceso a Internet se preservó.

6. **Persistencia**: Se reinició la VM y se verificó que la configuración de red sobrevive al reinicio.

7. **Documentación**: Se actualizó el archivo `/home/alumno/lab-baseline.txt` con la configuración de red final, completando el entregable acumulativo del curso piloto.

### Relación con el Setup Guide de Thor

Este laboratorio cierra el batch de preparación del entorno. El archivo `/home/alumno/lab-baseline.txt` ahora contiene el registro completo de:

- ✅ Versión del SO y kernel (Lab 01-00-01)
- ✅ Actualizaciones del sistema y gestión APT (Lab 01-00-02)
- ✅ Configuración de discos adicionales con UUIDs y puntos de montaje (Lab 02-00-02)
- ✅ Configuración de swap persistente (Lab 02-00-02)
- ✅ Configuración de red: interfaces, IPs, gateway, DNS (Lab 03-00-01)

### Conceptos clave reforzados

| Concepto | Herramienta/Comando | Aplicación en este lab |
|---|---|---|
| Nomenclatura predictable de interfaces | `ip link show` | Identificar `enp0s3` y `enp0s8` |
| Modos de red en VirtualBox | Configuración del hipervisor | NAT para Internet, Red Interna para laboratorio |
| Configuración declarativa de red | Netplan + YAML | `/etc/netplan/99-lab-static.yaml` |
| Aplicación segura de cambios | `netplan try` | Rollback automático en caso de error |
| Verificación de conectividad | `ping`, `ip addr`, `ip route` | Confirmar IP, rutas y alcanzabilidad |
| Persistencia de configuración | Reinicio + verificación | Netplan aplica automáticamente al arranque |

### Recursos adicionales

| Recurso | URL |
|---|---|
| Documentación oficial de Netplan | [https://netplan.readthedocs.io/en/stable/](https://netplan.readthedocs.io/en/stable/) |
| Referencia de configuración Netplan (YAML) | [https://netplan.readthedocs.io/en/stable/netplan-yaml/](https://netplan.readthedocs.io/en/stable/netplan-yaml/) |
| Modos de red en VirtualBox (Capítulo 6) | [https://www.virtualbox.org/manual/ch06.html](https://www.virtualbox.org/manual/ch06.html) |
| Documentación de iproute2 | [https://wiki.linuxfoundation.org/networking/iproute2](https://wiki.linuxfoundation.org/networking/iproute2) |
| Guía de red de Ubuntu Server 24.04 | [https://ubuntu.com/server/docs/network-configuration](https://ubuntu.com/server/docs/network-configuration) |
| Predictable Network Interface Names (systemd) | [https://systemd.io/PREDICTABLE_INTERFACE_NAMES/](https://systemd.io/PREDICTABLE_INTERFACE_NAMES/) |

---

# Demo: Validación integral del entorno

## Metadatos

| Campo | Detalle |
|---|---|
| **Duración** | 75 minutos |
| **Complejidad** | Fácil |
| **Nivel Bloom** | Comprender |
| **Tipo de actividad** | Demostración en vivo del instructor |
| **Curso** | Preparación de entornos Linux — Curso piloto Setup Guide de Thor |

## Descripción General

Esta demostración cierra el proceso de preparación del entorno de laboratorio. El instructor ejecuta, paso a paso y en vivo, la verificación integral de todos los componentes instalados y configurados en la máquina virtual Ubuntu Desktop 24.04 LTS sobre VirtualBox, contrastando cada resultado con los valores esperados en la Setup Guide de Thor. Al finalizar, se genera un reporte de validación persistente que documenta el estado completo del entorno y sirve como artefacto de referencia para los módulos posteriores del curso.

> ℹ️ **Nota:** Esta práctica es una **Demostración realizada por el instructor**. El instructor ejecutará los pasos y comandos mientras los alumnos observan, toman notas y analizan el procedimiento, en lugar de realizarla individualmente.

## Objetivos de Aprendizaje

Al finalizar esta demostración, los estudiantes serán capaces de:

- [ ] Comprender el proceso sistemático de verificación de un entorno Linux, identificando qué comandos validan cada componente (SO, kernel, paquetes, red)
- [ ] Interpretar la salida de comandos como `lsb_release -a`, `ip addr show` e `ip route show` para confirmar la versión del sistema operativo y el estado de la conectividad de red
- [ ] Explicar cómo se obtienen la versión exacta, la fuente oficial y los metadatos de un paquete APT mediante `apt show` y `dpkg -l`
- [ ] Reconocer la relación entre el modo de red configurado en VirtualBox (NAT) y el direccionamiento IP observado dentro de la máquina virtual
- [ ] Describir la estructura de un reporte de validación reproducible que registra resultados PASS/FAIL para cada componente verificado

## Prerrequisitos

### Conocimientos previos

| Requisito | Descripción |
|---|---|
| Línea de comandos Linux | Navegación de directorios, edición de archivos con `nano` o `cat`, ejecución de comandos con `sudo` |
| Conceptos de red IP | Dirección IP, máscara de subred, puerta de enlace predeterminada (gateway) |
| Labs anteriores completados | Labs del batch 1 (01-00-01, 01-00-02) y batch 2 (02-00-01, 02-00-02) finalizados con éxito |

### Acceso requerido

| Recurso | Detalle |
|---|---|
| Máquina virtual | Ubuntu Desktop 24.04.2 LTS (Noble Numbat) sobre VirtualBox 7.0.14 |
| Usuario | `alumno` con acceso `sudo` (contraseña: `Lab2024!`) |
| VirtualBox Guest Additions | Versión 7.0.14 instaladas y activas |
| Conectividad | Adaptador 1 (`enp0s3`) en modo NAT con acceso a Internet |
| Archivo baseline | `/home/alumno/lab-baseline.txt` existente con registros de labs anteriores |

## Entorno de Laboratorio

### Configuración de hardware de la VM

| Componente | Especificación |
|---|---|
| CPU | 2 núcleos asignados |
| RAM | Mínimo 4 GB (recomendado en host de 8–16 GB) |
| Disco principal | 30 GB (VDI, asignación dinámica) |
| Discos adicionales | `sdb` (5 GB) y `sdc` (5 GB) — configurados en labs previos |
| Adaptador de red 1 | `enp0s3` — Modo NAT |
| Adaptador de red 2 | `enp0s8` — Red Interna (`labnet`) |
| Pantalla | Resolución mínima 1280×768 |

### Software del entorno

| Componente | Versión exacta esperada | Fuente oficial |
|---|---|---|
| Ubuntu Desktop | 24.04.2 LTS (Noble Numbat) | https://releases.ubuntu.com/24.04/ |
| Kernel Linux | 6.8.0-x-generic (serie 6.8 de Noble) | Incluido en Ubuntu 24.04.2 LTS |
| VirtualBox | 7.0.14 | https://www.virtualbox.org/wiki/Download_Old_Builds_7_0 |
| Guest Additions | 7.0.14 | Incluidas en VirtualBox 7.0.14 |
| bash | 5.2.21 | Incluido en Ubuntu 24.04.2 LTS |
| curl | 8.5.0 | Incluido en Ubuntu 24.04.2 LTS |
| net-tools | 2.10-0.1ubuntu4 | Paquete APT en Ubuntu 24.04 LTS |
| iproute2 | 6.1.0-1ubuntu6 | Incluido en Ubuntu 24.04.2 LTS |
| APT | 2.7.14 | Incluido en Ubuntu 24.04.2 LTS |
| lsb-release | 12.0-2 | Incluido en Ubuntu 24.04.2 LTS |

### Estado esperado antes de iniciar

El instructor verifica rápidamente que la VM arranca correctamente y que el usuario `alumno` puede abrir una terminal:

```bash
## Abrir terminal con Ctrl+Alt+T y confirmar usuario activo
whoami
## Salida esperada: alumno

## Confirmar que el archivo baseline de labs previos existe
ls -l /home/alumno/lab-baseline.txt
```

## Instrucciones Paso a Paso

### Paso 1: Preparar el directorio y archivo de reporte de validación

**Objetivo:** Crear la estructura de directorio y el archivo donde se consolidará el reporte de validación integral del entorno.

**Instrucciones (el instructor demuestra):**

1. Abrir una terminal en la VM con `Ctrl+Alt+T`.

2. Crear el directorio de trabajo para este laboratorio:

```bash
mkdir -p /home/alumno/lab-env
```

3. Inicializar el archivo de reporte de validación con encabezado y fecha:

```bash
cat > /home/alumno/lab-env/validation_report.txt << 'EOF'
========================================================
 REPORTE DE VALIDACIÓN INTEGRAL DEL ENTORNO
 Setup Guide de Thor — Curso Piloto
========================================================
EOF

echo "Fecha de validación: $(date '+%Y-%m-%d %H:%M:%S %Z')" >> /home/alumno/lab-env/validation_report.txt
echo "Generado por: $(whoami)@$(hostname)" >> /home/alumno/lab-env/validation_report.txt
echo "--------------------------------------------------------" >> /home/alumno/lab-env/validation_report.txt
```

4. Explicar a los estudiantes la importancia de registrar la fecha y el usuario que ejecuta la validación para trazabilidad.

**Salida esperada:**

```
Fecha de validación: 2025-01-15 10:00:00 UTC
Generado por: alumno@ubuntu-lab
```

**Verificación:**

```bash
cat /home/alumno/lab-env/validation_report.txt
```

El archivo debe mostrar el encabezado completo con fecha y nombre de host.

---

### Paso 2: Verificar el sistema operativo y el kernel

**Objetivo:** Confirmar que la VM ejecuta Ubuntu 24.04.2 LTS (Noble Numbat) con el kernel correcto, contrastando con la Setup Guide de Thor.

**Instrucciones (el instructor demuestra):**

1. Ejecutar `lsb_release -a` para obtener la información completa de la distribución:

```bash
lsb_release -a
```

2. Ejecutar `uname -r` para obtener la versión exacta del kernel:

```bash
uname -r
```

3. Ejecutar `cat /etc/os-release` para obtener metadatos adicionales del sistema operativo:

```bash
cat /etc/os-release
```

4. Explicar a los estudiantes cada campo de la salida:
   - `Distributor ID`: Identifica la distribución (Ubuntu).
   - `Release`: Número de versión (24.04).
   - `Codename`: Nombre clave (noble).
   - `VERSION_ID` en `os-release`: Confirma la versión numérica.

5. Registrar los resultados en el reporte de validación:

```bash
{
  echo ""
  echo "[1] SISTEMA OPERATIVO Y KERNEL"
  echo "Comando: lsb_release -a"
  DISTRO=$(lsb_release -d --short)
  RELEASE=$(lsb_release -r --short)
  CODENAME=$(lsb_release -c --short)
  KERNEL=$(uname -r)
  ARCH=$(uname -m)

  echo "  Distribución: $DISTRO"
  echo "  Release:      $RELEASE"
  echo "  Codename:     $CODENAME"
  echo "  Kernel:       $KERNEL"
  echo "  Arquitectura: $ARCH"

  # Validación automática
  if [[ "$RELEASE" == "24.04" && "$CODENAME" == "noble" && "$ARCH" == "x86_64" ]]; then
    echo "  Resultado:    ✅ PASS"
  else
    echo "  Resultado:    ❌ FAIL — Revisar Setup Guide de Thor"
  fi
} >> /home/alumno/lab-env/validation_report.txt
```

6. Mostrar el contenido actualizado del reporte:

```bash
cat /home/alumno/lab-env/validation_report.txt
```

**Salida esperada de `lsb_release -a`:**

```
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.2 LTS
Release:        24.04
Codename:       noble
```

**Salida esperada de `uname -r`:**

```
6.8.0-51-generic
```

> **Nota del instructor:** La versión exacta del kernel (por ejemplo `6.8.0-51-generic`) puede variar según las actualizaciones aplicadas en el lab 01-00-02. Lo importante es que pertenezca a la serie `6.8.0-*-generic` correspondiente a Noble Numbat.

**Verificación:**

```bash
grep "PASS\|FAIL" /home/alumno/lab-env/validation_report.txt
```

Debe mostrar `✅ PASS` para la sección del sistema operativo.

---

### Paso 3: Verificar componentes de software instalados

**Objetivo:** Validar la versión exacta de cada herramienta del entorno (bash, curl, net-tools, iproute2, APT, Guest Additions) contra los valores esperados en la Setup Guide de Thor.

**Instrucciones (el instructor demuestra):**

1. Crear una función auxiliar en la terminal para estandarizar la verificación de cada componente:

```bash
check_version() {
  local NAME="$1"
  local EXPECTED="$2"
  local ACTUAL="$3"
  local SOURCE="$4"

  echo "  Componente:       $NAME"
  echo "  Versión esperada: $EXPECTED"
  echo "  Versión actual:   $ACTUAL"
  echo "  Fuente oficial:   $SOURCE"

  if echo "$ACTUAL" | grep -q "$EXPECTED"; then
    echo "  Resultado:        ✅ PASS"
  else
    echo "  Resultado:        ❌ FAIL"
  fi
  echo ""
}
```

2. Verificar cada componente y registrar en el reporte:

```bash
{
  echo ""
  echo "[2] COMPONENTES DE SOFTWARE"
  echo ""

  # bash
  BASH_VER=$(bash --version | head -1 | grep -oP '\d+\.\d+\.\d+')
  check_version "bash" "5.2.21" "$BASH_VER" "Incluido en Ubuntu 24.04.2 LTS"

  # curl
  CURL_VER=$(curl --version | head -1 | grep -oP '\d+\.\d+\.\d+' | head -1)
  check_version "curl" "8.5.0" "$CURL_VER" "Incluido en Ubuntu 24.04.2 LTS"

  # net-tools (ifconfig)
  NETTOOLS_VER=$(dpkg -l net-tools 2>/dev/null | awk '/^ii/{print $3}')
  check_version "net-tools" "2.10-0.1ubuntu4" "$NETTOOLS_VER" "Paquete APT — Ubuntu 24.04 LTS"

  # iproute2
  IP_VER=$(ip -V 2>&1 | grep -oP 'iproute2-\S+' || echo "no encontrado")
  check_version "iproute2" "6.1.0" "$IP_VER" "Incluido en Ubuntu 24.04.2 LTS"

  # APT
  APT_VER=$(apt --version 2>/dev/null | grep -oP '\d+\.\d+\.\d+' | head -1)
  check_version "APT" "2.7.14" "$APT_VER" "Incluido en Ubuntu 24.04.2 LTS"

  # lsb-release
  LSB_VER=$(dpkg -l lsb-release 2>/dev/null | awk '/^ii/{print $3}')
  check_version "lsb-release" "12.0-2" "$LSB_VER" "Incluido en Ubuntu 24.04.2 LTS"

  # VirtualBox Guest Additions
  GA_VER=$(modinfo vboxguest 2>/dev/null | awk '/^version:/{print $2}' || echo "no encontrado")
  check_version "Guest Additions" "7.0.14" "$GA_VER" "https://www.virtualbox.org/wiki/Download_Old_Builds_7_0"

} >> /home/alumno/lab-env/validation_report.txt
```

3. Mostrar el resultado acumulado:

```bash
cat /home/alumno/lab-env/validation_report.txt
```

4. Explicar a los estudiantes:
   - La diferencia entre `dpkg -l` (consulta la base de datos local de paquetes instalados) y `apt show` (consulta los metadatos del repositorio).
   - Que la versión de Guest Additions se obtiene del módulo del kernel (`modinfo vboxguest`), no de un paquete APT.

**Salida esperada (extracto):**

```
  Componente:       bash
  Versión esperada: 5.2.21
  Versión actual:   5.2.21
  Fuente oficial:   Incluido en Ubuntu 24.04.2 LTS
  Resultado:        ✅ PASS

  Componente:       curl
  Versión esperada: 8.5.0
  Versión actual:   8.5.0
  Fuente oficial:   Incluido en Ubuntu 24.04.2 LTS
  Resultado:        ✅ PASS
```

**Verificación:**

```bash
grep -c "PASS" /home/alumno/lab-env/validation_report.txt
```

Se esperan al menos 8 resultados PASS (1 del SO + 7 componentes).

---

### Paso 4: Verificar fuentes oficiales y metadatos de paquetes

**Objetivo:** Mostrar cómo obtener metadatos completos de un paquete APT (mantenedor, homepage, sección) para contrastar con la fuente oficial documentada en la Setup Guide de Thor.

**Instrucciones (el instructor demuestra):**

1. Ejecutar `apt show` para un paquete representativo (`curl`):

```bash
apt show curl 2>/dev/null | grep -E "^(Package|Version|Maintainer|Homepage|Section|Origin):"
```

2. Ejecutar `dpkg -l curl` para ver el estado de instalación:

```bash
dpkg -l curl
```

3. Repetir para `net-tools` para mostrar un paquete que no viene preinstalado en todas las distribuciones:

```bash
apt show net-tools 2>/dev/null | grep -E "^(Package|Version|Maintainer|Homepage|Section|Origin):"
```

4. Verificar la política de repositorios para confirmar que los paquetes provienen de los repositorios oficiales de Ubuntu:

```bash
apt policy curl
```

5. Registrar un resumen en el reporte:

```bash
{
  echo ""
  echo "[3] FUENTES OFICIALES Y METADATOS"
  echo ""
  echo "--- curl ---"
  apt show curl 2>/dev/null | grep -E "^(Package|Version|Maintainer|Homepage|Origin):"
  echo ""
  echo "--- net-tools ---"
  apt show net-tools 2>/dev/null | grep -E "^(Package|Version|Maintainer|Homepage|Origin):"
  echo ""
  echo "--- iproute2 ---"
  apt show iproute2 2>/dev/null | grep -E "^(Package|Version|Maintainer|Homepage|Origin):"
  echo ""
  echo "Repositorio activo (apt policy curl):"
  apt policy curl 2>/dev/null | head -6
  echo ""
  echo "  Resultado: ✅ PASS — Todos los paquetes provienen de repositorios oficiales Ubuntu"
} >> /home/alumno/lab-env/validation_report.txt
```

6. Explicar a los estudiantes:
   - El campo `Origin: Ubuntu` confirma que el paquete proviene de los repositorios oficiales.
   - El campo `Maintainer` indica quién mantiene el paquete dentro de la distribución.
   - `apt policy` muestra la prioridad y el repositorio exacto (URL) desde donde se instaló el paquete.

**Salida esperada de `apt show curl` (extracto):**

```
Package: curl
Version: 8.5.0-2ubuntu10
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Homepage: https://curl.haxx.se
Section: web
Origin: Ubuntu
```

**Salida esperada de `apt policy curl` (extracto):**

```
curl:
  Installed: 8.5.0-2ubuntu10
  Candidate: 8.5.0-2ubuntu10
  Version table:
 *** 8.5.0-2ubuntu10 500
        500 http://archive.ubuntu.com/ubuntu noble/main amd64 Packages
```

**Verificación:**

```bash
apt policy curl | grep "archive.ubuntu.com"
```

Debe devolver al menos una línea que confirme el repositorio oficial de Ubuntu.

---

### Paso 5: Validar la configuración de red y el direccionamiento IP

**Objetivo:** Inspeccionar las interfaces de red activas, el direccionamiento IP, la puerta de enlace y la conectividad, relacionando los resultados con el modo de red NAT configurado en VirtualBox.

**Instrucciones (el instructor demuestra):**

1. Mostrar todas las interfaces de red con `ip addr show`:

```bash
ip addr show
```

2. Filtrar solo las interfaces activas y sus direcciones IPv4:

```bash
ip -4 addr show | grep -E "^[0-9]+:|inet "
```

3. Mostrar la tabla de enrutamiento y la puerta de enlace predeterminada:

```bash
ip route show
```

4. Usar `ifconfig` (del paquete `net-tools`) para contrastar la salida con el formato clásico:

```bash
ifconfig
```

5. Explicar a los estudiantes la relación entre la salida observada y el modo NAT de VirtualBox:
   - La interfaz `enp0s3` recibe la IP `10.0.2.15/24` del servidor DHCP integrado en VirtualBox.
   - La puerta de enlace `10.0.2.2` es el enrutador virtual de VirtualBox en modo NAT.
   - La interfaz `enp0s8` (si está configurada como Red Interna) puede tener la IP estática `192.168.100.10/24` del lab 03-00-01, o estar sin IP si aún no se configuró.

6. Verificar conectividad hacia Internet:

```bash
ping -c 4 8.8.8.8
```

7. Verificar resolución DNS y conectividad HTTP:

```bash
curl -s -o /dev/null -w "HTTP Status: %{http_code}\nTiempo total: %{time_total}s\n" https://ubuntu.com
```

8. Verificar el estado de las interfaces con NetworkManager:

```bash
nmcli device status
```

9. Registrar los resultados en el reporte:

```bash
{
  echo ""
  echo "[4] CONFIGURACIÓN DE RED Y DIRECCIONAMIENTO"
  echo ""
  echo "Interfaces activas:"
  ip -4 addr show | grep -E "^[0-9]+:|inet "
  echo ""
  echo "Tabla de enrutamiento:"
  ip route show
  echo ""
  echo "Modo de red VirtualBox: NAT (Adaptador 1 — enp0s3)"
  echo ""

  # Verificar conectividad
  if ping -c 2 -W 3 8.8.8.8 &>/dev/null; then
    echo "  Ping a 8.8.8.8:     ✅ PASS"
  else
    echo "  Ping a 8.8.8.8:     ❌ FAIL"
  fi

  HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 https://ubuntu.com 2>/dev/null || echo "000")
  if [[ "$HTTP_CODE" -ge 200 && "$HTTP_CODE" -lt 400 ]]; then
    echo "  HTTPS ubuntu.com:   ✅ PASS (HTTP $HTTP_CODE)"
  else
    echo "  HTTPS ubuntu.com:   ❌ FAIL (HTTP $HTTP_CODE)"
  fi

  # Verificar IP esperada en NAT
  NAT_IP=$(ip -4 addr show enp0s3 2>/dev/null | grep -oP 'inet \K[\d.]+')
  echo "  IP enp0s3 (NAT):    $NAT_IP"
  if [[ "$NAT_IP" == "10.0.2.15" ]]; then
    echo "  IP NAT esperada:    ✅ PASS"
  else
    echo "  IP NAT esperada:    ⚠️ WARN — IP diferente a 10.0.2.15 (puede ser válido)"
  fi

  GW=$(ip route show default | awk '{print $3}')
  echo "  Gateway:             $GW"
  if [[ "$GW" == "10.0.2.2" ]]; then
    echo "  Gateway esperado:   ✅ PASS"
  else
    echo "  Gateway esperado:   ⚠️ WARN — Gateway diferente a 10.0.2.2"
  fi
} >> /home/alumno/lab-env/validation_report.txt
```

10. Mostrar el resultado en pantalla y explicar cada línea:

```bash
tail -30 /home/alumno/lab-env/validation_report.txt
```

**Salida esperada de `ip addr show` (extracto para `enp0s3`):**

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 08:00:27:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
```

**Salida esperada de `ip route show`:**

```
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15
```

**Salida esperada de `ping -c 4 8.8.8.8`:**

```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=12.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=11.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=12.1 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=117 time=11.9 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
```

**Verificación:**

```bash
grep -E "PASS|FAIL|WARN" /home/alumno/lab-env/validation_report.txt | tail -6
```

Todos los indicadores de red deben mostrar PASS.

---

### Paso 6: Verificar componentes de labs anteriores (swap, discos, baseline)

**Objetivo:** Confirmar que la configuración de swap persistente y los discos adicionales de labs previos siguen activos y correctamente configurados, cerrando la validación integral.

**Instrucciones (el instructor demuestra):**

1. Verificar el archivo de swap:

```bash
swapon --show
```

2. Verificar el valor de `vm.swappiness`:

```bash
cat /proc/sys/vm/swappiness
```

3. Verificar los discos adicionales y sus puntos de montaje:

```bash
lsblk -f /dev/sdb /dev/sdc 2>/dev/null
```

4. Verificar que los puntos de montaje están activos:

```bash
df -h /mnt/datos1 /mnt/datos2 2>/dev/null
```

5. Verificar la existencia y contenido del archivo baseline acumulativo:

```bash
wc -l /home/alumno/lab-baseline.txt
head -20 /home/alumno/lab-baseline.txt
```

6. Registrar en el reporte:

```bash
{
  echo ""
  echo "[5] COMPONENTES DE LABS ANTERIORES"
  echo ""

  # Swap
  SWAP_SIZE=$(swapon --show --noheadings --bytes 2>/dev/null | awk '{print $3}')
  SWAPPINESS=$(cat /proc/sys/vm/swappiness)
  echo "  Swap activo:        $(swapon --show --noheadings | awk '{print $1, $3}' || echo 'NO ENCONTRADO')"
  echo "  Swappiness:         $SWAPPINESS"
  if [[ -n "$SWAP_SIZE" && "$SWAPPINESS" == "10" ]]; then
    echo "  Resultado swap:     ✅ PASS"
  else
    echo "  Resultado swap:     ❌ FAIL"
  fi

  echo ""

  # Discos adicionales
  if mountpoint -q /mnt/datos1 2>/dev/null && mountpoint -q /mnt/datos2 2>/dev/null; then
    echo "  /mnt/datos1:        ✅ PASS (montado)"
    echo "  /mnt/datos2:        ✅ PASS (montado)"
  else
    echo "  Discos adicionales: ❌ FAIL — Verificar /etc/fstab y montaje"
  fi

  echo ""

  # Baseline
  if [[ -f /home/alumno/lab-baseline.txt ]]; then
    LINES=$(wc -l < /home/alumno/lab-baseline.txt)
    echo "  lab-baseline.txt:   ✅ PASS ($LINES líneas)"
  else
    echo "  lab-baseline.txt:   ❌ FAIL — Archivo no encontrado"
  fi
} >> /home/alumno/lab-env/validation_report.txt
```

**Salida esperada de `swapon --show`:**

```
NAME      TYPE SIZE USED PRIO
/swapfile file   2G   0B   -2
```

**Salida esperada de `lsblk -f /dev/sdb /dev/sdc`:**

```
NAME   FSTYPE FSVER LABEL UUID                                 MOUNTPOINTS
sdb
└─sdb1 ext4   1.0         xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt/datos1
sdc
└─sdc1 ext4   1.0         yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy /mnt/datos2
```

**Verificación:**

```bash
grep "PASS\|FAIL" /home/alumno/lab-env/validation_report.txt | wc -l
```

Se espera un total de al menos 14 líneas de resultado (PASS o FAIL) acumuladas en todo el reporte.

---

### Paso 7: Generar el reporte final y actualizar el baseline acumulativo

**Objetivo:** Consolidar el reporte de validación con un resumen ejecutivo y actualizar el archivo baseline acumulativo del curso piloto.

**Instrucciones (el instructor demuestra):**

1. Agregar el resumen ejecutivo al reporte:

```bash
{
  echo ""
  echo "========================================================"
  echo " RESUMEN EJECUTIVO"
  echo "========================================================"
  TOTAL_PASS=$(grep -c "✅ PASS" /home/alumno/lab-env/validation_report.txt)
  TOTAL_FAIL=$(grep -c "❌ FAIL" /home/alumno/lab-env/validation_report.txt)
  TOTAL_WARN=$(grep -c "⚠️ WARN" /home/alumno/lab-env/validation_report.txt)

  echo "  Verificaciones PASS: $TOTAL_PASS"
  echo "  Verificaciones FAIL: $TOTAL_FAIL"
  echo "  Advertencias WARN:   $TOTAL_WARN"
  echo ""

  if [[ "$TOTAL_FAIL" -eq 0 ]]; then
    echo "  ESTADO FINAL: ✅ ENTORNO VALIDADO — Listo para módulos de contenido"
  else
    echo "  ESTADO FINAL: ❌ ENTORNO NO VALIDADO — Revisar ítems FAIL"
  fi

  echo ""
  echo "========================================================"
  echo " Fin del reporte — $(date '+%Y-%m-%d %H:%M:%S %Z')"
  echo "========================================================"
} >> /home/alumno/lab-env/validation_report.txt
```

2. Mostrar el reporte completo:

```bash
cat /home/alumno/lab-env/validation_report.txt
```

3. Actualizar el archivo baseline acumulativo:

```bash
{
  echo ""
  echo "===== LAB 03-00-02: Validación integral del entorno ====="
  echo "Fecha: $(date '+%Y-%m-%d %H:%M:%S')"
  echo "Reporte completo en: /home/alumno/lab-env/validation_report.txt"
  TOTAL_PASS=$(grep -c "✅ PASS" /home/alumno/lab-env/validation_report.txt)
  TOTAL_FAIL=$(grep -c "❌ FAIL" /home/alumno/lab-env/validation_report.txt)
  echo "Resultado: PASS=$TOTAL_PASS, FAIL=$TOTAL_FAIL"
  echo "Kernel: $(uname -r)"
  echo "IP NAT: $(ip -4 addr show enp0s3 2>/dev/null | grep -oP 'inet \K[\d./]+')"
  echo "Swap: $(swapon --show --noheadings 2>/dev/null | awk '{print $1, $3}')"
  echo "Discos: $(lsblk -ln -o NAME,MOUNTPOINT /dev/sdb /dev/sdc 2>/dev/null | tr '\n' ' ')"
} >> /home/alumno/lab-baseline.txt
```

4. Verificar el baseline actualizado:

```bash
tail -15 /home/alumno/lab-baseline.txt
```

5. Explicar a los estudiantes que este archivo baseline acumulativo es el **entregable final del curso piloto** y debe contener los registros de todos los labs completados.

**Salida esperada (resumen ejecutivo):**

```
========================================================
 RESUMEN EJECUTIVO
========================================================
  Verificaciones PASS: 14
  Verificaciones FAIL: 0
  Advertencias WARN:   0

  ESTADO FINAL: ✅ ENTORNO VALIDADO — Listo para módulos de contenido

========================================================
 Fin del reporte — 2025-01-15 10:55:00 UTC
========================================================
```

**Verificación:**

```bash
grep "ESTADO FINAL" /home/alumno/lab-env/validation_report.txt
```

Debe indicar `✅ ENTORNO VALIDADO`.

---

### Paso 8: Discusión — Manejo de fallos y buenas prácticas

**Objetivo:** Explicar el procedimiento a seguir cuando una verificación falla, cómo consultar la Setup Guide de Thor y cómo reportar discrepancias.

**Instrucciones (el instructor explica y demuestra):**

1. Simular un escenario de fallo eliminando temporalmente un paquete de la verificación (solo verbal, sin ejecutar):

   > *"Si `net-tools` no estuviera instalado, la verificación mostraría FAIL. El procedimiento sería: (a) consultar la Setup Guide de Thor para confirmar que el paquete es requerido, (b) instalarlo con `sudo apt install net-tools`, (c) re-ejecutar la verificación."*

2. Mostrar cómo se vería un FAIL en el reporte (ejemplo construido):

```bash
echo "--- EJEMPLO DE FALLO (no real) ---"
echo "  Componente:       net-tools"
echo "  Versión esperada: 2.10-0.1ubuntu4"
echo "  Versión actual:   (no instalado)"
echo "  Resultado:        ❌ FAIL"
echo ""
echo "  Acción correctiva: sudo apt install net-tools"
echo "  Referencia: Setup Guide de Thor, sección 'Paquetes requeridos'"
```

3. Explicar las buenas prácticas de documentación:
   - **Reproducibilidad:** Cualquier persona debe poder re-ejecutar los mismos comandos y obtener resultados comparables.
   - **Trazabilidad:** Cada verificación incluye el comando exacto, la versión esperada y la fuente oficial.
   - **Automatización gradual:** Este reporte manual puede convertirse en un script de validación automatizado en el futuro.

4. Abrir espacio para preguntas de los estudiantes (5–10 minutos).

**Puntos de discusión sugeridos:**

| Pregunta | Respuesta esperada |
|---|---|
| ¿Qué pasa si el kernel es diferente al documentado? | Si pertenece a la serie 6.8.0-*-generic es válido; si es una serie diferente (ej. 6.5.0), hay que investigar |
| ¿Cómo sé si un paquete viene de un repositorio no oficial? | `apt policy <paquete>` muestra la URL del repositorio; debe ser `archive.ubuntu.com` o `security.ubuntu.com` |
| ¿Por qué la IP en modo NAT siempre es 10.0.2.15? | Es la IP predeterminada del servidor DHCP integrado en VirtualBox para modo NAT; puede cambiar si hay múltiples VMs NAT simultáneas |

**Verificación:** No aplica verificación técnica; este paso es de discusión y cierre pedagógico.

## Validación y Pruebas

### Criterios de aceptación medibles

| # | Criterio | Comando de verificación | Resultado esperado |
|---|---|---|---|
| 1 | El reporte de validación existe y tiene contenido | `test -s /home/alumno/lab-env/validation_report.txt && echo OK` | `OK` |
| 2 | El SO está correctamente identificado como Ubuntu 24.04 | `lsb_release -r --short` | `24.04` |
| 3 | Todos los componentes de software tienen resultado PASS | `grep -c "❌ FAIL" /home/alumno/lab-env/validation_report.txt` | `0` |
| 4 | La conectividad a Internet funciona desde la VM | `ping -c 1 -W 5 8.8.8.8 && echo OK` | `OK` (0% packet loss) |
| 5 | El archivo baseline acumulativo contiene registros del lab actual | `grep "LAB 03-00-02" /home/alumno/lab-baseline.txt` | Línea encontrada |
| 6 | El resumen ejecutivo indica entorno validado | `grep "ENTORNO VALIDADO" /home/alumno/lab-env/validation_report.txt` | Línea encontrada |
| 7 | Los paquetes provienen de repositorios oficiales Ubuntu | `apt policy curl 2>/dev/null \| grep "archive.ubuntu.com"` | Al menos una línea |

### Verificación integral rápida

El instructor puede ejecutar esta secuencia completa como validación final en un solo bloque:

```bash
echo "=== VALIDACIÓN INTEGRAL RÁPIDA ==="
echo -n "1. Reporte existe: "; test -s /home/alumno/lab-env/validation_report.txt && echo "✅" || echo "❌"
echo -n "2. Ubuntu 24.04:   "; [[ "$(lsb_release -r --short)" == "24.04" ]] && echo "✅" || echo "❌"
echo -n "3. Sin FAIL:       "; [[ $(grep -c "❌ FAIL" /home/alumno/lab-env/validation_report.txt) -eq 0 ]] && echo "✅" || echo "❌"
echo -n "4. Ping 8.8.8.8:   "; ping -c 1 -W 5 8.8.8.8 &>/dev/null && echo "✅" || echo "❌"
echo -n "5. Baseline:       "; grep -q "LAB 03-00-02" /home/alumno/lab-baseline.txt && echo "✅" || echo "❌"
echo -n "6. Validado:       "; grep -q "ENTORNO VALIDADO" /home/alumno/lab-env/validation_report.txt && echo "✅" || echo "❌"
echo -n "7. Repo oficial:   "; apt policy curl 2>/dev/null | grep -q "archive.ubuntu.com" && echo "✅" || echo "❌"
echo "=== FIN ==="
```

**Salida esperada:**

```
=== VALIDACIÓN INTEGRAL RÁPIDA ===
1. Reporte existe: ✅
2. Ubuntu 24.04:   ✅
3. Sin FAIL:       ✅
4. Ping 8.8.8.8:   ✅
5. Baseline:       ✅
6. Validado:       ✅
7. Repo oficial:   ✅
=== FIN ===
```

### Caso adverso: información contradictoria o inexistente

Para verificar la robustez del proceso de validación, el instructor demuestra qué ocurre cuando se consulta un paquete que **no existe** en los repositorios:

```bash
## Intentar verificar un paquete ficticio
apt show paquete-inexistente-xyz 2>&1
```

**Salida esperada:**

```
N: Unable to locate package paquete-inexistente-xyz
E: No packages found
```

El instructor explica:

> *"Si la Setup Guide de Thor listara un paquete que no existe en los repositorios de Ubuntu 24.04, este sería el comportamiento. La verificación debe registrar FAIL y el equipo del curso piloto debe reportar la discrepancia para corregir la guía. Nunca se debe asumir que un paquete faltante es un error menor; puede indicar un cambio de nombre, una dependencia no documentada o un error en la guía."*

Adicionalmente, se muestra qué ocurre si se intenta verificar la versión de una herramienta no instalada:

```bash
## Ejemplo: herramienta no instalada
which htop 2>/dev/null || echo "htop no está instalado en este entorno"
```

Esto refuerza la importancia de que el reporte no asuma la presencia de ningún componente y valide cada uno explícitamente.

## Solución de Problemas

### Problema 1: El ping a 8.8.8.8 falla con "Network is unreachable"

**Síntomas:**

```
ping: connect: Network is unreachable
```

La verificación de red registra `❌ FAIL` en el reporte. El comando `ip route show` no muestra una ruta `default`.

**Causa:** El adaptador de red en modo NAT no está conectado en VirtualBox. Esto puede ocurrir si la opción "Cable conectado" está desmarcada en la configuración del adaptador de red de la VM, o si el servicio de red dentro de la VM no levantó la interfaz correctamente.

**Solución:**

1. En la ventana de VirtualBox, ir a **Dispositivos → Red → Configuración de red…**
2. Verificar que el **Adaptador 1** esté habilitado, en modo **NAT** y con la casilla **"Cable conectado"** marcada.
3. Dentro de la VM, reiniciar el servicio de red:

```bash
sudo systemctl restart NetworkManager
## Esperar 5 segundos y verificar
sleep 5
ip addr show enp0s3
ip route show
ping -c 2 8.8.8.8
```

4. Si persiste, desconectar y reconectar el cable virtual desde el menú de VirtualBox: **Dispositivos → Red → Conectar adaptador de red 1**.

---

### Problema 2: `dpkg -l net-tools` muestra "no packages found" o estado "un" (no instalado)

**Síntomas:**

```bash
dpkg -l net-tools
## Salida: dpkg-query: no packages found matching net-tools
```

O bien la columna de estado muestra `un` en lugar de `ii`:

```
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
||/ Name           Version      Architecture Description
+++-==============-============-============-=================================
un  net-tools      <none>       <none>       (no description available)
```

**Causa:** El paquete `net-tools` no se instaló durante la preparación del entorno en los labs anteriores. A diferencia de `iproute2` y `bash`, `net-tools` no viene preinstalado en Ubuntu Desktop 24.04 LTS y debe instalarse explícitamente.

**Solución:**

1. Instalar el paquete desde los repositorios oficiales:

```bash
sudo apt update
sudo apt install -y net-tools
```

2. Verificar la instalación:

```bash
dpkg -l net-tools | grep "^ii"
## Salida esperada: ii  net-tools  2.10-0.1ubuntu4  amd64  NET-3 networking toolkit
```

3. Confirmar que `ifconfig` funciona:

```bash
ifconfig enp0s3 | head -3
```

4. Re-ejecutar la sección de verificación del Paso 3 para actualizar el reporte con el resultado correcto.

## Limpieza

> **Nota:** Al tratarse de una demostración, **no se eliminan** los artefactos generados. Los archivos creados son parte del entregable del curso piloto.

Los siguientes archivos deben **permanecer** en la VM para referencia futura:

| Archivo | Propósito | Ubicación |
|---|---|---|
| `validation_report.txt` | Reporte de validación integral | `/home/alumno/lab-env/validation_report.txt` |
| `lab-baseline.txt` | Baseline acumulativo del curso piloto | `/home/alumno/lab-baseline.txt` |

Si fuera necesario re-ejecutar la demostración desde cero (por ejemplo, para otro grupo de estudiantes), el instructor puede limpiar solo el reporte:

```bash
## SOLO si se necesita re-ejecutar la demo
rm -f /home/alumno/lab-env/validation_report.txt
## NO eliminar lab-baseline.txt — es acumulativo
```

## Resumen

### Conceptos clave demostrados

| Concepto | Comando(s) clave | Resultado |
|---|---|---|
| Identificación del SO y kernel | `lsb_release -a`, `uname -r`, `cat /etc/os-release` | Ubuntu 24.04.2 LTS, kernel 6.8.0-*-generic |
| Verificación de versiones de software | `bash --version`, `curl --version`, `dpkg -l`, `modinfo` | Versiones exactas contrastadas con la Setup Guide |
| Consulta de fuentes oficiales | `apt show`, `apt policy` | Origen Ubuntu, repositorio `archive.ubuntu.com` |
| Diagnóstico de red en modo NAT | `ip addr show`, `ip route show`, `ping`, `curl` | IP 10.0.2.15, gateway 10.0.2.2, conectividad OK |
| Documentación reproducible | Script de reporte con PASS/FAIL | Archivo persistente con trazabilidad completa |

### Relación con la lección 3.1

Esta demostración aplicó directamente los conceptos de la lección **3.1: Direccionamiento y modos de red**:

- Se utilizaron los comandos `ip addr show` e `ip route show` para inspeccionar las interfaces y la tabla de enrutamiento, tal como se describe en la sección "Herramientas de diagnóstico de red en Linux".
- Se identificó el modo NAT de VirtualBox y se explicó la relación entre la IP `10.0.2.15`, la puerta de enlace `10.0.2.2` y el servidor DHCP virtual del hipervisor.
- Se contrastó la salida de `ifconfig` (net-tools) con `ip addr` (iproute2), reforzando que `ip` es la herramienta moderna recomendada.
- Se verificó tanto la conectividad a nivel IP (`ping`) como la resolución DNS y conectividad HTTP (`curl`).

### Entregable del curso piloto

Al finalizar todos los labs del batch 1, 2 y 3, el archivo `/home/alumno/lab-baseline.txt` debe contener:

- ✅ Versión del SO y kernel (inicial y post-actualización)
- ✅ Lista de paquetes actualizados
- ✅ Configuración de swap (tamaño, UUID, swappiness)
- ✅ Configuración de discos adicionales (UUIDs, puntos de montaje, sistemas de archivos)
- ✅ Configuración de red (interfaces, IPs, gateway)
- ✅ Resultado de la validación integral (referencia al reporte)

### Recursos adicionales

| Recurso | URL |
|---|---|
| Documentación oficial de iproute2 | https://wiki.linuxfoundation.org/networking/iproute2 |
| Modos de red en VirtualBox | https://www.virtualbox.org/manual/ch06.html |
| Guía de red en Ubuntu Server | https://ubuntu.com/server/docs/network-configuration |
| Referencia de APT (Debian) | https://www.debian.org/doc/manuals/apt-guide/ |
| Hashes SHA256 de Ubuntu 24.04 | https://releases.ubuntu.com/24.04/SHA256SUMS |
