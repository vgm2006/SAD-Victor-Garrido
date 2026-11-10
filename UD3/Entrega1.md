# Guía de Realización: Administración de Permisos y ACLs en Linux


## Tarea 1 — Preparar el Entorno

### Paso 1: Comprobar el soporte de ACL
Primero verificamos que el paquete `acl` está instalado en el sistema.

```bash
dpkg -s acl
```

> **Explicación:** El comando `dpkg -s` consulta el estado de un paquete en distribuciones basadas en Debian/Ubuntu. Debe indicar `Status: install ok installed`. Si no estuviera instalado, se ejecuta `sudo apt update && sudo apt install acl`.


<img width="692" height="374" alt="image" src="https://github.com/user-attachments/assets/999f2b0f-37fc-4a50-a609-49b4e25a6abb" />


---

### Paso 2: Crear grupos y usuarios
Creamos la estructura de tres grupos y nueve usuarios.


<img width="582" height="354" alt="image" src="https://github.com/user-attachments/assets/eb3dbc69-d913-4e5b-939c-9aae56a4f3c5" />


> **Explicación:** 
> * `-m` crea el directorio personal (`/home/usuarioX`).
> * `-g` asigna el grupo primario especificado al usuario recién creado.

#### Comprobación:


<img width="665" height="176" alt="image" src="https://github.com/user-attachments/assets/bffec257-a8a8-4467-9039-5e385aed71ed" />







### Paso 3: Crear la estructura de ficheros

# Creamos los recursos sobre los que aplicaremos las reglas de acceso.
<img width="442" height="45" alt="image" src="https://github.com/user-attachments/assets/74922841-ca70-4918-ad67-7fd03038274e" />


# Crear la carpeta de planos y los ficheros simulados
<img width="873" height="67" alt="image" src="https://github.com/user-attachments/assets/281c8316-a4b7-473e-8edd-dd7f35149c1b" />


> **Explicación:** `chmod +x` le otorga bit de ejecución al fichero `hmi_planta`. Los archivos dentro de `planos/` se crean como ficheros regulares vacíos.

#### Comprobación:


<img width="552" height="18" alt="image" src="https://github.com/user-attachments/assets/dd914225-baa9-4b08-9baf-9e211bb314d7" />


<img width="519" height="21" alt="image" src="https://github.com/user-attachments/assets/08801ae0-1f11-4aae-b9cc-455006a15598" />


<img width="603" height="178" alt="image" src="https://github.com/user-attachments/assets/19d94881-dded-453a-9e5c-55a2fee72f2e" />


---

## Tarea 2 — Permisos Base (Propietario / Grupo / Resto)

### Paso 4 y 5: Asignar propietario y aplicar permisos restrictivos
Asignamos un propietario administrativo (ej. `root`) y restringimos el acceso general (permisos tradicionales Unix) para obligar a que cualquier acceso específico de los usuarios o grupos de la práctica se resuelva mediante la tabla de ACLs.


<img width="513" height="87" alt="image" src="https://github.com/user-attachments/assets/fa407db9-d3fa-4350-898b-014ac6db7914" />


<img width="573" height="48" alt="image" src="https://github.com/user-attachments/assets/1e96040a-42cb-410a-b3ec-fccb4c7a81b9" />


> **Explicación:** Al asignar permisos `700` (`rwx------`), únicamente el usuario `root` tiene acceso por los permisos estándar de Linux. El resto del mundo (`others`) y el grupo propietario no tienen permisos (`---`). De este modo, cualquier otro usuario dependerá exclusivamente de la ACL que le configuremos.

#### Comprobación:
```bash
ls -ld hmi_planta planos/
ls -l planos/
```

* **📸 Captura 4:** Listado de permisos POSIX tradicionales donde se confirme la pertenencia a `root:root` y las máscaras restrictivas (`-rwx------` / `drwx------`).

---

## Tarea 3 — Configurar las ACL con `setfacl`

> *Nota: Adapta los nombres de grupos/usuarios y los permisos (`rwx`, `rx`, `r--`) a la tabla/diagrama de la sección 3 de tu enunciado.*

### Paso 6: Configurar ACL sobre `hmi_planta`
Aplicamos las reglas de acceso al ejecutable.

```bash
# Ejemplo: Permitir ejecución y lectura a grupo1 y permiso de solo lectura a usuario4
sudo setfacl -m g:grupo1:rx hmi_planta
sudo setfacl -m u:usuario4:rx hmi_planta
```

> **Explicación:** El parámetro `-m` (*modify*) añade o modifica una entrada dentro de la ACL del recurso especificado sin alterar las demás entradas.

---

### Paso 7: Configurar ACL sobre la carpeta `planos/` y ficheros existentes
Aplicamos permisos de forma recursiva a la estructura de archivos existente.

```bash
# Ejemplo: Permitir lectura y traversado (rx) a grupo2 en la carpeta y su contenido
sudo setfacl -R -m g:grupo2:rx planos/

# Ejemplo: Conceder permisos de lectura/escritura/ejecución (rwx) a usuario1 sobre toda la carpeta
sudo setfacl -R -m u:usuario1:rwx planos/
```

> **Explicación:** El modificador `-R` (*recursive*) aplica las reglas de manera descendente tanto al directorio como a todos los elementos contenidos en él en ese momento.

---

### Paso 8: Configurar ACL por defecto (`setfacl -d`)
Configuramos la herencia de permisos para elementos que se creen a futuro dentro de la carpeta `planos/`.

```bash
# Definir ACL por defecto en planos/
sudo setfacl -d -m g:grupo2:rx planos/
sudo setfacl -d -m u:usuario1:rwx planos/
```

> **Explicación:** La opción `-d` (*default*) establece las reglas de herencia del directorio. Todo nuevo archivo o subdirectorio creado dentro de `planos/` heredará automáticamente estas ACLs sin necesidad de ejecutar `setfacl` de nuevo.

---

## Tarea 4 — Verificación y Evidencias

### Paso 9: Comprobar la estructura con `getfacl`
Inspeccionamos las tablas de control de acceso aplicadas a cada recurso.

```bash
# Comprobación del ejecutable
getfacl hmi_planta

# Comprobación recursiva del directorio planos y su contenido
getfacl -R planos/
```

> **Explicación:** `getfacl` muestra el propietario, el grupo propietario, los permisos habituales y la lista detallada de entradas de usuarios y grupos adicionales junto con la máscara activa.

* **📸 Captura 5:** Salida en pantalla del comando `getfacl hmi_planta`.
* **📸 Captura 6:** Salida completa del comando `getfacl -R planos/`.

---

### Paso 10: Verificación de acceso por cada perfil de usuario

Para comprobar que los permisos se cumplen de acuerdo con el diagrama, iniciamos sesión con los usuarios mediante `su -l <usuario>` y ejecutamos pruebas de acceso.

# 1. ¿Por qué los permisos tradicionales de Unix son insuficientes?
El modelo clásico de Linux (usuario, grupo y otros) resulta limitante porque únicamente contempla una única entidad por cada categoría (un solo dueño y un único grupo asociado). Es imposible definir políticas complejas y personalizadas para múltiples perfiles de forma simultánea.

Caso práctico: Imaginemos el fichero hmi_planta. Requerimos que el personal de planta (operarios) tenga acceso de ejecución, que Néstor posea privilegios de lectura y ejecución, y que Lucía cuente con control total (lectura, escritura y ejecución). Los mecanismos de Unix estándar se quedan cortos ante esta variedad de combinaciones, obligando a recurrir a las ACL (Listas de Control de Acceso), las cuales abren la puerta a definir reglas personalizadas para cada usuario o colectivo de manera independiente.

# 2. Función de la máscara (mask) en las ACL
La mask actúa como un techo o límite superior para los permisos efectivos de todos los usuarios y grupos específicos declarados en la ACL (a excepción del propietario original del fichero).

Impacto real: Imaginemos esta situación:

Plaintext
user:lucia:rwx   mask::r-x
A pesar de que el perfil de Lucía figure con el bloque completo rwx, el permiso de escritura quedará neutralizado debido a la restricción impuesta por la máscara (r-x). En consecuencia, los privilegios reales que entran en vigor surgen de la intersección (o cálculo lógico) entre los permisos asignados y el valor actual de la máscara.

# 3. Diferencia clave entre la ACL de acceso y la ACL por defecto
ACL de acceso (Access ACL):

Definición: Es el conjunto de reglas activas que rigen de forma inmediata sobre un archivo o directorio ya existente.

Ámbito: Se consulta cada vez que se intenta interactuar con el recurso.

ACL por defecto (Default ACL):

Definición: Funciona como una regla de herencia que únicamente se puede aplicar a directorios.

Ámbito: No modifica los permisos del directorio contenedor, sino que se propaga automáticamente hacia abajo, dictando qué ACL heredarán los nuevos ficheros o carpetas que se creen en su interior.

# 4. Alta de un nuevo operario y reutilización de permisos
Si el entorno se ha estructurado correctamente empleando grupos, la integración de un nuevo empleado resulta sumamente ágil:

### Creación de la cuenta de usuario (por ejemplo, pedro):

```Bash
sudo useradd -m -s /bin/bash pedro
```
Inscripción en el grupo correspondiente:

```Bash
sudo usermod -aG operarios pedro
```
Validación de la pertenencia:
```
Bash
id pedro
```
Gracias a que el fichero hmi_planta ya tiene configurado el acceso a través del grupo (g:operarios:--x), no hace falta modificar de nuevo las ACL. El recién incorporado heredará de forma automática dichos privilegios por el mero hecho de formar parte del grupo.

Para comprobar que todo funciona correctamente, puedes ejecutar:
```
Bash
sudo -u pedro test -x /home/hmi_planta && echo "Pedro tiene permiso de ejecución"
```
