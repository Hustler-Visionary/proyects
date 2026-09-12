# Glosario (sin jerga)

_Los términos de `README.md` y `decisions-log.md`, explicados para alguien que no programa. Si un término técnico te frenó, probablemente está acá._

| Término | Qué significa, en criollo |
|---|---|
| **Monorepo** | Un solo repositorio (una sola carpeta versionada) que contiene *varios* programas relacionados, en vez de tener un repositorio separado por cada uno. Es como tener todos los planos de un edificio en una sola carpeta, aunque cada piso sea distinto. |
| **Paquete** | Una unidad de código con nombre propio, que se puede compilar y probar por separado (ej. `@tst-autonomous/kernel`). Cada carpeta de este mapa con su propio `package.json` es un paquete. |
| **Dependencia** ("A depende de B") | A necesita que B exista y funcione para poder hacer su trabajo. Si B se rompe, A también se rompe. La regla de este repo es que las dependencias solo van "hacia abajo" en la jerarquía — nunca al revés. |
| **Nivel / capa** | Un grupo de piezas que comparten el mismo tipo de responsabilidad (ver la tabla del edificio en `README.md`, sección 1). |
| **Orquestación** | Coordinar el arranque, conexión y apagado de varias piezas para que funcionen juntas — como un director de orquesta, no como un músico. `infra/` orquesta procesos (los prende, los conecta por red); `execution/` orquesta *trabajo* (decide qué se ejecuta y en qué orden). |
| **Puerto (de red)** | La "puerta" numerada por la que un programa escucha pedidos. `apps/api` escucha en el puerto 4000, `apps/web` en el 3000 — como dos locales con distinta dirección en la misma cuadra. |
| **API / GraphQL** | La forma en que un programa le permite a otro pedirle datos o pedirle que haga algo. GraphQL es un estilo particular de API donde quien pregunta especifica exactamente qué datos quiere. |
| **JWT (token)** | Una "credencial firmada" que probás una vez (al iniciar sesión) y después mostrás en cada pedido para no tener que volver a poner tu contraseña. |
| **RBAC** | Sigla de "control de acceso basado en roles": las reglas de qué puede hacer alguien dependen de *qué rol* tiene (ej. "ADMIN" vs "USER"), no de quién es específicamente. |
| **CASL** | Una librería que expresa reglas de permiso más finas que un rol simple — por ejemplo "un USER puede editar *solo sus propias* tareas", algo que un rol por sí solo no alcanza a describir. |
| **ORM (Drizzle)** | Una capa que te deja escribir consultas a la base de datos en el mismo lenguaje de programación del resto del código, en vez de escribir SQL a mano en todos lados. |
| **Migración (de base de datos)** | Un cambio versionado y guardado a la estructura de la base de datos (ej. "agregar la tabla de usuarios") — como un historial de reformas edilicias, para poder aplicar los mismos cambios en cualquier copia del edificio. |
| **Docker / contenedor** | Una forma de empaquetar un programa junto con todo lo que necesita para correr, de manera que funcione igual en cualquier computadora. Como enviar un electrodoméstico ya armado, en vez de mandar las piezas sueltas con instrucciones. |
| **Docker Compose** | Una lista de "qué contenedores prender y cómo conectarlos entre sí" para desarrollo local — hoy usado acá solo para levantar Postgres. |
| **Kubernetes (k8s)** | El "director de orquesta" para contenedores a gran escala: decide en qué máquina corre cada contenedor, los reinicia si se caen, los escala si hay mucha demanda. Es el reemplazo, a futuro, de Docker Compose para producción — todavía no está en este repo. |
| **PVC (PersistentVolumeClaim)** | El equivalente, en Kubernetes, a "una carpeta de datos que sobrevive aunque el contenedor se reinicie". La disciplina de este repo (una carpeta `data/<servicio>/` por servicio) está pensada para que, el día que exista Kubernetes acá, cada carpeta se convierta directamente en una PVC del mismo nombre. |
| **Governance / Compliance** | Las reglas que dicen qué se puede hacer y qué no, y quién lo autoriza — como el área de "cumplimiento normativo" de una empresa, pero para el software: qué acción necesita aprobación, qué queda prohibido por política, qué se audita. |
| **Bounded context** (contexto delimitado) | Un área de negocio con su propio vocabulario y reglas, que no necesita saber los detalles internos de las demás áreas — como cómo "Finanzas" y "Legal" pueden usar la palabra "contrato" con significados algo distintos, cada una dueña del suyo. Cada carpeta en `services/` es uno de estos. |
| **Regla de dependencia** | El principio de que el código "genérico y estable" (el kernel) nunca debe saber nada del código "específico y cambiante" (una app puntual) — solo al revés. Es lo que evita que cambiar una pantalla rompa una regla de negocio. |
