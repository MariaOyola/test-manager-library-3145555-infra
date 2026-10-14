# Manager Library Infra

Infraestructura base para Manager Library.

## HU-01 - Health check

Esta rama prepara PostgreSQL 16 vacio para que el backend Laravel sea el unico responsable de crear el esquema mediante migraciones.

```bash
docker compose up -d
```

## Reglas del spec

- Este repositorio no contiene `CREATE TABLE`.
- La base arranca vacia.
- El volumen `manager_library_pgdata` conserva los datos entre arranques.
- El backend debe crear tablas, indices, restricciones y semillas desde sus migraciones.

## Servicio

| Servicio | Puerto | Descripcion |
|---|---:|---|
| `db` | `5432` | PostgreSQL 16 para Manager Library |

Cadena de conexion esperada por el backend:

```text
postgresql://library:library@localhost:5432/library
```
