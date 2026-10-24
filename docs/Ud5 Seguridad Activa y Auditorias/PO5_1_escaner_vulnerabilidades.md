# Práctica Opcional 5.1: Auditoría Automatizada de Vulnerabilidades con OpenVAS / Greenbone

!!! note "Datos de la Práctica"
    * **Módulo:** Seguridad y Alta Disponibilidad (SAD - 0378)
    * **Unidad Didáctica:** UD5 - Seguridad Activa y Auditorías en Redes Corporativas
    * **Requisitos:** Una máquina virtual Linux (Debian o Ubuntu) con la suite de auditoría **Greenbone Vulnerability Management (GVM / OpenVAS)** instalada, y una máquina virtual de pruebas como objetivo en una red aislada (GNS3, VirtualBox o KVM).
    * **Privilegios:** Acceso de superusuario (`sudo`) en el servidor auditor.


!!! warning "Practica Opcional"
    Esta práctica se presenta como opcional para aquel alumnado que quiera ir un paso más alla.
    El motivo es que la instalación de Greenbone es intensiva en recursos de almacenamiento(+15GB) y un tiempo alto de descarga.
---

## 1. Objetivos de la Práctica

* Comprender la función de los **escáneres automatizados de vulnerabilidades** en la fase de auditoría técnica y gestión continua del riesgo informático.
* Conocer la arquitectura de la suite **Greenbone / OpenVAS** y el mecanismo de actualización de su base de datos de pruebas (*NVT - Network Vulnerability Tests* / CVE).
* Configurar objetivos de escaneo (*Targets*), credenciales de auditoría y perfiles de análisis (*Scan Configs*).
* Lanzar y monitorizar tareas de escaneo automatizado contra activos de la red local.
* Interpretar y clasificar los hallazgos técnicos según la escala de gravedad estándar **CVSSv3** (*Common Vulnerability Scoring System*).
* Generar y analizar informes técnicos y ejecutivos de auditoría para la propuesta de planes de remediación y parches de seguridad.

---

## 2. Introducción Teórica: Gestión Automatizada de Vulnerabilidades

A diferencia de un escáner de puertos como Nmap (que identifica puertos abiertos pero no evalúa en profundidad la explotabilidad de sus fallos), un **escáner de vulnerabilidades** es un sistema automatizado que interactúa activamente con los servicios remotos probando miles de patrones de vulnerabilidad conocidos (*NVTs* - *Network Vulnerability Tests*).

```
  [ GVM / OPENVAS ] ─── (Sondas de Vulnerabilidad / NVTs) ───> [ SISTEMA OBJETIVO ]
  (Servidor Auditor)                                           (Puertos y Servicios)
         │                                                            │
         ▼                                                            ▼
  [ BASE DE DATOS CVE / NVD ] <─── (Correlación de Fallos) ───────────┘
```

### Arquitectura de Greenbone Community Edition (GVM / OpenVAS)

La solución corporativa Greenbone se estructura en varios componentes modulares:

1. **GSA (*Greenbone Security Assistant*):** Interfaz web de usuario (habitualmente expuesta en el puerto `9392` por HTTPS) desde la cual se gestionan objetivos, tareas, credenciales e informes.
2. **GVMD (*Greenbone Vulnerability Management Daemon*):** Demonio central encargado de orquestar la lógica de negocio, almacenar la configuración en la base de datos PostgreSQL y coordinar las tareas.
3. **OpenVAS Scanner:** Motor de ejecución que lanza las pruebas de seguridad (*scripts NVT* escritos en lenguaje NASL) contra los equipos auditados.
4. **Feed de Vulnerabilidades (*Greenbone Community Feed*):** Repositorio actualizado continuamente con miles de pruebas NVT y referencias directas a los registros internacionales CVE y puntuaciones CVSS.

Aquí puedes acceder a [Greenbone Community Edition – Documentation](https://greenbone.github.io/)

---

## 3. Desarrollo Paso a Paso de la Práctica

### Tarea 1: Instalación de Greenbone Community Edition

En primer lugar necesitas instalar Greenbone Community Edition. Puedes hacerlo de 2 formas, ambas disponibles en [Greenbone Community Edition – Documentation](https://greenbone.github.io/):

* En contenedores Docker - Si tienes linux y dominas docker es la opción recomendada.
* En Kali linux - Si tienes un Kali linux esta opción será más rápida. También puedes usar una máquina virtual Kali Linux (en VirtualBox o GNS3) y realizar ahí la instalación.


!!! warning "Espacio disponible"
    En cualquiera de los casos necesitarás descargar más de 15GB de datos del sistema. Y después sincronizar las bases de datos de vulnerabilidades de varios GB


### Tarea 2: Verificación del Estado del Servicio GVM/OpenVAS y Acceso Web

La verificación será algo parecido a lo siguiente. En cualquier caso, sigue el proceso de verificación indicado en los manuales de instalación.


1. En el servidor Linux donde se encuentra instalado GVM/OpenVAS, verifica que todos los servicios y demonios auxiliares se encuentran operativos:
    ```bash
    sudo gvm-check-setup
    ```
2. Si alguno de los servicios se encuentra detenido, inicia la suite completa:
    ```bash
    sudo gvm-start
    ```
3. Abre un navegador web en el equipo auditor (o desde un cliente con acceso de red al servidor) e ingresa a la consola web de administración:
    ```text
    https://127.0.0.1:9392  (o la IP del servidor: https://192.168.1.100:9392)
    ```
    *(Acepta la advertencia de seguridad del certificado SSL autofirmado).*
4. Autentícate con las credenciales de administrador configuradas durante el despliegue (`admin` y su contraseña asignada).

---

### Tarea 2: Definición del Objetivo de Auditoría (*Target*)

Antes de lanzar un escaneo, es necesario registrar formalmente el equipo o rango de red que será auditado:

1. En el menú superior de GSA, navega a **Configuration → Targets**.
2. Haz clic en el icono azul de **New Target** (esquina superior izquierda).
3. Rellena los datos de la máquina de pruebas objetivo:
    * **Name:** `Objetivo_Laboratorio_SAD`
    * **Hosts:** La dirección IP de tu máquina objetivo (ej. `192.168.1.105`).
    * **Port List:** Selecciona `All IANA assigned TCP` (o `OpenVAS Default`).
    * **Alive Test:** Selecciona `ICMP & TCP-ACK Service Ping`.
4. Haz clic en **Save** para registrar el objetivo en el inventario.

---

### Tarea 3: Configuración y Lanzamiento de la Tarea de Escaneo (*Task*)

1. En el menú superior de navegación, accede a **Scans → Tasks**.
2. Haz clic en el icono de varita mágica o en **New Task**.
3. Configura los parámetros del análisis:
    * **Name:** `Auditoria_Servicios_Laboratorio`
    * **Scan Targets:** Selecciona el objetivo creado en la Tarea 2 (`Objetivo_Laboratorio_SAD`).
    * **Scan Config:** Selecciona **Full and Fast** (perfil recomendado que optimiza la velocidad ejecutando únicamente las pruebas relevantes para los servicios detectados).
4. Haz clic en **Save** para guardar la tarea.
5. En la lista de tareas, localiza la tarea recién creada y pulsa el botón verde **Start** (icono de reproducir).
6. Observa el progreso del escaneo en la columna **Status** (el porcentaje avanzará del 0% al 100%).
    * *Nota:* Este proceso puede demorarse entre 5 y 15 minutos según el número de servicios expuestos en la máquina auditada.

---

### Tarea 4: Análisis de Hallazgos y Clasificación por Severidad CVSS

Una vez completada la tarea (estado `Done`), haz clic sobre el nombre de la tarea para inspeccionar el informe de resultados:

1. Accede a la pestaña **Results**.
2. Observa la barra de severidad que clasifica los hallazgos en cuatro niveles:
    * **High (Alto / Crítico):** Puntuación CVSS 7.0 - 10.0 (vulnerabilidades graves que permiten ejecución remota de código, escalada de privilegios o compromiso total).
    * **Medium (Medio):** Puntuación CVSS 4.0 - 6.9 (fugas de información sensible, denegaciones de servicio parciales o configuraciones débiles).
    * **Low (Bajo):** Puntuación CVSS 0.1 - 3.9 (desconfiguraciones menores o revelación de banners).
    * **Log / Info:** Información descriptiva sobre puertos, protocolos y versiones detectadas.
3. Haz clic sobre una vulnerabilidad clasificada como **High** o **Medium** (por ejemplo, accesos anónimos en FTP, versiones obsoletas de OpenSSH/Apache o suites de cifrado débiles en SSL/TLS).
4. Analiza los apartados de la ficha técnica:
    * **Summary:** Descripción detallada del fallo detectado.
    * **Vulnerability Detection Method:** Evidencia técnica y respuestas recibidas por el motor OpenVAS para confirmar la vulnerabilidad.
    * **Solution:** Recomendación técnica del fabricante para mitigar o solucionar el riesgo (actualización de paquetes, modificación de directivas de configuración, etc.).

---

### Tarea 5: Generación y Exportación del Informe de Auditoría

1. En la vista de resultados de la tarea, haz clic en la pestaña **Reports**.
2. Selecciona el informe correspondiente a la ejecución realizada.
3. En la esquina superior izquierda, despliega el selector de formato de exportación y elige **PDF Executive** o **PDF Report** (o alternativamente HTML).
4. Haz clic en el botón de descarga (icono de flecha hacia abajo).
5. Abre y examina el archivo PDF generado:
    * Identifica el resumen ejecutivo con gráficos de distribución por severidad.
    * Revisa el desglose técnico detallado de cada vulnerabilidad detectada.

---

## 4. Cheatsheet de Comandos y Navegación GVM / OpenVAS

| Acción / Objetivo | Entorno / Herramienta | Sintaxis o Ruta de Navegación |
| :--- | :--- | :--- |
| **Verificar estado de componentes** | Consola Linux | `sudo gvm-check-setup` |
| **Iniciar servicios de GVM** | Consola Linux | `sudo gvm-start` |
| **Detener servicios de GVM** | Consola Linux | `sudo gvm-stop` |
| **Actualizar base de datos NVT/CVE** | Consola Linux | `sudo gvm-feed-update` |
| **Crear nuevo objetivo de red** | Interfaz Web GSA | **Configuration → Targets → New Target** |
| **Crear y ejecutar escaneo** | Interfaz Web GSA | **Scans → Tasks → New Task → Start** |
| **Exportar informe técnico/ejecutivo** | Interfaz Web GSA | **Reports → Seleccionar Report → Download PDF** |

---

## 5. Cuestiones de Autoevaluación

1. **Diferencia Técnica:** ¿En qué se diferencia fundamentalmente una auditoría realizada con un escáner de puertos (como Nmap) frente a una evaluación con un escáner automatizado de vulnerabilidades (como OpenVAS)?
2. **Interpretación CVSS:** Si un informe de OpenVAS califica una vulnerabilidad con una puntuación CVSSv3 de 9.8 en un servidor expuesto a Internet, ¿qué nivel de urgencia tiene su corrección y qué tipo de impacto técnico suele representar?
3. **Escaneo con Credenciales vs. Sin Credenciales:** ¿Qué ventajas técnicas aporta configurar credenciales de acceso administrativo (SSH o SMB) en OpenVAS (*Authenticated Scan*) frente a realizar un escaneo puramente externo sin credenciales?
4. **Mantenimiento del Escáner:** ¿Por qué es crítico ejecutar periódicamente la actualización de feeds (`gvm-feed-update`) en el servidor de auditoría antes de realizar una evaluación de seguridad en una organización?

---

## 6. Recursos y Enlaces de Referencia

* **[Greenbone Community Edition Documentation](https://greenbone.github.io/docs/latest/):** Documentación oficial completa para instalación y administración de GVM.
* **[NIST National Vulnerability Database (NVD)](https://nvd.nist.gov/):** Catálogo oficial de vulnerabilidades y métricas CVSS.
* **[FIRST Common Vulnerability Scoring System (CVSS v3.1)](https://www.first.org/cvss/v3.1/specification-document):** Guía estándar de cálculo y métricas de severidad.
