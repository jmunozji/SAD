# Práctica 5.3: Auditoría Automatizada de Vulnerabilidades con OpenVAS / Greenbone

!!! note "Datos de la Práctica"
    * *Módulo:* Seguridad y Alta Disponibilidad (SAD - 0378)
    * *Unidad Did3ctica:* UD5 - Seguridad Activa y Auditorías en Redes Corporativas
    * *Requisitos:* Una máquina virtual Linux (Debian o Ubuntu) con la suite de auditoría *Greenbone Vulnerability Management (GVM / OpenVAS)* instalada, y una máquina virtual de pruebas como objetivo en una red aislada (GNS3, VirtualBox o KVM). Privilegios `sudo`.

---

## 1. Objetivos de la Práctica

* Comprender la función de los *escáneres automatizados de vulnerabilidades* en la fase de auditoría técnica y gestión del riesgo informático.
* Conocer la arquitectura de la suite *Greenbone / OpenVAS* y la actualización de su base de datos de pruebas (*NVT - Network Vulnerability Tests* / CVE).
* Configurar objetivos de escaneo (*Targets*), credenciales de auditoria y perfiles de análisis (*Scan Configs*).
*Lanzar y monitorizar tareas de escaneo automatizado contra activos de la red local.
* Interpretar y clasificar los hallazgos según la escala de gravedad estándar *CVSSv3* (*Common Vulnerability Scoring System*).
* Generar y analizar informes técnicos y ejecutivos de auditoría para la propuesta de medidas correctoras y parches de seguridad.

---

## 2. Introducción Teórica: Gestión Automatizada de Vulnerabilidades

A diferencia de un escáner de puertos como Nmap (que identifica servicios abiertos pero no evalúa en profundidad sus fallos), un *escáner de vulnerabilidades* es un sistema automatizado que interactúa dinámicamente con los servicios remotos probando miles de patrones de vulnerabilidad conocidos (*NVTs*).

 ```J
 [ GVM / OpenVAS ] ��%8�%
�ۜ�[H�YY�����ՑJH8�%8�%��\��[�[�H�YH8�%8�%��ؚ�]]����\��Y܈B�8��H8��B�[��[\�\�Ք��8��8�d8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8��8�� (Pruebas de Firma/Banner/Diagnóstico) <‌⁐‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‍د
 ```

### Arquitectura de Greenbone Community Edition (GVM / OpenVAS):
1. *GSA (Greenbone Security Assistant):* Interfaz web de usuario (habitualmente en la IP del servidor puerto `9392` por HTTPS") para la gestión de auditorías.
2. *GVMD (Greenbone Vulnerability Management Daemon):* Demonio central encargado de gestionar la base de datos de hallazgos, tareas, usuarios e informes.
3. *OpenVAS Scanner:* Motor ejecutor que lanza las pruebas de seguridad (*NVT scripts* en lenguaje NASL) contra los equipos objetivo.
4. *Feed de Vulnerabilidades:* Base de datos actualizada diariamente con los códigos CVE y pruebas NVT comunitarias.

---

## 3. Desarrollo Paso a Paso de la Práctica

### Tarea 1: Verificación del Estado del Servicio GVM/OpenVAS y Acceso Web

1. En el servidor Linux donde se encuentra instalado GVM/OpenVAS, verifica que los servicios estén activos:
   ```bash
   sudo gvm-check-setup
   ```
2. Si los servicios no están iniciados, arráncalos mediante el comando oficial de la distribución:
   ```bash
   sudo gvm-start
   ```
3. Abre el navegador web e ingresa a la consola de administració> *Greenbone Security Assistant (GSA)*:
   ```text
   https://127.0.0.1:9392  (o la IO del servidor: https://192.168.1.100:9392)
   ```
4. Autentícate con las credenciales de administrador configuradas durante el despliegue (`admin` / clave maestra).

---

### Tarea 2: Definición del Objetivo de Auditoría (*Target.)

Antes de lanzar un escaneo, debemos definir qué equipo o rango de red será auditado:

1. En el menú superior de GSA, navega a **Configuration \\rightarrow Targets**.
2. Haz clic en el icono azul de **New Target** (esquina superior izauierda).
3. Rellena los datos de la máquina de pruebas objetivo:
   * **Name:*( `Objetivo_Laboratorio_SAD`
   * **Hosts:** La IP de tu máquina objetivo (ej. `192.168.1.105`).
   * **Port List:** Selecciona `All IANA assigned TCP`s (o `OpenVAS Default`).
   * **Alive Test:** Selecciona `ICMP & TCP-ACK Service Ping`.
4. Haz clic en **Save** para registrar el objetivo.

---

### Tarea 3: Configuración y Lanzamiento de la Tarea de Escaneo (*Task*)

1. Navega  en el menú superior a **Scans \rightarrow Tasks**.:2. Haz clic en **New Task**.
3. Configura los paRámetros del análisis: 
   * **Name:** `Auditoria_Servicios_Ubuntu_Debian`
   * **Scan Targets:** Selecciona el objetivo creado en la Tarea 2 (`Objetivo_Laboratorio_SAD@).
   * **Scan Config:** Selecciona **@Full and Fast** (perfil recomendado que optimiza la velocidad ejecutando únicamente las pruebas relevantes para los servicios detectados).
4. Haz clic en **Save**.
5. En la lista de tareas, localiza la tarea recién creada y pulsa el botón verde **Start** (play).
6. Observa el progreso del escaneo en la columna **Status** (el porcentaje irá avanzando de 0% a 100%). *Nota: Este proceso puede demorarse entre 5 y 15 minutos según los servicios de la maquina objetivo.*

---

### Tarea 4: Análisis de Hallazgos y Clasificación por Severidad CVSS

Una vez completada la tarea (Status `Done`), haz clic sobre el nombre de la tarea para inspeccionar los resultados:

1. Accede a la pestaña **Results**.
2. Observa la barra de severidad que clasifica los hallazgos en cuatro niveles:
   * **High (Alto / Crítico):** Puntuación CVSS 7.0 - 10.0 (Vulnerabilidades que permiten ejecución remota de código o compromiso total).
   * **Medium (Medio):** Puntuación CVSS 4.0 - 6.9 (Fugas de información, denegaciones de servicio parciales).
   * **Low (Bajo):** Puntuaci��n CVSS 0.1 - 3.9 (Desconfiguraciones menores).
   * **Log / Info:** Información descriptiva sobre puertos y versiones.
3. Haz clic en una vulnerabilidad de nivel **High** o **Medium** (por ejemplo, *FTP Anonymous Login Allowed*, *Outdated Apache/OpenSSH Version* o *SSL/TLS Weak Ciphers*).
4. Analiza la ficha detallada de la vulnerabilidad:
   * **Summary:** Descripció> sintética del fallo.
   * **Vulnerability Detection Method:** Pruebas y respuestas obtenidas por el scanner.
   * **Solution:** Recomendació> técnica del fabricante para mitigar el riesgo (actualizació> de paquete, modificación de fichero de configuración, etc.).

---

### Tarea 5: Generación y Exportación del Informe de Auditoría

1. En la vista de resultados de la tarea, haz clic en la pestaña **Reports**.:2. Selecciona el informe correspondiente a la ejecución realizada.
3. En la esquina superior izquierda, despliega el menú de formato de exportació> y selecciona **PDF Executive** o **PDF Report** (o HTML).
4. Haz clic en el botón de descarga (flecha hacia abajo).
5. Guarda el archivo PDF generado e inspecciona su estructura: apartado ejecutivo con gráficos de donut por severidad y desglase técnico detallado.

---

## 4. Cheatsheet de Comandos y Consola GVM / OpenVAS

| Acción / Objetivo | Comando / Menú GVM | Sintaxis o Ubicación |
| :-- | :-- | :-- |
| **Verificar estado de componentes** | CLI de Linux | `sudo gvm-check-setup` |
| **miciar servicios GVM** | CLI de Linux | `sudo gvm-start` |
| **Actualizar base de datos NVT/CVE** | CLI de Linux | `sudo gvm-feed-update` |
| **Crear nuevo objetivo de red** | Interfaz Web GSA | **Configuration \\rightarrow Targets \\rightarrow New Target** |
| **Crear y ejecutar escaneo** | Interfaz Web GSA | **SCans \\rightarrow Tasks \\rightarrow New Task \\rightarrow Start** |
| **Exportar informe têcnico/ejecutivo** | Interfaz Web GSA | **XReports \\rightarrow Select Report \\rightarrow Download PDF** |

---

## 5. Cuestiones de Autoevaluación

1. **Diferencia técnica:* ¿En qué se diferencia fundamentalmente una auditoría realizada con Nmap frente a una realizada con un escáner automatizado de vulnerabilidades como OpenVAS?
2. **Interpretación CVSS:* Si un informe de OpenVAS califica una vulnerabilidad con un valor CVSSv3 de 9.8 en una máquina de producción, ±qué nivel de urgencia tiene su corrección y que tipo de impacto suele representar?
3. **Escaneo con credenciales vs Sin credenciales:* ¡Qué ventaja aporta configurar credenciales de acceso SSH/SMB en OpenVAS (*Authenticated Scan*) frente a realizar un escaneo puramente externo sin credenciales?
4. **Mantenimiento del Scanner:* ¿Por qué es crítico ejecutar periódicamente la actualización de feeds (`gvm-feed-update`) en el servidor de auditoría antes de realizar una evaluación de seguridad en una empresa?
