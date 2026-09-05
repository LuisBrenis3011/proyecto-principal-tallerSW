# API Contract — Tareas (To-Do)

Contrato de la API que conecta frontend ↔ backend ↔ base de datos para la
funcionalidad de CRUD de tareas.

## 1. Generalidades

| Ítem         | Valor                              |
|--------------|------------------------------------|
| Base URL     | `/api/tasks`                       |
| Formato      | JSON (`application/json`)          |
| Errores      | RFC 7807 (`application/problem+json`) |
| Convención   | camelCase                          |
| Identificador| `UUID` (formato canónico)          |
| Fechas       | ISO-8601 (`2026-09-05T10:00:00Z`)  |

## 2. Endpoints

| Método | Ruta              | Éxito | Errores |
|--------|-------------------|-------|---------|
| GET    | `/api/tasks`      | 200   | —       |
| GET    | `/api/tasks/{id}` | 200   | 400, 404 |
| POST   | `/api/tasks`      | 201   | 400     |
| PUT    | `/api/tasks/{id}` | 200   | 400, 404 |
| DELETE | `/api/tasks/{id}` | 204   | 400, 404 |

## 3. Modelos de datos

### 3.1 `TaskRequest` (entrada para POST/PUT)

```json
{
  "title": "Comprar pan",
  "done": false
}
```

| Campo   | Tipo     | Obligatorio | Reglas                          |
|---------|----------|-------------|---------------------------------|
| `title` | `string` | Sí (POST)   | 1–255 caracteres tras `trim`    |
| `done`  | `boolean`| No          | Default `false` en POST; en PUT reemplaza el valor |

### 3.2 `TaskResponse` (salida de todos los endpoints)

```json
{
  "id": "7b2f2f1e-9c3a-4a5d-9c6b-8e7e6d5c4b3a",
  "title": "Comprar pan",
  "done": false,
  "createdAt": "2026-09-05T10:00:00Z"
}
```

| Campo       | Tipo      | Descripción                       |
|-------------|-----------|-----------------------------------|
| `id`        | `string`  | UUID de la tarea                  |
| `title`     | `string`  | Título de la tarea                |
| `done`      | `boolean` | Estado de completado              |
| `createdAt` | `string`  | Fecha de creación (ISO-8601)      |

### 3.3 `ProblemDetail` (errores)

```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "Task not found: 7b2f2f1e-9c3a-4a5d-9c6b-8e7e6d5c4b3a",
  "instance": "/api/tasks/7b2f2f1e-9c3a-4a5d-9c6b-8e7e6d5c4b3a"
}
```

## 4. Ejemplos

### POST `/api/tasks`

Request:
```json
{ "title": "Estudiar Spring WebFlux" }
```

Response `201 Created`:
```json
{
  "id": "a1b2c3d4-0000-0000-0000-000000000001",
  "title": "Estudiar Spring WebFlux",
  "done": false,
  "createdAt": "2026-09-05T12:00:00Z"
}
```
Cabeceras: `Location: /api/tasks/a1b2c3d4-0000-0000-0000-000000000001`

### GET `/api/tasks`

Response `200 OK`:
```json
[
  {
    "id": "a1b2c3d4-0000-0000-0000-000000000001",
    "title": "Estudiar Spring WebFlux",
    "done": true,
    "createdAt": "2026-09-05T12:00:00Z"
  }
]
```

### PUT `/api/tasks/{id}`

Request:
```json
{ "title": "Estudiar Spring WebFlux", "done": true }
```

Response `200 OK`: `TaskResponse` con los valores actualizados (mismo `id` y `createdAt`).

### DELETE `/api/tasks/{id}`

Response `204 No Content` (sin cuerpo).

## 5. Esquema de base de datos

Flyway `V1__create_tasks.sql`:

```sql
CREATE TABLE IF NOT EXISTS tasks (
    id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title      VARCHAR(255) NOT NULL,
    done       BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## 6. Conexión (backend, `application.properties`)

| Clave                 | Valor (desarrollo local)                        |
|-----------------------|--------------------------------------------------|
| `spring.r2dbc.url`    | `r2dbc:postgresql://localhost:5432/taller_sw`    |
| `spring.r2dbc.username`| `postgres`                                       |
| `spring.r2dbc.password`| credencial de desarrollo (no commitear secretos) |
| `spring.flyway.url`   | `jdbc:postgresql://localhost:5432/taller_sw`     |
| `spring.flyway.user`  | `postgres`                                       |

> Los valores sensibles se inyectan por variables de entorno; no se versionan
> contraseñas reales.

## 7. Consumo desde el frontend

- Cliente HTTP en `frontend/src/api/tasks.ts` usando `fetch`.
- Base URL configurable con `VITE_API_URL`; en desarrollo se usa el proxy de Vite:
  `/api → http://localhost:8080`.
- Tipos compartidos en `frontend/src/types/Task.ts` (sin `any`).
