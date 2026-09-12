# Context Map — TST Autonomous / REPO_OS

**Documento:** Mapa de responsabilidades arquitectónicas del monorepo.
**Alcance:** Todos los directorios de primer nivel del repositorio (`infra/`, `apps/`, `execution/`, `services/`, `packages/`).
**Propósito:** Establecer, de forma verificable y sin ambigüedad, a qué nivel arquitectónico pertenece cada unidad de código, qué responsabilidad tiene ese nivel, y qué regla de dependencia rige entre niveles.
**Audiencia:** Ingeniería (referencia técnica autoritativa) y stakeholders no técnicos (sección 1, resumen ejecutivo).

---

## 1. Resumen ejecutivo

Este repositorio aloja una plataforma completa — interfaz de usuario, backend, motores de orquestación, reglas de negocio e infraestructura de despliegue — organizada en **cinco niveles de responsabilidad**. Cada nivel tiene un criterio de pertenencia único y verificable: un directorio pertenece a un nivel si y solo si cumple ese criterio, no por convención ni por dónde "parece" que debería estar.

| Nivel | Directorio | Responsabilidad | Puede ser modificado sin afectar | No puede depender de |
|---|---|---|---|---|
| 0 — Orquestación | `infra/` | Construcción, despliegue y conectividad de red de los procesos ejecutables | Ninguna lógica de negocio | (no aplica — no importa código fuente de ningún nivel) |
| 1 — Aplicaciones | `apps/` | Procesos ejecutables independientes, cada uno expuesto en un puerto de red propio | Otras aplicaciones | Nada — es el nivel más externo |
| 2 — Orquestación de ejecución | `execution/` | Coordinación de cómo se secuencia y enruta el trabajo entre servicios de dominio | Aplicaciones que lo consumen | `apps/` |
| 3 — Servicios de dominio | `services/` | Reglas de negocio de un contexto delimitado, sin conocimiento de quién las invoca | `execution/`, `apps/` | `execution/`, `apps/` |
| 4 — Núcleo compartido | `packages/` | Primitivas de las que cualquier otro nivel puede depender | Todo lo anterior | Cualquier otro nivel |

**Principio arquitectónico único que gobierna esta tabla (regla de dependencia):** una dependencia de código fuente solo puede apuntar de un nivel numéricamente menor hacia uno numéricamente mayor (por ejemplo, `apps/` → `services/` es válido; `packages/` → `apps/` no lo es, bajo ninguna circunstancia). El nivel 0 (`infra/`) queda fuera de esta regla porque no participa del grafo de dependencias de código: no importa ningún paquete, solo produce y conecta artefactos ya compilados (imágenes de contenedor).

Esta regla es la base de la separación de responsabilidades: el nivel más genérico (`packages/`) nunca puede quedar acoplado a una decisión específica de una aplicación puntual.

---

## 2. Criterios de clasificación por nivel

Cada nivel se define por un criterio de inclusión y uno de exclusión, no por descripción general. Esto es lo que elimina la ambigüedad de "a qué nivel pertenece esto": se aplica el criterio, no el juicio subjetivo.

### Nivel 0 — Orquestación (`infra/`)

- **Criterio de inclusión:** el directorio define cómo se construye, despliega, conecta en red o persiste un artefacto ya compilado (manifiestos de Docker Compose, futuros manifiestos de Kubernetes, definiciones de volúmenes).
- **Criterio de exclusión:** si el directorio contiene una sola línea de regla de negocio (validación de dominio, cálculo, política de autorización), no pertenece a este nivel, sin excepción.
- **Estado actual:** `infra/docker/` (Docker Compose para Postgres local). No existe todavía un directorio de manifiestos de Kubernetes.

### Nivel 1 — Aplicaciones (`apps/`)

- **Criterio de inclusión:** el directorio produce un proceso ejecutable independiente, con su propio punto de entrada (`main.ts`, `next start`, etc.) y su propio puerto de red.
- **Criterio de exclusión:** una librería que no se ejecuta por sí sola, sino que es importada por otro paquete, no pertenece a este nivel aunque tenga lógica compleja.
- **Estado actual:** `apps/api` (backend NestJS, puerto 4000), `apps/web` (frontend Next.js, puerto 3000).

### Nivel 2 — Orquestación de ejecución (`execution/`)

- **Criterio de inclusión:** el directorio coordina la secuencia, el enrutamiento o la simulación de ejecución de trabajo *entre* servicios de dominio, sin poseer reglas de negocio de un dominio específico.
- **Criterio de exclusión:** si el directorio modela las reglas propias de un área de negocio (por ejemplo, qué constituye una política aprobada), pertenece a `services/`, no a este nivel, independientemente de cuán "central" parezca su función.
- **Estado actual:** `execution/graph-orchestration` (enrutamiento tipo Markov), `execution/runtime-fabric` (simulación de ejecución en runtime).

### Nivel 3 — Servicios de dominio (`services/`)

- **Criterio de inclusión:** el directorio modela las reglas de negocio de un contexto delimitado (bounded context) específico, y es agnóstico respecto de qué proceso lo invoca o cómo se presenta su resultado.
- **Criterio de exclusión:** si el directorio solo construye o despliega procesos, pertenece a `infra/`; si solo orquesta la ejecución de otros servicios sin reglas propias, pertenece a `execution/`.
- **Estado actual:** `governance-compliance`, `repo-knowledge`, `agent-cognition`, `product-narrative`, `macro-apps`, `platform-integration`.

### Nivel 4 — Núcleo compartido (`packages/`)

- **Criterio de inclusión:** el directorio provee primitivas de propósito general, sin conocimiento de ningún contexto de negocio, consumibles por cualquier paquete de cualquier nivel superior.
- **Criterio de exclusión:** si el directorio depende de cualquier otro paquete del monorepo, no pertenece a este nivel, sin excepción — la ausencia de dependencias internas es la condición que lo define.
- **Estado actual:** `packages/kernel` (bus de trazabilidad de eventos).

---

## 3. Estructura de directorios

```
.
├── infra/                          NIVEL 0 — Orquestación
│   └── docker/                     Despliegue local de Postgres vía Docker Compose
│       ├── compose.yaml
│       ├── data/postgres/          Volumen de datos (excluido de control de versiones)
│       └── README.md               Disciplina operativa del nivel de orquestación
│
├── apps/                           NIVEL 1 — Aplicaciones
│   ├── api/                        Backend NestJS: GraphQL, JWT, RBAC, CASL, Zod, Drizzle/Postgres
│   └── web/                        Frontend Next.js: interfaz REPO_OS
│
├── execution/                      NIVEL 2 — Orquestación de ejecución
│   ├── graph-orchestration/        Enrutamiento de trabajo, overlays de estado operacional
│   └── runtime-fabric/             Motor de simulación de ejecución en runtime
│
├── services/                       NIVEL 3 — Servicios de dominio
│   ├── governance-compliance/      Motor de políticas, gobernanza constitucional, cumplimiento de seguridad
│   ├── repo-knowledge/             Lectura de repositorio y grafo de conocimiento
│   ├── agent-cognition/            Comandos de agente y razonamiento cognitivo
│   ├── product-narrative/          Narrativa de producto, demostraciones, reproducción de eventos
│   ├── macro-apps/                 Estado de interfaz para el shell del lienzo (canvas)
│   └── platform-integration/       Integración con plataformas externas y control MCP
│
└── packages/                       NIVEL 4 — Núcleo compartido
    └── kernel/                     Bus de trazabilidad de eventos
```

---

## 4. Grafo de dependencias

```
                    NIVEL 0 — infra/
                    (fuera del grafo de dependencias de código;
                     construye y conecta artefactos compilados)
                                │
                                │ construye y ejecuta
                                ▼
        ┌───────────────────────────────────────────────────┐
        │  NIVEL 1 — apps/                                    │
        │  apps/api  (puerto 4000)    apps/web  (puerto 3000) │
        └──────────────────┬──────────────────┬───────────────┘
                           │                  │ depende de
                           │                  ▼
                           │   ┌─────────────────────────────────┐
                           │   │  NIVEL 2 — execution/            │
                           │   │  graph-orchestration              │
                           │   │  runtime-fabric                   │
                           │   └────────────────┬───────────────────┘
                           │                    │ depende de
                           │                    ▼
                           │   ┌─────────────────────────────────────┐
                           └──▶│  NIVEL 3 — services/                │
                               │  governance-compliance · repo-knowledge│
                               │  agent-cognition · product-narrative  │
                               │  macro-apps · platform-integration    │
                               └────────────────┬─────────────────────┘
                                                │ depende de
                                                ▼
                               ┌─────────────────────────────────┐
                               │  NIVEL 4 — packages/               │
                               │  kernel                             │
                               │  (no depende de ningún otro nivel) │
                               └─────────────────────────────────┘
```

**Lectura del grafo:** toda flecha representa "depende de" y apunta exclusivamente en sentido descendente (de nivel numérico menor a mayor). No existe, ni debe existir, una flecha en sentido inverso. `apps/api` es una excepción visible en el estado actual: no declara dependencia de ningún paquete del monorepo (ver sección 6).

---

## 5. Inventario de paquetes

| # | Ruta | Identificador de paquete | Nivel | Responsabilidad | Dependencias internas declaradas | Líneas de código |
|---|---|---|---|---|---|---|
| 1 | `infra/docker` | (no es un paquete de código) | 0 | Orquestación de Postgres vía Docker Compose | — | — |
| 2 | `apps/api` | `@tst-autonomous/api` | 1 | Backend NestJS: GraphQL, JWT, RBAC, CASL, Zod, Postgres/Drizzle | Ninguna (ver sección 6) | 1084 |
| 3 | `apps/web` | `web` | 1 | Interfaz REPO_OS: grafo del repositorio y mensajería sobre NATS JetStream | governance-compliance, graph-orchestration, macro-apps, product-narrative, repo-knowledge | 2812 |
| 4 | `execution/graph-orchestration` | `@tst-autonomous/graph-orchestration` | 2 | Enrutamiento de trabajo (Markov routing), overlays de estado operacional | repo-knowledge, governance-compliance, kernel | 135 |
| 5 | `execution/runtime-fabric` | `@tst-autonomous/runtime-fabric` | 2 | Motor de simulación de ejecución en runtime | Ninguna | 782 |
| 6 | `services/governance-compliance` | `@tst-autonomous/governance-compliance` | 3 | Motor de políticas, gobernanza constitucional, cumplimiento de seguridad | kernel | 1245 |
| 7 | `services/repo-knowledge` | `@tst-autonomous/repo-knowledge` | 3 | Lectura de repositorio, grafo de conocimiento, memoria persistente | kernel, governance-compliance | 500 |
| 8 | `services/agent-cognition` | `@tst-autonomous/agent-cognition` | 3 | Comandos de agente y razonamiento cognitivo | kernel, governance-compliance, repo-knowledge, macro-apps | 378 |
| 9 | `services/product-narrative` | `@tst-autonomous/product-narrative` | 3 | Narrativa de producto, demostraciones, reproducción de eventos (alcance amplio — ver `decisions-log.md`) | runtime-fabric | 813 |
| 10 | `services/macro-apps` | `@tst-autonomous/macro-apps` | 3 | Estado de interfaz para el shell del lienzo | kernel, governance-compliance, repo-knowledge | 227 |
| 11 | `services/platform-integration` | `@tst-autonomous/platform-integration` | 3 | Integración con plataformas externas, control MCP, definición de stack canónico | Ninguna | 739 |
| 12 | `packages/kernel` | `@tst-autonomous/kernel` | 4 | Bus de trazabilidad de eventos | Ninguna | 502 |

---

## 6. Estado actual declarado (sin inferencias)

Las siguientes afirmaciones describen el estado verificado del código al momento de este documento. No deben interpretarse como planes ni como trabajo en curso salvo que se indique explícitamente.

1. **`services/governance-compliance` no está conectado a `apps/api`.** `apps/api` implementa su propio mecanismo de autorización (CASL/RBAC, en `apps/api/src/casl/`), independiente del motor de políticas de `services/governance-compliance`. La integración entre ambos no existe en el código actual.
2. **No existe infraestructura de Kubernetes en este repositorio.** `infra/docker/` es exclusivamente Docker Compose. La convención de nombrar cada volumen de datos igual que su servicio (documentada en `infra/docker/README.md`) fue diseñada para facilitar una futura migración a `PersistentVolumeClaim`, pero esa migración no ha comenzado.
3. **`apps/api` no declara ninguna dependencia interna del monorepo** (ver fila 2 de la sección 5). Es la única excepción a la expectativa general de que las aplicaciones consuman servicios de dominio.

---

## 7. Documentos relacionados

- **`decisions-log.md`** — Registro de decisiones de reclasificación arquitectónica: qué cambió, con qué evidencia, y qué verificación se ejecutó.
- **`glossary.md`** — Definiciones formales de la terminología empleada en este documento.
