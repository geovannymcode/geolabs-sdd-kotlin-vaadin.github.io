# Taller · SDD con Kotlin, Spring Boot y Vaadin

Guía paso a paso (mkdocs material) para construir el feature "crear cliente" siguiendo el flujo spec-driven, con Spring Boot 4.1 en capas (split api/internal) y Vaadin 25. Traducción del ejemplo de Loiane Groner: Java a Kotlin y Angular a Vaadin.

## Servir localmente

```bash
pip install mkdocs-material
mkdocs serve
```

Abre http://localhost:8000

## Fases

- `docs/index.md` — intro, requisitos, el flujo SDD
- `docs/01-spec.md` — la especificación antes del editor
- `docs/02-persistencia.md` — T-001: entidad, repositorio, Flyway, Testcontainers
- `docs/03-servicio.md` — T-002: servicio (api/internal), DTOs, email único
- `docs/04-rest.md` — T-003: controller y ProblemDetail
- `docs/05-vaadin.md` — T-004 a T-006: la UI en Vaadin con Karibu
- `docs/06-validacion.md` — validación y trazabilidad
- `docs/referencias.md` — créditos

El proyecto que reproduce esta guía está en el repo `sdd-kotlin-vaadin`.
