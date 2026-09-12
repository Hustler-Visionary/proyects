# Glossary

**Propósito:** Definiciones formales de la terminología técnica empleada en `README.md` y `decisions-log.md`, para lectores sin formación en ingeniería de software.

| Término | Definición |
|---|---|
| **Monorepo** | Repositorio único que contiene múltiples unidades de código relacionadas (aplicaciones, servicios, librerías), gestionadas bajo un mismo control de versiones, en contraposición a mantener un repositorio independiente por cada unidad. |
| **Paquete** | Unidad de código con identidad propia (nombre, versión, manifiesto de configuración) que puede compilarse y probarse de forma aislada. Cada directorio de este mapa que contiene su propio `package.json` es un paquete. |
| **Dependencia** | Relación en la que un paquete A requiere que un paquete B exista y funcione correctamente para operar. Una falla en B se propaga a A. La arquitectura de este repositorio exige que las dependencias solo se establezcan en sentido descendente según la jerarquía de niveles (ver `README.md` §1). |
| **Nivel arquitectónico** | Agrupación de paquetes que comparten un mismo criterio de responsabilidad, definido de forma explícita y verificable (ver `README.md` §2). |
| **Regla de dependencia** | Principio, tomado de la arquitectura hexagonal y de Clean Architecture, según el cual el código de propósito general nunca debe depender del código específico de una aplicación puntual — únicamente a la inversa. Es la base formal que sostiene la separación entre niveles. |
| **Orquestación (de procesos)** | Coordinación del ciclo de vida de procesos ejecutables: construcción, inicio, conexión de red y finalización. Responsabilidad del Nivel 0 (`infra/`). |
| **Orquestación (de ejecución de trabajo)** | Coordinación de la secuencia y el enrutamiento de tareas entre servicios de dominio, sin poseer reglas de negocio propias. Responsabilidad del Nivel 2 (`execution/`). Se distingue de la orquestación de procesos en que opera sobre unidades de trabajo lógicas, no sobre procesos del sistema operativo. |
| **Puerto de red** | Número mediante el cual un proceso identifica el canal de comunicación por el que acepta solicitudes entrantes. `apps/api` opera en el puerto 4000; `apps/web`, en el puerto 3000. |
| **API** | Interfaz mediante la cual un sistema expone funcionalidad o datos para ser consumidos por otro sistema. |
| **GraphQL** | Especificación de API en la que el consumidor declara explícitamente qué campos de datos requiere en cada solicitud, en contraste con los endpoints de forma fija de una API REST convencional. |
| **JWT (JSON Web Token)** | Credencial firmada digitalmente, emitida una vez tras la autenticación, que se adjunta a cada solicitud posterior para acreditar identidad sin requerir el reenvío de credenciales primarias. |
| **RBAC (Role-Based Access Control)** | Modelo de control de acceso en el que los permisos se determinan por el rol asignado a un sujeto (por ejemplo, `ADMIN`, `USER`), no por su identidad individual. |
| **CASL** | Biblioteca de control de acceso basado en atributos y condiciones, que permite expresar reglas de autorización más granulares que un rol por sí solo — por ejemplo, restringir una acción a los registros de propiedad del propio sujeto. |
| **ORM (Object-Relational Mapping)** | Capa de software que permite representar y consultar datos de una base de datos relacional mediante el lenguaje de programación de la aplicación, sin escribir sentencias SQL de forma directa. En este repositorio, dicha capa es Drizzle. |
| **Migración (de base de datos)** | Cambio versionado y reproducible aplicado a la estructura de una base de datos, registrado de forma que pueda aplicarse de manera idéntica en cualquier entorno. |
| **Contenedor** | Unidad de software que empaqueta código y todas sus dependencias de ejecución, garantizando comportamiento equivalente en cualquier entorno compatible. |
| **Docker Compose** | Herramienta que define, mediante un archivo de configuración declarativo, un conjunto de contenedores y su conectividad de red, empleada en este repositorio para el entorno de desarrollo local. |
| **Kubernetes** | Sistema de orquestación de contenedores a escala de producción: determina en qué nodo se ejecuta cada contenedor, gestiona su reinicio ante fallos y su escalamiento ante demanda. No forma parte todavía de este repositorio. |
| **PersistentVolumeClaim (PVC)** | En Kubernetes, solicitud de almacenamiento persistente que sobrevive al ciclo de vida de un contenedor individual. La convención de nombrar cada directorio de datos igual que su servicio (`infra/docker/README.md`) anticipa una futura correspondencia directa entre esos directorios y sus respectivas PVC. |
| **Gobernanza y cumplimiento (Governance & Compliance)** | Conjunto de reglas que determinan qué acciones están permitidas, cuáles requieren aprobación explícita y cuáles quedan prohibidas por política, junto con el registro auditable de dichas decisiones. |
| **Bounded context (contexto delimitado)** | Concepto de Domain-Driven Design que designa un área de negocio con vocabulario y reglas propias, cuyo modelo interno no necesita ser consistente con el de otras áreas. Cada paquete de `services/` constituye uno de estos contextos. |
| **ADR (Architecture Decision Record)** | Formato estandarizado de documentación que registra una decisión arquitectónica junto con su contexto, evidencia, alternativas consideradas y consecuencias. Empleado en `decisions-log.md`. |
