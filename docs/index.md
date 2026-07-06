# **Spec-Driven Development con Kotlin, Spring Boot y Vaadin**

## **Por qué este taller**

Promptear sirve para explorar. Le pides algo a la IA, te tira código, y para tantear una idea está bien. El problema empieza cuando ese código tiene que pasar a revisión, a pruebas, a producción. Ahí la conversación deja de alcanzar.

Spec-Driven Development resuelve eso: en vez de que la intención viva en un chat que mañana cierras, vive en un archivo dentro del repo. Una especificación, versionada al lado del código. Y un agente que la sigue, paso a paso, sin inventar.

La idea no es nueva. Lo nuevo acá es el stack. Casi todos los ejemplos de SDD son Java con Angular o React. En este taller lo hacemos con **Kotlin, Spring Boot y Vaadin**. Construimos un feature completo, crear un cliente, pero todo en la JVM. Sin JavaScript. Sin framework de frontend aparte.

## **Lo que vas a hacer, no a leer**

Esto es un taller de verdad. No vas a escribir el código a mano. Vas a correr comandos, responder preguntas, y ver a Claude generar el feature guiado por la spec. Tú diriges, la IA ejecuta.

El flujo completo, en comandos:

```bash
/onboard                 # clasifica el proyecto y prepara el terreno
/wire-harness            # arma las barandas: tests, cobertura, guardrail
/spec <feature>          # escribe la especificacion
/spec-review <feature>   # la audita: PASS o FAIL
/plan <feature>          # la parte en tareas con forma de TDD
/build T-001             # construye una tarea: red, green, refactor
/build T-006             # ... una por una
/validate                # corre todo el harness + trazabilidad
/review                  # auditoria final contra la spec
```

Cada comando deja un artefacto en `.specs/`. Al final, levantas el proyecto y lo ves corriendo.

## **El feature: crear un cliente**

Un cliente con cinco datos: **nombre, apellido, email, dirección y teléfono**. Suena simple, pero tiene todo lo interesante. Validación de campos, un correo que debe ser único, un contrato REST con sus códigos, y una pantalla de verdad en Vaadin.

## **A dónde vamos a llegar**

```bash
sdd-kotlin-vaadin/
 ├── 📁 src/main/kotlin/com/geovannycode/
 │    ├── SddKotlinVaadinApplication.kt
 │    └── 📁 customer/
 │         ├── 📁 api/        # la vitrina del feature
 │         ├── 📁 internal/   # la bodega (la implementacion)
 │         └── 📁 ui/         # la vista de Vaadin
 ├── 📁 src/main/resources/db/migration/   # las migraciones de Flyway
 └── 📁 .specs/2026-05-09-create-new-customer/   # el rastro spec-driven
```

En simple, la idea de **api / internal**: piensa en una tienda. La `api` es la vitrina, lo que el cliente ve y usa. El `internal` es la bodega, donde está la maquinaria y a la que nadie entra de afuera.

## **El harness, la pieza que casi nadie nombra**

El harness es el arnés que mantiene a la IA dentro de los límites que tú quieres. Tiene dos lados. El feedforward es lo que le das antes: la spec, el contrato, las reglas. El feedback es lo que la frena después: los tests, la validación, el guardrail de arquitectura.

En simple: el harness es la baranda de la escalera. No te impide subir, te frena antes de caerte. La IA escribe rápido; la baranda evita que ese código rápido se vaya por un barranco.

## **Requisitos**

- Sintaxis básica de Kotlin o Java, idea de API REST y de bases relacionales.
- **JDK 25**, **Docker Desktop**, **IntelliJ IDEA** con **Claude Code**.
- Versiones del stack: **Spring Boot 4.1.0, Kotlin 2.3.21, Vaadin 25.2.0, PostgreSQL 17**.

!!! tip "La regla de oro"
    El agente nunca inventa. Cada duda es una pregunta que respondes antes de avanzar. Si la spec tiene preguntas sin responder, la spec no está lista. Es la diferencia entre dirigir a la IA y rogarle.

Empezamos por preparar el terreno: el andamio y el `.claude`.
