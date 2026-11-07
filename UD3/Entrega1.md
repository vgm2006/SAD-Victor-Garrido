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

```bash
# 1. Crear los grupos
sudo groupadd grupo1
sudo groupadd grupo2
sudo groupadd grupo3

# 2. Crear los usuarios y asignarlos a sus respectivos grupos
sudo useradd -m -g grupo1 usuario1
sudo useradd -m -g grupo1 usuario2
sudo useradd -m -g grupo1 usuario3

sudo useradd -m -g grupo2 usuario4
sudo useradd -m -g grupo2 usuario5
sudo useradd -m -g grupo2 usuario6

sudo useradd -m -g grupo3 usuario7
sudo useradd -m -g grupo3 usuario8
sudo useradd -m -g grupo3 usuario9
```

> **Explicación:** 
> * `-m` crea el directorio personal (`/home/usuarioX`).
> * `-g` asigna el grupo primario especificado al usuario recién creado.

#### Comprobación:
```bash
getent group grupo1 grupo2 grupo3
id usuario1 usuario4 usuario7
```


<img width="451" height="86" alt="image" src="https://github.com/user-attachments/assets/ddad7845-6ac4-48a0-97da-d5ea66488529" />



### Paso 3: Crear la estructura de ficheros
Creamos los recursos sobre los que aplicaremos las reglas de acceso.

```bash
# Crear el script/ejecutable
touch hmi_planta
chmod +x hmi_planta

# Crear la carpeta de planos y los ficheros simulados
mkdir planos
touch planos/autoclave.dwg planos/caldera.dwg planos/destilacion.dwg
```

> **Explicación:** `chmod +x` le otorga bit de ejecución al fichero `hmi_planta`. Los archivos dentro de `planos/` se crean como ficheros regulares vacíos.

#### Comprobación:
```bash
ls -la
ls -la planos/
```

* **📸 Captura 3:** Listado detallado (`ls -la`) que muestre la presencia del ejecutable `hmi_planta` y la carpeta `planos/` con sus tres ficheros `.dwg`.

---

## Tarea 2 — Permisos Base (Propietario / Grupo / Resto)

### Paso 4 y 5: Asignar propietario y aplicar permisos restrictivos
Asignamos un propietario administrativo (ej. `root`) y restringimos el acceso general (permisos tradicionales Unix) para obligar a que cualquier acceso específico de los usuarios o grupos de la práctica se resuelva mediante la tabla de ACLs.

```bash
# 1. Asignar propietario root tanto a los ficheros como a la carpeta
sudo chown -R root:root hmi_planta planos/

# 2. Permisos base restrictivos para el ejecutable (700 o 750)
sudo chmod 700 hmi_planta

# 3. Permisos base restrictivos para la carpeta y su contenido (700 para directorio, 600 para archivos)
sudo chmod 700 planos/
sudo chmod 600 planos/*.dwg
```

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

#### Test 1: Usuario con permiso de ejecución sobre `hmi_planta`
```bash
sudo su -l usuario1
./hmi_planta
exit
```
* **📸 Captura 7:** Ejecución exitosa de `hmi_planta` como `usuario1`.

#### Test 2: Usuario SIN permiso de ejecución sobre `hmi_planta`
```bash
sudo su -l usuario7
./hmi_planta
exit
```
* **📸 Captura 8:** Error de `Permiso denegado` (*Permission denied*) al intentar ejecutar `hmi_planta` como `usuario7`.

#### Test 3: Usuario con permisos de lectura y creación en `planos/`
```bash
sudo su -l usuario1
ls -la planos/
cat planos/autoclave.dwg
touch planos/nuevo_plano.dwg
exit
```
* **📸 Captura 9:** Confirmación de lectura y creación correcta del archivo `nuevo_plano.dwg`.

#### Test 4: Comprobación de la Herencia (ACL Default)
Verificar que el nuevo archivo creado en el paso anterior ha heredado automáticamente la ACL:
```bash
getfacl planos/nuevo_plano.dwg
```
* **📸 Captura 10:** Salida de `getfacl` sobre `planos/nuevo_plano.dwg` demostrando que heredó la ACL configurada con `-d`.

#### Test 5: Usuario SIN permisos sobre la carpeta `planos/`
```bash
sudo su -l usuario9
ls -la planos/
exit
```
* **📸 Captura 11:** Mensaje de error `ls: cannot open directory 'planos/': Permission denied`.
