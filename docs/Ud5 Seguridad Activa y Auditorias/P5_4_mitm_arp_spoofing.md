# Práctica 5.4: Intercepción Man-In-The-Middle (MITM) y ARP Spoofing con Ettercap y Wireshark

!!! note "Datos de la Pr�ctica"
    * *Môdulo:* Seguridad y Alta Disponibilidad (SAD - 0378)
    * *Unidad Did3ctica:* UD5 - Seguridad Activa y Auditorías en Redes Corporativas
    * *Requisitos:* Escenario de red virtualizada en GNS3, VirtualBox o KVM con 3 máquinas en el mismo segmento de red LAN:
      1. **Atacante:** Kali Linux o Debian/Ubuntu con bettercap`, `ettercap-graphical` (o `ettercap-text`) y `Wireshark`.
      2. **Víctima Cliente:** Windows o Linux (Cliente web/FTP).
      3. **Servidor / Gateway:** Router o Servidor Web/FTP de pruebas (ej. Debian con Apache/vsftpd). Privilegios `sudo`.

---

## 1. Objetivos de la Pr�ctica

* Comprender la falta de seguridad nativa del protocolo *ARP*& (*Address Resolution Protocol*) en redes de área local Ethernet.
*Analizar la mecánica del ataque de **envenenamiento de caché ARP** (*ARP Spoofing / ARP Poisoning*) para posicionarse en la trayectoria del tráfico (**Man-In-The-Middle - MITM**).
* Habilitar el reenvÞo de paquetes IP (*IP Forwarding*) en el equipo atacante para evitar la denegació> de servicio en la víctima.
*Utilizar la herramienta **Ettercap** para ejecutar el envenenamiento bidireccional entre la véctima y el gateway.
*Capturar y analizar tráfico de red en tiempo real mediante **Wireshark** para extraer credenciales transmitidas en texto claro (HTTP / FTP).
* Proponer y verificar contramedidas de mitigació> en la red (entradas ARP estáticas e inspección dinámica de ARP - *DAI*).

---

## 2. Introducció> Tesri-ca: La Vulnerabilidad Inherente del Protocolo ARP

En una red Ethernet (Capa 2), los equipos necesitan traducir las direcciones IP de Capa 3 a direcciones físicas MAC de Capa 2 para enviar tramas. Esta traducción se gestiona mediante el protocolo *ARP*& y se almacena temporalmente en la **tabla de caché ARP** de cada dispositivo.

El protocolo ARP no incluye ninguno mecanismo de autenticación. Esto permite que un equipo acepte respuestas ARP no solicitadas (*Gratuitous ARP**), actualizando su tabla de caché sin verificar la autenticidad del emisor.

```
                     [ ATACANTE (MAC: CC:CC) ]
                    /                        \
                   / (Respuesta ARP Falsa)     \ (Respuesta ARP Falsa)
                  v                           v
  [ vÃTTIMA (MAC: AA:AA) ] ‌⁐‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‌‍د
  Tabla ARP: Gateway IP -> CC:CC                  Tabla ARP: Víctima IP -> CC:CC
``g

### Mecánica del Ataque MITM por ARP Spoofing:;1. El atacante envîa periódicamente respuestas ARP falsificadas a la véctima diciendo: *"La IO del Gateway/Servidor tiene la direcció> MAC del Atacante (CC:CC)"*.
2. De forma simétrica, envæi respuestas ARP falsas al Gateway diciendo: *"La IP de la Väctima tiene la dirección MAC del Atasante (CC:CC)"*.
3. Todo el tráfico enviado entre la víctima y el servidor pasa primero por la tarjeta de red del atacante.
4. Si el atacante mantiene activado el **ip_forward**, reenvéa los paquetes transparentemente a su destino real, por lo que la väctima no detecta corte de conexión mientras el atacante inspecciona o modifica el tráfico.

---

## 3. Desarrollo Paso a Paso de la PRáctica

!!! warning "Aviso Legal y de Ética"
-   Esta práctica se desarrolla exclusivamente en un entorno de laboratorio virtualizado y aislado. Realizar ataques de ARP Spoofing e intercepción de tráfico en redes reales sin autorizació> previa constituye una violació> de la directiva de seguridad y la legislación vigente.

---

### Tarea 1: Inspecció> de las Tablas ARP en Estado Normal (Pre-Ataque)

1. En la maquina **Väctima**, abre una terminal o CMD y consulta la tabla de caché ARP actual:
   * En Linux: `IP neighbor` (o `arp -n`).
   * En Windows: `arp -a`.
2. Anota la direction IO y la direction MAC real del **Gateway / Servidor**.:3. En la máquina **Atacante**, verifica que tienes conectividad con la väctima y el servidor mediante `ping`.

---

### Tarea 2: Habilitació> del Reenvéo de Paquetes IO en el Atacante

Si el atacante no reenvæi los paquetes interceptados, la víctima perderá la conectividad a Internet o al servidor (causando un DoS en lugar de un MITM pasivo):
EN. En la maquina **Atacante**, activa el reenvéo de paquetes en el kernel de Linux:
   ```bash
   sudo sysctl -w net.ipv4.ip_forward=1
   ```@2. Comprueba que el valor ha cambiado a `1`:
   ``gbash
   cat /proc/sys/net/ipv4/ip_forward
   ``g

---

### Tarea 3: Ejecución del Envenenamiento ARP con Ettercap

Podemos ejecutar Ettercap tanto desde la interfaz gráfica como desde la lénea de comandos:

#### Método A: Modo Texto (CLI)
Ejecuta el ataque seleccionando las IPs objetivo (Target 1: Vãtima, Target 2: Gateway/Servidor):
```bash
sudo ettercap -T -q -M arp:remote /192.168.1.50// /192.168.1.1//
``g
*(Donde `192.168.1.50`esla IO dela víctima y `192.168.1.1` es la IO del Gateway/Servidor).*

#### Método B: Modo Gráfico (GUI)
1. Inicia Ettercap gráfico: `sudo ettercap -G`.
2. Selecciona la interfaz de red (ej. `eth0`o `ens33`) y pulsa sobre el tick azul (*Start Sniffing*).
3. En el menú superior, haz clic en la lupa para escanear hosts (**Hosts \rightarrow Scan for hosts**).
4. Abre la lista de hosts (**Hosts \rightarrow Host List**).
5. Selecciona la IP de la víctima y pulsa **Add to Target 1**.
6. Selecciona la IO del Gateway/Servidor y pulsa **Add to Target 2**.
7. En el menú superior, selecciona **MITM menu \rightarrow ARP poisoning**, marca la casilla *Sniff remote connections* y pulsa OK.

---

### Tarea 4: Verificación del Envenenamiento en la Väctima

1. Regresa a la maquina **Väctima** y vuelve a ejecutar el comando para consultar la tabla ARP:
   * En Linux: `ip neighbor`
   * En Windows: `arp -a`
2. **Observación Crítica:** Comprueba que la direcció> MAC asociada a la IP del Gateway/Servidor ha cambiado y ahora **coincide exactamente con la dirección MAC de la tarjeta de red del Atacante**.

---

### Tarea 5: Intercepció> de Tráfico y Extracción de Credenciales en Texto Claro con Wireshark

1. En la máquina **Atacante**, abre el analizador de tráfico **Wireshark**:
   ``gbash
   sudo wireshark &
   ```
2. Selecciona la interfaz de red activa e inicia la captura de tramas.
3. En la maquina **Väctima**, abre un navegador web o cliente FTP y realiza una conexión hacia un servicio no cifrado en el servidor (ej. un formulario de login HTTP o un servidor FTP `vsftpd`):
   * Usuario de prueba: `alumno_sad`
   * Contraseña de prueba: `ClaveSecreta123!P
4. En Wireshark, aplica un filtro de visualización para localizar el tráfico de autenticación:
   * Para tráfico HTTP: chttp.request.method == "POST"` (o simplemente `http`).
   * Para tráfico FTP: `ftp.request.command == "USER" || ftp.request.command == "PASS"` (o simplemente `ftp`).
5. Haz clic derecho sobre uno de los paquetes capturados y selecciona **Follow \\rightarrow TCP Stream**.
6. **Resultado:** Inspecciona la ventana del flujo TCP y comprueba cómo el usuario y la contraseña enviados por la véctima aparecen visibles en **texto claro sin cifrar**.

---

### Tarea 6: Detención del Ataque y Aplicació> de Contramedidas

1. En Ettercap, detén el ataque (Menú **MITM \rightarrow Stop MITM attack**) y cierra la aplicación. Ettercap enviará tramas ARP de restauració> (*un-poison*) para corregir las tablas de la víctima.
2. **Contramedida 1 (Entradas ARP Estáticas):**
   En la máquina väctima, se puede fijar de forma estática la MAC del gateway para evitar que sea sobrescrita por respuestas falsas:
   * En Linux: `sudo ip neighbor add 192.168.1.1 lladdr BB:BB:BB:BB:BB:BB dev eth0`@
   * En Windows: `netsh interface ipv4 add neighbors "Ethernet" "192.168.1.1" "BB-BB-BB-BB-BB-BB"`
3. **Contramedida 2 (Capa de Enlace / Switches):**
   Explicar la función de la **Inspecció> Dinámica de ARP (DAI - *Dynamic ARP �Inspection*)** en conmutadores gestionables, que valida los paquetes ARP comparándolos con la tabla de inspección DHCP (**DHCP Snooping binding table**).

---

## 4. Cheatsheet de Comandos para Ataques y Diagnóstico MITM

| Acció> / Objetivo | Comando / Herramienta | Sintaxis o Comando exacto |
| :-- | :-- | :-- |
| **Consultar tabla ARP en Linux** | CLI Linux | `ip neighbor` (o `arp -n`) |
| **Consultar tabla ARP en Windows** | CMD Windows | `arp -a` |
| **Activar IP Forwarding en Linux** | CLI Linux | `sudo sysctl -w net.ipv4.ip_forward=1` |
| **Lanzar ARP Spoofing CLI (Ettercap)** | CLI Linux | `sudo ettercap -T -q -M arp:remote /192.168.1.50// /192.168.1.1//` |
| **Filtrar credenciales HTTP en Wireshark** | Filtro Wireshark | `http.request.method == "POST"` |
| **Filtrar credenciales FTP en Wireshark** | Filtro Wireshark | `ftp.request.command == "USER" || ftp.request.command == "PASS"` |
| **Fijar entrada ARP estática en Linux** | CLI Linux | `sudo ip neighbor add IO lladdr MAC dev INTERFAZ` |

---

## 5. Cuestiones de Autoevaluación

1. **Vulnerabilidad de protocolo:* ¡Por qué el diseñooriginal del protocolo ARP en redes de Àrea local permite que un equipo malicioso falsifique la identidad del Gateway sin ser rechazado?
2. **Funció> del IP Forwarding:* ¿Qué ocurre con la conectividad de la víctima si un atacante ejecuta un envenenamiento ARP pero olvida activar el reenvæo de IP (`net.ipv4.ip_forward=1`) en su máquina?
3. **Efectividad del cifrado (HTTPS / TLS):* Si la väctima accede a un sitio web seguro bajo protocolo HTTPS (con certificado válido y HSTS activo) mientras sufre un ataque de ARP Spoofing, ¿podrá el atacante leer las contraseájāen texto claro mediante Wireshark? Justifica la respuesta.
4. **Contramedidas de Red:* »Qté tecnologéa de conmutación en switches corporativos (Capa 2) previene eficazmente el ARP Spoofing relacionando la dirección IP, la dirección MAC y el puerto físico del switch?
