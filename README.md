# Manager Library Infra

PostgreSQL 16 según `spec-.net` y `spec-python` de `test-manager-library-3145555-docs`.

## Alcance

Rama local: `hu-01-dev`. HU-01 es iniciar sesión; `/health` pertenece a T-01. Este Compose despliega solo PostgreSQL. RNF-5 y T-20 (motor, API y frontend con un comando) están pendientes.

Constitution artículo V y ADR-001 de ambas variantes asignan tablas, índices, restricciones y cinco categorías a las migraciones del backend: EF Core en .NET o Alembic en Python. Infra prepara el motor vacío. El esqueleto Laravel existente está fuera de esos planes.

## 1. Preparar

Iniciar Docker Engine/Desktop con contenedores Linux y Compose v2. Tener libre el puerto 5432. Abrir la terminal en este repositorio:

```bash
docker version
docker compose version
```

## 2. Configurar

En PowerShell:

```powershell
Copy-Item .env.example .env
notepad .env
```

En Bash: `cp .env.example .env` y editar el archivo.

Completar `POSTGRES_PASSWORD` con una contraseña propia. `POSTGRES_DB` y `POSTGRES_USER` también son obligatorios. Compose falla si falta alguna. El archivo real `.env` está ignorado por Git.

## 3. Desplegar PostgreSQL

```bash
docker compose up -d --wait
```

Descarga `postgres:16-alpine`, crea la red `manager-library` y el volumen persistente, y espera al health check. No ejecuta migraciones.

## 4. Verificar

```bash
docker compose ps
docker compose logs --tail 50 db
docker compose exec db psql -U library -d library
```

Sustituir usuario y base si se cambiaron en .env. Dentro de psql:

```sql
SELECT tablename FROM pg_tables WHERE schemaname = 'public';
```

Con volumen nuevo, antes de migraciones, devuelve **0 filas**. Salir con `\q`. Un volumen reutilizado conserva tablas. El health check verifica disponibilidad del motor, no cumplimiento de HU.

## 5. Conectar y completar el sistema

| API ejecutada en | Host de PostgreSQL | Puerto |
|---|---|---|
| Equipo local | localhost | 5432 |
| Contenedor conectado a manager-library | db | 5432 |

Formato del plan:

```text
postgresql://<usuario>:<contraseña>@<host>:5432/<base>
```

Codificar caracteres reservados de credenciales para una URI. La aplicación requiere además `JWT_SECRET`, `ADMIN_USERNAME`, `ADMIN_PASSWORD` y `CORS_ORIGIN`; `COVERS_DIR` es opcional. La aplicación provisiona el admin inicial desde entorno.

Para T-20 falta alinear e implementar una variante del backend, sus migraciones y arranque T-17, implementar frontend T-22 e incorporar ambos al Compose. No hay todavía migraciones EF Core/Alembic ni un comando de despliegue completo verificable. Los datos de demostración los crea tool por API (T-21).

## 6. Detener y reiniciar

```bash
docker compose down
docker compose up -d --wait
```

`down` conserva datos. El volumen lógico `manager_library_pgdata` recibe el prefijo del proyecto Compose. `down -v` elimina los datos; no usarlo para una parada normal. Cambiar variables POSTGRES no cambia credenciales de una base inicializada.

## Problemas frecuentes

- Puerto ocupado: liberar 5432 o ajustar el puerto publicado y la conexión local.
- Docker no responde: iniciar Docker Desktop/Engine.
- Variable ausente: completar .env en este directorio.
- Tablas presentes: el volumen ya tenía datos.

Rendimiento, concurrencia, transporte cifrado de contraseñas y errores HTTP (RNF-1 a RNF-4) requieren evidencia del sistema implementado; este Compose local no los acredita.
