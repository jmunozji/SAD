# UD3. Criptografía y Mecanismos de Certificación Digital

!!! note "Objetivos de la Unidad (Duración: 8 Horas)"
    * Aplicar técnicas criptográficas en el almacenamiento y la transmisión de la información para garantizar la seguridad de los datos (CE 1.g).
    * Proteger la privacidad, autenticidad y confidencialidad mediante el uso de certificados digitales y correo seguro (CE 2.f).
    * Comprender y diferenciar los métodos criptográficos clásicos y modernos (simétricos, asimétricos e híbridos).
    * Analizar las propiedades matemáticas y aplicaciones prácticas de las funciones hash (resumen digital).
    * Dominar el mecanismo técnico de la firma digital y el funcionamiento de una Infraestructura de Clave Pública (PKI) bajo el estándar X.509.
    * Conocer la arquitectura criptográfica del DNI electrónico (DNIe) y los protocolos seguros SSL/TLS/HTTPS.

---

## 3.1. Introducción y Fundamentos de Criptografía

### 3.1.1. ¿Por qué cifrar en la Sociedad de la Información?
Desde la antigüedad, la humanidad ha tenido la necesidad de proteger el intercambio de mensajes confidenciales. En la era digital actual, este requerimiento se vuelve crítico debido a la exposición masiva de los datos en canales de comunicación no seguros (redes Wi-Fi abiertas, correo electrónico, voz sobre IP, redes móviles) y el uso constante de dispositivos de almacenamiento portátiles (pendrives, discos externos, teléfonos inteligentes) propensos a pérdidas o robos.

### 3.1.2. Criptografía Clásica: Historia y Clasificación
El término **criptografía** proviene del griego *kryptós* (oculto) y *graphía* (escritura): la ciencia que estudia la transformación de un texto claro en un texto cifrado ininteligible para terceros no autorizados.

Históricamente, los métodos de cifrado clásico se dividen en dos grandes familias:

1. **Sistemas de Transposición:** Consisten en descolocar o reordenar los caracteres del mensaje sin alterar las letras originales.
    * **La Escítala Espartana (Siglo V a.C.):** Primer método criptográfico documentado. Consistía en enrollar una cinta de cuero sobre un bastón de madera con un diámetro específico. Al escribir a lo largo del bastón y desenrollar la cinta, las letras quedaban desordenadas de forma aleatoria, siendo únicamente legible al enrollarla en una vara del mismo diámetro exacto.
    ![Escítala](img/escitala.png)
      Como podemos ver en la imagen, el mensaje es «es el primer método de encriptación conocido», pero en la cinta lo que se podría leer es «EMCCSEROETINLOPOPDTCROAIIDCDMEIOEEORNN»
    * **Transposición simple y múltiple:** Métodos matriciales donde el texto se escribe en filas y se lee en columnas según una clave acordada.
        Ejemplo:

        Mensaje: ATACAR HOY

        Clave: SOL

        Se distribuyen las 8 letras del mensaje A-T-A-C-A-R-H-O-Y horizontalmente. La última fila queda con 2 letras, así que se añade una X para cerrar el bloque:

        ![transposicion](img/transposicion.png)

        Se leen las columnas de arriba abajo según la prioridad numérica de la clave. Primero la columna cuya letra en la clave es anterior alfabéticamente, en este caso la L. Luego la siguiente, que sería la O y finalmente la última, la S:

           * Columna 1 (letra L): A R Y X

           * Columna 2 (letra O): T A O X

           * Columna 3 (letra S): A C H X

        Criptograma final: ARYX TAOX ACHX (o agrupado: ARYXTAOXACHX).

        En la transposición múltiple se aplica una nueva transposición al mensaje encriptado del paso anterior con una nueva clave


2. **Sistemas de Sustitución:** Consisten en reemplazar unos caracteres o símbolos del alfabeto original por otros según una regla predefinida.
    * **Tablero de Polibio (Siglo II a.C.):** Sustitución por pares de letras o números que representan las coordenadas (fila y columna) de una matriz de 5x5 donde se ubica cada letra del alfabeto.
    ![Polibio](img/polibios.png)
    * **Cifrador del César (Siglo I a.C.):** Sustitución monoalfabética utilizada por Julio César. Consiste en desplazar cada letra del alfabeto un número fijo *n* de posiciones hacia la derecha (en el ejemplo **n=3**).
       ![César](img/cesar.png)
        
        Mensaje del César a Cleopatra: «sic amote ut sin ete iam viverem non posit» (de tal manera te amo que sin ti no podría vivir).
        
        Para traducir el mensaje necesitamos los dos alfabetos el claro y el cifrado, que son los dos alfabetos latinos del cifrador del César.
        
        Si nos fijamos en el alfabeto cifrado el mensaje oculto debe corresponderse con el siguiente: VMF DPRXI YX VMQ IXI MDP ZMZIUIP QRQ SRVMX

    * **Cifrador de Alberti (Siglo XV) y Vigenère (Siglo XVI):** Sustitución polialfabética. León Battista Alberti propuso alternar entre varios alfabetos, pero fue Blaise de Vigenère quien XVI desarrolle la idea de Alberti. 
    
        El cifrador de Vigenère utiliza veintiséis alfabetos cifrados, obteniéndose cada uno de ellos comenzando con la siguiente letra del anterior, es decir, el primer alfabeto cifrado se corresponde con el cifrador del César con un cambio de una posición, de la misma manera para el segundo alfabeto, cifrado con el cifrador del César de dos posiciones. 
      
        Para cifrar hace falta una palabra clave que se va repitiendo hasta llegar a la longitud del mensaje a cifrar. Cada letra del mensaje original nos da la columna y cada letra de la clave la fila, que nos perminten obtener la letra en el mensaje cifrado.
      
        ![Vigenère](img/vigenere.jpeg)


!!! info "Esteganografía: El arte del ocultamiento"
    A diferencia de la criptografía (que hace ininteligible el mensaje pero no oculta su existencia), la **esteganografía** busca ocultar la presencia del propio mensaje dentro de un archivo portador de apariencia inofensiva (como una imagen PNG, un archivo de audio WAV o un archivo ejecutable).
    
    *Una herramienta clásica de línea de comandos en GNU/Linux para realizar ejercicios de esteganografía es `steghide`.*

### 3.1.3. Criptoanálisis de los Cifrados Clásicos
El **criptoanálisis** es la disciplina que estudia las técnicas para romper o descifrar códigos sin conocer la clave legítima:

* **Ataques de Fuerza Bruta:** Probar sistemáticamente todas las claves posibles. El cifrador del César solo posee 25 claves posibles en el alfabeto latino, por lo que es trivialmente roturable por fuerza bruta en segundos.

* **Análisis de Frecuencias:** En los cifrados por sustitución monoalfabética, la frecuencia de aparición de cada letra se mantiene. En español, las letras 'E', 'A' y 'O' son las más comunes; al analizar las frecuencias del texto cifrado, se puede deducir la correspondencia con las letras claras.

* **Método Kasiski:** Permite romper la criptografía polialfabética de Vigenère identificando repeticiones de patrones en el texto cifrado para deducir la longitud de la clave utilizada.

### 3.1.4. El Principio de Kerckhoffs
En el siglo XIX, Auguste Kerckhoffs estableció un axioma fundamental que rige toda la ciberseguridad moderna:

!!! quote "Principio de Kerckhoffs"
    "La seguridad de un sistema de cifrado debe recaer exclusivamente en el secreto de la **clave**, y no en el secreto del **algoritmo**."

Es decir, aunque el algoritmo criptográfico sea de dominio público y su código sea analizado por toda la comunidad internacional (código abierto), un atacante no debe ser capaz de descifrar el mensaje si desconoce la clave secreta utilizada.

### 3.1.5. Servicios de Seguridad
La criptografía moderna sirve para garantizar cuatro servicios fundamentales de la información:

* **Confidencialidad:** Garantizar que los datos solo sean legibles por receptores autorizados.
* **Integridad:** Garantizar que los datos no hayan sido alterados o manipulados en tránsito.
* **Autenticidad:** Confirmar la identidad legítima del emisor del mensaje.
* **No Repudio:** Impedir que el emisor de una comunicación o transacción pueda negar su autoría.

---

## 3.2. Criptografía Simétrica (Clave Secreta)

### 3.2.1. Concepto y Funcionamiento
La **criptografía simétrica** (o de clave privada/secreta) se basa en el uso de **una única clave compartida** tanto para el proceso de cifrado como para el de descifrado. Tanto el emisor como el receptor deben ponerse de acuerdo de forma previa sobre qué clave utilizarán.

![Clave simétrica](img/ClaveSimetrica.png)

Un ejemplo histórico de cifrado electromecánico simétrico fue la máquina **Enigma** utilizada durante la Segunda Guerra Mundial, cuyo código fue roto por el equipo de matemáticos de Alan Turing en Bletchley Park.

### 3.2.2. Clasificación de Algoritmos Simétricos
Atendiendo a la forma en que se procesan los datos en memoria, los algoritmos simétricos se clasifican en:

1. **Algoritmos de Bloque:** Dividen la información de entrada en bloques de tamaño fijo (por ejemplo, 64 o 128 bits) y cifran cada bloque de manera independiente antes de ensamblar el resultado. * *Ejemplos:* 
  
      * **DES** (Data Encryption Standard, 56 bits - obsoleto)
      * **3DES** (Triple DES - obsoleto) y 
      * **AES** (Advanced Encryption Standard / Rijndael, con tamaños de clave de 128, 192 o 256 bits, estándar global actual).

2. **Algoritmos de Flujo:** Cifran la información de forma continua, bit a bit o byte a byte, conforme se van generando o transmitiendo los datos. Son ideales para transmisiones de voz o vídeo en tiempo real. *Ejemplos:*
   
      * **RC4** (utilizado históricamente en WEP/WPA, hoy desaconsejado) y 
      * **ChaCha20** (muy rápido en dispositivos móviles).

### 3.2.3. Desventajas y Limitaciones de la Criptografía Simétrica
Aunque los algoritmos simétricos modernos como AES son extremadamente rápidos y eficientes en términos de rendimiento computacional, presentan dos inconvenientes críticos:

1. **El Problema del Intercambio Inicial de Claves:** ¿Cómo se transmite de forma segura la clave compartida por primera vez a través de una red insegura como Internet para que el receptor pueda descifrar el mensaje?
2. **Escalabilidad y Gestión de Claves:** El número de claves necesarias para que *N* usuarios puedan comunicarse de forma segura de manera individualizada crece de forma exponencial según la fórmula:
   ![Número de claves en criptografía simétrica](img/SimetricaNumClaves.png)
   *Para una red de solo 1.000 usuarios, se requeriría generar y almacenar de forma segura **499.500 claves simétricas diferentes**.*

### 3.2.4. Práctica de criptografía simétrica

!!! info "Ahora que conoces la teoría es el momento de practicar."
    **[Práctica 3.1: Cifrado Simétrico con GnuPG (GPG)](P01.md)**
    
       *En esta práctica utilizarás la herramienta GNU Privacy Guard (`gpg`) desde la línea de comandos en GNU/Linux para cifrar y   descifrar archivos de texto utilizando algoritmos simétricos robustos (como AES256) mediante claves secretas compartidas.*

---

## 3.3. Criptografía Asimétrica (Clave Pública)

### 3.3.1. Concepto y Par de Claves
A mediados de la década de 1970 (con las investigaciones de Diffie, Hellman, Rivest, Shamir y Adleman), surgió la **criptografía asimétrica** o de clave pública para resolver las debilidades del cifrado simétrico.

En un sistema asimétrico, cada participante dispone de **un par de claves matemáticamente enlazadas**:

* **Clave Pública (.pub):** Es de libre acceso y se distribuye abiertamente a cualquier usuario o servidor con el que queramos comunicarnos.
* **Clave Privada (.priv):** Es secreta y debe ser custodiada bajo el control exclusivo de su propietario. Nunca debe transmitirse ni revelarse a nadie.

![Criptografía asimétrica](img/ClaveAsimetrica.png)


### 3.3.2. Funciones Unidireccionales con Trampa
Las claves se generan simultáneamente utilizando propiedades matemáticas de **funciones de un solo sentido con trampa** (*trapdoor one-way functions*): es computacionalmente fácil calcular la clave pública a partir de los datos iniciales, pero resulta **computacionalmente imposible** calcular la clave privada a partir de la clave pública conocida.

Por "computacionalmente imposible" se entiende que, con la potencia de cálculo actual, un ataque de fuerza bruta tardaría décadas o siglos en despejar la clave.

### 3.3.3. Ventajas y Algoritmos Asimétricos Principales

* **Ventajas Principales:**
    1. *Resuelve el intercambio de claves:* Ya no es necesario enviar secretos por canales inseguros; basta con publicar la clave pública.
    2. *Alta Escalabilidad:* Para *N* usuarios, solo se requieren *N* pares de claves (*2N* claves en total en todo el sistema).

    ![Número de claves en criptografía asimétrica](img/AsimetricaNumClaves.png)

* **Algoritmos Asimétricos Destacados:**
    * **RSA:** Basado en la dificultad matemática de factorizar el producto de dos números primos muy grandes. Es el estándar más extendido.
    * **ElGamal:** Basado en la complejidad del problema del logaritmo discreto.
    * **ECC (Criptografía de Curva Elíptica):** Ofrece el mismo nivel de seguridad que RSA utilizando claves significativamente más cortas, reduciendo la carga de procesamiento y consumo de batería en dispositivos IoT y móviles.

### 3.3.4. Desventajas de la Criptografía Asimétrica

Las principales desventajas de la criptografía asimétrica son:

* **Lentitud Computacional:** Los algoritmos de clave pública son entre 100 y 1.000 veces más lentos que los simétricos debido a la complejidad de las operaciones matemáticas con números gigantescos.
* **Mayor Tamaño de Datos:** El texto cifrado resulta sensiblemente mayor que el texto original en claro.

### 3.3.5. Práctica de criptografía asimétrica y gestión de cables con GPG


!!! info "Ahora que conoces la teoría es el momento de practicar."
    **[Práctica 3.2: Criptografía Asimétrica y Gestión de Claves con GPG](P02.md)**
    
       *En esta práctica generarás tu propio par de claves asimétricas (K_pub y K_priv) utilizando GnuPG, importarás la clave pública de otras personas y cifrarás un documento para garantizar que solo el receptor legítimo pueda descifrarlo con su clave privada.*


---

## 3.4. Criptografía Híbrida

### 3.4.1. La Solución Combinada
Dado que el cifrado simétrico es muy rápido pero inseguro al compartir claves, y el asimétrico es muy seguro pero excesivamente lento, la criptografía moderna utiliza en la práctica un enfoque combinado denominado **Criptografía Híbrida**.

La criptografía híbrida utiliza la **criptografía asimétrica** únicamente para intercambiar de forma segura una clave aleatoria temporal, y la **criptografía simétrica** para cifrar el volumen real del documento o la sesión de red.

### 3.4.2. Mecánica de una Comunicación Híbrida Paso a Paso
Imaginemos que el Emisor quiere enviar un archivo voluminoso al Receptor:

1. **Generación de Clave de Sesión:** El Emisor genera automáticamente una clave simétrica aleatoria efímera (llamada **clave de sesión**, ej. AES-256).
2. **Cifrado del Mensaje (Simétrico):** El Emisor cifra el archivo de datos utilizando la clave de sesión simétrica (operación muy rápida).
3. **Cifrado de la Clave de Sesión (Asimétrico):** El Emisor obtiene la clave pública del Receptor (K_pub_Receptor) y cifra con ella únicamente la pequeña clave de sesión.
4. **Envío Conjunto:** El Emisor transmite al Receptor el archivo cifrado simétricamente junto con la clave de sesión cifrada asimétricamente.
5. **Descifrado por el Receptor:** El Receptor utiliza su clave privada (K_priv_Receptor) para descifrar la clave de sesión y, con esta última, descifra instantáneamente el archivo completo.

![Criptografía híbrida](img/CriptografiaHibrida.png)


Este sistema es la base del funcionamiento de protocolos seguros como **PGP/GPG, SSH y HTTPS/TLS**.

---

## 3.5. Funciones Hash o Resumen Digital

Empecemos viendo este video introductorio a las funciones hash:

![type:video](https://www.youtube.com/embed/FRBIc0udwv0)

### 3.5.1. Concepto de Función Hash
Una **función hash** (o función de resumen) es un algoritmo matemático que transforma cualquier conjunto de datos de entrada (desde una sola palabra hasta una imagen ISO de varios Gigabytes) en una cadena alfanumérica de **longitud fija** que actúa a modo de *huella digital* única del contenido.

A diferencia de los cifrados simétricos y asimétricos, las funciones hash **no utilizan claves** y son **procesos estrictamente irreversibles** (unidireccionales).

### 3.5.2. Las 6 Propiedades Criptográficas Fundamentales (Píldora UPM)
Para que un algoritmo hash sea considerado criptográficamente seguro, debe cumplir obligatoriamente seis propiedades:

1. **Unidireccionalidad:** Dado un resumen h(M), debe ser computacionalmente imposible calcular o recuperar el mensaje original M.
2. **Compresión:** Independientemente del tamaño de la entrada, la salida siempre tendrá exactamente el mismo número de bits (por ejemplo, SHA-256 siempre entrega 256 bits).
3. **Facilidad y Rapidez de Cálculo:** El cálculo de h(M) a partir de M debe ser ágil en tiempo de ejecución.
4. **Difusión de Bits o Efecto Avalancha:** Si se modifica tan solo un bit o carácter del mensaje original M, el nuevo hash resultante cambiará aproximadamente en el 50% de sus bits de forma impredecible.
5. **Resistencia Débil a Colisiones:** Conocido un mensaje M, es computacionalmente imposible encontrar otro mensaje distinto M' tal que h(M) = h(M').
6. **Resistencia Fuerte a Colisiones:** Es computacionalmente imposible encontrar un par aleatorio de mensajes distintos (M, M') que produzcan el mismo resumen h(M) = h(M') (evitando ataques basados en la *Paradoja del Cumpleaños*).

### 3.5.3. Evolución de los Algoritmos Hash
* **MD5 (128 bits) y SHA-1 (160 bits):** **Obsoletos**. Se han demostrado colisiones prácticas en laboratorio, por lo que están desaconsejados para su uso en ciberseguridad.
* **SHA-2 (SHA-256, SHA-512):** Estándar actual ampliamente utilizado en firmas digitales, certificados y cadenas de bloques (*blockchain*).
* **SHA-3:** Última familia estandarizada por el NIST con una estructura matemática completamente renovada (algoritmo Keccak).

### 3.5.4. Aplicaciones Principales de los Hashes
* **Verificación de Integridad:** Permite comprobar si una ISO descargada de Internet se ha transmitido sin corrupciones o si un archivo en disco ha sido modificado por malware.
* **Almacenamiento Seguro de Contraseñas:** Las contraseñas de usuarios en sistemas operativos (como `/etc/shadow` en Linux) nunca se guardan en texto claro. Se almacena el hash de la clave combinado con una cadena aleatoria (*salting*) utilizando funciones deliberadamente lentas (como **PBKDF2, bcrypt o Argon2**) para dificultar ataques por diccionario o fuerza bruta.
* **Optimización de la Firma Digital:** Permite firmar digitalmente documentos de cientos de Megabytes aplicando la cifra asimétrica únicamente sobre los pocos bits del resumen hash. (Esto lo veremos en el próximo apartado.)

!!! note "Práctica"
    **Práctica Moodle: Funciones hash**
        
    *Ahora puedes hacer la práctica "[Funciones Hash](P03.md)"

---

## 3.6. Firma Digital

### 3.6.1. Concepto y Servicios
La **firma digital** es un mecanismo criptográfico que equivale a la firma manuscrita tradicional en el ámbito electrónico. Su objetivo **no es proporcionar confidencialidad** (no oculta el documento), sino garantizar de forma simultánea tres servicios:

* **Autenticidad:** Prueba de forma inequívoca la identidad del emisor.
* **Integridad:** Confirma que el documento no ha sufrido ninguna modificación desde que fue firmado.
* **No Repudio en Origen:** El emisor no puede negar haber firmado el documento.

### 3.6.2. Mecánica de Generación de la Firma
Para firmar un documento digital, el emisor realiza el siguiente procedimiento:

1. Aplica una función hash (ej. SHA-256) sobre el documento original para obtener su resumen h_1.
2. Cifra el resumen h_1 utilizando su propia **clave privada** (K_priv_Emisor). Este resumen cifrado constituye la **firma digital**.
3. Se envía el documento original en claro junto con la firma digital adjunta.

![Firma digital](img/FirmaDigital.png)

!!! tip Uso de la clave privada en la firma digital
    Fíjate que hasta ahora sólo habíamos usado la clave privada para descifrar mensajes encriptados mediante la clave pública del receptor.

    En la firma digital es el propio usuario el que utiliza su clave privada para firmar el mensaje y será el receptor el que usará la clave pública del firmante para comprobar la firma realizada.

### 3.6.3. Mecánica de Verificación por el Receptor
Cuando el receptor recibe el documento y la firma, realiza el siguiente proceso informático:

1. Descifra la firma digital utilizando la **clave pública del emisor** (K_pub_Emisor), obteniendo el resumen hash original h_1.
2. Calcula de nuevo el resumen hash sobre el documento recibido utilizando la misma función hash, obteniendo h_2.
3. **Comparación:**
    * Si h_1 = h_2: La firma es **VÁLIDA** (el emisor posee la clave privada y el documento está intacto).
    * Si h_1 distinto de h_2: La firma es **NULA** (el documento fue modificado o la clave no corresponde al emisor).

La siguiente imagen muestra el proceso completo de firma y verificación.

![Firma digital completa](img/FirmaDigital2.png)

!!! note "Práctica"
    **Práctica Moodle: Firma digital**
        
    *Ahora puedes hacer la práctica "[Firma digital](P04.md)*

---

## 3.7. Infraestructura de Clave Pública (PKI) y Modelos de Confianza

### 3.7.1. El Problema Fundamental: La Confianza en la Clave Pública
La criptografía asimétrica y la firma digital proporcionan herramientas matemáticas eficaces para garantizar la confidencialidad, la autenticidad, la integridad y el no repudio. Sin embargo, todos estos servicios colapsan si no podemos resolver la pregunta clave: **¿cómo garantizamos que una clave pública pertenece realmente a quien dice ser y no a un impostor?**

Cualquier usuario o atacante puede generar libremente un par de claves asimétricas en su ordenador y publicar la clave pública en Internet afirmando falsamente ser un banco, una administración pública o un compañero de trabajo. Si un emisor cifra un mensaje con la clave pública de un suplantador (ataque *Man-in-the-Middle* / MITM), el atacante podrá interceptar y descifrar la información con su clave privada correspondiente.

Para resolver este dilema de confianza surgen dos aproximaciones fundamentales:

1. **Red de Confianza (*Web of Trust* - PGP/GPG):** Modelo descentralizado donde los propios usuarios firman las claves públicas de las personas a las que conocen personalmente. Es un modelo colaborativo ideal para comunidades técnicas, pero inviable a gran escala para el comercio electrónico o la administración pública.
2. **Jerarquía de Confianza Centralizada (PKI):** Se delega la verificación en entidades de máxima confianza legal y técnica denominadas **Autoridades de Certificación (AC / CA)**, que asumen la responsabilidad de autenticar de forma unívoca la vinculación entre una identidad real y su clave pública mediante un documento electrónico firmado denominado **certificado digital**.

---

### 3.7.2. Definición y Componentes de una PKI
Una **Infraestructura de Clave Pública (PKI - *Public Key Infrastructure*)** se define como el conjunto integrado de hardware, software, políticas, procedimientos, personas y marcos normativos necesarios para crear, gestionar, almacenar, distribuir, validar y revocar certificados digitales de forma segura y legalmente vinculante.

Una arquitectura PKI completa está compuesta por los siguientes roles y componentes:

1. **Autoridad de Certificación (AC / CA - *Certification Authority*):** Es el corazón de la PKI. Es la entidad de máxima confianza responsable de emitir, firmar digitalmente con su clave privada, renovar y revocar los certificados digitales de usuarios, servidores y servicios.
2. **Autoridad de Registro (AR / RA - *Registration Authority*):** Entidad delegada responsable de verificar de forma fehaciente la identidad física o jurídica del solicitante (de manera presencial o mediante mecanismos telemáticos acreditados) antes de autorizar a la CA la expedición del certificado.
3. **Autoridad de Validación (AV / VA - *Validation Authority*):** Componente que proporciona información en tiempo real sobre la validez o revocación de un certificado mediante el protocolo **OCSP** (*Online Certificate Status Protocol*).
4. **Repositorios y Directorios:** Servidores y bases de datos públicas accesibles en red (mediante LDAP o HTTP) donde se publican los certificados emitidos y las **Listas de Revocación de Certificados (CRL - *Certificate Revocation Lists*)**.
5. **Titulares o Suscriptores (*Subjects / Holders*):** Personas físicas, entidades jurídicas, servidores web o dispositivos a cuyo nombre se emite el certificado digital y que son propietarios exclusivos de su clave privada.
6. **Terceros de Confianza / Partes Utilizadoras (*Relying Parties*):** Usuarios, aplicaciones, navegadores web o sistemas operativos que reciben un certificado digital y confían en la CA emisora para verificar firmas digitales o establecer conexiones cifradas seguras.

![Estructura y componentes de una Infraestructura de Clave Pública PKI](img/pki.png)

---

### 3.7.3. Jerarquías de Certificación y Cadena de Confianza (*Trust Chain*)

En entornos corporativos y en la arquitectura global de Internet, una PKI nunca se implementa con una única Autoridad de Certificación que firma todos los certificados directamente. Para garantizar la seguridad operativa y mitigar riesgos catastróficos, se utilizan **jerarquías de certificación multinivel**:

```text
               ┌────────────────────────────────────────┐
               │              CA RAÍZ (Root CA)         │
               │   • Clave autofirmada (Self-Signed)    │
               │   • Fuera de línea (Offline Vault)     │
               └───────────────────┬────────────────────┘
                                   │ Firma
                                   ▼
               ┌────────────────────────────────────────┐
               │     CA INTERMEDIA (Intermediate CA)     │
               │   • Certificado firmado por la CA Raíz │
               │   • En línea (Emisión operativa)       │
               └───────────────────┬────────────────────┘
                                   │ Firma
                                   ▼
               ┌────────────────────────────────────────┐
               │    CERTIFICADO FINAL (End-Entity / Leaf)│
               │   • Servidor Web (HTTPS), Usuario, VPN │
               │   • NO puede firmar otros certificados │
               └────────────────────────────────────────┘
```

#### 1. CA Raíz (*Root CA*):
* Es la autoridad suprema en la cima de la jerarquía.
* Su propio certificado está **autofirmado** (*Self-Signed*): el emisor (*Issuer*) y el sujeto (*Subject*) son exactamente la misma entidad, y el certificado se firma con su propia clave privada.
* **Seguridad Crítica:** Si la clave privada de una CA Raíz se viese comprometida, toda la infraestructura se derrumbaría. Por ello, la CA Raíz **se mantiene fuera de línea (*offline*)**, en una máquina aislada físicamente de la red (en una cámara o centro de datos de alta seguridad), encendiéndose únicamente de forma esporádica para firmar certificados de CAs subordinadas.

#### 2. CAs Intermedias o Subordinadas (*Intermediate / Issuing CAs*):
* Son entidades delegadas cuyos certificados han sido firmados por la CA Raíz.
* Operan conectadas a la red (*online*) para atender las solicitudes cotidianas de emisión y revocación.
* Permiten **segregar funciones y acotar riesgos**: una organización puede tener una CA intermedia exclusiva para servidores web, otra para puestos de trabajo/VPN y otra para firma de documentos. Si una de ellas resulta comprometida, solo se revoca esa CA intermedia, manteniendo intacta la CA Raíz.

#### 3. Certificados Finales u Hoja (*End-Entity / Leaf Certificates*):
* Son los certificados emitidos a los servidores web (`www.empresa.es`), usuarios o servicios.
* Disponen de la restricción de extensiones X.509 `Basic Constraints: CA:FALSE`, lo que les prohíbe taxativamente actuar como autoridad para firmar otros certificados.

#### ¿Cómo valida un cliente la Cadena de Confianza?
Cuando tu navegador visita un sitio web protegido con HTTPS, el servidor web no solo entrega su certificado final, sino también el **paquete de certificados intermedios (*Certificate Bundle / Chain*)**. El navegador realiza una comprobación recursiva ascendente:
1. Comprueba la firma del certificado del servidor con la clave pública de la **CA Intermedia**.
2. Comprueba la firma de la CA Intermedia con la clave pública de la **CA Raíz**.
3. Verifica que la **CA Raíz** figure en su almacén local de autoridades preinstaladas de confianza (*Root Store*). Si la encuentra, la cadena queda anclada y la conexión se declara segura. Si falta algún certificado intermedio en la cadena, el navegador emitirá una advertencia de seguridad.

---

### 3.7.4. Almacenes de Confianza (*Root Stores*)
¿Por qué tu sistema operativo o tu navegador web confían de forma predeterminada en entidades como DigiCert, Sectigo o la FNMT sin pedirte confirmación previa?

Porque los sistemas operativos y los navegadores web incorporan de fábrica un **Almacén de Certificados Raíz de Confianza (*Root Store*)**:

* **Windows:** Almacén de certificados de Microsoft (`certmgr.msc` para usuario y `certlm.msc` para equipo local).
* **GNU/Linux (Debian/Ubuntu):** Directorio del sistema `/etc/ssl/certs/` gestionado mediante el paquete `ca-certificates` y la utilidad `update-ca-certificates`.
* **Mozilla Firefox:** Utiliza su propio almacén interno de certificados independiente del sistema operativo (en *Ajustes $\rightarrow$ Privacidad y Seguridad $\rightarrow$ Certificados*).
* **Google Chrome / Chromium:** Tradicionalmente utilizaba el almacén del sistema operativo anfitrión, integrando recientemente el *Chrome Root Store*.

!!! warning "El paso crítico en una PKI privada interna"
    Cuando desplegamos una PKI propia en una red corporativa o en un laboratorio didáctico (como en la **Práctica 3.5** con `easy-rsa`), ningún navegador reconocerá a nuestra CA de forma automática porque no forma parte del consorcio público de CAs. 
    
    Por ello, el paso obligatorio para que los clientes naveguen sin advertencias de peligro es **distribuir e importar el certificado público de nuestra CA (`ca.crt`) en el almacén de certificados de confianza** de cada equipo cliente.

---

### 3.7.5. Casos Reales de PKI: La FNMT y el DNI Electrónico (DNIe)

#### A) Solicitud del Certificado de Ciudadano ante la FNMT
Para comprender la coordinación práctica de los roles de una PKI en España, veamos el flujo real de obtención del certificado de Persona Física de la **Fábrica Nacional de Moneda y Timbre (FNMT-RCM)**:

1. **Titular / Suscriptor (*Subject*):** El ciudadano (ej. Laura Gómez). Desde su navegador solicita el certificado. Su equipo genera localmente el par de claves ($K_{pub}$ y $K_{priv}$) y envía telemáticamente a la FNMT una solicitud CSR con su clave pública.
2. **Autoridad de Registro (AR / RA):** Las oficinas de registro autorizadas (Agencia Tributaria - AEAT, Seguridad Social, Ayuntamientos). Laura acude presencialmente con su DNI para que un funcionario compruebe que su identidad real coincide con los datos de la solicitud.
3. **Autoridad de Certificación (AC / CA):** La FNMT. Una vez validada la identidad por la AR, la FNMT genera el certificado X.509, lo **firma digitalmente con su clave privada de CA** y lo deja disponible para su descarga telemática.
4. **Terceros de Confianza (*Relying Parties*):** Las sedes electrónicas de la administración (DGT, Hacienda, Carpeta Ciudadana). Cuando Laura accede a la DGT, presenta su certificado. La DGT valida la firma comprobando que proviene de la FNMT, cuya clave pública ya tiene en su almacén de confianza.
5. **Autoridad de Validación (AV / VA) y CRL:** La sede de la DGT consulta en tiempo real al respondedor **OCSP** de la FNMT para certificar que el documento no ha sido revocado.

#### B) Estudio de Caso: El DNI Electrónico (DNIe)
El **DNIe** español es un ejemplo avanzado de PKI corporativa nacional sobre tarjeta inteligente criptográfica (*smartcard*):

* **Chip Criptográfico Seguro:** Incorpora un microprocesador que almacena de forma blindada dos pares de claves y dos certificados X.509 independientes:
    1. **Certificado de Autenticación:** Garantiza la identidad del titular en accesos web y telemáticos.
    2. **Certificado de Firma:** Empleado para realizar firmas electrónicas cualificadas sobre documentos con idéntico valor legal que la firma manuscrita.
* **Principio de Separación de Claves:** Se generan claves privadas distintas para autenticación y para firma. De este modo, un compromiso accidental de la clave de autenticación durante una sesión web no compromete la capacidad legal de la clave de firma.
* **Roles PKI en el DNIe:** Las comisarías de la Policía Nacional actúan como **Autoridades de Registro (AR)** (toma presencial de huella y datos), y la Dirección General de la Policía (DGP) opera como la **Autoridad de Certificación (CA)** que firma los certificados grabados en el chip.

---

## 3.8. Certificados Digitales X.509 y Operativa de Administración

### 3.8.1. Anatomía del Estándar X.509 v3
Un **certificado digital** es el documento electrónico normalizado emitido por una CA que acredita que una determinada clave pública pertenece a una persona, servicio o servidor web.

El formato estándar internacional está definido por la especificación **UIT-T X.509 v3** (RFC 5280). La información técnica contenida en el certificado incluye los siguientes campos fundamentales:

| Campo X.509 | Descripción técnica |
| :--- | :--- |
| **Versión** | Versión del estándar (habitualmente v3). |
| **Número de Serie** | Identificador hexadecimal único asignado por la CA emisora. |
| **Algoritmo de Firma** | Algoritmo y función hash empleados por la CA (ej. `sha256WithRSAEncryption` o `ecdsa-with-SHA384`). |
| **Emisor (*Issuer*)** | Identidad de la Autoridad de Certificación que expide y firma el documento (ej. `CN=ACCVRAIZ1, O=ACCV, C=ES`). |
| **Período de Validez** | Fechas límites exactas UTC: *Not Before* (inicio) y *Not After* (expiración). |
| **Sujeto (*Subject*)** | Identidad del titular (ej. nombre/NIF para personas, o FQDN para servidores web). |
| **Clave Pública del Sujeto** | Algoritmo criptográfico utilizado (RSA, ECC) y los bytes numéricos de la clave pública del titular. |
| **Extensiones X.509 v3** | Parámetros avanzados: usos permitidos de la clave (*Key Usage*, *Extended Key Usage*), restricciones básicas (*Basic Constraints: CA:FALSE*) y nombres alternativos (*Subject Alternative Name*). |
| **Firma Digital de la CA** | Resumen hash de todos los campos anteriores **cifrado con la clave privada de la Autoridad de Certificación**. |

![Campos e información contenida en un certificado digital X.509 v3](img/CertificadoDigital.png)

---

### 3.8.2. Evolución del Sujeto: De Common Name (CN) a Subject Alternative Name (SAN)

Históricamente, los certificados de servidor web utilizaban el campo **Common Name (CN)** del *Subject* para indicar el dominio protegido (por ejemplo, `CN = www.electroasir.es`).

Sin embargo, los estándares modernos de la IETF y los navegadores actuales (Chrome, Firefox, Edge, Safari):

1. **Han declarado obsoleto (*deprecated*) el uso exclusivo de CN:** Si un certificado web no incluye la extensión SAN, los navegadores emiten una alerta de seguridad por certificado no confiable (`ERR_CERT_COMMON_NAME_INVALID`).
2. **Exigen obligatoriamente la extensión SAN (*Subject Alternative Name*):** Permite listar explícitamente uno o múltiples nombres de dominio (DNS), direcciones IP o correos electrónicos para los que el certificado es válido. Por ejemplo:
   ```text
   X509v3 Subject Alternative Name:
       DNS:electroasir.es, DNS:www.electroasir.es, DNS:tienda.electroasir.es, IP Address:192.168.1.100
   ```

---

### 3.8.3. Clasificación de Certificados de Servidor Web

En la administración de servicios web y comercio electrónico, los certificados SSL/TLS se clasifican bajo dos prismas:

#### A) Según el Grado de Validación de la Identidad:
* **Domain Validated (DV):** Nivel básico y automatizado. La CA solo comprueba que el solicitante tiene control administrativo sobre el dominio web (respondiendo a un reto DNS o publicando un archivo temporal en la raíz web). Se expiden en minutos. Son los certificados estándar generados por proveedores como Let's Encrypt.
* **Organization Validated (OV):** Nivel corporativo intermedio. Además del dominio, la CA verifica documentalmente la existencia legal y fiscal de la empresa titular en registros mercantiles oficiales.
* **Extended Validation (EV):** Máximo nivel de verificación. La CA realiza una investigación exhaustiva y auditada de la legitimidad física y legal de la organización.

#### B) Según el Alcance de Nombres de Dominio:
* **Certificado Monodominio:** Válido únicamente para un único nombre FQDN específico (ej. `portal.iescaminas.es`).
* **Certificado Multidominio (SAN):** Protege varios dominios y subdominios completamente diferentes en un solo certificado (ej. `iescaminas.es`, `aulavirtual.es`, `matriculas.edu`).
* **Certificado Comodín (*Wildcard*):** Protege todos los subdominios de primer nivel de un dominio utilizando un asterisco (`*.iescaminas.es`). Es válido automáticamente para `correo.iescaminas.es`, `ftp.iescaminas.es`, `vpn.iescaminas.es`, etc. Ahorra costes de gestión, aunque si su clave privada se compromete, todos los subdominios quedan expuestos simultáneamente.

!!! info "La Revolución de Let's Encrypt y el Protocolo ACME"
    Hasta 2015, obtener un certificado de servidor web requería un desembolso económico anual y procesos manuales lentos. La fundación sin ánimo de lucro **Let's Encrypt** transformó Internet al ofrecer certificados DV **totalmente gratuitos y renovables automáticamente** mediante el protocolo abierto **ACME** (*Automated Certificate Management Environment*).
    
    A través de clientes por línea de comandos como `certbot`, los servidores web Linux pueden solicitar, validar mediante retos automáticos (HTTP-01 o DNS-01), instalar y renovar sus certificados SSL/TLS cada 90 días sin intervención humana.

---

### 3.8.4. Formatos y Extensiones de Archivos en Sistemas

Para un administrador de sistemas, uno de los retos prácticos más habituales consiste en lidiar con los diferentes formatos de empaquetado y codificación de claves y certificados. 

La siguiente tabla resume los formatos estándar y su uso operativo:

| Formato / Estándar | Extensiones comunes | Codificación | ¿Contiene Clave Privada? | Entornos y Usos Principales |
| :--- | :---: | :---: | :---: | :--- |
| **PEM**<br>*(Privacy-Enhanced Mail)* | `.pem`, `.crt`, `.cer`, `.key` | **ASCII Base64**<br>(Texto legible) | **Opcional**<br>(frecuentemente separada en `.key`) | **Estándar universal en GNU/Linux, Apache, Nginx y Docker.**<br>Encabezados legibles: `-----BEGIN CERTIFICATE-----` o `-----BEGIN PRIVATE KEY-----`. |
| **DER**<br>*(Distinguished Encoding Rules)* | `.der`, `.cer` | **Binario puro**<br>(ASN.1) | No suele contener clave privada | Utilizado habitualmente en plataformas **Java** (KeyStore) y dispositivos de red embebidos. |
| **PKCS#12 / PFX**<br>*(Personal Information Exchange)* | `.p12`, `.pfx` | **Binario cifrado con contraseña** | **SÍ (Siempre)**<br>(Certificado + Clave Privada + Cadena) | **Estándar para copias de seguridad de certificados personales** (FNMT), importación en navegadores y servidores **Microsoft Windows (IIS / Active Directory)**. |
| **PKCS#7 / P7B** | `.p7b`, `.p7c` | Base64 o Binario | **NUNCA**<br>(Solo certificados públicos) | Utilizado para distribuir **cadenas completas de certificación** y listas CRL sin peligro de exponer claves privadas. |

!!! tip "Comandos OpenSSL indispensables para el Administrador de Sistemas"
    * **Ver el contenido legible de un certificado PEM:**
      ```bash
      openssl x509 -in certificado.crt -text -noout
      ```
    * **Comprobar las fechas de validez de un certificado:**
      ```bash
      openssl x509 -in certificado.crt -dates -noout
      ```
    * **Exportar un contenedor PKCS#12 (`.p12`) a formato PEM (separando certificado y clave privada):**
      ```bash
      # Extraer el certificado público:
      openssl pkcs12 -in backup.p12 -clcerts -nokeys -out certificado.crt
      # Extraer la clave privada:
      openssl pkcs12 -in backup.p12 -nocerts -nodes -out privada.key
      ```
    * **Empaquetar un certificado y su clave privada en un contenedor `.p12` protegido:**
      ```bash
      openssl pkcs12 -export -out paquete.p12 -inkey privada.key -in certificado.crt
      ```

---

### 3.8.5. Ciclo de Vida: El Proceso CSR (*Certificate Signing Request*)

Para obtener un certificado digital firmado sin exponer jamás la clave privada ante terceros, la industria sigue un procedimiento riguroso de dos etapas basado en una solicitud **CSR (PKCS#10)**:

```text
┌───────────────────────────────────────┐                    ┌────────────────────────────────────────┐
│     SOLICITANTE (Servidor Web)        │                    │   AUTORIDAD DE CERTIFICACIÓN (CA)      │
└──────────────────┬────────────────────┘                    └───────────────────┬────────────────────┘
                   │                                                             │
  1. Genera par de claves localmente:                                            │
     - Clave privada (srv.key) [SECRETO]                                         │
     - Clave pública                                                             │
                   │                                                             │
  2. Empaqueta clave pública + datos en                                          │
     la solicitud CSR (srv.req)                                                  │
                   │                                                             │
  3. Envía ÚNICAMENTE el fichero CSR ───────────────────────────────────────────>│
     (por scp, web o API)                                                        │ 4. Verifica identidad (AR)
                                                                                 │
                                                                                 │ 5. Firma el CSR con la clave
                                                                                 │    privada de la CA (ca.key)
                                                                                 │    y genera el certificado (srv.crt)
                                                                                 │
  6. Recibe el certificado srv.crt <─────────────────────────────────────────────│ 
                   │                                                             │
  7. Configura Apache/Nginx con:                                                 │
     - Su clave privada local (srv.key)                                          │
     - El certificado firmado (srv.crt)                                          │
```

!!! danger "Regla de Oro en Administración de PKI"
    **La clave privada NUNCA debe abandonar el equipo donde se generó.** La CA únicamente necesita recibir la solicitud CSR (que contiene la clave pública y los datos de identidad). Si alguien te pide que le envíes tu fichero `.key` para firmarte un certificado, el principio de seguridad se ha roto por completo.

*(Este es exactamente el procedimiento técnico que se ejecuta paso a paso en la [Práctica 3.5](P05.md) entre el `ServidorHTTPS` y el `ServidorAC`).*

---

### 3.8.6. Revocación de Certificados y Comprobación de Estado

Un certificado digital emitido puede perder su validez antes de la fecha de caducidad fijada (*Not After*) si concurren circunstancias críticas: robo o compromiso de la clave privada, cese laboral de un empleado titular, cambio de dominio web o cierre de la organización. Para invalidarlo, la CA procede a su **revocación**.

Para que los clientes comprueben si un certificado presentado sigue siendo válido, existen tres mecanismos sucesivos:

#### 1. Listas de Revocación de Certificados (CRL - *Certificate Revocation Lists*):
* Son listas públicas en texto firmado digitalmente por la CA que contienen los números de serie de todos los certificados revocados junto con la fecha y motivo de anulación.
* **Inconvenientes:** 
  * Ficheros crecientes en tamaño que saturan la red.
  * Los clientes las descargan periódicamente y las guardan en caché; si un certificado se revoca entre dos descargas, el cliente seguirá aceptándolo durante horas o días.
  * Si el servidor web de la CRL no responde, el cliente sufre retrasos severos de conexión.

#### 2. Protocolo de Estado de Certificados en Línea (OCSP - *Online Certificate Status Protocol*):
* Sustituye la descarga de listas gigantescas por una consulta HTTP puntual cliente-servidor: el navegador consulta al **servidor OCSP Responder** de la CA por el estado específico del número de serie que está validando. El servidor responde inmediatamente: `Good` (Válido), `Revoked` (Revocado) o `Unknown` (Desconocido).
* **Inconvenientes:**
  * **Pérdida de Privacidad:** La CA se entera en tiempo real de qué sitios web visita cada usuario.
  * **Sobrecarga y Latencia:** Cada conexión web inicial a una página nueva requiere una petición HTTP previa contra el servidor de la CA.

#### 3. Grapado OCSP (*OCSP Stapling*): La Solución Moderna
* Para solucionar los problemas de latencia y privacidad de OCSP, se ideó el **Grapado OCSP** (RFC 6066):
* Ya no es el cliente quien consulta a la CA. Es el **propio servidor web (Apache, Nginx)** el que se conecta periódicamente al servidor OCSP de la CA, obtiene una respuesta de estado firmada con sello de tiempo (*timestamp*) y la almacena en caché.
* Cuando un cliente inicia la conexión HTTPS, el servidor web adjunta ("grapa") directamente esa prueba de vigencia de la CA dentro del propio apretón de manos TLS.
* **Resultado:** Validación instantánea, sin retrasos y con total privacidad para el cliente.

---

### 3.8.7. Cuadro de Práctica del Bloque

!!! note "Práctica de Laboratorio"
    !!! info "Puesta en práctica de la infraestructura PKI corporativa"
        **[Práctica 3.5: PKI - Gestión de Claves con Easy-RSA en Servidores Separados](P05.md)**
        
        *En esta práctica desplegarás tu propia Infraestructura de Clave Pública completa utilizando máquinas virtuales Debian independientes: levantarás una Autoridad de Certificación con `easy-rsa`, generarás peticiones CSR en un servidor web Apache sin comprometer claves privadas, firmarás y desplegarás los certificados X.509, e importarás la CA raíz en los clientes para validar la navegación HTTPS segura sin alertas.*

---

## 3.9. Protocolos Seguros (SSL/TLS), Negociación y Buenas Prácticas

### 3.9.1. Arquitectura de SSL/TLS y HTTPS
Los protocolos **SSL** (*Secure Sockets Layer*) y su sucesor moderno **TLS** (*Transport Layer Security*) operan en el modelo TCP/IP como una capa intermedia de seguridad insertada entre los protocolos de la capa de aplicación (HTTP, SMTP, IMAP, FTP) y la capa de transporte TCP:

```text
    ┌────────────────────────────────────────────────────────┐
    │  Capa de Aplicación:  HTTP   |   SMTP   |   IMAP / POP │
    ├────────────────────────────────────────────────────────┤
    │  Capa de Seguridad:            SSL / TLS               │
    ├────────────────────────────────────────────────────────┤
    │  Capa de Transporte:                 TCP               │
    ├────────────────────────────────────────────────────────┤
    │  Capa de Red:                        IP                │
    └────────────────────────────────────────────────────────┘
```

Cuando HTTP opera sobre TLS, pasa a denominarse **HTTPS** (utilizando por defecto el puerto **TCP 443** en lugar del puerto TCP 80). Todo el contenido transmitido (cabeceras, URLs, cookies de sesión, contraseñas y datos HTML) viaja íntegramente cifrado a través de la red.

![Funcionamiento de https](img/https2.png)

#### Evolución Histórica de los Protocolos:
* **SSL 2.0 (1995) y SSL 3.0 (1996):** Desarrollados originalmente por Netscape. **Totalmente prohibidos y obsoletos** por graves fallos de diseño estructural.
* **TLS 1.0 (1999) y TLS 1.1 (2006):** Primeros estándares IETF. Declarados formalmente obsoletos en 2021 (RFC 8996).
* **TLS 1.2 (2008):** Estándar muy robusto y maduro que introdujo soporte para cifrados SHA-256 y suites autenticadas modernas. Continúa ampliamente desplegado.
* **TLS 1.3 (2018 - RFC 8446):** La versión actual de referencia. Rediseñó por completo el protocolo para hacerlo mucho más rápido y seguro, eliminando de raíz cualquier algoritmo criptográfico antiguo o susceptible de vulnerabilidades.

---

### 3.9.2. El Apretón de Manos (TLS Handshake) Paso a Paso
El **TLS Handshake** es el procedimiento inicial mediante el cual el cliente y el servidor se autentican mutuamente y acuerdan una clave secreta de sesión antes de transmitir el primer byte de datos.

Es la demostración práctica definitiva de la **Criptografía Híbrida**:

```text
CLIENTE (Navegador)                                            SERVIDOR WEB (HTTPS)
       │                                                                 │
       │ 1. ClientHello (Versiones TLS soportadas,                       │
       │    Ciphersuites disponibles, valor aleatorio R_c)               │
       ├────────────────────────────────────────────────────────────────>│
       │                                                                 │
       │ 2. ServerHello (Ciphersuite elegida, valor aleatorio R_s)       │
       │ 3. Certificate (Certificado X.509 del servidor + cadena CA)     │
       │ 4. ServerKeyExchange (Parámetros Diffie-Hellman efímeros ECDHE) │
       │ 5. ServerHelloDone                                              │
       │<────────────────────────────────────────────────────────────────┤
       │                                                                 │
[Verifica certificado y cadena]                                          │
       │                                                                 │
       │ 6. ClientKeyExchange (Parámetros DH del cliente)                │
       │ 7. ChangeCipherSpec (A partir de ahora, todo viaja cifrado)     │
       │ 8. Finished (Hash autenticado del intercambio)                  │
       ├────────────────────────────────────────────────────────────────>│
       │                                                                 │
[Ambos derivan idéntica Clave Simétrica de Sesión (K_sesion)]            │
       │                                                                 │
       │ 9. ChangeCipherSpec                                             │
       │ 10. Finished                                                    │
       │<────────────────────────────────────────────────────────────────┤
       │                                                                 │
       │═════════════════════════════════════════════════════════════════│
       │    CANAL SEGURO HTTPS: Cifrado Simétrico (AES-GCM / ChaCha20)   │
       │═════════════════════════════════════════════════════════════════│
```

1. **Negociación Inicial (*Hello*):** El cliente y el servidor acuerdan la versión más alta de TLS soportada por ambos y la suite criptográfica (*Cipher Suite*) que utilizarán.
2. **Autenticación del Servidor:** El servidor entrega su certificado X.509. El cliente valida que el certificado no esté caducado, que el dominio coincida con la URL navegada (campo SAN), que no esté revocado y que la firma de la CA ancle en una autoridad de su *Root Store*.
3. **Intercambio de Claves (Diffie-Hellman Efímero):** Mediante el algoritmo matemático ECDHE, el cliente y el servidor acuerdan de forma segura una **clave simétrica de sesión efímera** sin enviarla nunca directamente por la red.
4. **Transmisión de Datos:** Una vez establecida la clave simétrica, la comunicación conmuta a **criptografía simétrica de alta velocidad** (ej. AES-256-GCM o ChaCha20-Poly1305), donde cada bloque de datos incluye un código de autenticación (MAC) para asegurar su integridad.

---

### 3.9.3. Avances de TLS 1.3 y Secreto Perfecto hacia Adelante (PFS)

TLS 1.3 introdujo mejoras revolucionarias frente a las versiones anteriores:

#### 1. Reducción Radical de la Latencia (Handshake 1-RTT y 0-RTT):
* En TLS 1.2, el establecimiento del canal seguro requería **dos viajes de ida y vuelta completos (*2-RTT - Round Trip Time*)** antes de poder solicitar la página web.
* TLS 1.3 redujo el intercambio a **un solo viaje (*1-RTT*)**, acelerando de forma notable la velocidad de carga de las páginas web. Además, en conexiones recurrentes admite el modo *0-RTT Resumption*, enviando la petición web en el primer paquete.

#### 2. Limpieza Radical de Cifrados Obsoletos:
TLS 1.3 eliminó completamente el soporte para algoritmos antiguos y problemáticos: se prohibió el intercambio de claves basado en RSA estático, los hashes MD5 y SHA-1, el cifrador de flujo RC4, el cifrador 3DES y los modos de bloque CBC (vulnerables a ataques de relleno *padding oracle*).

#### 3. Secreto Perfecto hacia Adelante (*Perfect Forward Secrecy* - PFS):
En sistemas antiguos sin PFS, si un atacante grababa durante años el tráfico cifrado de un servidor web y, posteriormente, conseguía robar la clave privada del servidor (o forzaba su entrega judicialmente), podía descifrar retroactivamente todas las sesiones históricas grabadas.

Con **PFS** (obligatorio en TLS 1.3 a través de Diffie-Hellman efímero ECDHE):
* Cada sesión web utiliza una clave simétrica temporal completamente independiente que se destruye de la memoria RAM en cuanto se cierra la conexión.
* Aunque un atacante robe la clave privada del servidor web en el futuro, **le resultará matemáticamente imposible descifrar las conversaciones grabadas en el pasado**.

---

### 3.9.4. Vulnerabilidades Históricas y Mecanismos de Protección Actuales

A lo largo de la evolución de SSL y TLS, diversos incidentes de seguridad han motivado el endurecimiento de los navegadores y estándares:

#### Vulnerabilidades Notables:
* **Heartbleed (2014):** Grave fallo de validación de límites en la extensión *Heartbeat* de la librería OpenSSL. Permitía a cualquier atacante en Internet leer bloques de 64 KB de la memoria RAM del servidor web, filtrando claves privadas maestras, nombres de usuario, contraseñas y cookies de sesión sin dejar ningún rastro en los registros del sistema.
* **POODLE y Ataques de Degradación (*Downgrade Attacks*):** Ataques Man-in-the-Middle donde el atacante interceptaba la negociación TLS y simulaba fallos de red para forzar al navegador y al servidor a renegociar la conexión utilizando el protocolo obsoleto SSL 3.0, explotando a continuación debilidades conocidas en el cifrado CBC.

#### Mecanismos de Protección Obligatorios en la Actualidad:
* **Certificate Transparency (CT - RFC 6962):** Framework público de auditoría impulsado por Google. Todas las CAs públicas están obligadas a registrar públicamente cada certificado emitido en libros de registro abiertos (*CT Logs*) basados en árboles criptográficos de Merkle. Los navegadores modernos rechazan cualquier certificado HTTPS que no incorpore los sellos **SCT** (*Signed Certificate Timestamps*), impidiendo que una CA fraudulenta emita certificados secretos sin ser detectada en cuestión de minutos.
* **HSTS (*HTTP Strict Transport Security*):** Cabecera de respuesta HTTP (`Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`) que indica al navegador que **únicamente debe comunicarse con ese dominio mediante HTTPS**, denegando cualquier intento de conexión en HTTP claro e impidiendo que el usuario pueda saltarse advertencias de certificados no válidos.