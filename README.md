# 🗂️ Pentaract Deployment

Prueba deploy de [Pentaract](https://github.com/Dominux/Pentaract).

## ¿Qué es Pentaract?

Pentaract es un sistema de almacenamiento en la nube que guarda los archivos en un canal privado de Telegram, sin usar espacio en disco del servidor ni servicios de almacenamiento pagos.

## Requisitos

- [Git](https://git-scm.com/downloads)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Una cuenta de Telegram
- Un bot de Telegram (creado con [@BotFather](https://t.me/BotFather))
- Un canal privado de Telegram con el bot como administrador

## Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/leanluca/pentaract.git
cd pentaract
```

### 2. Crear el archivo `.env`

Copiá el archivo de ejemplo y completá los valores:

```bash
cp .env.example .env
```

Editá el `.env` con tus datos:

```env
PORT=8000
WORKERS=4
CHANNEL_CAPACITY=32
SUPERUSER_EMAIL=tu@email.com
SUPERUSER_PASS=tu_contraseña_segura
ACCESS_TOKEN_EXPIRE_IN_SECS=1800
REFRESH_TOKEN_EXPIRE_IN_DAYS=14
SECRET_KEY=<generado con openssl>
TELEGRAM_API_BASE_URL=https://api.telegram.org

DATABASE_USER=pentaract
DATABASE_PASSWORD=pentaract
DATABASE_NAME=pentaract
DATABASE_HOST=db
DATABASE_PORT=5432
```

Para generar el `SECRET_KEY` en Windows:

```bash
& "C:\Program Files\Git\usr\bin\openssl.exe" rand -hex 32
```
Tambien podes usar cualquier generador random hex 32.

### 3. Levantar los contenedores

```bash
docker compose up -d
```

Abrí el navegador en **http://localhost:8000** e ingresá con las credenciales del `.env`.

## Configuración de Telegram

### Crear el bot

1. Abrí [@BotFather](https://t.me/BotFather) en Telegram
2. Ejecutá `/newbot` y seguí las instrucciones
3. Guardá el **token** que te devuelve

### Crear el canal

1. Creá un canal **privado** en Telegram
2. Durante la creación, agregá el bot como miembro
3. Promové el bot a **administrador** con todos los permisos

### Obtener el ID del canal

1. Enviá un mensaje en el canal
2. Reenviáselo a tu bot
3. Abrí en el navegador:
```
https://api.telegram.org/bot<TOKEN>/getUpdates
```
4. Buscá el campo `"forward_from_chat"` → `"id"` en el JSON

### ⚠️ Importante — Chat ID sin el 100

El ID del canal que devuelve Telegram tiene el formato `-1003510256731`.  
Al registrar el storage en Pentaract, **debés ingresar el ID sin el `100`**:

```
Telegram devuelve:  -1003518256431
Ingresar en Pentaract:  -3518256431
```

Esto es porque el código de Pentaract agrega el `100` automáticamente. Si lo ingresás completo, queda duplicado y Telegram no encuentra el canal.

## Uso

1. Entrá a **Storages** y creá uno nuevo con el chat_id **sin el 100**
2. Entrá a **Storage Workers** y registrá el token del bot
3. ¡Listo! Ya podés subir, descargar y organizar archivos

## Bugs conocidos

### El storage desaparece de la UI al borrar todos los archivos

Si borrás todos los archivos de un storage, la UI deja de mostrarlo (bug del proyecto).

**Solución:** Evitá borrar todos los archivos. Si ocurre, recuperalo así:

```bash
docker exec -it pentaract_db psql -U pentaract -d pentaract
```

```sql
DELETE FROM storage_workers WHERE storage_id = '<id>';
DELETE FROM access WHERE storage_id = '<id>';
DELETE FROM storages WHERE id = '<id>';
```

Luego recreá el storage y el worker desde la UI.

## Seguridad

El archivo `.env` contiene información sensible (contraseñas, tokens, secret key) y **no debe subirse al repositorio**.

Asegurate de tener un `.gitignore` con lo siguiente:

```
.env
```

Si accidentalmente subiste el `.env`, ejecutá:

```bash
git rm --cached .env
git add .gitignore
git commit -m "Remove .env from tracking and add .gitignore"
git push
```

Se recomienda además crear un archivo `.env.example` con las claves pero sin los valores, para que otros sepan qué variables configurar:

```env
PORT=
WORKERS=
CHANNEL_CAPACITY=
SUPERUSER_EMAIL=
SUPERUSER_PASS=
ACCESS_TOKEN_EXPIRE_IN_SECS=
REFRESH_TOKEN_EXPIRE_IN_DAYS=
SECRET_KEY=
TELEGRAM_API_BASE_URL=
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_NAME=
DATABASE_HOST=
DATABASE_PORT=
```

## Comandos útiles

```bash
# Levantar
docker compose up -d

# Detener
docker compose down

# Ver logs
docker logs -f pentaract
```

## Créditos

- [Pentaract](https://github.com/Dominux/Pentaract) por [Dominux](https://github.com/Dominux)