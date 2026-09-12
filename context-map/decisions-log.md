# Decisions Log

_Cada entrada dice qué cambió, por qué, y qué evidencia lo respalda — para que una reclasificación futura no tenga que adivinar el razonamiento de esta._

---

## 2026-09-12 — `infra/platform-integration` → `services/platform-integration`

**Qué cambió:** el paquete completo (`@tst-autonomous/platform-integration`) se movió de `infra/` a `services/`, con `git mv` (historial preservado). Se actualizaron las 4 referencias reales al path viejo: `tsconfig.json` raíz (project reference), `Taskfile.yml` y `Dockerfile` propios del paquete, y `scripts/scaffold-package.mjs` (el registro que usa el scaffolding para futuros paquetes).

**Por qué:** el commit original que creó esta carpeta (`57a6941`, "Split src/domain into DDD bounded-context packages") la describió como "platform/integration adapters ... no business logic" — pero el contenido real no respalda esa descripción:

- Su estructura es idéntica a la de cualquier paquete en `services/`: pares `engine.ts` + `types.ts` por submódulo (`mcp-fabric`, `external-integration`, `reality-bridge`, `saas-control-plane`, `stack-definition`).
- No contiene absolutamente nada de despliegue: sin Kubernetes, sin llamadas de red reales, sin nada que orqueste contenedores. Su propio `Dockerfile` es genérico (el mismo patrón turbo-prune que usa cada paquete del monorepo para poder buildearse solo) — no es "infraestructura" en el sentido de este repo, solo el empaquetado estándar de cualquier paquete.
- Su contenido modela **reglas de negocio** sobre gobernanza de integraciones: cómo se registra una herramienta MCP, qué permisos/sandbox aplican, qué stack es canónico — exactamente el tipo de contenido que vive en `services/governance-compliance` o `services/repo-knowledge`.

En otras palabras: el nombre de la carpeta (`infra/`) prometía "cómo se despliega esto", pero el contenido real es "qué reglas de negocio rigen la integración con plataformas externas". Con la definición de `infra/` que se estableció para este repo — *orquestación y despliegue de apps y services, no lógica de negocio* — la ubicación original era una inconsistencia real, no una cuestión de gusto.

**Evidencia verificada antes de mover (no fue una suposición):**
- `grep` de imports relativos cruzando límites de paquete (`../../../`) en todo `apps/*/src`, `services/*/src`, `execution/*/src`: **cero resultados** — confirma que mover el paquete es puramente mecánico (nadie hace un import "oculto" que dependa de la ruta física).
- Solo 4 archivos en todo el repo referenciaban el path `infra/platform-integration` como string literal (los ya listados arriba) — no había sorpresas escondidas.
- Post-movimiento: `pnpm install` (regenera el lockfile con el nuevo path) + `pnpm exec turbo run build test` sobre las 21 tareas del monorepo → **21/21 exitosas**, incluidos los 13 tests propios de `platform-integration` corriendo desde su nueva ubicación.

**Qué NO se tocó, y por qué:** `execution/` se evaluó y se decidió **mantener separado** de `services/`, aunque también es "lógica de negocio" en sentido amplio. La diferencia real: `execution/graph-orchestration` y `execution/runtime-fabric` no modelan reglas de un dominio de negocio específico — coordinan/simulan *cómo* se ejecuta el trabajo entre servicios (capa de orquestación/aplicación en términos de Clean Architecture), mientras que cada paquete en `services/` modela reglas de *un* área de negocio (gobernanza, conocimiento del repo, narrativa). Es una distinción de responsabilidad real, no solo organizativa — se documenta acá para que quede explícita, no implícita.

---

## Notas abiertas (no son decisiones tomadas, son observaciones para una futura pasada)

Estas dos no se tocaron en esta pasada porque no había una inconsistencia tan clara como la de `platform-integration` — mover código sin una razón concreta sería cambio innecesario, no limpieza:

- **`services/product-narrative`** agrupa bajo un solo paquete cosas bastante distintas: narrativa/storytelling, escenarios de demo, replay, **y también** `product-workflows`/`product-consolidation` (que suenan más a "flujo de producto" que a "narrativa"). Es el paquete de alcance más amplio del repo (813 líneas en un solo bounded context). Candidato razonable a partirse en dos si crece más.
- **`services/macro-apps`** contiene estado de UI (store de Zustand, selectors, controller de canvas) más que reglas de negocio puras — se parece más a una capa de presentación/estado que a un "servicio de dominio" en el sentido estricto de los otros. Funciona porque `apps/web` lo consume directo, pero el nombre no comunica bien qué hace.

## Cómo agregar una entrada nueva

Cada entrada de este log responde tres preguntas, en este orden: **qué** cambió (con paths exactos), **por qué** (evidencia, no intuición), y **qué se verificó** después del cambio (comandos reales corridos, no "debería funcionar"). Si no podés responder las tres, todavía no es momento de mover el archivo.
