# Intranet Docente: backend (API REST)

Backend de la **intranet de gestión docente** del IES Ramón Arcas Meca, dentro del Proyecto Intermodular del CFGS de Desarrollo de Aplicaciones Web (curso 2026/2027).

Esta es la base de la **Fase 1: Cimientos** de la memoria técnica. Incluye:

- Autenticación con **JWT** (access token y refresh token con rotación) y sesiones revocables.
- **Roles y permisos** comprobados con dependencias de FastAPI (`Depends()`).
- **Registro de auditoría** de las acciones relevantes (login, logout y alta de usuarios).
- Alta manual de usuarios por un administrador.
- Inicialización automática de la base de datos MongoDB: índices, roles por defecto y administrador inicial.

## Tecnologías

| Capa | Tecnología |
|---|---|
| Lenguaje | Python 3.10 o superior |
| Framework | FastAPI y Uvicorn |
| Base de datos | MongoDB (driver `pymongo`) |
| Seguridad | PyJWT (HS256) y `pwdlib` con Argon2 para las contraseñas |
| Configuración | `pydantic-settings` y fichero `.env` |
| Pruebas | pytest y `mongomock` (no necesita MongoDB real) |

## Estructura

```
.
├── app/
│   ├── main.py            # Crea la app FastAPI, conecta a MongoDB y registra las rutas
│   ├── api/               # Endpoints: auth, users, roles
│   ├── core/              # Configuración (.env) y seguridad (JWT, hash, permisos)
│   ├── db/                # Acceso a MongoDB e inicialización de la base de datos
│   ├── models/            # Forma de los documentos de MongoDB
│   ├── schemas/           # Modelos Pydantic de entrada y salida de la API
│   └── services/          # Lógica de negocio
├── tests/                 # Pruebas con pytest y smoke test contra MongoDB real
├── docker-compose.yml     # MongoDB en Docker (opcional)
├── requirements.txt
├── .env.example           # Plantilla de configuración (copiar a .env)
└── LICENSE
```

---

## Puesta en marcha en local

Los comandos están escritos para **PowerShell en Windows**. En Linux o macOS se indica la alternativa cuando cambia.

### Requisitos previos

- **Python 3.10 o superior**. Compruébalo con `py --version` (Windows) o `python3 --version`.
- **Git**, para clonar el repositorio.
- **MongoDB** con una de estas dos opciones (ver el paso 4):
  - **Opción A:** Docker Desktop. Es la opción recomendada.
  - **Opción B:** MongoDB Community Server instalado en el sistema.

### 1. Clonar el repositorio

```powershell
git clone <URL-del-repositorio> intranet-backend
Set-Location intranet-backend
```

### 2. Crear y activar el entorno virtual

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

En Linux o macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Si PowerShell bloquea la activación por la política de ejecución, ejecuta antes esto solo en la sesión actual:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

El prompt debe empezar por `(.venv)`. La carpeta `.venv/` **no se sube al repositorio**, porque está en `.gitignore`.

### 3. Instalar las dependencias

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Montar la base de datos MongoDB

La API espera MongoDB en la URI que indique `MONGO_URI` en el `.env`. La plantilla usa **`mongodb://localhost:27017`**.

No hace falta crear a mano ni la base de datos ni las colecciones: **la API las crea al arrancar** (ver el apartado *Qué hace la API con la base de datos*). Solo necesitas un servidor MongoDB en marcha.

#### Opción A: MongoDB con Docker (recomendada)

Con Docker Desktop abierto, desde la raíz del proyecto:

```powershell
docker compose up -d
```

Esto descarga la imagen `mongo:7.0`, crea el contenedor `intranet-mongodb` y expone MongoDB en el puerto **27017** de tu equipo. Los datos se guardan en el volumen `mongodb_data`, así que no se pierden al parar el contenedor.

Comandos útiles:

```powershell
docker compose ps              # ver si el contenedor está en marcha
docker compose logs mongodb    # ver los logs de MongoDB
docker compose stop            # parar MongoDB (conserva los datos)
docker compose down -v         # borrar contenedor y datos (empezar de cero)
```

#### Opción B: MongoDB instalado en el sistema

1. Descarga e instala **MongoDB Community Server** desde <https://www.mongodb.com/try/download/community>. En Windows, deja marcada la opción *Install MongoDB as a Service*.
2. MongoDB escucha por defecto en el puerto **27017**. Hay dos formas de hacerlo coincidir con el `.env`:
   - Cambiar `MONGO_URI=mongodb://localhost:27017` en tu `.env` (más sencillo), **o**
   - Cambiar el puerto del servicio a 27017 en `mongod.cfg` (`net.port: 27017`) y reiniciar el servicio.
3. Comprueba que el servicio está en marcha:

   ```powershell
   Get-Service MongoDB
   ```

**Opcional:** con [MongoDB Compass](https://www.mongodb.com/products/compass) o `mongosh` puedes conectarte a la misma URI e inspeccionar las colecciones.

### 5. Configurar las variables de entorno

Copia la plantilla:

```powershell
Copy-Item .env.example .env
```

En Linux o macOS: `cp .env.example .env`.

Edita `.env` y rellena como mínimo estos valores:

| Variable | Descripción |
|---|---|
| `MONGO_URI` | URI de MongoDB. `mongodb://localhost:27017` con Docker, o `27017` con la instalación por defecto. |
| `DATABASE_NAME` | Nombre de la base de datos. Por defecto, `intranet`. |
| `SECRET_KEY` | **Obligatoria.** Clave para firmar los JWT, de **32 caracteres como mínimo**. |
| `JWT_ALGORITHM` | Solo se admite `HS256`. |
| `JWT_ACCESS_TOKEN_EXPIRE_MINUTES` | Duración del access token. Por defecto, 30 minutos. |
| `JWT_REFRESH_TOKEN_EXPIRE_DAYS` | Duración del refresh token. Por defecto, 7 días. |
| `INITIAL_ADMIN_EMAIL` | Email del primer administrador. |
| `INITIAL_ADMIN_NAME` | Nombre del primer administrador. |
| `INITIAL_ADMIN_PASSWORD` | Contraseña del primer administrador, de **12 caracteres como mínimo**. |

Para generar una `SECRET_KEY` segura:

```powershell
python -c "import secrets; print(secrets.token_urlsafe(48))"
```

Las tres variables `INITIAL_ADMIN_*` deben rellenarse **todas o ninguna**. El administrador inicial solo se crea si la colección `users` está vacía. Después del primer arranque puedes dejarlas vacías.

> **Importante:** `.env` contiene secretos y **nunca se sube al repositorio**, porque ya está en `.gitignore`. Solo se versiona `.env.example`.

### 6. Arrancar la API

Con el entorno virtual activado y MongoDB en marcha:

```powershell
uvicorn app.main:app --reload
```

- API: <http://127.0.0.1:8000>
- Documentación interactiva (Swagger): <http://127.0.0.1:8000/docs>
- Documentación alternativa (ReDoc): <http://127.0.0.1:8000/redoc>

Si MongoDB no está accesible o la `SECRET_KEY` no es válida, la API **no arranca** y muestra el error en la consola (ver *Solución de problemas*).

### 7. Probar que funciona

1. Abre <http://127.0.0.1:8000/docs>.
2. Ejecuta `POST /api/intranet/v1/auth/login` con el administrador inicial:

   ```json
   { "email": "admin@tu-centro.es", "password": "la-contraseña-del-.env" }
   ```

3. Copia el `access_token` de la respuesta, pulsa **Authorize** (arriba a la derecha) y pégalo.
4. Prueba `GET /api/intranet/v1/users/me` y `GET /api/intranet/v1/roles`.

---

## Qué hace la API con la base de datos

Cada vez que arranca, `app/db/init_db.py` deja MongoDB preparada. Puede ejecutarse varias veces sin duplicar nada:

**Colecciones**

| Colección | Contenido |
|---|---|
| `users` | Usuarios: nombre, email, hash de la contraseña, roles y si están activos. |
| `roles` | Roles con su lista de permisos. |
| `sessions` | Sesiones de login (refresh tokens activos o revocados). |
| `audit_logs` | Registro de auditoría: acción, resultado, actor, objetivo y fecha. |

**Índices**

- `users.email`: **único**, para que no haya dos usuarios con el mismo email.
- `sessions.expires_at`: índice **TTL**, con el que MongoDB borra solo las sesiones caducadas.
- `audit_logs.occurred_at`: para consultar la auditoría por fecha.

**Roles por defecto**

| Rol | Permisos |
|---|---|
| `admin` | `*` (acceso total) |
| `profesor` | `announcements.read` |
| `director` | `users.read`, `roles.read`, `announcements.read` |
| `staff` | `users.read`, `roles.read` |

**Administrador inicial:** se crea con los datos `INITIAL_ADMIN_*` si no hay ningún usuario.

## Endpoints disponibles

Prefijo común: `/api/intranet/v1`

| Método | Ruta | Acceso | Descripción |
|---|---|---|---|
| POST | `/auth/login` | Público | Login con email y contraseña. Devuelve el access token y el refresh token. |
| POST | `/auth/refresh` | Público (con refresh token) | Renueva los tokens. El refresh token anterior queda invalidado. |
| POST | `/auth/logout` | Autenticado | Revoca la sesión actual. |
| GET | `/users/me` | Autenticado | Perfil del usuario actual. |
| GET | `/users` | Permiso `users.read` | Lista los usuarios (`limit`, `skip`). |
| POST | `/users/register` | Permiso `users.write` | Alta manual de un usuario. |
| GET | `/roles` | Autenticado | Lista los roles. |
| GET | `/roles/{role_name}/permissions` | Autenticado | Permisos de un rol. |

## Pruebas

**Pruebas unitarias.** No necesitan MongoDB, porque usan `mongomock`:

```powershell
pytest
```

**Smoke test contra MongoDB real.** Necesita MongoDB en marcha y el `.env` con `MONGO_URI` y `SECRET_KEY`. Crea una base de datos temporal, arranca la API, recorre el flujo completo (login, permisos, refresh y logout) y **borra la base temporal** al terminar. Por seguridad, solo acepta MongoDB en `localhost`.

```powershell
python tests/smoke_mongodb.py
```

## Solución de problemas

| Síntoma | Causa probable | Solución |
|---|---|---|
| `ServerSelectionTimeoutError` al arrancar | MongoDB no está en marcha o el puerto no coincide | Comprueba `docker compose ps` o `Get-Service MongoDB`, y revisa el puerto de `MONGO_URI`. |
| `SECRET_KEY debe tener al menos 32 caracteres` | `SECRET_KEY` vacía o demasiado corta | Genera una con el comando del paso 5. |
| `Para crear el administrador inicial configura...` | Solo has rellenado algunas variables `INITIAL_ADMIN_*` | Rellena las tres o déjalas todas vacías. |
| `INITIAL_ADMIN_PASSWORD debe tener al menos 12 caracteres` | Contraseña demasiado corta | Usa 12 caracteres o más. |
| Aviso `No hay usuarios...` en la consola | Base vacía y sin variables `INITIAL_ADMIN_*` | Rellénalas y reinicia la API. |
| `uvicorn` no se reconoce | El entorno virtual no está activado | Ejecuta `.\.venv\Scripts\Activate.ps1`. |
| Login devuelve 401 con el admin inicial | El admin no se creó porque ya había usuarios | Usa ese usuario, o empieza de cero con `docker compose down -v`. |

## Licencia

Distribuido bajo licencia **MIT**. Ver [LICENSE](LICENSE).
