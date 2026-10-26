# Práctica 5.5: Despliegue de un Servidor de Autenticación RADIUS con FreeRADIUS

!!! note "Datos de la Práctica"
    * **Módulo:** Seguridad y Alta Disponibilidad (SAD - 0378)
    * **Unidad Didáctica:** UD5 - Seguridad Activa y Auditorías en Redes Corporativas
    * **Requisitos:** Un servidor Linux (Debian o Ubuntu) para albergar FreeRADIUS 3.0, un punto de acceso/router inalámbrico o simulador en GNS3/VirtualBox (con soporte WPA2/WPA3 Enterprise / IEEE 802.1X), y un equipo cliente/móvil suplicante. Privilegios `sudo`.

---

## 1. Objetivos de la Práctica

* Comprender el funcionamiento del protocolo de autenticación centralizada **RADIUS** (*Remote Authentication Dial-In User Service*) y la arquitectura **AAA** (*Authentication, Authorization, Accounting*).
* Desplegar y configurar el servicio **FreeRADIUS 3.0** en una distribución Linux.
* Configurar usuarios, contraseñas y parámetros de red en el archivo `/etc/freeradius/3.0/users`.
* Registrar equipos de red (Puntos de Acceso / Routers NAS - *Network Access Server*) como clientes autorizados en `/etc/freeradius/3.0/clients.conf` mediante clave compartida (*secret*).
* Habilitar la auditoría de accesos y registro de trazas de autenticación (*Accounting*) en FreeRADIUS.
* Configurar un punto de acceso inalámbrico en modo **WPA2-Enterprise / WPA3-Enterprise (IEEE 802.1X)** vinculado al servidor RADIUS.
* Realizar pruebas de autenticación desde un dispositivo cliente (suplicante) y auditar los accesos registrados en `/var/log/freeradius/radius.log`.

---

## 2. Introducción Teórica: La Arquitectura AAA e IEEE 802.1X

En redes corporativas, el uso de claves compartidas (WPA2-Personal / PSK) presenta graves inconvenientes: todos los empleados usan la misma contraseña, no se puede identificar individualmente quién accede y cambiar la clave exige reconfigurar todos los dispositivos.

Para resolver esto se emplea el estándar **IEEE 802.1X** con un servidor **RADIUS** (Modelo AAA):

```mermaid
flowchart LR
    Sup["Suplicante / Cliente<br>Laptop / Smartphone<br>Credenciales nominales"] -->|EAP-PEAP / TLS| Auth["Autenticador / AP<br>Punto de Acceso NAS<br>Filtro de puerto 802.1X"]
    Auth -->|Protocolo RADIUS / UDP| Rad["Servidor FreeRADIUS<br>Servicio AAA<br>Base de datos local / LDAP"]

    style Sup fill:#e1f5fe,stroke:#0288d1,stroke-width:2px
    style Auth fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    style Rad fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
```

### Componentes de la Arquitectura:

1. **Suplicante (Cliente):** El dispositivo (PC, smartphone) que solicita acceso a la red proporcionando credenciales individuales.
2. **Autenticador (NAS - *Network Access Server*):** El switch o punto de acceso Wi-Fi. Bloquea todo el tráfico salvo las tramas de autenticación EAP hacia el servidor RADIUS.
3. **Servidor de Autenticación (FreeRADIUS):** Valida las credenciales recibidas, consulta las políticas de acceso y responde con un mensaje `Access-Accept` o `Access-Reject`.

### Posibles Escenarios de Montaje de la Práctica

Para la realización del laboratorio puedes optar por uno de los dos escenarios siguientes en función del equipamiento disponible:

#### Opción A: Escenario con Punto de Acceso dedicado (AP)
En este montaje el Punto de Acceso actúa como autenticador conectado directamente a la misma red local que el servidor FreeRADIUS:

![Escenario RADIUS con Punto de Acceso](img/radius_escenario_ap.svg)

#### Opción B: Escenario con Router Inalámbrico
En este montaje se emplea un router Wi-Fi doméstico o de laboratorio configurado para delegar la autenticación de su red inalámbrica en el servidor FreeRADIUS:

![Escenario RADIUS con Router Inalámbrico](img/radius_escenario_1.svg)

!!! tip "Emulación sin dispositivo físico"
    Si no dispones de un punto de acceso físico en casa o en el aula, puedes realizar pruebas de configuración y explorar las opciones de seguridad WPA-Enterprise mediante el [Emulador Web de Punto de Acceso D-Link](https://anweb.dlink.ru/).

---

## 3. Desarrollo Paso a Paso de la Práctica

### Tarea 1: Instalación de FreeRADIUS y Utilidades

1. En el servidor Linux destinado a RADIUS, actualiza los repositorios e instala FreeRADIUS y sus herramientas de prueba (`freeradius-utils`):
   ```bash
   sudo apt update
   sudo apt install freeradius freeradius-utils -y
   ```
2. Comprueba que el servicio se encuentra activo y escuchando en los puertos UDP **1812** (Autenticación) y **1813** (Anotación/Accounting):
   ```bash
   sudo systemctl status freeradius
   sudo ss -tulpn | grep freeradius
   ```

---

### Tarea 2: Configuración de Usuarios y Credenciales en `users`

1. Abre el archivo de usuarios de FreeRADIUS:
   ```bash
   sudo nano /etc/freeradius/3.0/users
   ```
2. Añade al principio del archivo la definición de dos usuarios de prueba con contraseñas en claro (`Cleartext-Password`):
   ```text
   # Usuario de prueba 1
   testing Cleartext-Password := "1234"

   # Usuario de prueba nominal (SAD)
   roman Cleartext-Password := "ClaveSegura2026!"
   ```
3. Guarda los cambios (`Ctrl + O`) y sal (`Ctrl + X`).
4. Reinicia el servicio FreeRADIUS:
   ```bash
   sudo systemctl restart freeradius
   ```

---

### Tarea 3: Verificación Local del Servidor con `radtest`

Antes de conectar el punto de acceso inalámbrico, probaremos localmente que la base de datos de usuarios responde correctamente:

1. Ejecuta el comando de diagnóstico `radtest` simulando una petición de autenticación:
   ```bash
   radtest roman ClaveSegura2026! 127.0.0.1 1812 testing123
   ```
   Los parámetros del comando son:

    * Usuario que has dado de alta en users: roman
    * Su contraseña: ClaveSegura2026!
    * Nombre del cliente, en este caso, localhost o 127.0.0.1
    * Un número de puerto. Vale cualquiera.
    * Contraseña que usa localhost para validase frente al servidor freeradius ( directiva secret de clients.conf).

2. **Resultado esperado:**
   ```text
   Sent Access-Request Id 78 from 0.0.0.0:54321 to 127.0.0.1:1812
   Received Access-Accept Id 78 from 127.0.0.1:1812 length 20
   ```
   Si la clave es errónea, devolverá `Received Access-Reject`.

---

### Tarea 4: Configuración de Clientes NAS en `clients.conf`

Para que el punto de acceso Wi-Fi o router pueda enviar peticiones al servidor RADIUS, debemos registrar su dirección IP y una clave compartida (*Secret*):

1. Abre el archivo de clientes:
   ```bash
   sudo nano /etc/freeradius/3.0/clients.conf
   ```
2. Añade un nuevo bloque para registrar tu Punto de Acceso / Router Wi-Fi (ejemplo con IP `192.168.122.99`):
   ```text
   client router_wifi_ap {
       ipaddr = 192.168.122.0/24
       secret = ElCaminas             
       shortname = MikroTik 
   }
   ```

!!! tip "IP del AP"
    Obtendremos la IP del punto de acceso más adelante. De momento puedes poner el rango entero de red local. Cuando configuremos el Router/AP podríamos cambiarla por la IP del equipo.      


3. Guarda y reinicia FreeRADIUS:
   ```bash
   sudo systemctl restart freeradius
   ```

---

### Tarea 5: Activación de Logs de Auditoría (*Accounting*)

1. Abre el fichero principal de configuración `/etc/freeradius/3.0/radiusd.conf`:
   ```bash
   sudo nano /etc/freeradius/3.0/radiusd.conf
   ```
2. Localiza la sección `log { ... }` y asegúrate de habilitar el registro de intentos de acceso exitosos y fallidos. Busca las distintas directivas y modifícalas si es necesario. Comprueba en la ayuda del propio archivo para qué sirve cada una:
   ```text
   log {
       destination = files
       file = ${logdir}/radius.log
       auth = yes
       auth_badpass = yes
       auth_goodpass = yes
   }
   ```
3. Reinicia el servicio: `sudo systemctl restart freeradius`.

---

### Tarea 6: Configuración del Punto de Acceso Inalámbrico (WPA2/WPA3 Enterprise)

Si tenemos un AP que podamos conectar a la misma red que está conectado nuestro servidor FreeRadis podremos hacer una prueba real de validación de cliente móvil. En primer lugar deberemos configurar en AP.

1. Accede a la interfaz web de administración de tu router/AP Wi-Fi.
2. En la sección de configuración de la red inalámbrica, selecciona el modo de seguridad:

    * **Modo de Seguridad:** WPA2-Enterprise / WPA3-Enterprise (IEEE 802.1X).
    * **Dirección IP del Servidor RADIUS:** La IP de tu servidor Linux FreeRADIUS (ej. `192.168.122.X`).
    * **Puerto RADIUS:** `1812` UDP.
    * **Clave compartida (Secret):** `ClaveSecretaRADIUS2026` (debe coincidir exactamente con la configurada en `clients.conf`).

![Configuración WPA2-Enterprise en Punto de Acceso](img/radius_escenario_ap_emu.png)

---

#### Validación de Conexión desde el Suplicante y Auditoría de Logs

1. Desde un smartphone o PC cliente, busca la red Wi-Fi corporativa emitida por el AP.
2. Al conectar, el sistema operativo solicitará los parámetros de autenticación Enterprise (EAP-PEAP):

    * **Método EAP:** PEAP (o TTLS).
    * **Autenticación de Fase 2 (Inner):** MSCHAPv2 (o GTC).
    * **Certificado CA:** *"No validar"* (o seleccionar certificado de pruebas).
    * **Identidad / Usuario:** `roman`
    * **Contraseña:** `ClaveSegura2026!`

| Conexión en Smartphone Android | Parámetros de Seguridad Avanzados |
| :---: | :---: |
| ![Configuración Wi-Fi Enterprise en móvil](img/radius_cliente_movil.png) | ![Detalles de la conexión y seguridad](img/radius_ap_config.png) |

3. Conéctate a la red y comprueba que se valida la identidad y se obtiene dirección IP para navegar.
4. En el servidor FreeRADIUS, audita el registro de accesos en tiempo real:
    ```bash
    sudo tail -f /var/log/freeradius/radius.log
    ```
5. Realiza una prueba introduciendo una contraseña incorrecta y verifica que queda registrado el intento fallido (`Auth: Login incorrect...`).

### Tarea 7: Conexión de un router MikroTik

Si no tenemos un AP físico podremos emularlo con un router MikroTik. Simularemos la conexión inalámbrica con una conexión cableada en su interfaz eth2.

Para simular el escenario de autenticación WPA2/WPA3 Enterprise (802.1X/EAP) mediante cable en RouterOS, no se utiliza la pestaña Wireless (ya que ether2 es un puerto Ethernet físico), sino el servicio Dot1X. El Router MikroTik actúa como NAS / Autenticador 802.1X, delegando la validación de credenciales al FreeRADIUS conectado en ether1, mientras que el equipo cliente en ether2 debe actuar como Suplicante (Supplicant).

En primer lugar abre una consola en el MikroTik para obtener su dirección IP. 

1. Te pedirá usuario: `admin`
2. Password: déjalo en blanco
3. Te pedirá que le asignes contraseña al usuario admin, repite `admin` 2 veces
4. Ahora ya puedes obenter la IP con `ip/address/print` 
5. Desde tu host accede a la IP obtenida para gestionar el mikrotik desde Webfig, su entorno gráfico.

A continuación se detalla la configuración paso a paso a través de WinBox o WebFig:

1. Configura la IP de la red local en eth2 en QuickSet. Activa también DHCP y NAT.
    ![QuickSet](img/QuickSet.png)

2. Registrar el servidor RADIUS en RouterOS
    Define el servidor FreeRADIUS como backend de autenticación para servicios Dot1X.

    1. En el menú lateral, abre RADIUS.
    2. Haz clic en + (Add):
    3. Configura el resto de parámetros como en la imagen:
    ![ServerRadiusMikrotic](img/ServerRadiusMikroTik.png)
    4. Haz clic en OK.

3. Configurar el puerto ether2 como autenticador 802.1X
    Activa el servidor Dot1X en la interfaz cableada eth2 que da acceso al cliente de prueba.
    1. En el menú lateral, ve a Dot1X.
    2. En la pestaña Server, haz clic en + (Add):
    3. Configura según la imagen.
    ![dot1x](img/dot1x.png)
    4. Haz clic en OK.

Nota sobre el tráfico: Mientras el puerto no esté autenticado, ether2 permanecerá bloqueado a nivel de capa 2 para cualquier tráfico que no sean tramas EAPOL (EAP over LAN). Al autenticarse correctamente, el estado de la interfaz pasará a autorizado y permitirá el paso del tráfico de red.

#### Validación de Conexión desde el Suplicante y Auditoría de Logs

Como suplicante usaremos un Debian conectado al interfaz eth2 del router MikroTik.

Para actuar como suplicante (supplicant) cableado en Linux se requiere el demonio `wpa_supplicant`, el cual está disponible o es instalable en Debian.

Ejemplo de configuración en el Debian (conectado a ether2)
Instalar el software suplicante:

``` Bash
sudo apt update && sudo apt install -y wpasupplicant
```

Crear el archivo de configuración 802.1X:
Crea un archivo temporal (por ejemplo, /etc/wpa_supplicant/dot1x.conf):

```bash
ctrl_interface=/run/wpa_supplicant
ap_scan=0
fast_reauth=1

network={
    key_mgmt=IEEE8021X
    eap=PEAP
    phase2="auth=MSCHAPV2"
    identity="roman"
    password="ClaveSegura2026!"
}
```

Nota: La directiva ap_scan=0 es obligatoria para conexiones cableadas (Ethernet), ya que indica a wpa_supplicant que no busque balizas WiFi (beacons) y opere directamente sobre tramas EAPOL cableadas.

Lanzar la autenticación:
Suponiendo que la interfaz del Debian conectada a ether2 sea eth0:

```Bash
sudo wpa_supplicant -D wired -i ens4 -c /etc/wpa_supplicant/dot1x.conf -dd &
```

-D wired: Indica el driver para interfaces cableadas.

-dd: Modo depuración para ver paso a paso el intercambio de tramas EAP (EAP-Request/Identity, intercambio de claves, EAP-Success).

Puedes reiniciar el servicio networking para asegurar que se tira el interfaz y se vuelve a levantar correctamente.

#### Comprobación en el MikroTik

Una vez que el puerto pasa a estado autorizado, el conmutador de MikroTik permite el tráfico normal. Puedes comprobarlo en el servicio dot1x en las pestañas State y Active.

![State](img/dot1xState.png)

![Active](img/dot1xActive.png)

#### Comprobación en el FreeRadius

Comprueba el log del Freeradius para comprobar si se ha realizado bien la autenticación del suplicante:

```bash
cat /var/log/freeradius/radius.log
```

---

## 4. Cheatsheet de Administración de FreeRADIUS

| Acción / Objetivo | Fichero o Comando | Sintaxis / Detalle |
| :--- | :--- | :--- |
| **Instalar FreeRADIUS** | Comando `apt` | `sudo apt install freeradius freeradius-utils` |
| **Añadir usuario y clave** | `/etc/freeradius/3.0/users` | `usuario Cleartext-Password := "contraseña"` |
| **Registrar cliente NAS / AP** | `/etc/freeradius/3.0/clients.conf` | `client AP { ipaddr = 192.168.1.X; secret = clave }` |
| **Prueba local de credenciales** | Comando `radtest` | `radtest <user> <pass> <host> <port> <secret>` |
| **Modo Depuración (Debug Manual)** | Comando `freeradius -X` | Ejecuta el servidor en primer plano mostrando trazas detalladas. |
| **Inspección de Logs de Acceso** | `/var/log/freeradius/radius.log` | `sudo tail -f /var/log/freeradius/radius.log` |

---

## 5. Cuestiones de Autoevaluación

1. **Ventaja organizativa:** ¿Por qué la arquitectura WPA2/WPA3 Enterprise basada en RADIUS es infinitamente más segura en una empresa que el uso de una clave Wi-Fi compartida (PSK)?
2. **Función del Secret:** ¿Qué función cumple la clave secreta compartida (*secret*) configurada en `clients.conf` entre el servidor RADIUS y el Punto de Acceso AP?
3. **Uso del modo depuración (`freeradius -X`):** Si un cliente no puede autenticarse a través del AP, ¿por qué se recomienda detener el servicio `systemctl` y ejecutar `sudo freeradius -X` para diagnosticar el problema?
4. **Auditoría e Incidentes:** ¿Qué información valiosa queda registrada en `/var/log/freeradius/radius.log` cuando un usuario descontento intenta acceder a la red fuera de su horario laboral?

---

## 6. Recursos y Enlaces de Referencia

* **[FreeRADIUS Documentation & WPA-HOWTO](https://wiki.freeradius.org/guide/WPA-HOWTO):** Guía técnica oficial para configuración de WPA-Enterprise y certificados TLS.
* **[Tabla de Compatibilidad de Protocolos EAP y FreeRADIUS](http://deployingradius.com/documents/protocols/compatibility.html):** Resumen de compatibilidad entre clientes, métodos EAP (PEAP, TLS, TTLS) y almacenes de contraseñas.
* **[Emulador Web de Punto de Acceso D-Link](https://anweb.dlink.ru/):** Entorno interactivo para simular la interfaz web de routers y puntos de acceso Wi-Fi.
