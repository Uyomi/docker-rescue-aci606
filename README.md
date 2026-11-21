# ACI606 - Docker Rescue Challenge

Aplicación multi-servicio recuperada y desplegada con Docker Compose.

```
Navegador -> Frontend (Nginx :80) -> Backend (FastAPI :8000) -> PostgreSQL (:5432)
              publicado en :8080      red interna de Compose     volumen pgdata
```

## Requisitos
- Docker y Docker Compose v2 (probado en GitHub Codespaces).

## Configuración
```bash
cp .env.example .env
```
Variables: `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `FRONTEND_PORT` (por defecto 8080).
Si no existe `.env`, Compose usa los valores por defecto de `compose.yaml`, así que
`docker compose up --build -d` funciona igualmente. El `.env` no se versiona; solo `.env.example`.

## Iniciar
```bash
docker compose up --build -d
docker compose ps
```
Los tres servicios deben quedar `Up`; `db` y `backend` en `(healthy)`.

## Verificar
```bash
curl -i localhost:8080/api/health          # 200 {"status":"ok","database":"reachable"}
curl -s -X POST localhost:8080/api/items -H 'Content-Type: application/json' -d '{"name":"prueba"}'
curl -s localhost:8080/api/items           # lista los registros
docker compose logs <servicio>             # frontend | backend | db
```
Interfaz web: abrir el puerto 8080 (en Codespaces, pestaña Ports).

## Detener
```bash
docker compose down        # conserva los datos (volumen pgdata)
docker compose down -v     # borra también los datos
```

## Recrear
```bash
docker compose down
docker compose up --build -d
```
Los datos sobreviven mientras no se use `down -v`.

## Reparaciones realizadas
| ID | Falla | Corrección |
|----|-------|-----------|
| R-01 | uvicorn ejecutaba `app:app`, pero el módulo es `main.py` | `CMD ... main:app` |
| R-02 | Faltaba el driver de PostgreSQL | `psycopg2-binary` en requirements.txt |
| R-03 | `DATABASE_URL` apuntaba a `localhost` | host `db` (nombre del servicio) |
| R-04 | Contraseña incorrecta en el backend | credenciales iguales a las de la base |
| R-05 | Puerto publicado `8080:8080`, Nginx escucha en 80 | `8080:80` |
| R-06 | `proxy_pass` a `localhost:8000` | `proxy_pass http://backend:8000/` |
| R-07 | Backend arrancaba antes de que PostgreSQL estuviera listo | healthchecks + `depends_on: service_healthy` |
| R-08 | Datos perdidos al recrear el contenedor `db` | volumen nombrado `pgdata` |

La evidencia de cada incidencia está en `evidencias/`.