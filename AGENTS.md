# AGENTS.md

Guía de trabajo para agentes de IA que contribuyen a este repositorio.

## Contexto del proyecto

Monorepo full-stack del **Grupo 7** (taller de software):

- `backend/` — API REST reactiva con Spring Boot (WebFlux) y Java 21.
- `frontend/` — SPA con React 19 + TypeScript + Vite.
- `specs/` — especificaciones y documentación. Consulta `specs/overview.md` antes de empezar.

## Reglas generales

1. No asumir librerías o dependencias que no estén ya declaradas en `build.gradle` o `package.json`.
2. No agregar comentarios innecesarios al código (a menos que se soliciten explícitamente).
3. Seguir las convenciones existentes de cada carpeta (naming, estilo, estructura).
4. No escribir secretos, claves ni credenciales en el código ni en commits.

## Backend

- Java 21. El paquete base es `com.grupo7.seguridad_spring`.
- Controladores y servicios deben usar reactividad (`Mono`/`Flux` de Project Reactor).
- Build y test con el wrapper de Gradle:
  - Windows: `.\gradlew.bat test`
  - Unix: `./gradlew test`
- No cambies la versión de Spring Boot ni el group id sin pedirlo.

## Frontend

- TypeScript estricto; evita `any`.
- Componentes funcionales con hooks de React 19.
- Comandos (desde `frontend/`):
  - `npm run dev` — servidor de desarrollo
  - `npm run build` — compilación (`tsc -b && vite build`)
  - `npm run lint` — ESLint
- Usa `npm` (existe `package-lock.json` y `yarn.lock`; evita mezclar gestores).

## Verificación obligatoria

Antes de dar por terminada una tarea de código:

1. Backend: ejecutar `.\gradlew.bat test`.
2. Frontend: ejecutar `npm run lint` y `npm run build`.
3. Reporta cualquier error de lint o typecheck que no puedas resolver.
