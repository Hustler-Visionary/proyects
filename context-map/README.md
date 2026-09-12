# Context Map — TST Autonomous / REPO_OS

_Mapa de responsabilidades del monorepo: qué es cada carpeta, quién depende de quién, y por qué está organizado así. Pensado para leerse sin conocer el código de antemano — sea o no seas quien lo programa._

> Si solo vas a leer una sección, que sea la 2 ("Los 5 niveles"). Todo lo demás es detalle de esa idea.

---

## 1. La idea en una frase

Este repositorio junta **una plataforma completa** (backend, frontend, motores de orquestación, reglas de negocio, y la infraestructura que los corre) en un solo lugar, pero cada pieza vive en un nivel claro según **qué responsabilidad tiene**, no según cuándo se escribió o quién la tocó.

Una analogía: pensá en un edificio de oficinas.

| Nivel del edificio | Nivel de este repo | Qué hace |
|---|---|---|
| 🏗️ Cuarto de máquinas (sótano) | `infra/` | Enciende las luces, el agua, la electricidad. No sabe qué negocio funciona arriba. |
| 🏢 Recepción / mostradores al público | `apps/` | Por acá entra la gente (usuarios, otros sistemas). Cada mostrador tiene su propia puerta (puerto de red). |
| 🧭 Oficina de coordinación | `execution/` | No atiende al público directamente; decide cómo se reparte y ejecuta el trabajo entre departamentos. |
| 🗂️ Departamentos (legal, finanzas, soporte...) | `services/` | Cada uno sabe las reglas de su propia área. No le importa cómo llegó el pedido. |
| 🧰 Caja de herramientas compartida | `packages/` | Lo mínimo que todos los departamentos usan (una regla, un formulario común). No depende de nadie. |

Esa tabla **es** la arquitectura. Todo lo de abajo es la versión técnica de la misma idea.

---

## 2. Los 5 niveles (versión técnica)

```
                    ┌───────────────────────────────────────────────┐
                    │  NIVEL 0 · ORQUESTACIÓN — infra/               │
                    │  Construye, despliega y conecta. Cero reglas   │
                    │  de negocio. Hoy: Docker Compose. Mañana:      │
                    │  también manifiestos de Kubernetes, acá mismo. │
                    └───────────────────────┬─────────────────────────┘
                                            │ construye y corre
                                            ▼
        ┌─────────────────────────────────────────────────────────────────┐
        │  NIVEL 1 · APLICACIONES DESPLEGABLES — apps/                    │
        │  Procesos independientes, cada uno con su propio puerto de red. │
        │  apps/api  (NestJS · GraphQL · puerto 4000)                     │
        │  apps/web  (Next.js · REPO_OS · puerto 3000)                    │
        └───────────────┬───────────────────────────────┬─────────────────┘
                        │                               │ usa
                        │ (sin consumir services aún)   ▼
                        │              ┌─────────────────────────────────────┐
                        │              │  NIVEL 2 · ORQUESTACIÓN DE EJECUCIÓN  │
                        │              │  — execution/                        │
                        │              │  Coordina/simula cómo fluye el       │
                        │              │  trabajo. Nadie de afuera le habla   │
                        │              │  directo.                            │
                        │              │  graph-orchestration · runtime-fabric│
                        │              └───────────────────┬───────────────────┘
                        │                                  │ usa
                        ▼                                  ▼
        ┌───────────────────────────────────────────────────────────────────┐
        │  NIVEL 3 · SERVICIOS DE DOMINIO — services/                        │
        │  Cada uno es un "departamento": reglas de negocio de un área,      │
        │  sin saber cómo llegó el pedido ni quién lo va a mostrar.          │
        │  governance-compliance · repo-knowledge · agent-cognition ·        │
        │  product-narrative · macro-apps · platform-integration            │
        └───────────────────────────────────┬─────────────────────────────────┘
                                            │ usa
                                            ▼
                        ┌─────────────────────────────────────┐
                        │  NIVEL 4 · KERNEL COMPARTIDO           │
                        │  — packages/                           │
                        │  Lo único de lo que todo puede depender│
                        │  packages/kernel (trace bus)            │
                        │  No depende de nada más del repo.      │
                        └─────────────────────────────────────┘
```

**La única regla que importa:** las flechas de dependencia solo apuntan hacia abajo. `apps/` puede usar `execution/` y `services/`; `execution/` puede usar `services/`; todos pueden usar `packages/`. Nunca al revés — `packages/kernel` no sabe que `apps/api` existe, y no debería. Esto es lo que en arquitectura se llama la **regla de dependencia** (hexagonal / clean architecture): lo genérico no depende de lo específico.

`infra/` está **fuera** de esa cadena por completo: no importa código de ningún nivel, solo sabe construir imágenes y exponer puertos.

---

## 3. El árbol real (solo lo que importa para este mapa)

```
.
├── infra/                          # NIVEL 0 — Orquestación
│   └── docker/                     #   Postgres vía Compose hoy; +k8s/ mañana
│       ├── compose.yaml
│       ├── data/postgres/          #   (gitignored — el nombre = futuro nombre de PVC)
│       └── README.md               #   documenta la disciplina a repetir por servicio
│
├── apps/                           # NIVEL 1 — Aplicaciones desplegables
│   ├── api/                        #   NestJS: GraphQL + JWT + RBAC + CASL + Zod + Drizzle/Postgres
│   └── web/                        #   Next.js: REPO_OS (grafo real + chat sobre NATS)
│
├── execution/                      # NIVEL 2 — Orquestación de ejecución
│   ├── graph-orchestration/        #   Enruta trabajo (Markov routing) + overlays HUD
│   └── runtime-fabric/             #   Motor de simulación runtime (event-sourcing, etc.)
│
├── services/                       # NIVEL 3 — Servicios de dominio (departamentos)
│   ├── governance-compliance/      #   Policy runtime, constitutional runtime, security-compliance
│   ├── repo-knowledge/             #   Lectura real de repo + knowledge graph
│   ├── agent-cognition/            #   Comandos de agente + razonamiento
│   ├── product-narrative/          #   Narrativa/demo/replay (alcance amplio — ver decisions-log)
│   ├── macro-apps/                 #   Estado UI (store/selectors) del canvas
│   └── platform-integration/       #   MCP fabric, integración externa (recién reubicado — ver decisions-log)
│
└── packages/                       # NIVEL 4 — Kernel compartido
    └── kernel/                     #   Trace bus + trace-stream
```

---

## 4. Tabla completa (para referencia rápida)

| # | Ruta | Nombre del paquete | Nivel | Responsabilidad en una línea | Depende de (dentro del repo) | Tamaño (líneas) |
|---|---|---|---|---|---|---|
| 1 | `infra/docker` | _(no es paquete de código)_ | 0 · Orquestación | Postgres vía Compose; futuro home de manifiestos k8s | — | — |
| 2 | `apps/api` | `@tst-autonomous/api` | 1 · Apps | Backend NestJS: GraphQL, JWT, RBAC, CASL, Zod, Postgres/Drizzle | _(ninguno del monorepo todavía — ver §6)_ | 1084 |
| 3 | `apps/web` | `web` | 1 · Apps | REPO_OS: grafo real del repo + chat sobre NATS JetStream | governance-compliance, graph-orchestration, macro-apps, product-narrative, repo-knowledge | 2812 |
| 4 | `execution/graph-orchestration` | `@tst-autonomous/graph-orchestration` | 2 · Execution | Enruta trabajo (Markov routing), overlays HUD/loop-prevention | repo-knowledge, governance-compliance, kernel | 135 |
| 5 | `execution/runtime-fabric` | `@tst-autonomous/runtime-fabric` | 2 · Execution | Motor de simulación runtime: event-sourcing, distributed-fabric, unified-runtime-kernel | _(ninguno)_ | 782 |
| 6 | `services/governance-compliance` | `@tst-autonomous/governance-compliance` | 3 · Services | Gobernanza/compliance: policy-runtime, constitutional-runtime, security-compliance | kernel | 1245 |
| 7 | `services/repo-knowledge` | `@tst-autonomous/repo-knowledge` | 3 · Services | Lectura real de repo, knowledge-graph, memoria | kernel, governance-compliance | 500 |
| 8 | `services/agent-cognition` | `@tst-autonomous/agent-cognition` | 3 · Services | Comandos de agente, razonamiento cognitivo | kernel, governance-compliance, repo-knowledge, macro-apps | 378 |
| 9 | `services/product-narrative` | `@tst-autonomous/product-narrative` | 3 · Services | Narrativa de producto/demo/replay | runtime-fabric | 813 |
| 10 | `services/macro-apps` | `@tst-autonomous/macro-apps` | 3 · Services | Estado UI (store/selectors) para el shell del canvas | kernel, governance-compliance, repo-knowledge | 227 |
| 11 | `services/platform-integration` | `@tst-autonomous/platform-integration` | 3 · Services | MCP fabric, integración externa, stack-definition | _(ninguno)_ | 739 |
| 12 | `packages/kernel` | `@tst-autonomous/kernel` | 4 · Kernel | Trace bus + trace-stream | _(ninguno)_ | 502 |

---

## 5. ¿Y "governance-compliance"? ¿Y Kubernetes?

Dos cosas que se piden seguido y conviene aclarar dónde están **hoy**, para no asumir de más:

- **`services/governance-compliance` ya existe** y contiene policy-runtime, constitutional-runtime y security-compliance reales — pero **no está conectado a `apps/api`**. Hoy `apps/api` solo usa su propia capa CASL/RBAC (`apps/api/src/casl/`), que es un sistema aparte y más simple. Conectar `apps/api` a `governance-compliance` de verdad (un guard/interceptor que consulte `policy-runtime` en cada mutation) es un paso pendiente, no algo que este mapa ya asuma hecho.
- **Kubernetes todavía no existe en el repo.** `infra/docker/` es Docker Compose puro. La disciplina de `infra/docker/README.md` (una carpeta de datos por servicio, nombrada igual que el servicio) está pensada explícitamente para que el día que se agregue `infra/k8s/`, cada `data/<servicio>/` se convierta en una `PersistentVolumeClaim` del mismo nombre — pero ese día todavía no llegó.

## 6. Un hueco real que vale la pena mirar

`apps/api` no depende hoy de **ningún** paquete del monorepo — ni de `packages/kernel`, ni de `services/governance-compliance`. Es una app NestJS aislada con su propia lógica (auth, CASL, Postgres). Esto no es un error, pero es la razón por la que "aplicar governance-compliance" a `apps/api` requiere trabajo real de integración, no solo un `import`.

---

## 7. Documentos relacionados en esta carpeta

- **`decisions-log.md`** — qué se movió/renombró en esta pasada, por qué, y con qué evidencia (no son decisiones "porque sí").
- **`glossary.md`** — los términos técnicos de este documento, explicados sin jerga, para quien no programa.
