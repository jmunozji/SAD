# UD1. Fundamentos de Seguridad Informática

!!! info "Objetivos de la Unidad"
    Al finalizar esta unidad didáctica, serás capaz de:
    
    *   **Diferenciar** con precisión entre seguridad informática y seguridad de la información.
    *   **Dominar** el vocabulario elemental de análisis de riesgos (activos, amenazas, vulnerabilidades, etc.).
    *   **Identificar** los servicios de seguridad fundamentales y los principios de la tríada CIA.
    *   **Clasificar** cualquier medida defensiva dentro de la matriz de seguridad física, lógica, activa y pasiva.
    *   **Comprender** el impacto crítico del factor humano y las principales técnicas de ingeniería social (*phishing*, *vishing*, *baiting*).

---

## 1. La sociedad de la información en que vivimos

Actualmente vivimos en lo que se conoce como la **Sociedad de la Información.** Para cualquier organización —ya sea con ánimo de lucro o sin él, pública o privada— el activo más valioso es la **información**. Incluso a nivel particular, todos dependemos enormemente de la información que manejamos a diario para nuestras actividades cotidianas.

En el ámbito educativo, por ejemplo, los centros dependen de aplicaciones de gestión docente para registrar calificaciones y asistencias, o de entornos de aprendizaje virtual (*e-learning*) como **Moodle** para publicar contenidos, programar exámenes y entregar actividades.

!!! example "Ejemplo local: Aplicación ITACA"
    En la Comunidad Valenciana, la plataforma **ITACA** centraliza toda la gestión administrativa y docente de los centros de secundaria y formación profesional. Un fallo de disponibilidad en este sistema paraliza de inmediato la actividad administrativa diaria de miles de docentes.

Es un grave error pensar que nuestros datos personales, ordenadores de aula o redes Wi-Fi carecen de interés para un ciberdelincuente. Desde el punto de vista defensivo, debemos asumir que **cualquier recurso conectado tiene valor**, aunque solo sea para ser instrumentalizado.

!!! danger "El peligro del malware silencioso: Servidores Zombie"
    Un ordenador comprometido mediante malware puede ser controlado de forma remota por un atacante desde un servidor de *Command & Control* (C&C). Este equipo pasa a ser un "zombie" dentro de una red de *bots* (botnet), utilizándose de forma masiva para:
    
    *   Lanzar ataques distribuidos de denegación de servicio (DDoS) contra grandes corporaciones.
    *   Enviar millones de correos de spam o correos maliciosos de *phishing*.
    *   Alojar de manera oculta contenido ilegal o altamente comprometido.

!!! quote "Caso real de investigación forense"
    Hace unos años, un docente tuvo que solicitar soporte urgente porque su ordenador corporativo funcionaba con extrema lentitud. Tras realizar un análisis forense y escanear el sistema con herramientas antimalware, se detectó que el equipo había sido infectado por un troyano. Este malware utilizaba silenciosamente la máquina de la víctima como un servidor de intercambio P2P para alojar y distribuir material audiovisual cifrado de carácter ilícito. 
    
    Si el Grupo de Delitos Telemáticos de la Guardia Civil hubiera rastreado la dirección IP del servidor, la investigación habría apuntado directamente al domicilio del docente afectado, quien, lógicamente, habría sido descubierto como una víctima colateral e inconsciente del ataque tras el análisis forense de su disco.

Por lo tanto, la **seguridad informática** no es una materia que deba tomarse a la ligera. Todo administrador de sistemas debe contar con un sólido criterio defensivo. En la seguridad, como en la vida, el **sentido común** es nuestra mejor salvaguarda. No sirve de nada aplicar medidas técnicas a lo loco (cortafuegos, antivirus, cifrados complejos) si no entendemos qué estamos protegiendo y de quién nos estamos defendiendo.

### 1.1. El auge del IoT y las Smart Cities (Internet of Threats)
El imparable despliegue del Internet de las Cosas (IoT) y la evolución hacia las *Smart Cities* han multiplicado exponencialmente la superficie de ataque de nuestras infraestructuras. Dispositivos cotidianos como cámaras de videovigilancia, electrodomésticos inteligentes o controladores de sensores carecen a menudo de medidas de seguridad nativas. En los círculos de ciberseguridad, ya se apoda humorísticamente al IoT como el **"Internet of Threats"** (Internet de las Amenazas).

!!! warning "El ciberataque histórico a Dyn (Octubre de 2016)"
    El 21 de octubre de 2016 se ejecutó uno de los mayores ciberataques de la historia utilizando la botnet **Mirai**. Compuesta por millones de dispositivos IoT domésticos vulnerables (principalmente routers domésticos, cámaras IP y monitores de bebés que venían con contraseñas por defecto de fabricante), la botnet saturó mediante tráfico DDoS los servidores DNS del proveedor Dyn, dejando fuera de servicio temporalmente a gigantes tecnológicos de la talla de Amazon, Spotify, Twitter, Netflix y HBO.

---

## 2. Seguridad de la información vs. Seguridad informática

Para asentar las bases teóricas de la asignatura, es crucial distinguir entre estos dos conceptos que, aunque íntimamente relacionados, operan en diferentes niveles de abstracción:

```
┌────────────────────────────────────────────────────────┐
│               SEGURIDAD DE LA INFORMACIÓN               │
│  (Políticas, personas, contratos, archivadores, leyes)   │
│                                                        │
│       ┌────────────────────────────────────────┐       │
│       │          SEGURIDAD INFORMÁTICA         │       │
│       │    (Servidores, cifrado, firewalls)    │       │
│       └────────────────────────────────────────┘       │
└────────────────────────────────────────────────────────┘
```

### Seguridad de la información
Consiste en el conjunto de medidas, procedimientos, políticas, contratos y metodologías, tanto humanas como organizativas o técnicas, que permiten proteger la integridad, confidencialidad y disponibilidad de la información, **independientemente del soporte** en el que se encuentre (digital, papel, conversaciones orales o memorandos).

### Seguridad informática
Es una especialización de la seguridad de la información que se enfoca estrictamente en salvaguardar la información contenida, procesada o transmitida a través de la **infraestructura informática y de telecomunicaciones** (servidores, redes lógicas, bases de datos, sistemas operativos y dispositivos conectados).

!!! info "En resumen"
    La *seguridad de la información* se encarga de proteger el **sistema de información** general de la organización, mientras que la *seguridad informática* protege los **sistemas informáticos** concretos que sirven de soporte técnico a ese flujo de información.

---

## 3. Citas célebres sobre seguridad informática

Antes de profundizar en la teoría, analizaremos algunas citas célebres de grandes figuras de la ciberseguridad. Sus reflexiones nos ayudarán a entender la mentalidad y filosofía necesarias para proteger un entorno corporativo:

!!! quote "Sobre la inexistencia del sistema 100% seguro — Gene Spafford"
    "El único sistema verdaderamente seguro es el que está apagado y desenchufado, encerrado en una caja fuerte de titanio revestido, enterrado en un búnker de hormigón, y rodeado por gas nervioso y guardias armados muy bien remunerados. Incluso entonces, no apostaría mi vida por él."
    
    *Gene Spafford, profesor y reputado experto en seguridad informática.*

!!! quote "Sobre la debilidad de los hábitos de los usuarios — Chris Pirillo"
    "Las contraseñas son como la ropa interior. No puedes dejar que nadie la vea, debes cambiarla regularmente y no debes compartirla con extraños."
    
    *Chris Pirillo, blogger y divulgador tecnológico.*

!!! quote "Sobre la falsa sensación de seguridad técnica — Bruce Schneier"
    "Si piensas que la tecnología puede solucionar tus problemas de seguridad, está claro que ni entiendes los problemas ni entiendes la tecnología."
    
    *Bruce Schneier, criptógrafo de renombre internacional.*

!!! quote "Sobre la necesidad de educar antes que blindar — Kevin Mitnick"
    "Las organizaciones gastan millones de dólares en firewalls y dispositivos de seguridad, pero tiran el dinero porque ninguna de estas medidas cubre el eslabón más débil de la cadena de seguridad: la gente que usa y administra los ordenadores."
    
    *Kevin Mitnick, histórico hacker defensivo y consultor.*

---

## 4. Principios básicos de la seguridad de la información (Tríada CIA)

La norma **ISO/IEC 27000** define los tres pilares fundamentales que sustentan cualquier estrategia defensiva, representados de forma clásica por las siglas inglesas **CIA** (*Confidentiality, Integrity, Availability*):

```
       ▲
      / \
     /   \
    /     \
   / TRÍADA\
  /   CIA   \
 /___________\
C             I
(Conf.)     (Integ.)
```

### 1. Confidencialidad
Garantiza que la información sea accesible exclusivamente para aquellas personas, entidades o procesos autorizados de forma legítima, impidiendo cualquier divulgación accidental o intencionada a terceros no autorizados.

### 2. Integridad
Asegura que la información y los datos se mantengan exactos, completos y libres de modificaciones no autorizadas, alteraciones accidentales, borrados o corrupciones desde su origen hasta su destino.

### 3. Disponibilidad
Asegura que los usuarios legítimos y autorizados tengan acceso ininterrumpido a la información y a los recursos del sistema en el momento preciso en que lo requieran.

---

### Servicios y propiedades adicionales de seguridad
Para completar la tríada CIA, los sistemas modernos requieren la implementación de servicios complementarios:

*   **Autenticación:** Proceso que permite verificar de forma inequívoca la identidad de un usuario, emisor o receptor (mediante contraseñas, certificados digitales o biometría) antes de concederle privilegios de acceso.
*   **Control de acceso:** Mecanismo técnico que restringe y supervisa las operaciones que un usuario autenticado puede realizar sobre los recursos del sistema, basándose en políticas de permisos y perfiles definidos.
*   **No repudio:** Propiedad tecnológica vinculada a la firma digital que impide que una entidad pueda negar haber participado en una comunicación o transacción. Se clasifica en:
    *   **En origen:** El emisor no puede negar el envío (p. ej., la firma electrónica de una declaración telemática a la Agencia Tributaria).
    *   **En destino:** El receptor no puede negar haber recibido la información corporativa.
*   **Trazabilidad (Auditoría):** Capacidad de registrar cronológica e inalterablemente todas las operaciones y eventos acontecidos en el sistema informático. Este registro (log) permite rastrear cualquier suceso hasta su origen, resultando indispensable para realizar un **análisis forense básico** tras un incidente.

---

## 5. Conceptos fundamentales en análisis de riesgos

Para diseñar y evaluar defensas, la seguridad de la información utiliza una terminología muy precisa. Entender la relación dinámica entre estos conceptos es clave para cualquier administrador de sistemas de red:

```
               ┌───────────────┐
               │    ACTIVO     │
               └───────┬───────┘
                       │ tiene
                       ▼
               ┌───────────────┐
               │VULNERABILIDAD │
               └───────┬───────┘
                       │ explotada por
                       ▼
 ┌───────────┐ ┌───────────────┐
 │  ATAQUE   ├─►   AMENAZA     │
 └───────────┘ └───────┬───────┘
                       │ causa
                       ▼
 ┌───────────┐ ┌───────────────┐
 │ DESASTRE  ◄─┤   IMPACTO     │ (conduce a un Riesgo)
 └───────────┘ └───────────────┘
```

*   **Activo (Asset):** Cualquier recurso físico, lógico o humano del sistema de información que resulta valioso para que la organización funcione de forma correcta y cumpla con sus metas (ejemplos: la base de datos de los alumnos, los servidores de virtualización, la reputación de la marca o la infraestructura física del CPD).
*   **Vulnerabilidad (Vulnerability):** Debilidad, fallo de diseño, mala configuración o grieta inherente en el sistema informático (hardware, software, factor humano o políticas) que puede ser explotada por un atacante para comprometer la seguridad.
*   **Amenaza (Threat):** Evento, suceso o acción (humana o ambiental) capaz de desencadenar un incidente en la organización, explotando una vulnerabilidad y provocando daños o pérdidas en sus activos. Las amenazas suelen clasificarse según su origen en:
    *   **Físicas / Ambientales:** Afectan al entorno de hardware e instalaciones (p. ej., incendios, inundaciones o cortes eléctricos).
    *   **Lógicas:** Afectan al software y los datos (p. ej., ataques de malware, inyecciones de código o exploits).
    *   **Pasivas:** Buscan espiar u obtener información sin alterar el sistema (p. ej., intercepción de comunicaciones en texto plano).
    *   **Activas:** Modifican los datos o servicios deliberadamente (p. ej., ataques de Man-in-the-Middle o DoS).
*   **Ataque (Attack):** Acción concreta ejecutada de manera deliberada por un adversario para intentar burlar las políticas de seguridad y explotar una vulnerabilidad sobre un activo, tenga éxito o no.
*   **Impacto (Impact):** Medida cuantitativa o cualitativa de las consecuencias de la materialización de una amenaza sobre un activo (pérdidas financieras, filtraciones de datos privados, daños legales o de reputación).
*   **Riesgo (Risk):** Probabilidad estadística de que una amenaza se materialice con éxito explotando una vulnerabilidad específica sobre un activo, provocando un impacto estimado.
    
    !!! tip "Fórmula intuitiva del Riesgo"
        $$\text{Riesgo} = \text{Probabilidad de materialización} \times \text{Impacto del suceso}$$
        Mientras que el *impacto* describe qué daño sufriríamos, el *riesgo* determina la probabilidad de que ese daño ocurra realmente en el tiempo.
*   **Desastre o Contingencia:** Interrupción severa de la operatividad del sistema informático y de los servicios de una empresa que imposibilita de manera temporal el desarrollo normal del negocio, requiriendo la activación de planes de contingencia o de recuperación.

---

## 6. Tipos de amenazas y clasificación de atacantes

### Atacantes pasivos vs. activos
*   **Atacante Pasivo:** Su única intención es escuchar u obtener información sensible de la red sin ser detectado. El sistema sigue funcionando perfectamente. Un ejemplo típico es el análisis de tráfico (*sniffing*) de una red local mediante **Wireshark** para interceptar credenciales enviadas en protocolos no seguros (como HTTP, FTP o Telnet).
*   **Atacante Activo:** Interviene en el canal de comunicaciones para modificar paquetes, alterar datos, inyectar transmisiones falsas o denegar servicios. Un ejemplo claro es un ataque **Man-in-the-Middle (MITM)** en una red Wi-Fi pública (un aeropuerto o estación), donde el ciberdelincuente suplanta la dirección MAC del router local mediante envenenamiento ARP (*ARP Spoofing*) para desviar todo el tráfico de la víctima hacia su ordenador, capturando contraseñas bancarias y sesiones HTTP activas.

---

### Taxonomía clásica de atacantes (Ciberactores)
Detrás de las amenazas —dejando a un lado los incidentes fortuitos o climáticos— existe una variada tipología de atacantes con distintas motivaciones, destrezas y objetivos profesionales:

*   **Hacker (White Hat):** Persona con un profundo conocimiento tecnológico que utiliza sus habilidades para descubrir vulnerabilidades de forma autorizada, con fines constructivos, ayudando a las empresas a fortificar sus infraestructuras mediante auditorías de seguridad y hacking ético.
*   **Cracker (Black Hat):** Actor malicioso que vulnera sistemas informáticos de forma ilegal y sin autorización con fines lucrativos, de sabotaje, robo de datos confidenciales o espionaje corporativo.
*   **Grey Hat:** Perfil intermedio que actúa de forma no autorizada pero sin fines destructivos. Por ejemplo, buscan fallos de seguridad en grandes corporaciones sin consentimiento y, tras encontrarlos, notifican a la empresa de manera informal a cambio de reconocimiento o una recompensa.
*   **Lamer y Script Kiddies:** Aficionados con escasos o nulos conocimientos técnicos de programación o redes que se limitan a descargar y ejecutar herramientas automáticas creadas por crackers reales en Internet, buscando notoriedad o realizar pequeños sabotajes sin comprender cómo funciona el exploit subyacente.
*   **Phreaker:** Especialistas orientados al hackeo, manipulación técnica y explotación de debilidades de los sistemas de telecomunicaciones y telefonía fija/móvil para realizar llamadas gratuitas o redirigir el tráfico de voz.
*   **Spammer:** Emisores masivos de correos electrónicos no solicitados (spam), habitualmente de carácter publicitario o con enlaces que dirigen a fraudes en la red.
*   **Phisher:** Ciberdelincuente especializado en suplantar la identidad corporativa de entidades de confianza para pescar credenciales privadas, tal como se analiza en la sección de ingeniería social.
*   **Scammer:** Estafadores de internet que utilizan la ingeniería social para engañar de forma directa a usuarios (p. ej., fraudes sentimentales, falsos premios de lotería o inversiones fraudulentas en criptomonedas).
*   **Ciberterroristas:** Grupos organizados y altamente cualificados respaldados a menudo por gobiernos u organizaciones extremistas que ejecutan ciberataques militares o sabotajes lógicos con el fin de dañar infraestructuras críticas nacionales de un país (comunicaciones, suministros de agua, redes de electricidad o centrales nucleares).

---

## 7. Clasificación de las medidas de seguridad

Cualquier mecanismo defensivo que decidamos aplicar en un sistema de información se clasifica atendiendo a una doble dimensión (criterio organizativo y criterio temporal):

### 1. Criterio Organizativo: Seguridad Física vs. Seguridad Lógica
*   **Seguridad Física:** Conjunto de medidas de protección perimetral, barreras físicas y controles ambientales destinadas a proteger los recursos **tangibles** de la empresa (CPDs, cableado estructurado, hardware de red y servidores) contra el acceso no autorizado de personas, vandalismo, robos o desastres ambientales (fuego, agua, picos de tensión).
*   **Seguridad Lógica:** Engloba los mecanismos de protección de activos **intangibles** de la organización (software, bases de datos, contraseñas, ficheros y procesos). Su objetivo es bloquear intrusiones lógicas y accesos remotos no autorizados a través de la red de datos.

### 2. Criterio Temporal (Momento de Acción): Seguridad Activa vs. Seguridad Pasiva
*   **Seguridad Activa:** Conjunto de medidas y herramientas de carácter **preventivo** destinadas a evitar que el incidente de seguridad llegue a ocurrir. Actúa antes del suceso (ejemplos: instalación de un cortafuegos, configuración de una directiva robusta de contraseñas locales o la implementación de autenticación multifactor).
*   **Seguridad Pasiva:** Conjunto de medidas de carácter **corrector o paliativo** que actúan tras la materialización de un ataque o fallo del sistema. Su objetivo no es evitar el desastre (el cual ya ha ocurrido), sino minimizar de inmediato el impacto del daño y permitir la rápida restauración del servicio (ejemplos: restauración de copias de seguridad de los sistemas, uso de redundancia RAID, o activación de un plan de recuperación de desastres).

---

### La analogía automovilística de la seguridad
Para que el alumnado asimile con total claridad la diferencia entre seguridad activa y pasiva, suele utilizarse la siguiente analogía con los sistemas de seguridad vial de un vehículo:

| Concepto | Ámbito Automovilístico | Ámbito Informático |
| :--- | :--- | :--- |
| **Seguridad Activa** (Evitar el accidente) | * Sistemas de frenado ABS.<br>* Control de estabilidad ESP.<br>* Dirección asistida.<br>* Neumáticos de calidad. | * Cortafuegos (firewalls) perimetrales.<br>* Sistemas antivirus actualizados.<br>* Robustez de contraseñas de red.<br>* Parches de seguridad del S.O. |
| **Seguridad Pasiva** (Minimizar el daño tras chocar) | * Despliegue de los airbags.<br>* Cinturones de seguridad pretensados.<br>* Carrocería deformable.<br>* Chasis de deformación programada. | * Copias de seguridad remotas.<br>* Clústeres de alta disponibilidad.<br>* Volúmenes RAID tolerantes a fallos.<br>* Análisis forense y logs. |

---

## 8. Aplicaciones de medidas de seguridad en el centro docente

A continuación, analizaremos de manera conceptual cómo se despliegan estos principios en el entorno real de un centro educativo:

### 8.1. Aplicación de Medidas de Seguridad Física
Los centros educativos modernos y los Centros de Procesamiento de Datos (CPD) aplican de forma integrada las siguientes protecciones físicas:

*   **Controles de presencia y accesos perimetrales:** Puertas blindadas, uso de tarjetas magnéticas, lectores RFID/NFC, cámaras de vigilancia CCTV y, en entornos corporativos críticos, sistemas de autenticación biométrica (como huella dactilar o reconocimiento venoso de palma Fujitsu PalmSecure).
*   **Sistemas de climatización y control ambiental:** Sensores de humedad, extractores y sistemas de aire acondicionado de precisión para mantener la temperatura óptima de los armarios de servidores rack en el cuarto de comunicaciones o telemática.
*   **Sistemas de extinción pasivos:** Uso de puertas cortafuegos, extintores de CO₂ (para evitar dañar la electrónica al apagar un conato de incendio) y sistemas de aspersión de agua nebulizada o gases limpios (como gas inergen) en CPDs que sofocan el fuego disminuyendo la concentración de oxígeno en la sala de forma controlada.

!!! note "Sistemas de Alimentación Ininterrumpida (SAI / UPS)"
    Un SAI es un componente de seguridad que actúa como un acumulador de energía (baterías). Ante una fluctuación o corte completo del suministro de red eléctrica del proveedor, el SAI estabiliza de inmediato la señal y suministra corriente de forma ininterrumpida a los servidores y equipos de comunicaciones conectados, dándoles el tiempo de autonomía necesario para que un agente de software automatizado realice un apagado ordenado del sistema operativo mediante un script.

!!! success "Caso práctico de campo: Incidencias de tormenta eléctrica"
    En centros educativos con instalaciones eléctricas deficientes o ubicaciones geográficas rurales (como la zona elevada del CIPFP de Cheste), es muy frecuente que las tormentas veraniegas provoquen picos severos de tensión por la caída de rayos en las proximidades. 
    
    Sin la presencia de un SAI con regulador de tensión incorporado o un protector de sobretensión, estos picos queman de inmediato fuentes de alimentación de servidores de aula, placas de switches o los propios puertos RJ45 de datos de la red LAN escolar. El SAI actúa, por tanto, como una medida de seguridad física (actúa sobre el cableado físico) y pasiva (mitiga el impacto de un corte eléctrico en curso).

---

### 8.2. Pincelada Conceptual de Medidas Lógicas
En esta primera unidad didáctica, el alumno debe conocer a nivel conceptual las medidas lógicas clave que se detallarán de forma práctica a lo largo del curso:

1.  **Antivirus (Antimalware):** Soluciones lógicas instaladas localmente destinadas a escanear en tiempo real la firma, comportamiento y heurística de los archivos que entran en memoria, sirviendo como contramedida de seguridad activa y lógica.
2.  **Cortafuegos (Firewalls):** Herramientas lógicas de seguridad activa y lógica perimetral que filtran selectivamente el tráfico de red, analizando las direcciones IP de origen/destino y los puertos lógicos TCP/UDP, bloqueando de inmediato cualquier conexión no autorizada hacia los servicios internos del centro educativo.
3.  **Cifrado de datos (Criptografía):** Aplicación práctica de algoritmos criptográficos para proteger datos locales confidenciales almacenados en discos duros, soportes extraíbles o archivos antes de ser transmitidos, asegurando que si un atacante accede físicamente a ellos de forma ilícita (p. ej., el robo de un pendrive con las notas), no pueda interpretar su contenido sin la clave privada.
4.  **Uso de Certificados Digitales:** Integración de certificados HTTPS (como Let's Encrypt) para fortificar las comunicaciones de cara al acceso cifrado a la intranet del centro o a las aulas virtuales.

---

## 9. El factor humano y la ingeniería social

## 9.1. El Factor Humano: El eslabón más débil y el "Cortafuegos Humano"

Como administradores de sistemas en red, es común centrar todos los esfuerzos en la implantación de medidas puramente técnicas y lógicas (antivirus, firewalls, copias de seguridad o cifrado). Sin embargo, la seguridad absoluta no existe. La inmensa mayoría de los incidentes graves de seguridad en entornos corporativos reales no se producen por un fallo en el software, sino por una debilidad en el componente más complejo de proteger: **el factor humano**.

El célebre experto e investigador en ciberseguridad **Bruce Schneier** resume la excesiva confianza en la tecnología con esta conocida frase:

!!! quote "Bruce Schneier"
    "Si piensas que la tecnología puede solucionar tus problemas de seguridad, está claro que ni entiendes los problemas ni entiendes la tecnología."

En la misma línea, **Kevin Mitnick** (histórico hacker de los años 90 y posterior consultor de ciberseguridad) destacaba que la inversión técnica es inútil si no se capacita a las personas que operan el sistema:

!!! quote "Kevin Mitnick"
    "Las organizaciones gastan millones de dólares en firewalls y dispositivos de seguridad, pero tiran el dinero porque ninguna de estas medidas cubre el eslabón más débil de la cadena de seguridad: la gente que usa y administra los ordenadores."

Por este motivo, en la ciberseguridad moderna se fomenta el concepto de **"Cortafuegos Humano"** (*Human Firewall*): la necesidad de concienciar, entrenar y formar de manera continua a los usuarios para que actúen como la primera línea de defensa activa de la empresa.

---

## 9.2. ¿Qué es la Ingeniería Social?

!!! info "Definición"
    La **ingeniería social** es el conjunto de técnicas de manipulación psicológica, engaño y persuasión utilizadas por los atacantes para conseguir que usuarios legítimos revelen información confidencial (como credenciales de acceso), realicen acciones dañinas para la organización o se salten los protocolos de seguridad establecidos.

A diferencia de los ataques técnicos que buscan vulnerabilidades en el código o en las malas configuraciones de los servicios, el ingeniero social explota **vulnerabilidades en el comportamiento y la psicología humana**. Los atacantes suelen aprovecharse de aspectos fundamentales de la conducta humana:

*   **La voluntad de ayudar:** Cooperar de buena fe ante un supuesto problema técnico urgente de un compañero.
*   **El respeto a la autoridad:** Ceder ante llamadas o correos que suplantan a directivos, auditores o personal de soporte técnico.
*   **El temor a las consecuencias:** Reaccionar con pánico ante amenazas de bloqueo de cuentas de usuario, sanciones administrativas o pérdidas financieras.
*   **La curiosidad:** El impulso natural de querer saber qué contiene un archivo o dispositivo desconocido.

---

## 9.3. Principales Vectores de Ataque de Ingeniería Social

Aunque los engaños pueden adoptar múltiples formas creativas, en la actualidad los administradores de sistemas deben saber identificar y mitigar de forma precisa tres vectores fundamentales:

### A) Phishing (Pesca de credenciales)

Es el vector de ataque más común, masivo y dañino en Internet. Consiste en el envío de correos electrónicos (o mensajes por otros canales como SMS, conocido como *Smishing*) que suplantan la identidad visual, logotipos y firmas de una entidad de total confianza (bancos, agencias gubernamentales, empresas de mensajería o servicios en la nube).

!!! danger "Mecánica del ataque"
    El mensaje suele contener un tono de extrema urgencia (ej. *"Su cuenta será suspendida en 24 horas por actividad sospechosa"*). Incluye un enlace que redirige a la víctima a una página web falsa clonada de forma idéntica por el atacante. Al introducir su usuario y contraseña, el atacante captura las credenciales en tiempo real.

### B) Vishing (Phishing de voz)

El **vishing** utiliza llamadas telefónicas para ejecutar el engaño. Es un método sumamente peligroso porque la voz humana transmite una falsa sensación de cercanía, urgencia y autoridad que un texto escrito difícilmente puede replicar.

!!! danger "Mecánica del ataque"
    El atacante llama suplantando, por ejemplo, al personal del departamento de soporte informático de la Conselleria, a un técnico de mantenimiento de red o a un gestor de seguridad bancaria. Con el pretexto de solucionar una "incidencia crítica de seguridad en su puesto de trabajo", persuade a la víctima para que le dicte su contraseña o le facilite el código de doble factor (MFA) que acaba de recibir en su móvil.

### C) Baiting (El señuelo físico)

El **baiting** aprovecha directamente la curiosidad física de los usuarios y su falta de precaución al conectar medios de almacenamiento externos.

!!! danger "Mecánica del ataque"
    El atacante abandona de forma deliberada un dispositivo físico de almacenamiento (habitualmente un pendrive USB) en un área común o estratégica de la empresa (aparcamiento, cafetería, recepción, aseos o sala de profesores). Para despertar el interés de la víctima, el USB suele llevar una etiqueta llamativa escrita a mano como *"Salarios Dirección 2026"* o *"Exámenes Finales"*. Cuando la víctima lo encuentra y lo conecta a su ordenador corporativo para ver qué contiene, se ejecuta automáticamente un script o archivo malicioso que compromete la seguridad de la máquina y de la red local.

---

## 9.4. Medidas de Prevención y Mitigación

La tecnología por sí sola no puede detener un ataque de ingeniería social. Las defensas más efectivas se basan en buenas prácticas organizativas:

!!! success "Buenas prácticas de defensa pasiva y activa"
    1. **Políticas corporativas de "No Solicitud":** Establecer por normativa interna que ningún administrador ni técnico de soporte solicitará jamás contraseñas, claves privadas o códigos MFA por teléfono o correo.
    2. **Verificación fuera de banda (*Out-of-band*):** Si se recibe una solicitud inusual o sospechosa de transferencia de fondos o cambio de claves por parte de un directivo o proveedor, se debe confirmar a través de un canal de comunicación alternativo y oficial (por ejemplo, llamando a un número de teléfono conocido y guardado de antemano, nunca al número que figura en el correo sospechoso).
    3. **Fomento de la cultura de reporte:** En lugar de castigar el error, las organizaciones deben animar a los empleados a reportar de forma inmediata cualquier llamada sospechosa o correo extraño al equipo de administración informática para que se puedan aplicar contramedidas tempranas.

---

## 10. Conclusiones

En esta unidad didáctica hemos aprendido a:

*   **Diferenciar** entre el concepto corporativo amplio de la *seguridad de la información* y la especialización tecnológica que supone la *seguridad informática*.
*   **Identificar** los principios lógicos que sustentan la ciberseguridad, dominando la tríada CIA (Confidencialidad, Integridad y Disponibilidad), así como los servicios adicionales de autenticación, control de accesos, no repudio y trazabilidad.
*   **Dominar** el lenguaje técnico propio del análisis de riesgos: caracterizar activos, amenazas lógicas y físicas, vulnerabilidades del sistema, impacto, ataques activos o pasivos, y riesgos potenciales.
*   **Clasificar** cualquier medida de protección según su naturaleza física/lógica y su momento de acción activo/pasivo.
*   **Reconocer** al factor humano como la capa más vulnerable y asimilar la ingeniería social y sus vectores tácticos (*phishing*, *vishing*, *baiting*) como amenazas críticas que requieren un "Cortafuegos Humano".
