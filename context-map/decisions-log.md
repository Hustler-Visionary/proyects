# Decisions Log

**Propósito:** Registro formal de decisiones de reclasificación arquitectónica. Cada entrada documenta contexto, decisión, evidencia y verificación, siguiendo el formato de un Architecture Decision Record (ADR), de modo que una reclasificación futura disponga del razonamiento completo y no deba reconstruirlo.

---

## ADR-001 — Reclasificación de `infra/platform-integration` a `services/platform-integration`

**Fecha:** 2026-09-12
**Estado:** Implementada y verificada.

### Contexto

El commit `57a6941` ("Split src/domain into DDD bounded-context packages") introdujo `infra/platform-integration` con la justificación textual: *"platform/integration adapters (MCP fabric, external integration, SaaS control plane, stack definition, reality bridge) — no business logic."*

Al aplicar el criterio de inclusión del Nivel 0 definido en `README.md` §2 ("el directorio define cómo se construye, despliega, conecta en red o persiste un artefacto ya compilado"), el contenido de `infra/platform-integration` no satisface ese criterio.

### Evidencia

1. **Estructura interna.** El paquete está compuesto por pares `engine.ts` + `types.ts` por submódulo (`mcp-fabric`, `external-integration`, `reality-bridge`, `saas-control-plane`, `stack-definition`) — estructura idéntica, archivo por archivo, a la de cualquier paquete en `services/`.
2. **Ausencia de contenido de despliegue.** El paquete no contiene manifiestos de orquestación de contenedores, definiciones de red, ni llamadas a APIs de infraestructura real. Su `Dockerfile` es el patrón genérico de empaquetado (`turbo prune`) que comparte cada paquete del monorepo — no constituye, por sí mismo, una responsabilidad de infraestructura.
3. **Naturaleza del contenido.** Los módulos modelan reglas de negocio sobre gobernanza de integraciones (registro de herramientas MCP, políticas de sandbox y permisos, definición de stack canónico) — el mismo tipo de contenido que reside en `services/governance-compliance`.

Conclusión: la ubicación original respondía a la intención declarada en el commit original, no al contenido real del paquete.

### Decisión

Mover `infra/platform-integration` a `services/platform-integration` mediante `git mv`, preservando el historial de commits.

### Verificación previa a la ejecución

- Búsqueda de imports relativos que crucen límites de paquete (`../../../`) en la totalidad de `apps/*/src`, `services/*/src`, `execution/*/src`: **cero coincidencias**. Esto descarta cualquier acoplamiento oculto que dependiera de la ruta física del paquete.
- Identificación exhaustiva de referencias al literal `infra/platform-integration` en el repositorio: 4 archivos (`tsconfig.json` raíz, `Taskfile.yml` y `Dockerfile` propios del paquete, `scripts/scaffold-package.mjs`). No se identificaron referencias adicionales no anticipadas.

### Cambios ejecutados

| Archivo | Cambio |
|---|---|
| `tsconfig.json` (raíz) | Referencia de proyecto actualizada de `infra/platform-integration` a `services/platform-integration` |
| `services/platform-integration/Taskfile.yml` | Argumento `PACKAGE_DIR` actualizado |
| `services/platform-integration/Dockerfile` | Rutas `COPY --from=builder` actualizadas |
| `scripts/scaffold-package.mjs` | Entrada del registro `REGISTRY` actualizada |

### Verificación posterior a la ejecución

- `pnpm install`: relockeo del lockfile con la nueva ruta del paquete de workspace, sin errores.
- `pnpm exec turbo run build test`: **21 de 21 tareas exitosas**, incluidas las 13 pruebas propias de `platform-integration` ejecutándose desde su nueva ubicación.

### Consecuencias

- `infra/` queda compuesto exclusivamente por contenido que satisface su criterio de inclusión (Nivel 0).
- `scripts/scaffold-package.mjs` refleja la ubicación correcta para cualquier paquete futuro que dependa de `platform-integration`.
- Ningún consumidor externo requirió cambios: ningún paquete del monorepo declara `platform-integration` como dependencia al momento de esta decisión.

---

## Observaciones registradas sin decisión asociada

Las siguientes observaciones se documentan por transparencia. No constituyen una decisión de reclasificación: la evidencia disponible no alcanza el mismo nivel de certeza que la del ADR-001, y una reorganización sin evidencia equivalente introduciría riesgo sin beneficio verificable.

| Paquete | Observación | Motivo para no actuar en esta pasada |
|---|---|---|
| `services/product-narrative` | Agrupa narrativa/demostraciones/reproducción de eventos junto con `product-workflows` y `product-consolidation`, que responden a una responsabilidad distinta ("flujo de producto" antes que "narrativa"). Es el paquete de mayor volumen del repositorio (813 líneas) dentro de un único bounded context. | No se identificó una inconsistencia entre nombre y contenido con el mismo grado de certeza que en `platform-integration`; una división requeriría análisis adicional de los consumidores de cada submódulo. |
| `services/macro-apps` | Contiene principalmente estado de interfaz (store, selectors, controlador de lienzo) — más cercano a una capa de presentación que a reglas de negocio de un dominio. | El paquete satisface el criterio de inclusión de Nivel 3 en sentido amplio (es consumido como una unidad de dominio por `apps/web`); renombrar o reubicar requeriría antes definir un criterio explícito para separar "estado de interfaz" de "servicio de dominio", que no existe todavía en este documento. |

### Criterio para agregar una entrada nueva

Toda entrada nueva debe incluir, como mínimo: Contexto (qué situación motiva la revisión), Evidencia (verificable, no una apreciación), Decisión (acción concreta), Verificación previa y posterior (comandos ejecutados y resultado obtenido), y Consecuencias. Una entrada sin evidencia verificable se registra en la tabla de "Observaciones registradas sin decisión asociada", no como una decisión ejecutada.
