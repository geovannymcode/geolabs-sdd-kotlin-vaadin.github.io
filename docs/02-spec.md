# **Fase 1: La especificación (`/spec`)**

## **¿Qué vamos a hacer y por qué?**

Acá empieza lo spec-driven. La primera cosa que producimos no es código, es la especificación. El comando `/spec` toma la descripción del feature y la convierte en un documento estructurado: qué debe hacer, qué queda fuera, y qué falta por decidir.

En simple: la spec es el plano de la casa. Nadie pone ladrillos antes de tener el plano aprobado. Y si a mitad de obra descubres que faltaba un baño, tumbar pared cuesta carísimo. En software el costo se esconde mejor, pero está ahí.

## **1. Correr `/spec`**

Le pasas al comando la descripción del feature. Para el taller usamos crear un cliente con cinco datos.

```bash title="Claude Code"
/spec Crear un cliente con nombre, apellido, email, direccion y telefono,
con validacion de campos y email unico, mostrando errores antes de guardar
```

El agente escribe la spec en su carpeta, con los criterios numerados:

```markdown title="01-spec.md"
## Criterios de aceptacion
- AC-001 El formulario muestra nombre, apellido, email, direccion, telefono y el boton "Crear cliente".
- AC-002 Nombre obligatorio, maximo 255.
- AC-003 Apellido obligatorio, maximo 255.
- AC-004 Email obligatorio, formato valido, maximo 254.
- AC-005 Direccion obligatoria, maximo 255.
- AC-006 Telefono obligatorio, solo digitos con un + opcional al inicio, maximo 20.
- AC-007 Al enviar con un campo invalido, se muestra error inline y no se envia.
- AC-008 Al enviar valido, se persiste el cliente y se muestra notificacion de exito.
- AC-009 Tras crear con exito, el formulario se limpia.
- AC-010 Por REST, POST valido responde 201 con el id y el header Location.
- AC-011 Datos invalidos responden 400 con ProblemDetail.
- AC-012 Email ya registrado responde 409 con ProblemDetail.
- AC-013 El email unico no distingue mayusculas y se normaliza a minusculas.
```

Cada criterio es una afirmación verificable. Esto importa, porque más adelante cada uno se vuelve un test. La prueba rápida: ¿podrías escribir un test que diga "pasó" o "no pasó" para este criterio? Si no, reescríbelo.

## **2. Las preguntas abiertas**

Acá está la regla más importante de todo el flujo: **el agente no inventa**. Lo que no está claro, lo pregunta.

```markdown title="01-spec.md (preguntas)"
## Preguntas abiertas
- Q-001 Tras crear en la UI, a donde navega?
- Q-002 El email unico distingue mayusculas?
- Q-003 Como se ve la confirmacion de exito?
- Q-004 Que formato acepta el telefono?
- Q-005 El control de roles esta en alcance?
```

Las respondes y vuelves a correr `/spec` con las respuestas, hasta que no quede ninguna pendiente.

```bash title="Claude Code"
/spec Respuestas: Q-001 notificacion y limpiar el form; Q-002 no distingue
mayusculas; Q-003 notificacion de Vaadin con autocierre; Q-004 solo digitos con
un + opcional; Q-005 fuera de alcance
```

- **Puntos Clave**
    1. **Una pregunta sin responder bloquea la fase.** No pasas al plan hasta que todas tengan respuesta.
    2. **Las respuestas se vuelven decisiones.** Q-002 termina siendo el ADR de cómo garantizamos el email único.
    3. **El alcance se escribe.** Listar, editar y borrar clientes quedan como non-goals, para que la IA no invente features que nadie pidió.

## **3. Revisar la spec con `/spec-review`**

Una spec solo sirve si es clara. El comando `/spec-review` la audita contra una lista: objetivo en términos de usuario, criterios atómicos y verificables, non-goals explícitos, preguntas resueltas.

```bash title="Claude Code"
/spec-review 2026-05-09-create-new-customer
```

Escribe el resultado y te da un veredicto:

```text
Veredicto: PASS

Hallazgo (advisory, no bloquea):
  MINOR-001: AC-008 junta dos comportamientos (persistir y notificar).
  Como ambos quedaron definidos en Q-001 y Q-003, no bloquea el plan.
```

En simple: el review es el segundo par de ojos antes de construir. Si la spec no aguanta la lista, el momento de arreglarla es ahora, no cuando ya hay código encima.

!!! warning "Si el veredicto es FAIL"
    No avances. El review te lista qué resolver. Casi siempre es una pregunta abierta a medias o un criterio que no es verificable. Lo arreglas en la spec y vuelves a correr `/spec-review`.

!!! success "Checkpoint de la Fase 1"
    `01-spec.md` (sin preguntas pendientes) y `02-spec-review.md` con veredicto PASS. Con eso, la spec está lista para volverse un plan.

En la siguiente fase la partimos en tareas con `/plan`.
