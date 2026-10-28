# Práctica UD3: Configuración de ACL en Linux para caso TorrentSL


## 1. Preparación del Entorno y Creación de Usuarios
Se ha verificado el soporte de ACL en el sistema, creado los 3 grupos requeridos (`operarios`, `ingenieria`, `supervisores`) y los 9 usuarios asignados a sus respectivos grupos.

> 📷 **CAPTURA 1:** Inserta aquí la captura de pantalla ejecutando `dpkg -s acl` y mostrando la creación de grupos y usuarios (por ejemplo, listando `/etc/passwd` o usando `getent group`).
> *Indicar dónde poner la captura:* Colocar justo debajo del bloque de comandos de creación de usuarios.

---

## 2. Configuración de Permisos Base (Chmod)
Se han asignado los propietarios iniciales con permisos restrictivos de partida para forzar el uso de ACL.

> 📷 **CAPTURA 2:** Inserta aquí la captura del comando `ls -l hmi_planta` y `ls -ld planos/` mostrando los permisos base restrictivos (750/640).

---

## 3. Configuración de Listas de Control de Acceso (ACL)
Se han aplicado las directivas `setfacl` tanto para el ejecutable `hmi_planta` como para la estructura de la carpeta `planos/` siguiendo el diagrama de Torrent S.L.

> 📷 **CAPTURA 3:** Inserta aquí la captura aplicando los comandos `setfacl` en la terminal sin errores.

---

## 4. Verificación de Resultados

### Verificación con `getfacl`
Se comprueba que las ACL se han aplicado correctamente sobre los recursos.

> 📷 **CAPTURA 4.1:** Inserta aquí la salida del comando `getfacl hmi_planta`.
> 📷 **CAPTURA 4.2:** Inserta hieri la salida del comando `getfacl -R planos/`.

### Verificación de acceso por usuarios (`su`)
Se ha comprobado el acceso con diferentes usuarios para validar el cumplimiento de las políticas de seguridad.

> 📷 **CAPTURA 5:** Inserta aquí la captura probando el acceso con un usuario (ej. `su - alberto` ejecutando `./hmi_planta` o leyendo los planos permitidos).

---

## 5. Respuestas a las Preguntas de Reflexión

1. **¿Por qué no bastan los permisos clásicos de Unix?**
   - Los permisos tradicionales solo permiten un único propietario, un único grupo y el resto, lo cual es insuficiente para esquemas complejos con múltiples perfiles cruzados sobre un mismo recurso.
   
2. **¿Qué es la máscara (mask) de una ACL?**
   - Es el límite superior que restringe los permisos efectivos de los usuarios y grupos definidos en la ACL.

3. **¿Qué diferencia hay entre una ACL de acceso y una ACL por defecto?**
   - La de acceso actúa sobre elementos actuales, mientras que la por defecto se aplica en directorios para que los nuevos ficheros creados hereden dichos permisos automáticamente.

4. **Si mañana se incorpora un nuevo operario, ¿qué pasos habría que repetir?**
   - Crear el usuario, asignarlo al grupo `operarios` y asegurarse de que herede las reglas mediante las ACL por defecto configuradas en los directorios compartidos.
