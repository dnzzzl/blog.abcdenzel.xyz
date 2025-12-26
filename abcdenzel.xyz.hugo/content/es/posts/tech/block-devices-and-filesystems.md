+++
date = '2025-11-16T12:00:00-04:00'
draft = false
title = 'Dispositivos de Bloque y Sistemas de Archivos: Un Viaje de Aprendizaje'
tags = ['linux', 'filesystems', 'storage', 'tutorial']
categories = ['tech']
+++

Compré una nueva unidad NVMe y decidí que este era un momento tan bueno como cualquier otro para realmente entender lo que estaba haciendo en lugar de solo copiar comandos de Stack Overflow hasta que algo funcionara.

Esto es lo que aprendí sobre dispositivos de bloque, tablas de particiones, y por qué todos todavía pretenden que los sectores son de 512 bytes cuando no lo han sido por años.

## Encontrando la Unidad

Primer paso: descubrir dónde demonios está realmente la nueva unidad.

```bash
lsblk
```

Ahí estaba en `/dev/sda`, anunciándose como un dispositivo de 1.8TB sin sistema de archivos y sin particiones. Lienzo en blanco.

```
NAME          FSVER FSSIZE FSAVAIL FSTYPE   LABEL      MOUNTPOINTS                  STATE   PATH                  LOG-SEC PHY-SEC VENDOR
sda                                                                                 running /dev/sda                  512    4096 JMicron
```

Ese nombre de fabricante "JMicron" es el adaptador USB 3.2 que vino en la caja, no la unidad real (PNY CS2241 M.2, si te interesa).

## El Agujero de Conejo del 512e

Nota esos tamaños de sector: 512 bytes lógicos, 4096 bytes físicos. Esto se llama "512e" (emulación de 512 bytes), y es una de esas cosas de compatibilidad hacia atrás que parecía una buena idea en su momento.

**Tamaños de Sector Lógicos vs Físicos**

| Tamaño de Sector Lógico | Tamaño de Sector Físico |
|-------------------------|------------------------|
| 512 bytes              | 4096 bytes            |

El **tamaño de sector lógico** es con lo que el sistema operativo piensa que está trabajando. Es la unidad más pequeña de datos que puede leer o escribir.

El **tamaño de sector físico** es lo que el hardware realmente está usando.

Cuando estos no coinciden, el firmware de la unidad tiene que hacer traducción. ¿Quieres escribir 512 bytes al tercer sector lógico? El firmware tiene que:

1. Cargar el sector físico completo de 4096 bytes (sector 0) en un búfer
2. Modificar el fragmento apropiado de 512 bytes (posición 3 × 512)
3. Escribir el sector completo de 4096 bytes de vuelta al disco

Esto se llama un ciclo de lectura-modificación-escritura, y es más lento que simplemente escribir directamente a sectores nativos de 4K.

### ¿Podemos Arreglar Esto? ¿Deberíamos?

Técnicamente sí—puedes formatear algunas unidades para usar 4Kn (4K nativo) en lugar de 512e. ¿Prácticamente? El consenso de internet es: "Esta es una de esas 'correcciones' de alto riesgo y bajo retorno que no creo que valgan la pena."

Así que lo dejé en 512e y pasé al particionamiento.

## Tablas de Particiones: El Mapa que Tu Unidad Necesita

Antes de que puedas usar una unidad, necesitas una tabla de particiones que es básicamente un mapa que le dice al sistema "estos bytes son la partición 1, estos bytes son la partición 2" y así sucesivamente.

### GPT vs MBR

El estándar moderno es **GPT** (Tabla de Particiones GUID), que es parte de UEFI. Reemplaza el esquema MBR (Registro de Arranque Principal) más antiguo. Solo ten en cuenta que los sistemas modernos prefieren tablas de particiones GPT y no necesitarías el esquema más antiguo a menos que ya sepas lo que estás haciendo.

**Dónde Vive GPT:**

- **Encabezado GPT primario**: Primer sector del disco (LBA 1)
- **Encabezado GPT de respaldo**: Último sector del disco (LBA -1)
- **Entradas de partición**: Siguiendo el encabezado primario, con respaldos antes del encabezado de respaldo

Esta redundancia significa que si la tabla primaria se corrompe, puedes recuperar desde el respaldo. Genial.

**Características de GPT:**

- Soporta discos mayores a 2TB (a diferencia de MBR)
- Hasta 128 particiones por defecto (expandible)
- Cada partición identificada por un GUID único
- No existe concepto de particiones "primarias" vs "lógicas"—todas las particiones son iguales

### Creando una Tabla de Particiones

Verifica qué hay actualmente en la unidad:

```bash
sudo parted /dev/sda print
```

```plaintext
Error: /dev/sda: unrecognised disk label
Model: JMicron Tech (scsi)
Disk /dev/sda: 2000GB
Sector size (logical/physical): 512B/4096B
Partition Table: unknown
```

Aún no hay tabla de particiones. Creemos una.

```bash
sudo parted /dev/sda mklabel gpt
```

Esto crea una tabla de particiones GPT. Podrías usar `msdos` para MBR si te odias a ti mismo o necesitas compatibilidad con algo antiguo.

## Creando Particiones

Ahora podemos crear particiones reales. Decidí dividir la unidad: 512GB para archivos grandes con los que estoy trabajando actualmente, como mis checkpoints de modelos de IA, el resto para respaldos.

### Entendiendo `mkpart`

```bash
sudo parted /dev/sda mkpart primary ntfs 2s 25%
```

Desglosemos esto:

- `mkpart`: Crea una partición (no la formatea—eso viene después)
- `primary`: Técnicamente sin significado en GPT (término heredado de los días de MBR), pero requerido por la sintaxis de `parted`
- `ntfs`: Establece el GUID de tipo de partición a NTFS—esto es metadata, aún no es un sistema de archivos real
- `2s`: Comenzar en el sector 2 (el sector 1 es el encabezado GPT)
- `25%`: Terminar al 25% del disco

**Tipos de Partición en GPT:**

A diferencia del desastre de "primaria/lógica/extendida" de MBR, GPT usa GUIDs de Tipo de Partición para identificar el uso previsto:

- Partición de Sistema EFI (ESP) para arranque UEFI
- Partición Reservada de Microsoft (MSR) para Windows
- Sistemas de archivos Linux (ext4, btrfs, etc.)
- Windows NTFS
- Swap de Linux
- Y muchos más

El tipo de sistema de archivos que especificas en `mkpart` solo establece este GUID, no crea realmente un sistema de archivos. Pasé al menos 10 minutos preguntándome qué había pasado realmente.

## Formateo: Haciéndola Realmente Usable

Ahora tenemos particiones, pero están vacías. Hora de formatearlas con sistemas de archivos reales.

Opté por NTFS para compatibilidad multiplataforma (tanto Linux como Windows lo manejan bien).

```bash
sudo mkfs.ntfs -f /dev/sda1
```

La bandera `-f` hace un formato rápido (no pone a cero cada byte). Para una unidad nueva, esto está bien.

**Otras opciones de sistema de archivos:**

- `mkfs.ext4` para uso exclusivo en Linux (mejor rendimiento, mejores características)
- `mkfs.btrfs` si quieres snapshots y características avanzadas
- `mkfs.xfs` para archivos grandes y alto rendimiento
- `mkfs.fat -F 32` para máxima compatibilidad pero características limitadas

## Lo Que Aprendí

1. **512e está en todas partes** y probablemente está bien. Perseguir el formateo 4Kn no vale la pena el esfuerzo para la mayoría de los casos de uso, a menos que el firmware de tu unidad ya haya abandonado sus viejas costumbres de 512e.

2. **GPT es simple una vez que lo entiendes**: Encabezados primarios y de respaldo, entradas de partición entre ellos, GUIDs para identificar tipos de partición. Eso es todo.

3. **Las tablas de particiones y los sistemas de archivos son separados**: La tabla de particiones es solo un mapa. Los sistemas de archivos son lo que realmente organiza los datos en esas particiones.

4. **La sintaxis de `parted` es rara**: Hereda terminología de MBR ("primaria") incluso para GPT, donde no tiene significado. No le des muchas vueltas.

5. **NTFS es la elección pragmática** para unidades multiplataforma, aunque ext4 sería más rápido en Linux.

## Comandos Útiles

```bash
# Listar dispositivos de bloque y estado actual
lsblk

# Herramienta interactiva de particionamiento (interfaz de texto)
sudo fdisk /dev/sda

# Herramienta de particionamiento scriptable (la que usé)
sudo parted /dev/sda

# Formatear como NTFS
sudo mkfs.ntfs -f /dev/sda1

# Formatear como ext4
sudo mkfs.ext4 /dev/sda1

# Mostrar información detallada de dispositivos de bloque
sudo blkid

# Montar un sistema de archivos
sudo mount /dev/sda1 /mnt/mydrive
```

## Referencias

Si quieres profundizar más:

- [Especificación NVMe](https://www.nvmexpress.org/specifications) - El estándar NVMe real (para masoquistas)
- [¿Por qué los discos duros todavía usan sectores 512e?](https://superuser.com/questions/1667441/) - Buen hilo de Stack Overflow sobre 512e vs 4Kn
- `man parted` - Documentación realmente útil, sorprendentemente

---

La próxima vez probablemente olvidaré la mitad de esto y tendré que buscarlo de nuevo. Para eso son estas notas y blogposts ;)
