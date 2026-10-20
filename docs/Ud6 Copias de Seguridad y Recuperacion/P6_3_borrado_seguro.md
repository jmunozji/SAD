# Práctica 6.3: Borrado Seguro de Datos y Sanitización de Soportes

!!! note "Datos de la Práctica"
    * **Módulo:** Seguridad y Alta Disponibilidad (SAD - 0378)
    * **Unidad Didáctica:** UD6 - Copias de Seguridad y Recuperación de la Información
    * **Requisitos:** Máquina virtual Linux (Ubuntu Desktop, Ubuntu MATE o Debian) con privilegios de superusuario (`sudo`).
    * **Soporte de almacenamiento de prueba:** Memoria USB física (recomendado por defecto) formateada en FAT32 o ext4. Como alternativa, puede utilizarse un disco virtual secundario o una imagen *loopback*.
    * **Precaución crítica:** En las prácticas de sanitización se trabaja con comandos destructivos a bajo nivel. **Verifica siempre el identificador de la unidad de prueba (`/dev/sdb`, `/dev/sdc`, etc.) antes de ejecutar cualquier comando para evitar sobrescribir accidentalmente el disco del sistema operativo (`/dev/sda`).**

---

## 1. Objetivos

* Comprender los riesgos de fuga de información confidencial y vulneración del RGPD asociados al desecho, donación o reutilización de soportes de almacenamiento digital.
* Analizar los estándares internacionales de sanitización lógica de medios: **NIST SP 800-88 Rev. 1**, **DoD 5220.22-M** y el algoritmo clásico de **Peter Gutmann**.
* Utilizar herramientas nativas de consola (`shred`) para la destrucción lógica irreversible de archivos individuales mediante múltiples pasadas de sobrescritura.
* Sanitizar el espacio libre no asignado de un sistema de archivos mediante utilidades del paquete `secure-delete` (`sfill`).
* Ejecutar la sanitización integral de una unidad de almacenamiento completa mediante utilidades interactivas especializadas (`nwipe`) y comandos de bajo nivel (`dd`).
* Realizar auditorías forenses posteriores con **TestDisk** y **PhotoRec** para verificar y certificar empíricamente la imposibilidad de rescatar los datos borrados.
* Conocer las particularidades del borrado seguro en unidades de estado sólido (SSD) y chips de memoria Flash NAND frente a discos magnéticos tradicionales (*Wear Leveling*, TRIM y ATA Secure Erase).

---

## 2. Escenario y Fundamentos Técnicos

### La Trampa del Borrado Ordinario

Como se comprobó en la práctica anterior, el borrado estándar mediante el explorador de archivos o la instrucción `rm` se limita a desvincular los punteros de los directorios o marcar los sectores como "libres". La información binaria sigue existiendo intacta en los platos magnéticos o celdas de memoria hasta que otro fichero ocupe físicamente ese mismo espacio.

Para garantizar la confidencialidad, la seguridad informática recurre a la **sanitización lógica por sobrescritura (*Data Overwrite*)**, que consiste en escribir nuevos patrones de bits directamente sobre los sectores que ocupaba la información sensible.

```text
  [ Archivo Confidencial ] ──> [ Sobrescritura con shred / nwipe ] ──> [ Sectores con Ruido / Ceros ]
                                                                                   │
                                                                   ¿Es recuperable?
                                                                                   ▼
                                                                  [ Auditoría TestDisk / PhotoRec ]
                                                                  - Metadatos destruidos
                                                                  - Cero firmas legibles
                                                                  ✔ Sanitización CERTIFICADA
```

### Estándares Internacionales de Sanitización

| Estándar / Algoritmo | Procedimiento de Sobrescritura | Ámbito y Observaciones |
| :--- | :--- | :--- |
| **DoD 5220.22-M** | **3 pasadas:** 1ª pasada con ceros (`0x00`), 2ª pasada con unos (`0xFF`), y 3ª pasada con un valor pseudoaleatorio, verificando la última pasada. | Estándar militar histórico del Departamento de Defensa de EE.UU. Muy extendido en auditorías corporativas. |
| **Algoritmo Gutmann** | **35 pasadas:** Combina secuencias pseudoaleatorias con patrones específicos adaptados a la codificación magnética MFM y RLL. | Creado en 1996 por Peter Gutmann. Considerado hoy en día sobredimensionado e innecesario para discos modernos de alta densidad. |
| **NIST SP 800-88 Rev. 1** | Define tres niveles de tratamiento: **Clear** (sobrescritura lógica de sectores de usuario), **Purge** (sanitización a nivel de firmware/criptográfico) y **Destroy** (destrucción física). | El estándar de referencia actual en la industria y en normativas de protección de datos (RGPD / ENS). |

### Particularidades en Discos de Estado Sólido (SSD) y Memorias Flash

A diferencia de los discos magnéticos HDD (donde un sector lógico `LBA` se corresponde con una posición física fija en el plato), en los discos SSD el controlador interno aplica una capa intermedia llamada **FTL (*Flash Translation Layer*)** con algoritmos de **nivelación de desgaste (*Wear Leveling*)**:

* Cuando el sistema operativo ordena sobrescribir un bloque lógico con `shred`, el controlador del SSD redirige la escritura a un bloque físico nuevo menos desgastado y marca el bloque físico original como "invalido", pero los datos antiguos pueden permanecer legibles en la memoria NAND hasta que el proceso de recolección de basura (*Garbage Collection* / TRIM) los limpie físicamente.
* Por este motivo, la sanitización eficaz de SSDs no se realiza mediante herramientas de sobrescritura de sectores como `shred`, sino ejecutando las instrucciones nativas del firmware del disco: **ATA Secure Erase** (en unidades SATA mediante `hdparm`) o **NVMe Format / Sanitize** (en discos M.2 NVMe mediante `nvme-cli`), las cuales aplican una descarga de voltaje que resetea simultáneamente todas las celdas NAND o destruyen la clave de cifrado hardware del disco (*Cryptographic Erase*).

---

## 3. Preparación del Laboratorio

### Conexión del Soporte de Prueba

Utilizaremos un pendrive USB físico (o alternativamente un disco virtual secundario):

1. Conecta tu memoria USB al ordenador anfitrión.
2. En VirtualBox, accede al menú **Dispositivos → Dispositivos USB** y captura tu pendrive hacia la máquina virtual.
3. Abre una terminal y confirma el identificador asignado al dispositivo:

    ```bash
    lsblk -f
    ```

    *Identifica claramente tu dispositivo (por ejemplo, `/dev/sdb` con su partición `/dev/sdb1`).*

### Instalación de las Herramientas Forenses y de Sanitización

Instala los paquetes necesarios en el sistema Linux invitado:

```bash
sudo apt update
sudo apt install coreutils nwipe secure-delete testdisk -y
```

* `coreutils`: Proporciona las utilidades nativas `shred` y `dd`.
* `nwipe`: Herramienta de sanitización completa de discos basada en DBAN (*Darik's Boot and Nuke*).
* `secure-delete`: Colección de utilidades de borrado seguro avanzado (`srm`, `sfill`, `sdmem`, `sswap`).
* `testdisk`: Utilidad forense que utilizaremos para auditar la eficacia del borrado.

---

## 4. Desarrollo Paso a Paso de la Práctica

---

### Fase 1: Creación de Archivos Confidenciales de Prueba

1. Monta la partición de tu pendrive en un directorio temporal:

    ```bash
    sudo mkdir -p /mnt/pendrive
    sudo mount /dev/sdb1 /mnt/pendrive
    ```

2. Genera dentro del pendrive varios ficheros de prueba con información que simule datos corporativos confidenciales:

    ```bash
    cd /mnt/pendrive
    sudo bash -c 'echo "Claves de Acceso Root Servidores Produccion 2026: P@ssw0rdSecure!" > passwords_servidores.txt'
    sudo bash -c 'echo "Datos Contables y Cuentas Bancarias IBAN ES9121000418450200051332" > contabilidad_bancaria.txt'
    sudo bash -c 'echo "Registro de Auditoria de Empleados y Salarios" > nominas_personal.txt'
    
    # Descargar un archivo de imagen real para comprobar su cabecera binaria
    sudo wget -q https://upload.wikimedia.org/wikipedia/commons/thumb/3/35/Tux.svg/200px-Tux.svg.png -O logotipo_confidencial.png
    ```

3. Comprueba el listado de archivos creados y calcula sus resúmenes hash:

    ```bash
    ls -la /mnt/pendrive
    md5sum /mnt/pendrive/*.txt
    ```

4. Abre el gestor de archivos gráfico del sistema (Caja / Nautilus) y accede a la memoria USB para ver visualmente la presencia de los ficheros.

---

### Fase 2: Borrado Seguro de Archivos Individuales con `shred`

El comando `shred` sobrescribe repetidamente los bloques de datos asignados a los archivos especificados mediante patrones aleatorios criptográficos.

#### Parámetros Principales de `shred`:

* **`-n N`:** Número de pasadas de sobrescritura con datos aleatorios (por defecto son 3).
* **`-z`:** Pasada final con ceros (`0x00`) para enmascarar el proceso y ocultar que el fichero fue triturado.
* **`-u`:** Trunca y desvincula (elimina) el archivo del sistema de ficheros tras finalizar la sobrescritura.
* **`-v`:** Modo detallado (*verbose*), muestra en pantalla el progreso de cada pasada.

1. Ejecuta el borrado seguro de dos de los archivos confidenciales aplicando **3 pasadas aleatorias y una pasada final con ceros**:

    ```bash
    sudo shred -u -z -v -n 3 /mnt/pendrive/passwords_servidores.txt /mnt/pendrive/contabilidad_bancaria.txt
    ```

2. Observa cómo `shred` renombra temporalmente el archivo, sobrescribe sus sectores en cada pasada (`pass 1/4 (random)`, `pass 2/4 (random)`, `pass 3/4 (random)`, `pass 4/4 (000000)`) y finalmente lo elimina.

3. **Toma una captura de pantalla de la terminal mostrando la ejecución de `shred` y el progreso detallado de sus 4 pasadas.**

4. Comprueba en el explorador de archivos que los dos ficheros han desaparecido de la lista.

---

### Fase 3: Auditoría Forense tras la Ejecución de `shred`

¿Qué ha sucedido realmente con los archivos triturados? Vamos a comprobarlo mediante **TestDisk**:

1. Desmonta la unidad de prueba para realizar el análisis en modo solo lectura:

    ```bash
    sudo umount /mnt/pendrive
    ```

2. Inicia TestDisk:

    ```bash
    sudo testdisk
    ```

3. Selecciona `[ Create ]` → Elige el disco del pendrive (`/dev/sdb`) → Tipo `[ Intel ]` → `[ Advanced ] Filesystem Utils` → Selecciona la partición y entra en `[ Undelete ]`.

4. **Análisis de los resultados:**
    * Comprueba si TestDisk detecta las entradas de directorio correspondientes a los ficheros borrados (aparecerán en color rojo).
    * Marca uno de los archivos triturados con la tecla `:` y pulsa la tecla **`C`** para copiarlo a tu carpeta de usuario (ejemplo: `/home/usuario/recuperados_shred/`).
    * Sal de TestDisk y abre el archivo recuperado mediante un visor hexadecimal o la terminal:

    ```bash
    cat /home/usuario/recuperados_shred/passwords_servidores.txt
    xxd /home/usuario/recuperados_shred/passwords_servidores.txt | head -n 10
    ```

5. **Toma una captura de pantalla del visor o del comando `xxd`/`cat` donde se demuestre que el contenido original es completamente irrecuperable (aparecerán únicamente ceros `00` o secuencias de ruido aleatorio).**

---

### Fase 4: Sanitización de Espacio Libre con `sfill`

Cuando en un soporte se han eliminado archivos en el pasado mediante el comando `rm` estándar, sus datos binarios continúan residiendo en los clústeres no asignados del disco. La herramienta `sfill` permite sanitizar únicamente el espacio libre sin destruir los archivos legítimos activos:

1. Monta de nuevo la unidad:

    ```bash
    sudo mount /dev/sdb1 /mnt/pendrive
    ```

2. Ejecuta `sfill` en modo rápido sobre el punto de montaje:

    ```bash
    sudo sfill -v -z -l /mnt/pendrive
    ```

    * `-v`: Muestra el progreso en tiempo real.
    * `-z`: Limpieza final con ceros.
    * `-l`: Modo rápido (dos pasadas: una aleatoria y una de ceros), ideal para unidades de prueba.

3. Observa cómo `sfill` crea un archivo temporal que ocupa todo el espacio libre de la unidad, lo sobrescribe con patrones criptográficos y posteriormente lo elimina, dejando todos los sectores vacíos del disco completamente limpios.

---

### Fase 5: Sanitización Completa del Soporte con `nwipe`

En ocasiones se requiere sanitizar un disco entero (por ejemplo, antes de donar o retirar un equipo de la empresa). Para esta tarea utilizaremos **`nwipe`**, la implementación de código abierto del mítico software DBAN (*Darik's Boot and Nuke*):

1. Asegúrate de que la unidad está desmontada:

    ```bash
    sudo umount /mnt/pendrive
    ```

2. Ejecuta `nwipe` en la consola:

    ```bash
    sudo nwipe
    ```

3. **Navegación e interfaz de `nwipe`:**
    * Se abrirá una interfaz interactiva de consola ncurses que listará todos los discos del equipo.
    * Desplázate con las flechas del teclado hasta situarte sobre el disco del pendrive (ejemplo: `sdb`).
    * Pulsa la **barra espaciadora** para marcar la unidad con un asterisco `[*]`.
    * **¡ATENCIÓN CRÍTICA!**: Asegúrate de que **SOLO** está marcado el pendrive (`sdb`) y que el disco del sistema (`sda`) está desmarcado `[ ]`.

4. **Selección del método de borrado:**
    * Pulsa la tecla **`M`** (*Method*) para abrir el catálogo de algoritmos de sanitización:
        * *Quick Erase (1 pasada con ceros)*
        * *DoD Short (3 pasadas)*
        * *DoD 5220.22-M (7 pasadas)*
        * *Gutmann (35 pasadas)*
        * *PRNG Stream (sobrescritura pseudoaleatoria)*
    * Selecciona **`Quick Erase`** (o `DoD Short`) para realizar la prueba en el tiempo del laboratorio. Pulsa `Enter`.

5. **Ejecución del borrado masivo:**
    * Pulsa la tecla **`S`** (*Start*) para iniciar el proceso de destrucción de datos.
    * La aplicación mostrará una barra de progreso en tiempo real indicando la velocidad en MB/s, el porcentaje completado y el estado de los sectores.

6. **Toma una captura de pantalla durante o al finalizar el proceso de sanitización con `nwipe` donde se visualice el estado `SUCCESS / Finished`.**

7. Sal de `nwipe` pulsando la tecla `Enter` cuando finalice.

---

### Fase 6: Auditoría Forense Final con TestDisk y PhotoRec

1. Ejecuta nuevamente TestDisk sobre el disco completo:

    ```bash
    sudo testdisk
    ```

2. Selecciona `[ Create ]` → Elige el disco del pendrive (`/dev/sdb`).
3. Comprueba el diagnóstico que emite la herramienta:
    * TestDisk indicará que **no reconoce ninguna tabla de particiones** o que el disco aparece como espacio sin particionar (`None / Unknown partition table`).
4. Ejecuta PhotoRec sobre la unidad completa para intentar encontrar firmas de cabecera (*file carving*):

    ```bash
    sudo photorec /dev/sdb
    ```

5. Comprueba que PhotoRec finaliza el escaneo sin encontrar ningún archivo (0 archivos recuperados).
6. **Toma una captura de pantalla demostrando que las herramientas forenses no localizan ningún tipo de dato ni partición tras la sanitización.**

---

## 5. Análisis Técnico y Cuestiones

Responde de forma justificada a las siguientes cuestiones en tu memoria de prácticas:

1. **Comparativa `rm` vs. `shred`:**
    * Explica por qué un archivo eliminado con `rm` puede recuperarse intacto mediante herramientas como TestDisk, mientras que tras aplicar `shred` su recuperación es imposible.
    * ¿Qué función desempeña el parámetro `-z` en `shred` y por qué es una buena práctica de seguridad utilizarlo?
2. **Estándares de Sanitización:**
    * ¿En qué se diferencian el método **DoD 5220.22-M** y el método **Gutmann**?
    * ¿Por qué el método Gutmann de 35 pasadas se considera hoy en día obsoleto o innecesario en discos duros modernos de alta densidad? ¿Cuántas pasadas recomienda la normativa actual (NIST SP 800-88)?
3. **El Problema del Borrado en Discos de Estado Sólido (SSD):**
    * Explica por qué ejecutar herramientas de sobrescritura de bloques como `shred` o `dd` no garantiza la destrucción total de los datos en un SSD debido al *Wear Leveling* y la capa FTL.
    * ¿Qué comando o mecanismo oficial del fabricante debe emplearse para sanitizar de forma segura y certificada una unidad SSD SATA o NVMe?
4. **Valoración de la Regla de Destrucción:**
    * Si una empresa almacena datos bancarios y de salud de clientes, y un disco duro magnético sufre una avería mecánica que impide que el motor gire (haciendo imposible la sobrescritura por software con `nwipe`), ¿qué método físico de destrucción debe aplicarse para cumplir con el RGPD y el Esquema Nacional de Seguridad (ENS)?

---

## 6. Entregables

Elabora un único documento en formato **PDF** que incluya:

1. **Datos identificativos:** Nombre del alumno, fecha y módulo (SAD).
2. **Captura 1 (Ejecución de `shred`):** Salida detallada en terminal mostrando las 4 pasadas de sobrescritura del comando `shred -u -z -v -n 3`.
3. **Captura 2 (Auditoría de TestDisk tras `shred`):** Visualización del archivo recuperado con `cat` o `xxd` demostrando que el contenido original ha sido sustituido por ceros.
4. **Captura 3 (Ejecución de `nwipe`):** Pantalla de `nwipe` completada con éxito mostrando el método aplicado y la velocidad de borrado.
5. **Captura 4 (Auditoría final):** Salida de TestDisk o PhotoRec demostrando la ausencia total de datos o particiones recuperables.
6. **Respuestas argumentadas:** Resolución justificada de las cuatro cuestiones del apartado 5.
