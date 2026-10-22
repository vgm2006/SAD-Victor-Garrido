Parte A · ¿Está actualizado el sistema?
Trabajad en Torrent-Auditor.

A1. Anotad qué versión de sistema operativo y kernel tenéis:

lsb_release -a
uname -r
<img width="351" height="130" alt="image" src="https://github.com/user-attachments/assets/53af8481-a066-4301-9166-a2f716f5d079" />


A2. Pedid al sistema la lista de paquetes disponibles y qué se puede actualizar:

sudo apt update
apt list --upgradable
<img width="873" height="549" alt="image" src="https://github.com/user-attachments/assets/943e0a5a-b652-4d04-8c10-afd2ef9ba11a" />


Anotad: ¿cuántos paquetes tienen actualización pendiente?
<img width="778" height="41" alt="image" src="https://github.com/user-attachments/assets/b45e03df-8416-409d-b5c6-3fe038aba1cb" />



A3. Simulad la actualización sin aplicarla:

sudo apt-get -s upgrade | head -30
La opción -s es "simular". Fijaos en que dice qué instalaría, pero no toca nada.
<img width="800" height="683" alt="image" src="https://github.com/user-attachments/assets/84f5f350-a339-4c75-8cd9-b023f519d14b" />


A4. ¿Y si el sistema ya no recibe parches? Buscad en https://endoflife.date la fecha de fin de soporte de:

La versión de Ubuntu/Debian de vuestro Torrent-Auditor.
Ubuntu 8.04, la que tiene Torrent-Vulnerable (Metasploitable2).
Rellenad:

Sistema	Fecha de fin de soporte	¿Recibe parches hoy?
Torrent-Auditor: 31/05/2029	Sí	
Torrent-Vulnerable (Ubuntu 8.04): Según endoflife.date, Ubuntu 8.04 dejó de recibir soporte el 9 de mayo de 2013 en su edición Server
		
Pregunta A: En la oferta de empleo de Torrent (dossier de la UD02) decían que trabajaban con "Ubuntu 8.04". Aunque pusieran todas las contraseñas bien, ¿qué problema sigue sin tener solución?

Aunque todas las contraseñas sean seguras, Ubuntu 8.04 ya no recibe actualizaciones ni parches de seguridad oficiales desde 2013. Esto significa que las vulnerabilidades descubiertas posteriormente pueden seguir sin corregirse y los atacantes podrían 
aprovecharlas para acceder al sistema. La solución es migrar a una versión que todavía tenga soporte.

## Parte B · ¿De dónde vienen los programas instalados?
B1. Mirad de qué repositorios se descarga vuestro sistema:

cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/
Investiga el fichero donde están configurados los repositorios y comenta el significado de cada una de las líneas y su contenido.
<img width="785" height="160" alt="image" src="https://github.com/user-attachments/assets/774a7989-a25d-4391-bbcf-d17d751bdedb" />
<img width="666" height="142" alt="image" src="https://github.com/user-attachments/assets/e0963780-f754-48b3-8055-90018c1c953a" />

¿Qué hace? Muestra los repositorios desde los que Ubuntu descarga programas y actualizaciones.
<img width="762" height="325" alt="image" src="https://github.com/user-attachments/assets/a1fb377a-5705-4010-85e4-5059c8d078cb" />

El archivo `/etc/apt/sources.list.d/ubuntu.sources` contiene los repositorios de Ubuntu.

- **Types:** indica que se descargan paquetes compilados (`deb`).
- **URIs:** dirección del servidor desde donde se descargan los paquetes.
- **Suites:** indica la versión de Ubuntu y los tipos de actualizaciones (`noble`, `noble-updates`, `noble-backports` y `noble-security`).
- **Components:** clasifica el software en `main`, `restricted`, `universe` y `multiverse`.
- **Signed-By:** indica la clave utilizada para comprobar que los repositorios están firmados correctamente.

El sistema utiliza un repositorio principal y otro de actualizaciones de seguridad.

B2. apt solo instala paquetes firmados por claves en las que confía. Mirad dónde están esas claves:

ls /etc/apt/trusted.gpg.d/ /usr/share/keyrings/ 2>/dev/null
Nessus, por ejemplo, no viene de los repositorios de Ubuntu, lo descargasteis de Tenable. Si lo instalasteis con dpkg -i, apt no comprobó ninguna firma, la confianza la pusisteis vosotros.
<img width="786" height="363" alt="image" src="https://github.com/user-attachments/assets/20a1ba05-641d-4206-a665-f55a7dcc3419" />
Este comando muestra las claves que pueden utilizarse para verificar la autenticidad de los repositorios



Reflexiona: 
¿qué pasaría si un atacante introduce una clave falsa en /usr/share/keyrings/? 

Podría conseguir que se aceptasen paquetes maliciosos si esa clave se configura como confiable para un repositorio. 
Guardarla en /usr/share/keyrings/ por sí solo no basta.

¿y si intercepta o reemplaza la clave oficial de Ubuntu ubuntu-archive-keyrings.gpg?
odría comprometer la seguridad de las actualizaciones, permitiendo aceptar paquetes falsificados si el atacante también controla el repositorio y firma su contenido con la clave correspondiente.

Por eso es importante proteger las claves oficiales y verificar los paquetes instalados.

B3. Información de un paquete concreto. Elegid openssh-server o curl:

apt policy openssh-server
Anotad: versión instalada, versión candidata, repositorio de origen y el significado de la tabla de versiones.
<img width="895" height="307" alt="image" src="https://github.com/user-attachments/assets/6bffd3c1-e446-4fff-9587-068f6172dea6" />

B4. Buscad paquetes instalados que no proceden de ningún repositorio (instalados a mano):

apt list --installed 2>/dev/null | grep -i local
Cada línea que salga merece una pregunta: ¿quién lo instaló, de dónde, y quién lo verificó?
<img width="932" height="252" alt="image" src="https://github.com/user-attachments/assets/3c982c3f-f347-4d18-85a0-8b12b088ca25" />

He ejecutado el comando y he encontrado un paquete instalado localmente:

- **Paquete:** Nessus
- **Versión:** 10.12.4
- **Origen:** descargado de la página oficial de Tenable.
- **Instalación:** fuera de los repositorios oficiales de Ubuntu.

Al instalarlo manualmente, APT no verificó su firma mediante los repositorios configurados. Por eso es 
importante comprobar que el archivo procede de una fuente oficial y que no ha sido manipulado.


B5. Comprobad si algún fichero de un paquete ha cambiado desde que se instaló:

sudo dpkg -V
Si no sale nada, todo coincide con lo instalado.
Si salen líneas, la primera columna indica qué cambió (5 = el hash MD5 es distinto). Los ficheros de configuración (c) cambian de forma normal. Un binario (/usr/bin/...) modificado es una alarma.

¿Qué hace? Comprueba si los archivos de los paquetes instalados han cambiado respecto a la información registrada por dpkg.
- Si no aparece nada, no se han detectado diferencias.
- Si aparecen líneas con 5, significa que el hash MD5 no coincide.
- Si aparece una c, identifica un archivo de configuración, que puede haberse modificado legítimamente.
<img width="665" height="195" alt="image" src="https://github.com/user-attachments/assets/13f1904d-c4b6-4781-9c29-1a3e2f2a19f4" />

Pregunta B: dpkg -V ¿detectaría un rootkit que además hubiera modificado la base de datos de dpkg? ¿Por qué conviene comparar con una copia guardada fuera del sistema? (Pista: recordad la fase 4 de la intrusión.)
He ejecutado `sudo dpkg -V` y se han detectado varios archivos modificados y otros que faltan.

Los archivos pertenecen principalmente a Nessus, aunque también aparece modificado `cloud-init.service`.

El símbolo `5` indica que el hash MD5 del archivo no coincide con el registrado por dpkg, mientras que `missing` indica que falta un archivo.

Estos cambios no significan necesariamente que exista malware, pero conviene revisarlos.


## Parte C · Verificar una descarga: hash y firma
Descargaremos un programa pequeño y real, GNU Hello, junto a su firma y la lista de claves de GNU.

## C1. Descarga:
mkdir -p ~/verif && cd ~/verif
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz.sig
wget https://ftp.gnu.org/gnu/gnu-keyring.gpg
<img width="430" height="152" alt="image" src="https://github.com/user-attachments/assets/ca495190-4058-478d-bbba-ba2f50aeaea9" />
<img width="920" height="342" alt="image" src="https://github.com/user-attachments/assets/0fce2ec9-9d2b-483a-9828-c105da47012d" />

<img width="900" height="546" alt="image" src="https://github.com/user-attachments/assets/9501f29e-1049-4ee1-aa4c-15ad88603f2f" />
<img width="750" height="158" alt="image" src="https://github.com/user-attachments/assets/d37ed490-e9c0-420a-87a4-ab6c8a74ccdd" />




## C2. Calculad el hash:

sha256sum hello-2.12.1.tar.gz
Comparadlo con vuestro compañero de al lado. ¿Sale igual? Debería.
<img width="862" height="125" alt="image" src="https://github.com/user-attachments/assets/6ed237ae-67c3-45b2-85ea-6946c9dd6e16" />
Si tu compañero tiene exactamente el mismo archivo, debería obtener el mismo hash.


## C3. Verificad la firma usando la lista de claves de GNU:

gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-2.12.1.tar.gz.sig hello-2.12.1.tar.gz

Deberíais ver Good signature from .... También saldrá un aviso WARNING: This key is not certified with a trusted signature. Es normal, significa que vosotros no habéis dicho personalmente que esa clave sea de confianza. Anotad:
Nombre de quien firmó.
Fecha de la firma.
Identificador de la clave.

<img width="882" height="255" alt="image" src="https://github.com/user-attachments/assets/e4e7ec44-b23a-4bb9-a033-11bf59fb60d3" />

Resultado
Resultado	Good signature (firma válida)
Firmante	Reuben Thomas
Correo	rrt@sc3d.org
Fecha	29/05/2022, 23:05 UTC
Identificador de clave	4C8EF3DA3FD37230
Huella completa	24093F016FFE8602EF449BB84C8EF3DA3FD37230
¿Qué significa el aviso WARNING?
La firma es válida, pero GPG advierte que todavía no hemos confirmado personalmente que la clave pertenezca realmente al firmante.


## C4. El experimento: ¿qué pasa si alguien toca el fichero? Haced una copia y añadid un carácter:

cp hello-2.12.1.tar.gz hello-manipulado.tar.gz
echo "x" >> hello-manipulado.tar.gz
sha256sum hello-2.12.1.tar.gz hello-manipulado.tar.gz
cp hello-2.12.1.tar.gz.sig hello-manipulado.tar.gz.sig
gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-manipulado.tar.gz.sig hello-manipulado.tar.gz

<img width="870" height="152" alt="image" src="https://github.com/user-attachments/assets/7cbf063c-6fc3-4b5b-a6b4-2f53debf5305" />
Anotad: ¿cómo cambia el hash? ¿Qué dice ahora gpg? (Debe decir BAD signature.)

Archivo original: hash que empieza por 8d99142a...
Archivo manipulado: hash que empieza por 8c253f1a...
<img width="883" height="177" alt="image" src="https://github.com/user-attachments/assets/f50f8f1e-185e-4b29-9ed3-318145158224" />

Al modificar el archivo, su hash SHA-256 cambió completamente. Al verificarlo con GPG apareció BAD signature, indicando que la firma digital ya no coincide con el contenido del archivo. Esto demuestra que las firmas digitales permiten detectar modificaciones.

## C5. Pregunta C: completad:
| Situación | ¿Basta con el hash? | ¿Hace falta la firma? ¿Por qué? |
|---|---|---|
| El fichero se corrompió al descargarlo. | **Sí**, si se compara con un hash original obtenido de una fuente fiable. | **No es imprescindible.** El hash permite detectar si el archivo ha cambiado accidentalmente. |
| Un atacante cambió el fichero **y** el hash publicado en la misma web. | **No**, porque el atacante también ha falsificado la referencia. | **Sí.** Una firma válida, verificada con la clave pública auténtica del desarrollador, permite detectar la manipulación. |
| Descargas desde un espejo (*mirror*) que no es de confianza. | **No**, si el hash también procede del espejo y no se puede verificar de forma independiente. | **Sí, es recomendable.** La firma permite comprobar la integridad y el origen del archivo usando una clave pública auténtica. |

**Conclusión:** El hash sirve para comprobar la integridad respecto a una referencia fiable. La firma digital también permite verificar el origen del archivo, siempre que la clave pública sea auténtica.

 ## Parte D · Política de actualizaciones para Torrent

1. Uno de los tres socios será el responsable de supervisar las actualizaciones semanalmente.
2. Los parches de seguridad críticos se instalarán lo antes posible, después de comprobar su compatibilidad.
3. Las actualizaciones del servidor de impresión y del ERP se probarán antes y se realizará una copia de seguridad.
4. Las actualizaciones automáticas solo se permitirán en equipos donde no afecten a servicios importantes.
5. El software se descargará de repositorios oficiales o fuentes fiables, con autorización del responsable.
6. Los equipos sin soporte se sustituirán o actualizarán a una versión compatible y, mientras tanto, se aislarán.
