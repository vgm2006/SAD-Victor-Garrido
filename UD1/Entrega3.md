# Informe de auditoría de vulnerabilidades

## 1. Contexto y alcance

La empresa Estudio Técnico Torrent S.L. va a firmar un contrato de mantenimiento informático con una nueva empresa y quiere saber cómo de seguros están sus sistemas antes de firmar. Me han contratado como auditor externo para hacer una primera revisión.

Como los sistemas reales no existen, he analizado una máquina de laboratorio llamada **Metasploitable2**. Es un Ubuntu antiguo que tiene fallos de seguridad conocidos a propósito. Representa lo que me podría encontrar en una pyme que nunca ha revisado su seguridad.

Solo he escaneado la máquina que me dio el profesor, dentro de una red interna aislada (`red-torrent`). Escanear otros equipos sin permiso es un delito (art. 197 bis del Código Penal), así que no he tocado nada fuera de ese entorno.

## 2. Metodología

- **Herramienta:** Nessus Essentials (escáner de vulnerabilidades).
- **Tipo de escaneo:** Basic Network Scan.
- **Objetivo analizado:** Metasploitable2, con IP `192.168.56.2`.
- **Entorno:** dos máquinas virtuales en VirtualBox (el Auditor con Nessus y Metasploitable2) conectadas en la red interna `red-torrent`.

Pasos que seguí:

1. Importé las dos máquinas virtuales en VirtualBox.
2. Comprobé que las dos estaban en la red interna `red-torrent`, para que se vieran entre ellas pero no tuvieran salida a Internet ni a la red del centro.
3. Entré en `https://localhost:8834`, acepté el aviso del certificado y activé mi licencia con el código que me llegó al correo.
4. Arranqué Metasploitable2 y averigüé su IP con el comando `ip a`.
5. En Nessus creé un escaneo con ***New Scan → Basic Network Scan***, puse la IP en "Targets" y lo llamé `Auditoria-Torrent`.
6. Esperé a que terminara (unos 10-25 minutos) y revisé los resultados.

## 3. Resumen de resultados

El escaneo encontró las siguientes vulnerabilidades, clasificadas con la escala CVSS:

| Severidad   | Puntuación CVSS | Cantidad |
|-------------|-----------------|----------|
| Crítica     | 9.0 – 10.0      | 8        |
| Alta        | 7.0 – 8.9       | 4        |
| Media       | 4.0 – 6.9       | 22       |
| Baja        | 0.1 – 3.9       | 8        |
| Informativa | 0.0             |      |

Las vulnerabilidades con la puntuación más alta son:

| Vulnerabilidad | CVSS | Puerto |
|----------------|------|--------|
| Canonical Ubuntu Linux SEoL (8.04.x) | 10.0 | — |
| VNC Server 'password' Password | 10.0 | 5900/tcp |
| Debian OpenSSH/OpenSSL Weak RNG (CVE-2008-0166) | 10.0 | 22/tcp |
| Bind Shell Backdoor Detection | 9.8 | 1524/tcp |
| Apache Tomcat AJP Connector Request Injection (Ghostcat) | 9.8 | 8009/tcp |
| SSL Version 2 and 3 Protocol Detection | 9.8 | 443/tcp |

Lo que más me llama la atención es que hay muchas críticas, y casi todas permiten tomar el control de la máquina sin apenas esfuerzo.

## 4. Vulnerabilidades clasificadas

| Vulnerabilidad | Severidad / CVSS | Origen | Breve descripción |
|----------------|------------------|--------|-------------------|
| VNC Server 'password' Password | Crítica / 10.0 | Uso | El servicio de escritorio remoto VNC está protegido con la contraseña "password". El programa funciona bien, pero se ha dejado una contraseña débil y por defecto, así que el fallo es de configuración. |
| Debian OpenSSH/OpenSSL Weak RNG (CVE-2008-0166) | Crítica / 10.0 | Implementación | Una versión concreta de OpenSSL de Debian tenía un error en el código que hacía que las claves de cifrado SSH se generaran con muy poca aleatoriedad, y por eso se pueden adivinar. Es un error de programación en esa versión. |
| Samba Badlock (CVE-2016-2118) | Alta / 7.5 | Diseño | Es un fallo en cómo está pensado el protocolo que usa Samba para gestionar usuarios. Un atacante que se ponga en medio de la comunicación puede aprovecharlo. No es un fallo de configuración ni un simple error de código. |

## 5. Análisis en profundidad

Vulnerabilidad elegida: **VNC Server 'password' Password** (Crítica, CVSS 10.0).

**Qué es:** VNC es un programa que permite controlar un ordenador de forma remota, viendo su pantalla como si estuvieras delante. En esta máquina está activado y protegido solo con la contraseña `password`, una de las primeras que probaría cualquiera.

**Cómo se podría explotar:** un atacante que esté en la misma red solo tendría que abrir un cliente VNC, conectarse al puerto 5900 de la máquina y escribir `password` como contraseña. Con eso obtendría control gráfico completo del sistema, sin necesidad de ningún programa de hacking.

**Cómo se mitiga:**
- Cambiar la contraseña por una larga y difícil de adivinar.
- Si no se usa VNC, desactivarlo.
- Si hace falta, no dejarlo abierto a toda la red: limitarlo con el firewall o acceder solo a través de una VPN.

**Referencia:** plugin de Nessus "VNC Server 'password' Password". No tiene CVE porque no es un fallo del programa, sino de configuración.

## 6. Recomendaciones

Ordenadas de más a menos urgente:

1. **Cambiar las contraseñas por defecto y débiles:** Es lo más rápido de arreglar y lo que más riesgo quita.
2. **Cerrar los servicios peligrosos que no hacen falta**: la shell abierta del puerto 1524 y Telnet. Además, poner un firewall que solo deje pasar los puertos necesarios.
3. **Actualizar o sustituir el sistema operativo y los programas.** Ubuntu 8.04 hace años que no recibe parches de seguridad, así que muchos fallos no se pueden arreglar sin actualizar a una versión con soporte.

## 7. Conclusión

Con lo que he visto, no firmaría el contrato tal cual. El sistema tiene vulnerabilidades críticas que permiten tomar el control sin apenas esfuerzo, y eso es muy peligroso para los datos de la empresa, pero si se hicieran un par de modificaciones clave si que lo firmaria.
