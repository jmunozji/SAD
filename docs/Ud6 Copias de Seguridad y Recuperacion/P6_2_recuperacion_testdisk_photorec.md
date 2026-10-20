# Práctica 6.2: Recuperación de Archivos y Particiones con TestDisk y PhotoRec

!!! note "Datos de la Práctica"
    * **Módulo:** Seguridad y Alta Disponibilidad (SAD - 0378)
    * **Unidad Didáctica:** UD6 - Copias de Seguridad y Recuperación de la Información
    * **Requisitos:** Máquina virtual Linux (Ubuntu Desktop, Ubuntu MATE o Debian) con privilegios de superusuario (`sudo`).
    * **Soporte de almacenamiento de prueba:** Memoria USB física (opción por defecto recomendada) con una partición formateada en FAT32. Como alternativa en caso de no disponer de pendrive, puede utilizarse el disco virtual preconfigurado [`HD64MB.vdi`](recursos/HD64MB.vdi) o un archivo de imagen *loopback*.

---

## 1. Objetivos

* Comprender el funcionamiento interno del borrado lógico de ficheros en sistemas de archivos magnéticos y flash (FAT32, ext4).
* Diferenciar conceptual y operativamente los dos enfoques principales de recuperación de datos:
    * **Recuperación basada en metadatos y tabla de particiones:** Empleando **TestDisk**.
    * **Tallado binario de archivos (*File Carving*) por firmas de cabecera y pie:** Empleando **PhotoRec**.
* Montar, particionar y manipular unidades de almacenamiento secundarias aisladas en entornos de virtualización.
* Realizar el análisis forense y rescate de archivos eliminados intencionadamente desde la consola interactiva de TestDisk preservando la estructura de nombres original.
* Ejecutar un escaneado en bruto (*raw carving*) de sectores no asignados mediante PhotoRec y analizar su impacto sobre los nombres y metadatos de los archivos recuperados.
* Evaluar y aplicar la **regla de oro de la informática forense**: *nunca escribir ni guardar los datos recuperados en el mismo soporte que se está investigando*.

---

## 2. Escenario y Fundamentos Técnicos

### Mecánica del Borrado Lógico vs. Destrucción Física

Cuando un usuario o sistema operativo elimina un archivo de forma ordinaria (por ejemplo, vaciando la papelera o ejecutando `rm`), **la información no se destruye físicamente de los sectores de almacenamiento**:

* **En el sistema de archivos FAT32:** El sistema operativo localiza la entrada del archivo en la tabla de directorios y sustituye únicamente el primer byte de su nombre por el valor hexadecimal `0xE5` (marca de entrada borrada). A continuación, en la tabla de asignación FAT, marca los clústeres ocupados por el archivo como "libres" (`0x00000000`), indicando que pueden reutilizarse en futuras escrituras. Sin embargo, los datos binarios originales permanecen intactos en los clústeres del disco hasta que sean sobrescritos por nuevos ficheros.
* **En el sistema de archivos ext4:** El inodo del archivo se desvincula del directorio, se pone a cero su contador de enlaces (*hard links*) y los bloques de datos se marcan como disponibles en el mapa de bits del sistema de ficheros.

### Dos Enfoques Forenses: TestDisk frente a PhotoRec

Aunque ambas herramientas forman parte de la misma suite desarrollada por Christophe Grenier, su metodología de recuperación es radicalmente diferente:

```text
               ┌────────────────────────────────────────────────────────┐
               │          UNIDAD DE ALMACENAMIENTO DE PRUEBA            │
               │   [Tabla Particiones] [Entrada Directorio] [Clústeres] │
               └───────────────────────────┬────────────────────────────┘
                                           │
                 ┌─────────────────────────┴─────────────────────────┐
                 │                                                   │
                 ▼                                                   ▼
       ┌──────────────────┐                                ┌──────────────────┐
       │     TestDisk     │                                │     PhotoRec     │
       └─────────┬────────┘                                └─────────┬────────┘
                 │ Lee metadatos y entradas borradas                 │ Escaneo en bruto (Carving)
                 │ Busca la marca '0xE5'                             │ Ignora el sistema de ficheros
                 │ Reconstruye la cadena de clústeres                │ Busca Magic Numbers (Cabecera/Pie)
                 ▼                                                   ▼
       ✔ Recupera nombre original                          ✘ Pierde el nombre original
       ✔ Mantiene la ruta de carpetas                      ✘ Pierde jerarquía (recup_dir.*)
       ✔ Conserva fechas originales                        ✔ Funciona aun con filesystem destruido
```

| Característica | TestDisk | PhotoRec |
| :--- | :--- | :--- |
| **Enfoque técnico** | Análisis de metadatos, MBR/GPT y entradas de directorio borradas (*Undelete*). | Tallado binario de archivos (*File Carving*) por firmas de cabecera (*Magic Numbers*). |
| **Dependencia del Sistema de Ficheros** | **Alta:** Requiere que las estructuras del sistema de archivos no estén completamente corruptas o sobrescritas. | **Nula:** Ignora el sistema de archivos; funciona sobre particiones dañadas, formateadas o sin formato. |
| **Nombres de archivo recuperados** | **Sí:** Conserva el nombre original y extensión del fichero. | **No:** Asigna nombres secuenciales automáticos basados en la ubicación del sector (ej. `f0123456.jpg`). |
| **Estructura de directorios** | **Sí:** Permite reconstruir la jerarquía original de carpetas. | **No:** Agrupa los ficheros encontrados en carpetas genéricas (`recup_dir.1/`, `recup_dir.2/`). |
| **Manejo de fragmentación** | Alto (si la cadena de clústeres no está sobreescrita). | Limitado (asume clústeres contiguos si no hay metadatos). |

---

## 3. Preparación del Soporte de Prueba

Para realizar las operaciones de análisis y rescate forense sin poner en peligro el disco del sistema operativo, utilizaremos una unidad de almacenamiento externa o secundaria. La opción principal y recomendada es utilizar una memoria USB real:

### Opción A (Por defecto y recomendada): Memoria USB Física (Pendrive)

Esta es la modalidad preferida de trabajo, idéntica a una intervención de campo real:

1. Conecta un pendrive USB a tu ordenador anfitrión. Debe contener al menos una partición formateada en **FAT32** (o puedes formatearlo desde la propia máquina).
2. Asegúrate de tener instalado el paquete **VirtualBox Extension Pack** en el anfitrión para permitir el soporte de controladores USB 2.0 / USB 3.0.
3. Con la máquina virtual Linux en ejecución, accede a la barra de menú superior de VirtualBox:
    * Selecciona **Dispositivos → Dispositivos USB** y haz clic sobre el nombre de tu pendrive para capturarlo y redirigirlo al sistema invitado.
    * *(Opcional)*: También puedes definir un filtro permanente en **Configuración → USB → Añadir filtro a partir de dispositivo**.
4. En el terminal de la máquina virtual Linux, comprueba que el pendrive ha sido detectado correctamente:

    ```bash
    lsblk -f
    ```

    *Identifica el identificador asignado por el kernel (por ejemplo, `/dev/sdb` y su partición `/dev/sdb1`).*

---

### Opción B (Alternativa de laboratorio): Disco Virtual Preconfigurado `HD64MB.vdi`

Si no dispones de un pendrive físico en el aula o estás realizando la práctica en un entorno virtualizado sin acceso a puertos USB:

1. Descarga el archivo de disco virtual: **[HD64MB.vdi](recursos/HD64MB.vdi)** (4 MB, particionado en FAT32).
2. Con la máquina virtual **apagada**, accede en VirtualBox a **Configuración → Almacenamiento**.
3. En el **Controlador SATA** (o IDE), pulsa el icono de **"Añadir disco duro"** → **Añadir** → Selecciona el fichero descargado `HD64MB.vdi`.
4. Inicia la máquina virtual. El disco aparecerá reconocido en el sistema (habitualmente como `/dev/sdb`).

---

### Opción C (Alternativa por comandos): Creación de Disco Virtual mediante Loopback en Linux

Si deseas generar un soporte de pruebas temporal directamente desde la línea de comandos sin necesidad de apagar la máquina virtual ni conectar dispositivos físicos:

```bash
# Crear un fichero imagen vacío de 64 MB
dd if=/dev/zero of=~/disco_prueba.img bs=1M count=64

# Asociar la imagen a un dispositivo loop
sudo losetup -fP ~/disco_prueba.img

# Comprobar el dispositivo asignado (ejemplo: /dev/loop0)
losetup -a

# Crear una tabla de particiones MBR y una partición FAT32
sudo parted -s /dev/loop0 mklabel msdos
sudo parted -s /dev/loop0 mkpart primary fat32 1MiB 100%

# Formatear la partición en FAT32
sudo mkfs.vfat -F 32 /dev/loop0p1
```

---

## 4. Desarrollo Paso a Paso de la Práctica

---

### Fase 1: Instalación de Herramientas y Preparación de Datos

1. Inicia sesión en la máquina virtual Linux y abre un terminal.

2. Instala el paquete que incluye la suite forense:

    ```bash
    sudo apt update
    sudo apt install testdisk -y
    ```

    *El paquete `testdisk` instala de forma conjunta las utilidades de consola `testdisk` y `photorec`.*

3. Identifica el identificador de tu disco de prueba:

    ```bash
    lsblk -f
    ```

    *(Identifica la partición FAT32 de prueba, por ejemplo `/dev/sdb1` o `/dev/loop0p1`).*

4. Monta la partición en un punto de montaje temporal (por ejemplo `/mnt/pendrive`):

    ```bash
    sudo mkdir -p /mnt/pendrive
    sudo mount /dev/sdb1 /mnt/pendrive
    ```

5. **Generación de archivos de prueba:** Copia o genera dentro de la unidad de prueba al menos 4 ficheros de distintos tipos y extensiones (documentos de texto, imágenes y PDFs):

    ```bash
    cd /mnt/pendrive
    sudo bash -c 'echo "Informe de Auditoria de Redes 2026" > informe_auditoria.txt'
    sudo bash -c 'echo "Claves de Recuperacion de Cifrado" > claves_seguridad.txt'
    sudo bash -c 'echo "Registro de Eventos y Accesos del Servidor" > log_servidor.txt'
    
    # Descargar o copiar una imagen real para comprobar el file carving
    sudo wget -q https://upload.wikimedia.org/wikipedia/commons/thumb/3/35/Tux.svg/200px-Tux.svg.png -O logotipo_tux.png
    ```

6. Comprueba el listado de archivos en la unidad:

    ```bash
    ls -la /mnt/pendrive
    ```

7. **Simulación del desastre (Borrado de ficheros):** Elimina permanentemente 3 de los ficheros creados para simular una pérdida de información:

    ```bash
    sudo rm /mnt/pendrive/informe_auditoria.txt
    sudo rm /mnt/pendrive/claves_seguridad.txt
    sudo rm /mnt/pendrive/logotipo_tux.png
    ```

8. Comprueba que en `/mnt/pendrive` solo queda el archivo no eliminado.

9. **Desmontaje preventivo (Buena práctica forense):** Desmonta la unidad para evitar que el sistema operativo escriba escrituras accidentales en la partición durante el análisis:

    ```bash
    sudo umount /mnt/pendrive
    ```

10. **Creación del directorio de destino seguro:** Crea en tu carpeta personal los directorios donde almacenarás los archivos recuperados:

    ```bash
    mkdir -p ~/recuperados/testdisk ~/recuperados/photorec
    ```

---

### Fase 2: Recuperación con TestDisk

1. Inicia la herramienta con privilegios de administrador:

    ```bash
    sudo testdisk
    ```

2. **Creación del fichero de registro:**
    * En la primera pantalla, selecciona **`[ Create ]`** para registrar las operaciones en `testdisk.log`. Pulsa `Enter`.

3. **Selección del medio de almacenamiento:**
    * En la lista de discos detectados, desplázate con las flechas del teclado hasta seleccionar tu disco de prueba (ejemplo: `/dev/sdb` o `/dev/loop0`).
    * Selecciona **`[ Proceed ]`** y pulsa `Enter`.

4. **Tipo de tabla de particiones:**
    * TestDisk detectará automáticamente el tipo de tabla. Para discos convencionales y FAT32 suele ser **`[ Intel ]`** (Intel/PC partition). Pulsa `Enter`.

5. **Acceso a utilidades avanzadas del sistema de ficheros:**
    * En el menú principal, baja hasta la opción **`[ Advanced ] Filesystem Utils`** y pulsa `Enter`.

6. **Selección de la partición y opción Undelete:**
    * Aparecerá la lista de particiones del disco. Sitúate sobre la partición FAT32.
    * En el menú inferior, desplázate hacia la derecha y selecciona **`[ Undelete ]`**. Pulsa `Enter`.

7. **Exploración de entradas eliminadas:**
    * Aparecerá el explorador de archivos interno de TestDisk.
    * Los archivos activos aparecerán en blanco, mientras que los **archivos borrados aparecerán resaltados en color rojo** (con su nombre y extensión originales legibles).
    * **Toma una captura de pantalla de esta ventana donde se aprecien los archivos borrados en rojo.**

8. **Selección y recuperación de archivos:**
    * Utiliza la tecla **`:`** para marcar individualmente los archivos que deseas recuperar (cambiarán a color verde). También puedes pulsar **`a`** para seleccionarlos todos.
    * Una vez marcados, pulsa la tecla **`C`** (C mayúscula) para iniciar la copia.
    * TestDisk te pedirá seleccionar el directorio de destino. Navega por el árbol de directorios hasta situarte dentro de `/home/tu_usuario/recuperados/testdisk/`.
    * Cuando estés dentro de la carpeta deseada, pulsa la tecla **`C`** para confirmar la copia.
    * En la parte superior de la ventana aparecerá el mensaje en verde: `Copy done!`.

9. **Verificación en el sistema:**
    * Sal de TestDisk seleccionando `Quit` sucesivamente hasta volver a la consola de comandos.
    * Comprueba el contenido del directorio de rescate:

    ```bash
    ls -la ~/recuperados/testdisk/
    cat ~/recuperados/testdisk/informe_auditoria.txt
    ```

    * **Toma una captura de pantalla del listado de archivos recuperados y su contenido.**

---

### Fase 3: Recuperación con PhotoRec (Tallado Binario / File Carving)

1. Ejecuta la herramienta de tallado forense con privilegios de superusuario:

    ```bash
    sudo photorec
    ```

2. **Selección del soporte de almacenamiento:**
    * Selecciona la misma unidad de prueba (`/dev/sdb` o `/dev/loop0`) y pulsa **`[ Proceed ]`**.

3. **Tipo de tabla de particiones:**
    * Confirma el tipo detectado (**`[ Intel ]`**).

4. **Selección de partición y opciones de búsqueda:**
    * Sitúate sobre la partición de prueba.
    * En el menú inferior, puedes revisar las opciones y los tipos de archivo reconocidos con **`[ File Opt ]`** (verás cientos de extensiones soportadas por defecto: jpg, png, pdf, txt, zip...).
    * Selecciona **`[ Search ]`** y pulsa `Enter`.

5. **Tipo de sistema de archivos:**
    * PhotoRec te preguntará sobre el sistema de ficheros para optimizar la búsqueda de bloques:
        * Para ext2/ext3/ext4 selecciona *ext2/ext3*.
        * Para FAT, NTFS, exFAT o HFS+, selecciona **`[ Other ]`**. En nuestro caso, elige **`[ Other ]`** y pulsa `Enter`.

6. **Espacio a analizar:**
    * **`[ Free ]`**: Escanea únicamente el espacio no asignado (los clústeres marcados como libres donde estaban los ficheros borrados).
    * **`[ Whole ]`**: Analiza todos los sectores del disco sin excepción.
    * Selecciona **`[ Free ]`** (o `[ Whole ]`) y pulsa `Enter`.

7. **Directorio de salida:**
    * Navega por el sistema de ficheros hasta seleccionar la carpeta de destino: `/home/tu_usuario/recuperados/photorec/`.
    * Pulsa **`C`** para autorizar la extracción.

8. **Proceso de escaneado y extracción:**
    * PhotoRec comenzará el análisis sectorial en tiempo real, mostrando contadores de sectores escaneados y ficheros encontrados desglosados por tipo.
    * **Toma una captura de pantalla durante o al finalizar el proceso de extracción de PhotoRec.**

9. **Auditoría de los ficheros recuperados:**
    * Finalizada la tarea, pulsa `Enter` para salir de PhotoRec.
    * Inspecciona la estructura de carpetas y ficheros generada en el directorio de salida:

    ```bash
    ls -la ~/recuperados/photorec/
    ls -la ~/recuperados/photorec/recup_dir.1/
    ```

    * **Toma una captura de pantalla del contenido de la subcarpeta `recup_dir.1` donde se vean los nombres asignados a los archivos.**

---

## 5. Análisis Técnico y Cuestiones

Responde de forma justificada y detallada a las siguientes cuestiones en tu memoria de prácticas:

1. **Comparativa de Resultados:**
    * ¿Se han recuperado exactamente los mismos archivos con ambas herramientas?
    * ¿Los archivos recuperados por **TestDisk** conservan su nombre y extensión originales? ¿Y los recuperados por **PhotoRec**? Explica técnicamente por qué se produce esta diferencia según los metadatos.
2. **Organización del Almacenamiento:**
    * ¿Se guardan los ficheros recuperados por TestDisk directamente en la carpeta seleccionada o se crean subcarpetas?
    * ¿Cómo organiza PhotoRec los ficheros recuperados? ¿Por qué utiliza carpetas del tipo `recup_dir.1/`, `recup_dir.2/` y nombres del tipo `f0123456.txt`?
3. **Mecánica del File Carving:**
    * ¿En qué consiste un *Magic Number* o firma de cabecera de un archivo? Pon como ejemplo los números mágicos de un archivo JPEG (`FF D8 FF`) o PDF (`%PDF` / `25 50 44 46`).
    * ¿Qué sucedería si un archivo estuviese altamente fragmentado en el disco al intentar recuperarlo con PhotoRec? ¿Por qué la fragmentación es el mayor obstáculo del *file carving*?
4. **La Regla de Oro Forense:**
    * ¿Por qué es un error metodológico crítico guardar los archivos recuperados dentro de la misma partición o disco que se está investigando? ¿Qué consecuencias inmediatas tendría para el resto de archivos pendientes de rescatar?

---

## 6. Entregables

Elabora un único informe en formato **PDF** que contenga:

1. **Datos identificativos:** Nombre del alumno, fecha y módulo (SAD).
2. **Captura 1 (Exploración en TestDisk):** Ventana de `[ Undelete ]` donde se visualicen los ficheros eliminados resaltados en rojo.
3. **Captura 2 (Resultado de TestDisk):** Salida del comando `ls -la ~/recuperados/testdisk/` evidenciando que los archivos recuperados conservan sus nombres originales.
4. **Captura 3 (Ejecución de PhotoRec):** Pantalla de escaneado y extracción de PhotoRec mostrando el recuento de archivos clasificados.
5. **Captura 4 (Resultado de PhotoRec):** Salida del comando `ls -la ~/recuperados/photorec/recup_dir.1/` donde se aprecie la nomenclatura genérica secuencial generada.
6. **Respuestas argumentadas:** Resolución completa de las cuatro cuestiones del apartado 5.
