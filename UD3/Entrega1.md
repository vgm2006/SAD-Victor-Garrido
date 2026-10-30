# Configuración de ACL en Linux — Caso Torrent S.L.

> **Módulo 0378 · Seguridad y Alta Disponibilidad — 2º ASIX**
> UD3 · Seguridad lógica — Actividad práctica

## Índice

1. [Objetivo](#1-objetivo)
2. [Interpretación del diagrama y tabla de permisos](#2-interpretación-del-diagrama-y-tabla-de-permisos)
3. [Tarea 1 — Preparar el entorno](#3-tarea-1--preparar-el-entorno)
4. [Tarea 2 — Permisos base](#4-tarea-2--permisos-base-propietario--grupo--resto)
5. [Tarea 3 — Configurar las ACL](#5-tarea-3--configurar-las-acl-con-setfacl)
6. [Tarea 4 — Verificación](#6-tarea-4--verificación)
7. [Script completo](#7-script-completo)
8. [Preguntas de reflexión](#8-preguntas-de-reflexión)
9. [Capturas de pantalla](#9-capturas-de-pantalla)

---

## 1. Objetivo

Torrent S.L. tiene dos recursos que hay que proteger:

- `hmi_planta`: aplicación de control de planta (fichero ejecutable).
- `planos/`: carpeta compartida con `autoclave.dwg`, `caldera.dwg` y `destilacion.dwg`.

Hay que dar acceso a tres grupos de personal (**operarios**, **ingeniería** y **supervisores**) siguiendo exactamente el esquema del diagrama. Los permisos clásicos de Unix (propietario / grupo / resto) solo permiten **un** grupo propietario por fichero, así que se usan **ACL** (`setfacl` / `getfacl`).

| Grupo | Usuarios |
|---|---|
| `operarios` | alberto, oscar, hugo |
| `ingenieria` | nestor, victor, lucia |
| `supervisores` | carlos, mary, john |

---

## 2. Interpretación del diagrama y tabla de permisos

`r` = leer · `w` = escribir · `x` = ejecutar.

> ⚠️ **Revisa esta tabla contra tu diagrama.** En el diagrama hay flechas que salen de nombres concretos de un mismo grupo (por ejemplo, dos flechas distintas de ingeniería hacia `hmi_planta`, y dos flechas rojas de "leer" que parten de alberto y oscar hacia ficheros distintos). Aquí he supuesto la lectura más razonable; si al mirarlo tú ves otra asignación, solo tienes que cambiar el usuario/fichero en los comandos `setfacl` correspondientes.

### Tabla (a nivel de grupo)

| Grupo | Recurso | Permisos `rwx` | Fuente en el diagrama |
|---|---|---|---|
| operarios | `hmi_planta` | `--x` | flecha "ejecutar" |
| operarios | `planos/` | `r--` (solo en ficheros concretos) | flechas rojas "leer" |
| ingenieria | `hmi_planta` | `r-x` | flecha "leer, ejecutar" |
| ingenieria | `planos/` | `rw-` | flecha "leer, escribir" |
| supervisores | `planos/` | `r-x` | flecha "leer, ejecutar" |

### Casos individuales (ACL de usuario)

| Usuario | Recurso | Permisos | Fuente en el diagrama |
|---|---|---|---|
| alberto | `planos/autoclave.dwg` | `r--` | flecha roja "leer" |
| oscar | `planos/caldera.dwg` | `r--` | flecha roja "leer" |
| victor | `hmi_planta` | `rwx` | flecha "leer, escribir, ejecutar" |
| nestor | `planos/` y su contenido | `rwx` | flecha "leer, escribir, ejecutar" |

### Una aclaración importante sobre directorios

En una **carpeta**, el permiso `x` significa "poder entrar / atravesar" y `r` significa "poder listar". Para leer `autoclave.dwg` hace falta `x` sobre `planos/`, aunque el diagrama solo diga "leer". Por eso:

- sobre la **carpeta** `planos/` se da `x` a los grupos que necesitan acceder a ella (operarios con `--x`: pueden atravesarla pero **no listarla**, y solo abren los ficheros que tengan concedidos);
- sobre los **ficheros** se aplican los permisos reales del diagrama.

---

## 3. Tarea 1 — Preparar el entorno

### 3.1. Comprobar soporte de ACL

```bash
dpkg -s acl
```

Si no está instalado:

```bash
sudo apt update && sudo apt install -y acl
```

El sistema de ficheros también debe estar montado con ACL (en ext4 moderno viene activado por defecto). Se puede comprobar con:

```bash
mount | grep " / "
tune2fs -l $(findmnt -n -o SOURCE /) | grep -i "default mount options"
```

### 3.2. Crear grupos y usuarios

```bash
sudo groupadd operarios
sudo groupadd ingenieria
sudo groupadd supervisores

# Usuario de sistemas (propietario de los recursos)
sudo useradd -m -s /bin/bash sistemas

# Operarios
for u in alberto oscar hugo; do
  sudo useradd -m -s /bin/bash -g operarios "$u"
done

# Ingeniería
for u in nestor victor lucia; do
  sudo useradd -m -s /bin/bash -g ingenieria "$u"
done

# Supervisores
for u in carlos mary john; do
  sudo useradd -m -s /bin/bash -g supervisores "$u"
done
```

Asigna una contraseña a cada usuario (necesario para poder hacer `su -l`):

```bash
for u in sistemas alberto oscar hugo nestor victor lucia carlos mary john; do
  echo "$u:Torrent2026!" | sudo chpasswd
done
```

Comprueba que se han creado bien:

```bash
getent group operarios ingenieria supervisores
id alberto
id nestor
id carlos
```

### 3.3. Crear la estructura de ficheros

Trabajaremos en un directorio común, por ejemplo `/srv/torrent`:

```bash
sudo mkdir -p /srv/torrent/planos
cd /srv/torrent

# Fichero ejecutable hmi_planta
sudo tee hmi_planta > /dev/null <<'EOF'
#!/bin/bash
echo "HMI de planta - Torrent S.L. - en ejecución"
EOF

# Planos
sudo touch planos/autoclave.dwg planos/caldera.dwg planos/destilacion.dwg
```

> El directorio `/srv/torrent` debe poder atravesarse por todos los usuarios (`chmod 755 /srv/torrent`), porque si no nadie llegará a los recursos de dentro.

```bash
sudo chmod 755 /srv/torrent
```

---

## 4. Tarea 2 — Permisos base (propietario / grupo / resto)

Propietario: el usuario `sistemas`. Grupo propietario: `ingenieria` (es el grupo con más control sobre ambos recursos).

```bash
sudo chown sistemas:ingenieria /srv/torrent/hmi_planta
sudo chown -R sistemas:ingenieria /srv/torrent/planos

# Permisos de partida restrictivos: el resto = ningún acceso
sudo chmod 750 /srv/torrent/hmi_planta
sudo chmod 750 /srv/torrent/planos
sudo chmod 640 /srv/torrent/planos/*.dwg
```

Con esto solo `sistemas` y el grupo `ingenieria` tienen algo de acceso base; operarios y supervisores no pueden hacer nada hasta que se les conceda con ACL.

> **Orden importante:** hacer el `chmod` **antes** del `setfacl`. Cuando una ACL ya tiene entradas con nombre, `chmod` sobre el grupo modifica la *máscara* y no el permiso real del grupo, lo que lleva a resultados confusos.

---

## 5. Tarea 3 — Configurar las ACL con `setfacl`

Sintaxis:

```bash
setfacl -m g:grupo:rwx recurso      # ACL de acceso para un grupo
setfacl -m u:usuario:rwx recurso    # ACL de acceso para un usuario
setfacl -R -m g:grupo:rwx carpeta/  # recursivo
setfacl -d -m g:grupo:rwx carpeta/  # ACL por defecto (herencia)
```

### 5.1. ACL sobre `hmi_planta`

```bash
cd /srv/torrent

# Operarios: solo ejecutar
sudo setfacl -m g:operarios:--x hmi_planta

# Ingeniería: leer y ejecutar (ya es grupo propietario con 5 = r-x; lo hacemos explícito)
sudo setfacl -m g:ingenieria:r-x hmi_planta

# Victor (ingeniería): leer, escribir y ejecutar
sudo setfacl -m u:victor:rwx hmi_planta
```

### 5.2. ACL sobre la carpeta `planos/`

```bash
# Acceso a la carpeta (atravesar / listar)
sudo setfacl -m g:operarios:--x  planos      # solo atravesar, no listar
sudo setfacl -m g:ingenieria:rwx planos      # entrar, listar y crear/borrar ficheros
sudo setfacl -m g:supervisores:r-x planos    # entrar y listar
sudo setfacl -m u:nestor:rwx planos
```

### 5.3. ACL sobre los ficheros ya existentes

```bash
# Ingeniería: leer y escribir
sudo setfacl -m g:ingenieria:rw- planos/*.dwg

# Supervisores: leer y ejecutar
sudo setfacl -m g:supervisores:r-x planos/*.dwg

# Nestor: control total
sudo setfacl -m u:nestor:rwx planos/*.dwg

# Operarios: solo los ficheros concretos del diagrama
sudo setfacl -m u:alberto:r-- planos/autoclave.dwg
sudo setfacl -m u:oscar:r--   planos/caldera.dwg
```

> Alternativa si tu diagrama indica que **todo el grupo** operarios puede leer los planos: `sudo setfacl -m g:operarios:r-- planos/*.dwg` y, en la carpeta, `g:operarios:r-x`.

### 5.4. ACL por defecto (herencia para ficheros nuevos)

Los ficheros que se creen dentro de `planos/` no heredan las ACL de acceso del resto; solo heredan las **ACL por defecto** de la carpeta. Conviene aplicarla a los permisos que afectan a ficheros (no a los de la carpeta en sí):

```bash
sudo setfacl -d -m g:ingenieria:rw-   planos
sudo setfacl -d -m g:supervisores:r-x planos
sudo setfacl -d -m u:nestor:rwx       planos
```

No se aplican por defecto los permisos individuales de operarios (alberto y oscar), porque el diagrama los limita a ficheros concretos, no a todo lo que se cree.

Con esto, si ingeniería crea `nuevo.dwg` en `planos/`, ingeniería, supervisores y nestor tendrán los mismos permisos que en el resto de planos.

---

## 6. Tarea 4 — Verificación

### 6.1. Con `getfacl`

```bash
getfacl hmi_planta
getfacl -R planos/
```

Resultado esperado para `hmi_planta` (aproximado):

```text
# file: hmi_planta
# owner: sistemas
# group: ingenieria
user::rwx
user:victor:rwx
group::r-x
group:ingenieria:r-x
group:operarios:--x
mask::rwx
other::---
```

### 6.2. Probar como cada usuario

```bash
su -l alberto
cd /srv/torrent
./hmi_planta                    # debe funcionar
cat planos/autoclave.dwg        # debe funcionar
cat planos/caldera.dwg          # debe dar Permission denied
ls planos/                      # debe dar Permission denied
exit
```

Tabla de comprobaciones esperadas:

| Usuario | `./hmi_planta` | Leer planos | Escribir planos |
|---|---|---|---|
| alberto | ✅ ejecutar | ✅ solo `autoclave.dwg` | ❌ |
| oscar | ✅ ejecutar | ✅ solo `caldera.dwg` | ❌ |
| hugo | ✅ ejecutar | ❌ | ❌ |
| nestor | ✅ leer/ejecutar | ✅ | ✅ |
| victor | ✅ leer/escribir/ejecutar | ✅ | ✅ |
| lucia | ✅ leer/ejecutar (`cat` sí, editar no) | ✅ | ✅ |
| carlos / mary / john | ❌ | ✅ | ❌ |

Pruebas rápidas que conviene incluir en las capturas:

```bash
# Como lucia: puede escribir en un plano, pero no modificar hmi_planta
echo "prueba" >> /srv/torrent/planos/caldera.dwg    # OK
echo "prueba" >> /srv/torrent/hmi_planta            # Permission denied

# Como carlos (supervisor): puede leer pero no escribir
cat /srv/torrent/planos/caldera.dwg                 # OK
echo "x" >> /srv/torrent/planos/caldera.dwg         # Permission denied
/srv/torrent/hmi_planta                             # Permission denied
```

---

## 7. Script completo

Guárdalo como `configurar_acl.sh` y ejecútalo con `sudo bash configurar_acl.sh`.

```bash
#!/bin/bash
# =====================================================================
# Configuración de ACL en Linux - Caso Torrent S.L.
# Módulo 0378 - Seguridad y Alta Disponibilidad - 2º ASIX
# =====================================================================
set -e

BASE=/srv/torrent

# ---------------------------------------------------------------------
# TAREA 1 - PREPARAR EL ENTORNO
# ---------------------------------------------------------------------

# 1. Soporte de ACL
dpkg -s acl >/dev/null 2>&1 || apt-get install -y acl

# 2. Grupos
groupadd operarios
groupadd ingenieria
groupadd supervisores

# 3. Usuarios (cada uno con su grupo principal) + usuario de sistemas
useradd -m -s /bin/bash sistemas
for u in alberto oscar hugo;  do useradd -m -s /bin/bash -g operarios    "$u"; done
for u in nestor victor lucia; do useradd -m -s /bin/bash -g ingenieria   "$u"; done
for u in carlos mary john;    do useradd -m -s /bin/bash -g supervisores "$u"; done

# Contraseñas (necesarias para 'su -l')
for u in sistemas alberto oscar hugo nestor victor lucia carlos mary john; do
  echo "$u:Torrent2026!" | chpasswd
done

# 4. Estructura de ficheros
mkdir -p "$BASE/planos"
cat > "$BASE/hmi_planta" <<'EOF'
#!/bin/bash
echo "HMI de planta - Torrent S.L. - en ejecución"
EOF
touch "$BASE/planos/autoclave.dwg" "$BASE/planos/caldera.dwg" "$BASE/planos/destilacion.dwg"
chmod 755 "$BASE"

# ---------------------------------------------------------------------
# TAREA 2 - PERMISOS BASE (hacer antes que setfacl)
# ---------------------------------------------------------------------
chown sistemas:ingenieria "$BASE/hmi_planta"
chown -R sistemas:ingenieria "$BASE/planos"
chmod 750 "$BASE/hmi_planta"
chmod 750 "$BASE/planos"
chmod 640 "$BASE"/planos/*.dwg

# ---------------------------------------------------------------------
# TAREA 3 - ACL
# ---------------------------------------------------------------------
cd "$BASE"

# --- hmi_planta ---
setfacl -m g:operarios:--x  hmi_planta     # operarios: ejecutar
setfacl -m g:ingenieria:r-x hmi_planta     # ingeniería: leer + ejecutar
setfacl -m u:victor:rwx     hmi_planta     # victor: leer + escribir + ejecutar

# --- carpeta planos/ ---
setfacl -m g:operarios:--x    planos       # atravesar, sin listar
setfacl -m g:ingenieria:rwx   planos
setfacl -m g:supervisores:r-x planos
setfacl -m u:nestor:rwx       planos

# --- ficheros existentes ---
setfacl -m g:ingenieria:rw-    planos/*.dwg
setfacl -m g:supervisores:r-x  planos/*.dwg
setfacl -m u:nestor:rwx        planos/*.dwg
setfacl -m u:alberto:r--       planos/autoclave.dwg
setfacl -m u:oscar:r--         planos/caldera.dwg

# --- ACL por defecto: ficheros nuevos en planos/ ---
setfacl -d -m g:ingenieria:rw-   planos
setfacl -d -m g:supervisores:r-x planos
setfacl -d -m u:nestor:rwx       planos

# ---------------------------------------------------------------------
# TAREA 4 - VERIFICACIÓN
# ---------------------------------------------------------------------
getfacl hmi_planta
getfacl -R planos/
```

---

## 8. Preguntas de reflexión

### 1. ¿Por qué no bastan los permisos clásicos de Unix?

Porque cada fichero solo tiene **un propietario, un grupo y "el resto"**: tres conjuntos de permisos. Aquí hay **tres grupos distintos** con permisos diferentes sobre el mismo recurso, más excepciones por usuario.

Ejemplo: `planos/` necesita que ingeniería lea y escriba, que supervisores solo lean y ejecuten, y que operarios casi no accedan. Con permisos clásicos solo podríamos asignar un grupo (por ejemplo `ingenieria`) y el resto de grupos caerían todos en "otros", con los mismos permisos. Tampoco se podría dar a `alberto` lectura en `autoclave.dwg` sin dársela a todo el mundo.

### 2. ¿Qué es la máscara (mask) y cómo afecta?

La **máscara** es el límite máximo de permisos efectivos para todas las entradas de usuario con nombre, la del grupo propietario y las entradas de grupo con nombre. Los permisos reales de un usuario o grupo son la **intersección** entre su entrada y la máscara.

Ejemplo: si `group:ingenieria:rwx` pero `mask::r-x`, `getfacl` mostrará `#effective:r-x`: ingeniería **no** podrá escribir aunque su entrada diga `w`. La máscara se recalcula con `setfacl` (salvo `-n`) y se modifica con `chmod` sobre el grupo, así que conviene revisar `getfacl` después de cualquier `chmod`.

### 3. ¿Qué diferencia hay entre ACL de acceso y ACL por defecto?

- **ACL de acceso:** controlan quién puede hacer qué sobre **ese** fichero o carpeta, ahora mismo. Se usa en cualquier recurso.
- **ACL por defecto (`-d`):** solo existen en **directorios**; no afectan al propio directorio, sino que sirven de plantilla que **heredan** los ficheros y subcarpetas que se creen dentro después.

Se usa la de acceso para el estado actual y la por defecto para que los ficheros nuevos de `planos/` nazcan ya con los permisos correctos, sin tener que repetir `setfacl`.

### 4. Si se incorpora un nuevo operario, ¿qué pasos hay que repetir?

Solo hay que **añadirlo al grupo `operarios`**; al estar las ACL asignadas al grupo, hereda todo el acceso:

```bash
sudo useradd -m -s /bin/bash -g operarios nuevo_operario
sudo passwd nuevo_operario
```

No hay que tocar ningún `setfacl`. La excepción son los permisos que se dieron a usuarios concretos (por ejemplo, lectura de `autoclave.dwg` para `alberto`): esos **no** se heredan por pertenecer al grupo y habría que repetirlos para el nuevo usuario si se quisieran. Esta es la razón por la que es mejor asignar ACL a grupos y no a usuarios individuales siempre que sea posible.

---

## 9. Capturas de pantalla

Añade aquí tus capturas (carpeta `capturas/` del repositorio):

```markdown
![getfacl hmi_planta](capturas/getfacl_hmi_planta.png)
![getfacl planos](capturas/getfacl_planos.png)
```

---

## Estructura sugerida del repositorio

```text
.
├── README.md
├── configurar_acl.sh
└── capturas/
    ├── getfacl_hmi_planta.png
    └── getfacl_planos.png
```
