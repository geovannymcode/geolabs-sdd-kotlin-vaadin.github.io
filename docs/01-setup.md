# **Fase 0: El andamio y el `.claude`**

## **¿Qué vamos a hacer y por qué?**

Antes de la primera spec, el repo tiene que hablar el mismo idioma que el flujo. Eso significa dos cosas: un andamio limpio y los comandos del flujo instalados. Sin eso, la IA puede generar código, pero no dentro de un proceso.

## **1. El andamio, desde Spring Initializr**

Genera el proyecto base en [**start.spring.io**](https://start.spring.io){:target="_blank"}. Un solo proyecto.

Configúralo así:

- Project: **Maven**, Language: **Kotlin**, Spring Boot: **4.1.x**, Java: **25**.
- Dependencias: **Vaadin**, **Spring Web**, **Spring Data JPA**, **Validation**, **Flyway Migration**, **PostgreSQL Driver**.

Descárgalo, ábrelo en IntelliJ y arráncalo una vez para confirmar que vive.

!!! warning "El andamio no lo crea la IA"
    Esto es importante. El andamio sale de Spring Initializr, nunca de un asistente. El modelo tiene un corte de conocimiento y te arma el proyecto con versiones viejas. La IA entra después, para specs, diseño y código. Generar el esqueleto es trabajo de la herramienta oficial.

!!! warning "Dos cambios de Spring Boot 4 que te van a morder"
    El starter web ahora se llama `spring-boot-starter-webmvc`, no `-web`. Y Flyway necesita `spring-boot-starter-flyway` más `flyway-database-postgresql`. Si Initializr te los dejó como en Boot 3, los ajustas. El comando `/wire-harness` lo revisa por ti más adelante.

## **2. El `.claude`, el cerebro del flujo**

Acá está la pieza que vuelve esto spec-driven. Copia la carpeta `.claude` a la raíz del proyecto. Agrega el stack: Kotlin, Vaadin y arquitectura en capas.

Esto es lo que trae, y vale la pena que lo abras y lo mires:

```bash
.claude/
 ├── 📁 commands/      # los comandos slash que vas a correr
 │    ├── onboard.md
 │    ├── wire-harness.md
 │    ├── spec.md
 │    ├── spec-review.md
 │    ├── plan.md
 │    ├── build.md
 │    ├── validate.md
 │    └── review.md
 ├── 📁 templates/     # la plantilla de la spec
 ├── 📁 checklists/    # la lista contra la que se revisa la spec
 ├── settings.json     # permisos del agente
 └── README.md         # el orden de los comandos
```

Cada archivo de `commands/` es una receta. Cuando escribes `/spec` en Claude Code, el agente abre `commands/spec.md` y sigue esa receta. No es magia ni un prompt secreto, es un instructivo que tú puedes leer y cambiar.

Y eso es lo bonito: si mañana tu equipo tiene otra convención, abres el archivo del comando y lo ajustas. El proceso vive con el código, no en la cabeza de alguien.

!!! info "Por qué los comandos están afinados a tu stack"
    Cada receta ya sabe que tu stack es Kotlin con Vaadin y arquitectura en capas, no Java con Angular ni hexagonal. Por eso el `/build` te genera una entidad JPA simple y un servicio con su interfaz e implementación, sin value objects ni puertos de por medio.

Abre el proyecto con Claude Code. En la barra de comandos, al escribir `/`, deberías ver aparecer los ocho comandos. Si los ves, el `.claude` quedó bien instalado.

## **3. `/onboard`, clasificar el terreno**

El primer comando inspecciona el proyecto y lo clasifica. No construye nada todavía.

```bash title="Claude Code"
/onboard
```

El agente revisa el `pom.xml` y la estructura, detecta el stack, y escribe dos artefactos:

```bash
.specs/
 ├── _stack.json        # las tecnologias detectadas y sus versiones
 └── _onboarding.md     # el estado base y los huecos a resolver
```

Vas a ver un resumen parecido a este:

```text
Clasificacion: GREENFIELD. Solo el andamio de Spring Boot + Vaadin.

Stack:
  Kotlin 2.3 · Java 25 · Spring Boot 4.1 · Vaadin 25 · PostgreSQL 17 · Maven

Huecos a resolver antes de construir:
  1. Migracion: Flyway por cablear
  2. Tests de integracion: falta Testcontainers
  3. Test de la UI: falta Karibu
  4. Guardrail: falta ArchUnit para el limite api/internal

Siguiente: /wire-harness
```

Con plastilina: `/onboard` es el maestro de obra que llega al lote, mira qué hay y te dice qué falta antes de empezar a construir.

## **4. `/wire-harness`, armar las barandas**

Este comando arma el harness: las dependencias y la configuración que después van a frenar el código que no cumple.

```bash title="Claude Code"
/wire-harness
```

Antes de tocar nada, hace un chequeo previo. Si falta una decisión, se detiene y pregunta en vez de asumir. Por ejemplo, si hay un driver de base pero no se eligió herramienta de migración, te lo pregunta y deja la decisión escrita en un ADR.

Para este stack, deja cableado:

- Flyway con su starter de Boot 4 y el módulo de Postgres.
- El starter web correcto (`webmvc`).
- Testcontainers para los tests de integración.
- Karibu para testear la vista sin navegador.
- ArchUnit para el guardrail del límite api/internal.
- `ddl-auto: validate`, porque Flyway es el dueño del esquema.

!!! success "Checkpoint de la Fase 0"
    Deberías tener: el proyecto que arranca, la carpeta `.claude` con los ocho comandos visibles en Claude Code, y `.specs/` con `_stack.json` y `_onboarding.md`. Las barandas quedaron cableadas. Ya puedes escribir la primera spec.

En la siguiente fase escribimos la especificación con `/spec`.
