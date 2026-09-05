# Taller de Software — Overview

## 1. Descripción del proyecto

Monorepo del taller de software del **Grupo 7**. Consiste en una aplicación full-stack
para la gestión de tareas ("To-Do App") como ejemplo libre de referencia. El objetivo es
practicar los estándares de desarrollo: separación de responsabilidades, API REST
reactiva, tipado estricto y documentación de especificaciones.

## 2. Estructura del repositorio

```
taller_sw/
├── backend/     # API REST reactiva (Spring Boot + Java 21)
├── frontend/    # SPA (React + TypeScript + Vite)
├── specs/       # Documentación y especificaciones del proyecto
└── AGENTS.md    # Guía de trabajo para agentes de IA
```

## 3. Stack tecnológico

| Capa      | Tecnología                         | Versión        |
|-----------|------------------------------------|----------------|
| Backend   | Java                               | 21 (toolchain) |
| Backend   | Spring Boot (WebFlux)              | 4.1.1          |
| Backend   | Gradle (wrapper)                   | 8.x            |
| Frontend  | React                              | 19.2.x         |
| Frontend  | TypeScript                         | 6.0.x          |
| Frontend  | Vite                               | 8.2.x          |
| Linting   | ESLint (`typescript-eslint`)       | 10.x           |

## 4. Dominio de ejemplo: Tareas (To-Do)

### Entidad principal: `Task`

| Campo      | Tipo       | Descripción                                   |
|------------|------------|-----------------------------------------------|
| `id`       | `UUID`     | Identificador único                            |
| `title`    | `String`   | Título de la tarea (obligatorio)              |
| `done`     | `Boolean`  | Estado de completado (por defecto `false`)    |
| `createdAt`| `Instant`  | Fecha de creación                              |

### Endpoints del backend

| Método | Ruta           | Descripción                              |
|--------|----------------|------------------------------------------|
| GET    | `/api/tasks`   | Listar todas las tareas                  |
| GET    | `/api/tasks/{id}` | Obtener una tarea por ID              |
| POST   | `/api/tasks`   | Crear una tarea                          |
| PUT    | `/api/tasks/{id}` | Actualizar una tarea                  |
| DELETE | `/api/tasks/{id}` | Eliminar una tarea                   |

## 5. Especificaciones transversales

- **API reactiva**: el backend usa Spring WebFlux, por lo que los controladores deben
  retornar `Mono<T>` / `Flux<T>`.
- **Convención de respuestas**: JSON con camelCase; errores HTTP estándar (`404`, `400`, `500`).
- **Frontend tipado**: componentes funcionales con hooks; sin `any` en código nuevo.
- **Pruebas**: backend con JUnit 5 + WebFlux Test; frontend con tests por definir.
