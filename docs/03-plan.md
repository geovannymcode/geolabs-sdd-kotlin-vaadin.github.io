# **Fase 2: El plan (`/plan`)**

## **¿Qué vamos a hacer y por qué?**

Tenemos una spec aprobada. Ahora la volvemos ejecutable. El comando `/plan` la traduce a un diseño y la parte en tareas pequeñas, cada una atada a sus criterios y con forma de TDD.

Pedirle a la IA que construya todo el feature de una es como pedirle a alguien que se trague una arepa entera de un bocado. Partirlo en tareas chiquitas es más fácil de revisar, y es donde alcanzas a cazar los errores antes de que se rieguen.

## **1. Correr `/plan`**

```bash title="Claude Code"
/plan 2026-07-09-create-new-customer
```

El agente lee la spec y el review, y escribe varios artefactos:

```bash
.specs/2026-07-09-create-new-customer/
 ├── 03-design.md        # la arquitectura y el contrato
 ├── 04-tasks.md         # las tareas
 ├── 📁 adr/             # las decisiones registradas
 │    ├── ADR-001-feature-package-structure.md
 │    └── ADR-002-email-uniqueness-strategy.md
 └── .tdd-state.json     # el estado de cada tarea
```

## **2. El diseño: capas, no hexagonal**

El `03-design.md` define la arquitectura. Spring en capas, organizado por feature, con un split entre lo público y lo interno:

```bash
com.geovannycode.sdd.customer
 ├── 📁 api/        # la vitrina: lo que el resto de la app puede usar
 └── 📁 internal/   # la bodega: la implementacion, escondida
```

La `api` es la vitrina de la tienda, el `internal` es la bodega. El cliente compra en la vitrina y nunca entra a la bodega. El día que cambies algo de la bodega, nadie afuera se rompe.

!!! info "api / internal en Kotlin, un matiz que vale oro en la charla"
    En Java, los tipos de `internal` se esconden con package-private y lo refuerza el compilador. Kotlin no tiene package-private, así que el límite es una convención. ¿Cómo lo volvemos real? El plan registra un ADR que dice: lo convertimos en un test de ArchUnit. Una regla de arquitectura se vuelve un sensor del harness. No es debilidad de Kotlin, es ejemplo de por qué el harness importa.

## **3. Las tareas, en dos tracks**

El `04-tasks.md` parte el feature en seis tareas, en dos pistas que pueden ir en paralelo porque el contrato ya está definido.

```markdown title="04-tasks.md"
## Track A - Backend
- T-001 Persistencia: Customer (entidad), CustomerRepository, migracion V1.
- T-002 Servicio: CustomerService (api) + CustomerServiceImpl (internal), DTOs.
- T-003 HTTP: CustomerController, CustomerExceptionHandler (ProblemDetail).

## Track B - Frontend (Vaadin)
- T-004 Form: layout, modelo y campos de la vista.
- T-005 Validacion inline + boton condicional.
- T-006 Wiring: enviar al servicio, mapear errores, exito.
```

Cada tarea sabe qué archivos toca, qué criterios cubre y cuál es su test. Y el `.tdd-state.json` arranca con todas en `pending`:

```json title=".tdd-state.json"
{
  "feature_id": "2026-07-09-create-new-customer",
  "active_task": null,
  "tasks": {
    "T-001": { "phase": "pending", "acs_covered": ["AC-008", "AC-011"], "depends_on": [] }
  }
}
```

- **Ventajas de planear antes de codear**
    - **Paralelismo real**: con el contrato fijo, el front no espera al backend.
    - **TDD acotado**: cada tarea sabe qué probar y qué archivos tocar.
    - **Estado fuera de tu cabeza**: si vuelves el lunes, el `.tdd-state.json` te dice dónde quedaste.

!!! success "Checkpoint de la Fase 2"
    Tienes `03-design.md`, `04-tasks.md`, los dos ADR y `.tdd-state.json`. Las seis tareas están en `pending`. Ya puedes pedirle a Claude que construya la primera.

En la siguiente fase corremos `/build` y vemos a Claude generar el backend.
