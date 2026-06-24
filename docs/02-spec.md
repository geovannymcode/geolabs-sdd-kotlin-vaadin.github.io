# **Fase 1: La especificación (`/spec`)**

## **¿Qué vamos a hacer y por qué?**

Acá empieza lo spec-driven. La primera cosa que producimos no es código, es la especificación. El comando `/spec` toma la descripción del feature y la convierte en un documento estructurado: qué debe hacer, qué queda fuera, y qué falta por decidir.

Una spec es como acordar el precio antes de facturarle a un cliente. El sistema cobra igual, pero la conversación de "¿cuánto era?" llega después, y siempre llega cara. En software el costo se esconde mejor, pero está ahí.

## **1. Correr `/spec`**

Le pasas al comando la descripción del feature, o una URL de issue si la tienes. Para el taller usamos la historia de crear un cliente.

```bash title="Claude Code"
/spec Crear un cliente desde la app, con validacion de campos y email unico,
mostrando errores antes de guardar
```

El agente escribe la spec en su carpeta:

```bash
.specs/2026-05-09-create-new-customer/
 └── 01-spec.md
```

Y dentro vas a ver algo así, con los criterios numerados:

```markdown title="01-spec.md"
## Criterios de aceptacion
- AC-001 El formulario muestra cinco campos: First, Last, Email, Phone, Company.
- AC-002 El boton de crear esta deshabilitado hasta que lo obligatorio es valido.
- AC-005 Email obligatorio, formato valido, max 255, unico.
- AC-008 Con datos validos, el cliente se persiste.
- AC-009 Tras crear por REST, responde 201 con Location.
- AC-011 Email ya registrado responde 409 y, en la UI, marca el campo email.
- AC-013 Un fallo inesperado responde 500 con ProblemDetail.
```

Cada criterio es una afirmación verificable. Esto importa, porque más adelante cada uno se vuelve un test. La prueba rápida: ¿podrías escribir un test que diga "pasó" o "no pasó" para este criterio? Si no, reescríbelo.

## **2. Las preguntas abiertas**

Acá está la regla más importante de todo el flujo: **el agente no inventa**. Lo que no está claro, lo pregunta. El `/spec` te deja una lista de preguntas abiertas que tienes que responder antes de avanzar.

```markdown title="01-spec.md (preguntas)"
## Preguntas abiertas
- Q-001 Tras crear en la UI, a donde navega?
- Q-002 El email unico distingue mayusculas?
- Q-003 Como se ve la confirmacion de exito?
- Q-004 El control de roles esta en alcance?
- Q-005 El NFR de 500ms es end-to-end o de servidor?
```

Las respondes y vuelves a correr `/spec` con las respuestas, hasta que no quede ninguna pendiente.

```bash title="Claude Code"
/spec Respuestas: Q-001 notificacion y limpiar el form; Q-002 no distingue
mayusculas; Q-003 notificacion de Vaadin; Q-004 fuera de alcance; Q-005 de servidor
```

- **Puntos Clave**
    1. **Una pregunta sin responder bloquea la fase.** No pasas al plan hasta que todas tengan respuesta. Esto solo te ahorra la mitad de los bugs.
    2. **Las respuestas se vuelven decisiones.** Q-002 termina siendo el ADR de cómo garantizamos el email único.
    3. **El alcance se escribe.** Listar, editar y borrar clientes quedan como non-goals, para que la IA no se invente features que nadie pidió.

## **3. Revisar la spec con `/spec-review`**

Una spec solo sirve si es clara. El comando `/spec-review` la audita contra una lista: objetivo en términos de usuario, criterios atómicos y verificables, non-goals explícitos, preguntas resueltas.

```bash title="Claude Code"
/spec-review 2026-05-09-create-new-customer
```

Escribe el resultado y te da un veredicto:

```text
Veredicto: PASS

Hallazgo (advisory, no bloquea):
  MINOR-001: AC-009 junta dos comportamientos (confirmar y navegar).
  Como ambos quedaron definidos en Q-001 y Q-003, no bloquea el plan.
```

Con plastilina: el review es el segundo par de ojos antes de construir. Si la spec no aguanta la lista, el momento de arreglarla es ahora, no cuando ya hay código encima.

!!! warning "Si el veredicto es FAIL"
    No avances. El review te lista qué resolver. Casi siempre es una pregunta abierta que quedó a medias o un criterio que no es verificable. Lo arreglas en la spec y vuelves a correr `/spec-review`.

!!! success "Checkpoint de la Fase 1"
    `.specs/2026-05-09-create-new-customer/` con `01-spec.md` (sin preguntas pendientes) y `02-spec-review.md` con veredicto PASS. Con eso, la spec está lista para volverse un plan.

En la siguiente fase la partimos en tareas con `/plan`.
