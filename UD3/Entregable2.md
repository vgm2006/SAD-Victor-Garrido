# UD5 Control de acceso - Sesión guiada: Contraseñas, MFA y privilegios justos (Caso Torrent S.L.)

Este documento contiene la ejecución paso a paso de las actividades de control de acceso, políticas de contraseñas, autenticación multifactor (MFA) y gestión de privilegios con sudo.

## PARTE 1: Contraseñas robustas con PAM

### 1.1 Instalar e inspeccionar pwquality

**Instalamos la librería libpam-pwquality** para la comprobación de contraseñas:
```bash

sudo apt update && sudo apt install -y libpam-pwquality
```

**Comprobamos que en /etc/pam.d/common-password** figure la siguiente línea antes de pam\_unix.so:

```bash
password requisite pam\_pwquality.so retry=3
```
### 1.2 Configurar la política de calidad

Editamos el fichero /etc/security/pwquality.conf y **aplicamos/descomentamos los siguientes parámetros**:
```
minlen = 12 minclass = 3 dcredit = -1 ucredit = -1 lcredit = -1 ocredit = -1 dictcheck = 1 maxrepeat = 3 enforce\_for\_root
```
### 1.3 Verificación de la política de contraseñas

1.  **Intento con contraseña débil (usuario normal):** sudo passwd hugo (Introducir contraseña débil, ej. 1234 -> Rechazada por PAM)

> \[INSERTAR CAPTURA 1: Mensaje de error al intentar poner contraseña débil a hugo\]

2.  **Intento con contraseña fuerte (cumple política):** sudo passwd hugo (Introducir contraseña fuerte -> Aceptada correctamente)

> \[INSERTAR CAPTURA 2: Confirmación de cambio exitoso de contraseña para hugo\]

3.  **Prueba de protección sobre root:** Al haber activado **enforce\_for\_root**, intentamos cambiar la contraseña de root por una débil para validar que no puede saltarse la regla: sudo passwd root (Introducir contraseña débil -> Rechazada)

> \[INSERTAR CAPTURA 3: Rechazo de contraseña débil aplicada al usuario root\]

### 1.4 Configurar la caducidad con chage

**Aplicamos las políticas de expiración diferenciadas** según el grupo del usuario:

-   **Ingeniería y supervisores** (caducidad a los 60 días): nestor, victor, lucia, carlos, mary, john.
-   **Operarios** (caducidad a los 90 días): alberto, oscar, hugo.

```bash
for u in nestor victor lucia carlos mary john; do sudo chage -m 1 -M 60 -W 7 $u; done

for u in alberto oscar hugo; do sudo chage -m 1 -M 90 -W 7 $u; done
```
**Verificamos la política aplicada:** chage -l nestor

> \[INSERTAR CAPTURA 4: Salida del comando chage -l nestor mostrando la caducidad\]

## PARTE 2: MFA en SSH con Google Authenticator

### 2.1 Instalación y configuración por usuario

**Instalamos el módulo PAM** para Google Authenticator:

```bash
sudo apt install -y libpam-google-authenticator
```
Posteriormente, cada usuario que requiera MFA (por ejemplo, carlos) **ejecuta en su propia sesión**:
```bash
google-authenticator
```
**Respuestas seleccionadas durante el asistente:**

-   ¿Tokens time-based? -> **y**
-   ¿Actualizar ~/.google\_authenticator? -> **y**
-   ¿Impedir reutilización del mismo código? -> **y**
-   ¿Ampliar ventana de validez? -> **n**
-   ¿Limitar intentos (rate-limit)? -> **y**

> \[INSERTAR CAPTURA 5: Generación del código QR y los scratch codes en la terminal\]

### 2.2 Integración con SSH

Añadimos la regla al final de **/etc/pam.d/sshd**:

auth required pam\_google\_authenticator.so

Modificamos el fichero **/etc/ssh/sshd\_config** para habilitar la autenticación interactiva:
```
KbdInteractiveAuthentication yes UsePAM yes PasswordAuthentication yes
```
**Reiniciamos el servicio SSH:**
```bash
sudo systemctl restart ssh
```

### 2.3 Verificación de SSH + MFA

1.  **Nos conectamos por SSH** desde un cliente remoto: `ssh carlos@<IP\_SERVIDOR>`

> \[INSERTAR CAPTURA 6: Petición de contraseña y posterior solicitud del código verification code\]

2.  **Probamos a fallar el código TOTP** para comprobar que bloquea el acceso.

> \[INSERTAR CAPTURA 7: Acceso denegado tras introducir un código TOTP erróneo\]

### 2.4 Condicionar el MFA a un grupo específico (pam\_succeed\_if)

Para exigir MFA **únicamente a un grupo específico** (por ejemplo, mfa\_required), editamos **/etc/pam.d/sshd** y reemplazamos la regla genérica por:
```
auth \[success=1 default=ignore\] pam\_succeed\_if.so user ingroup mfa\_required auth required pam\_google\_authenticator.so
```
*(Si el usuario pertenece al grupo mfa\_required, la primera línea no salta la regla y se evalúa pam\_google\_authenticator.so. Si NO pertenece al grupo, salta 1 línea y omite la petición del MFA).*

> \[INSERTAR CAPTURA 8: Fichero /etc/pam.d/sshd configurado con pam\_succeed\_if\]

## PARTE 3: Privilegios justos con sudo

### 3.1 Creación del script de auditoría

Creamos el script **/usr/local/bin/auditoria\_planta.sh** y le asignamos permisos de ejecución:
```bash
sudo tee /usr/local/bin/auditoria\_planta.sh <<'EOF' #!/bin/bash echo "\[(date)\]Auditoriadeplantaejecutadapor(date)\] Auditoria de planta ejecutada por(date)\]Auditoriadeplantaejecutadapor(whoami)" EOF

sudo chmod 755 /usr/local/bin/auditoria\_planta.sh
```
### 3.2 Reglas de sudoers

Ejecutamos **sudo visudo -f /etc/sudoers.d/torrent** y definimos las siguientes reglas según las necesidades del caso:
```
%ingenieria ALL=(root) /usr/bin/systemctl restart hmi\_planta, /usr/bin/systemctl status hmi\_planta %supervisores ALL=(root) NOPASSWD: /usr/local/bin/auditoria\_planta.sh
```
*(Los **operarios** no se añaden al fichero ya que no disponen de ningún privilegio sudo).*

> \[INSERTAR CAPTURA 9: Contenido del archivo /etc/sudoers.d/torrent creado con visudo\]

### 3.3 Verificación de permisos y registros

1.  **Comprobar privilegios permitidos:** sudo -l -U nestor sudo -l -U carlos sudo -l -U alberto

> \[INSERTAR CAPTURA 10: Salida de los comandos sudo -l para los tres usuarios\]

2.  **Intento denegado (usuario alberto):** sudo systemctl restart hmi\_planta (Resultado: alberto is not in the sudoers file. This incident will be reported.)

> \[INSERTAR CAPTURA 11: Denegación de sudo al usuario alberto\]

3.  **Revisión de logs del sistema:** sudo journalctl -u sudo -n 20 (O bien: sudo tail -n 20 /var/log/auth.log)

> \[INSERTAR CAPTURA 12: Registro del intento no autorizado en los logs del sistema\]

4.  **Ejecución NOPASSWD (usuario carlos):** sudo /usr/local/bin/auditoria\_planta.sh

> \[INSERTAR CAPTURA 13: Ejecución correcta del script sin solicitar contraseña\]

## Preguntas de reflexión

### 1\. ¿Qué riesgo concreto cubre el MFA que una contraseña de 20 caracteres aleatorios no cubre?

Una contraseña extremadamente fuerte previene ataques de fuerza bruta o diccionarios, pero **no protege en caso de filtración o robo del secreto** (por ejemplo, phishing, keyloggers, sniffing en redes inseguras o reutilización de credenciales comprometidas en filtraciones de terceros). El **MFA** exige poseer el dispositivo físico que genera el código temporal (TOTP), garantizando que tener únicamente la contraseña no sea suficiente para acceder.

### 2\. Postura actual del NIST sobre la caducidad obligatoria de contraseñas

El **NIST (NIST SP 800-63B)** desaconseja obligar a cambiar las contraseñas de forma periódica (por ejemplo, cada 30 o 60 días) salvo que existan indicios claros de compromiso.

**Motivo:** Forzar rotaciones frecuentes genera un comportamiento en los usuarios donde eligen patrones predecibles o variaciones simples (ej. Password2024! -> Password2025!) o las apuntan en lugares inseguros (como notas físicas o archivos de texto plano), lo que acaba reduciendo la seguridad real en lugar de aumentar la protección.

### 3\. ¿Por qué sudo no sustituye a las ACL de sistema?

sudo y las ACL actúan en capas de seguridad distintas (**Defensa en Profundidad**):

-   **sudo** gestiona la **autorización a nivel de ejecución de procesos** como cuenta privilegiada (root).
-   **ACL** gestiona los **permisos de acceso a archivos del sistema de archivos** directamente.

**Ejemplo:** Si concedemos a un usuario acceso mediante sudo a un editor de texto o comando de lectura, pero el fichero carece de ACLs estructuradas, el usuario podría usar ese comando con privilegios para leer o sobreescribir ficheros sensibles del sistema que no debería tocar. De forma inversa, las ACL impiden que un proceso sin elevación lea o modifique información confidencial aunque el usuario no intente usar sudo.

### 4\. Recuperación ante pérdida de dispositivo MFA sin scratch codes

Si el usuario pierde el móvil y no dispone de los scratch codes, el administrador con acceso root debe intervenir mediante los siguientes pasos:

1.  **Acceder al servidor** mediante una sesión SSH/consola directa que tenga permisos root.
2.  **Eliminar o renovar el fichero de estado MFA** del usuario afectado: sudo rm /home/usuario/.google\_authenticator
3.  Solicitar al usuario que inicie sesión (ahora solo le pedirá su contraseña) y **volver a ejecutar google-authenticator** para generar una nueva clave secreta, registrar el nuevo dispositivo y guardar adecuadamente los nuevos scratch codes.
