# Arquitectura del Proyecto

Este documento describe la arquitectura de referencia del monorepo, usando como ejemplo
libre una aplicación de gestión de tareas (To-Do App). Ilustra la organización de
carpetas propuesta para el `backend` (Spring Boot WebFlux) y el `frontend` (React + TS).

## 1. Visión general

```
                         ┌───────────────┐
                         │    Navegador   │
                         └───────┬───────┘
                                 │ HTTP / JSON
                         ┌───────▼───────┐
                         │   Frontend     │  React 19 + TypeScript + Vite
                         │   (localhost)  │
                         └───────┬───────┘
                                 │ REST (fetch / axios)
                         ┌───────▼───────┐
                         │    Backend     │  Spring Boot WebFlux (Java 21)
                         │  /api/tasks    │
                         └───────┬───────┘
                                 │
                         (persistencia / repositorio en memoria o BD)
```

- **Frontend** es responsable de la UI, el estado de la sesión y el consumo de la API.
- **Backend** expone una API REST reactiva y concentra la lógica de negocio.
- La comunicación es siempre vía **HTTP con JSON** (camelCase).

## 2. Arquitectura del Backend

El backend sigue una arquitectura **en capas** dentro del paquete base
`com.grupo7.seguridad_spring`. Cada capa tiene una responsabilidad única y se comunica
mediante `Mono<T>` / `Flux<T>` (Programación Reactiva).

### Estructura de carpetas propuesta

```
backend/src/main/java/com/grupo7/seguridad_spring/
├── SeguridadSpringApplication.java     # Punto de entrada (@SpringBootApplication)
├── controller/
│   └── TaskController.java             # Capa de presentación: rutas HTTP y status codes
├── service/
│   └── TaskService.java                # Capa de negocio: validaciones y casos de uso
├── repository/
│   └── TaskRepository.java             # Capa de datos: acceso a la persistencia
├── model/
│   ├── Task.java                       # Entidad de dominio (POJO / record)
│   └── dto/
│       └── TaskRequest.java            # DTO de entrada (payload de POST/PUT)
└── config/
    └── CorsConfig.java                 # Configuración CORS para el frontend

backend/src/main/resources/
└── application.properties              # Configuración (puerto, CORS, etc.)
```

### Capas y responsabilidades

| Capa         | Paquete             | Responsabilidad                                              |
|--------------|---------------------|--------------------------------------------------------------|
| Presentación | `controller`        | Mapear rutas, parsear/validar payload, retornar `Mono`/`Flux` |
| Negocio      | `service`           | Reglas de negocio, validaciones, coordinación de repositorio |
| Datos        | `repository`        | Persistencia y consultas (interfaz reactiva)                 |
| Dominio      | `model` (+ `dto`)   | Entidades y objetos de transferencia sin lógica             |
| Infra        | `config`            | Configuración transversal (CORS, beans, etc.)               |

### Flujo de una petición

```
HTTP GET /api/tasks
   │
   ▼
TaskController.getTasks()        → Flux<Task>
   │
   ▼
TaskService.findAll()            → Flux<Task>
   │
   ▼
TaskRepository.findAll()         → Flux<Task>   (en memoria → ConcurrencyMap)
```

### Convenciones

- Controladores anotados con `@RestController` y `@RequestMapping("/api/tasks")`.
- Respuestas reactivas: `Mono<T>` (un recurso) o `Flux<T>` (colección).
- Errores estándar HTTP: `404 NOT_FOUND`, `400 BAD_REQUEST`, `500 INTERNAL_SERVER_ERROR`.
- La configuración sensible (puerto, orígenes permitidos) vive en `application.properties`.

## 3. Arquitectura del Frontend

El frontend es una SPA construida con React 19 y TypeScript, organizada por
**responsabilidad de archivo**.

### Estructura de carpetas propuesta

```
frontend/src/
├── main.tsx                   # Bootstrap de la app (ReactDOM.createRoot)
├── App.tsx                    # Componente raíz / ruteo de alto nivel
├── api/
│   └── tasks.ts               # Cliente HTTP de la API (tipado con interfaces)
├── components/
│   ├── TaskList.tsx           # Lista de tareas
│   └── TaskItem.tsx           # Item individual de tarea
├── hooks/
│   └── useTasks.ts            # Hook de estado y efectos (llamadas a la API)
├── types/
│   └── Task.ts                # Tipos e interfaces compartidos (sin any)
├── styles/                    # (o App.css/index.css según prefieras)
│   └── App.css
└── assets/
    └── (imágenes, svg, etc.)
```

### Responsabilidades

| Carpeta        | Responsabilidad                                        |
|----------------|--------------------------------------------------------|
| `api/`         | Funciones de acceso HTTP y mapeo de respuestas JSON    |
| `components/`  | Componentes funcionales de UI (presentación)           |
| `hooks/`       | Lógica de estado y efectos reutilizable (`useTasks`)   |
| `types/`       | Tipos e interfaces del dominio compartidos             |
| `assets/`      | Recursos estáticos (imágenes, estilos)                 |

### Flujo de datos en el frontend

```
Componente (TaskList)
      │  usa
      ▼
Hook (useTasks)  ── llama ──►  api/tasks.ts  ── fetch ──►  Backend /api/tasks
      │                           │
      ▼                           ▼
Estado React (useState)      tipos/types/Task.ts (validación de forma)
      │
      ▼
Render (JSX)
```

### Convenciones

- Componentes funcionales con hooks (`useState`, `useEffect`).
- TypeScript estricto; **prohibido** usar `any`.
- La lógica de red queda aislada en `api/`, nunca mezclada con los componentes.
- Tipos de dominio centralizados en `types/` para evitar duplicación.

## 4. Comunicación Frontend ↔ Backend

- Las URLs base del backend se configuran por entorno (p. ej. `VITE_API_URL`).
- El frontend consume los endpoints definidos en `specs/overview.md` (sección 4).
- El backend debe habilitar CORS para el origen del frontend (ver `config/CorsConfig`).
