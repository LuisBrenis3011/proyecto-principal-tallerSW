# Funcionalidad: CRUD de Tareas (To-Do)

## 1. Objetivo

Permitir gestionar tareas (crear, listar, consultar, actualizar y eliminar) mediante
una API REST reactiva, persistidas en PostgreSQL, con una SPA en React que consuma
dicha API.

## 2. Alcance

### Incluido

- Endpoints CRUD para el recurso `Task`.
- Persistencia reactiva en PostgreSQL vía Spring Data R2DBC.
- Migración del esquema de base de datos con Flyway.
- Frontend para listar, crear, marcar como hecha y eliminar tareas.

### No incluido (futuras iteraciones)

- Autenticación y usuarios.
- Asignación de tareas a usuarios.
- Fechas de vencimiento, prioridades o etiquetas.
- Paginación y filtros.

## 3. Entidad de dominio: `Task`

| Campo       | Tipo     | Reglas                                                          |
|-------------|----------|-----------------------------------------------------------------|
| `id`        | `UUID`   | Generado por la BD (`gen_random_uuid()`). De solo lectura.      |
| `title`     | `String` | Obligatorio, 1–255 caracteres. Se normaliza (`trim`).           |
| `done`      | `Boolean`| Estado de completado. Por defecto `false`.                      |
| `createdAt` | `Instant`| Fecha de creación. Generada por la BD (`now()`). Solo lectura.  |

## 4. Casos de uso

| ID    | Caso de uso                | Actor     |
|-------|----------------------------|-----------|
| UC-01 | Crear tarea                | Usuario   |
| UC-02 | Listar tareas              | Usuario   |
| UC-03 | Obtener tarea por id       | Usuario   |
| UC-04 | Actualizar tarea           | Usuario   |
| UC-05 | Eliminar tarea             | Usuario   |

## 5. Criterios de aceptación

### UC-01 — Crear tarea

- **Dado** un `title` válido (1–255 caracteres tras `trim`),
  **cuando** se hace `POST /api/tasks`,
  **entonces** se crea la tarea con `done = false`, se retorna `201` con el recurso
  y la cabecera `Location` apuntando a la nueva tarea.
- **Dado** un `title` vacío, nulo o de más de 255 caracteres,
  **cuando** se hace `POST /api/tasks`,
  **entonces** se retorna `400` con un `Problem Detail` explicando el campo inválido.

### UC-02 — Listar tareas

- **Dado** un conjunto de tareas existente,
  **cuando** se hace `GET /api/tasks`,
  **entonces** se retorna `200` con el arreglo de tareas (vacío si no hay ninguna).

### UC-03 — Obtener tarea por id

- **Dado** una tarea con id conocido,
  **cuando** se hace `GET /api/tasks/{id}`,
  **entonces** se retorna `200` con la tarea.
- **Dado** un id inexistente,
  **cuando** se hace `GET /api/tasks/{id}`,
  **entonces** se retorna `404`.
- **Dado** un id con formato no-UUID,
  **cuando** se hace `GET /api/tasks/{id}`,
  **entonces** se retorna `400`.

### UC-04 — Actualizar tarea

- **Dado** una tarea existente y un `title`/`done` válidos,
  **cuando** se hace `PUT /api/tasks/{id}`,
  **entonces** se retorna `200` con la tarea actualizada.
- **Dado** un id inexistente, **cuando** se hace `PUT`, **entonces** se retorna `404`.
- **Dado** datos inválidos, **cuando** se hace `PUT`, **entonces** se retorna `400`.

### UC-05 — Eliminar tarea

- **Dado** una tarea existente,
  **cuando** se hace `DELETE /api/tasks/{id}`,
  **entonces** se retorna `204` sin cuerpo.
- **Dado** un id inexistente, **cuando** se hace `DELETE`, **entonces** se retorna `404`.

## 6. Reglas de negocio

- El `title` es obligatorio y no puede estar en blanco; se almacena con `trim` aplicado.
- El `id` y `createdAt` son generados por la base de datos y nunca se aceptan del cliente.
- La actualización (`PUT`) reemplaza `title` y `done`; no modifica `id` ni `createdAt`.
- Una tarea recién creada siempre inicia con `done = false`.

## 7. Errores esperados

| Código | Situación                                          |
|--------|----------------------------------------------------|
| `400`  | Payload inválido o `id` con formato no-UUID.       |
| `404`  | Recurso no encontrado.                             |
| `500`  | Error interno no controlado.                       |

El formato de error sigue RFC 7807 (`application/problem+json`), ver `api-contract.md`.
