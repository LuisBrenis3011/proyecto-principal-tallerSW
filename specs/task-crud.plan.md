# Plan de implementación — CRUD de Tareas (To-Do)

Resultado final del plan. No quedan pendientes: todas las decisiones están cerradas.

## 1. Decisiones cerradas

| Decisión            | Resolución                                                      |
|---------------------|-----------------------------------------------------------------|
| Funcionalidad       | CRUD de Tareas (To-Do) end-to-end                               |
| Base de datos       | PostgreSQL 17 local (`localhost:5432`), base `taller_sw`        |
| Acceso a datos      | Spring Data R2DBC (reactivo) + `r2dbc-postgresql`               |
| Migraciones         | Flyway (`flyway-core` + `flyway-database-postgresql`)           |
| Arquitectura        | Hexagonal (ports & adapters)                                    |
| Frontend            | React 19 + TypeScript + Vite (fetch + proxy `/api`)             |
| Tests               | JUnit 5 + Mockito + `@WebFluxTest`                              |

## 2. Stack

- **Backend**: Java 21, Spring Boot 4.1.1 (WebFlux), reactivo de punta a punta.
- **Datos**: `spring-boot-starter-data-r2dbc`, `io.r2dbc:r2dbc-postgresql`.
- **Migraciones**: `org.flywaydb:flyway-core`, `org.flywaydb:flyway-database-postgresql`.
- **JDBC (solo para Flyway)**: `org.postgresql:postgresql`.
- **Frontend**: React 19 + TS + Vite (ya instalados).

## 3. Arquitectura backend (hexagonal)

Paquete base: `com.grupo7.seguridad_spring`, módulo de funcionalidad `task`.

```
task/
├── domain/
│   ├── Task.java                    # entidad de dominio
│   ├── TaskId.java                  # value object (UUID)
│   ├── TaskTitle.java               # value object (validación 1..255, trim)
│   └── TaskNotFoundException.java
├── application/
│   ├── port/in/
│   │   ├── CreateTaskUseCase.java
│   │   ├── GetTaskByIdUseCase.java
│   │   ├── ListTasksUseCase.java
│   │   ├── UpdateTaskUseCase.java
│   │   └── DeleteTaskUseCase.java
│   ├── port/out/
│   │   └── TaskRepository.java      # puerto de salida (persistencia)
│   └── service/
│       └── TaskService.java         # implementa puertos de entrada
├── infrastructure/
│   ├── persistence/
│   │   ├── TaskEntity.java          # @Table("tasks") R2DBC
│   │   └── R2dbcTaskRepository.java # adaptador Spring Data R2DBC
│   └── config/
│       └── CorsConfig.java
└── presentation/
    ├── controller/
    │   ├── TaskController.java
    │   └── GlobalExceptionHandler.java
    ├── dto/
    │   ├── TaskRequest.java
    │   ├── TaskResponse.java
    │   └── ErrorResponse.java
    └── mapper/
        └── TaskMapper.java
```

- **Dominio puro**: sin dependencias de Spring; los value objects encapsulan validación.
- **Puertos**: casos de uso (`port/in`) y persistencia (`port/out`) como interfaces.
- **Adaptadores**: `R2dbcTaskRepository` implementa `TaskRepository`; `TaskController`
  usa los puertos de entrada vía `TaskService`.
- **Conexión**: `application.properties` (`spring.r2dbc.*` + `spring.flyway.*`).

## 4. Esquema de BD (Flyway)

`backend/src/main/resources/db/migration/V1__create_tasks.sql`:

```sql
CREATE TABLE IF NOT EXISTS tasks (
    id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title      VARCHAR(255) NOT NULL,
    done       BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Se ejecuta automáticamente al arrancar el backend.

## 5. Frontend

```
frontend/src/
├── api/tasks.ts          # cliente fetch
├── components/
│   ├── TaskList.tsx
│   ├── TaskItem.tsx
│   └── TaskForm.tsx
├── hooks/useTasks.ts     # estado + efectos
├── types/Task.ts         # interfaces (sin any)
└── App.tsx               # reemplaza el template de Vite
```

- `vite.config.ts`: proxy `/api → http://localhost:8080`.

## 6. Tests

- `TaskServiceTest` (Mockito) — casos de uso.
- `TaskControllerTest` (`@WebFluxTest`) — contrato HTTP.
- `TaskTitleTest` — validación de dominio.

## 7. Orden de ejecución

1. Dependencias en `backend/build.gradle`.
2. Config BD en `application.properties`.
3. Migración Flyway `V1__create_tasks.sql` (y ejecutar el esquema).
4. Capa `domain`.
5. Puertos + `TaskService`.
6. Adaptador de persistencia.
7. Capa `presentation` (controller, DTO, mapper, excepciones, CORS).
8. Tests backend.
9. Frontend (types → api → hooks → components → App → proxy).
10. Verificación: `.\gradlew.bat test`, `npm run lint`, `npm run build`, y `curl`
    contra el backend con Postgres corriendo.
11. Proponer los cambios como diffs para revisión.

## 8. Documentos asociados

- `specs/task-crud.md` — especificación funcional.
- `specs/api-contract.md` — contrato de API (frontend ↔ backend ↔ BD).

## 9. Fuera de alcance (esta iteración)

Autenticación, usuarios, paginación, filtros, fechas de vencimiento y etiquetas.
