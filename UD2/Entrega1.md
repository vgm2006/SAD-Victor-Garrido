## Grupo 4. SMTP Spoofing



### Qué es

Un correo de suplantación de identidad para hacer creer al usuario que ha recibido un correo de alguien que en realidad no es el.

### Cómo se lleva a cabo

Se trata de modificar las cabeceras del correo

### Qué categoría(s) de amenaza compromete

Compromete a la AUTENTICIDAD

### Ejemplo o caso real

Un atacante se hizo pasar por el probeedor de una empresa para pedir dinero

### Medida de prevención

SPF, DKIM y DMARC



## Grupo 3. Capturas de cuentas de usuario y contraseñas



### Qué es
Es una vulnerabilidad activa que consiste en tomar el control total de credenciales legítimas aprovechando fallos de seguridad existentes.

### Cómo se lleva a cabo
Se ejecuta principalmente mediante el uso de herramientas de captura de tráfico de red como los sniffers(son herramientas que permiten capturar y analizar los datos que circulan por una red informática.), técnicas de ingeniería social como el phishing(es una técnica de engaño en la que un atacante se hace pasar por una persona o empresa de confianza para conseguir información privada), y la instalación de software malicioso registrador de pulsaciones (keyloggers, es un programa o dispositivo que registra las teclas que una persona pulsa en el teclado.).

### Qué categoría(s) de amenaza compromete
Compromete la categoría de Intercepción, cuando se capturan datos durante su transmisión, y la Suplantación de identidad, cuando las credenciales obtenidas se utilizan para hacerse pasar por otra persona.

### Ejemplo o caso real

En junio de 2025, hubo un caso que se considera la mayor exposición de datos de la historia, en el que se localizaron 30 conjuntos de datos en internet que sumaban 16.000 millones de registros de nombres de usuario, contraseñas, cookies de sesión y enlaces directos de inicio de sesión. 
Una gran cantidad de empresas de las mas famosas del sector fueron afectadas entre las que se encontraban multinacionales como Netflix, PayPal, Apple, Google, entre muchas otras.
Todo esto se originó mediante un malware instalado en los ordenadores de los usuarios, utilizando ingeniería social para que los propios usuarios lo descarguen e instalen sin darse cuenta.


### Medida de prevención
Utilizar contraseñas seguras y diferentes para cada cuenta, activar la autenticación de dos factores (2FA), evitar acceder a enlaces sospechosos y mantener actualizados el sistema operativo y los programas de seguridad.

### Fuente
INCIBE – Instituto Nacional de Ciberseguridad de España


## Grupo 1: Ip Spoofing



### Qué es

Consiste en robar paquetes a través de  falsificar una IP

### Cómo se lleva a cabo

Todos los paquetes de red tienen una cabecera con un destino y un origen, el atacante puede cambiar las 2 para dirigir hacia donde va el paquete

### Qué categoría(s) de amenaza compromete

Compromete principalmente AUTENTICIDAD, aunque también a la integridad e incluso podría comprometer a la disponibilidad

### Ejemplo o caso real

Un atacante consiguió una IP de confianza de un servidor a la que poder atacar cambiando la cabecera para luego cambiar la IP para que el servidor la acepte

### Medida de prevención

Filtrado de paquetes, la supervisión de firewalls



## Grupo 2: DNS Spoofing



### Qué es

Un ataque para redirigir el trafico de los usuarios hacia paginas web maliciosas

### Cómo se lleva a cabo

El atacante introduce datos falsos en la caché del servidor DNS para interceptar los datos

### Qué categoría(s) de amenaza compromete

Compromete tanto AUTENTICIDAD, aunque también a la INTEGRIDAD e incluso podría comprometer a la DISPONIBILIDAD

### Ejemplo o caso real

Unos hackers lanzaron un ataque a una aerolínea para que en vez de salir la pagina web de la aerolínea, saliera un error y una imagen de un lagarto

### Medida de prevención

Usar servicio DNS LLC, esto lo que hace es autentificar la página
