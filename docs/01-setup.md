# **Fase 0: El andamio y el `.claude`**

## **¿Qué vamos a hacer y por qué?**

Antes de la primera spec, el repo tiene que hablar el mismo idioma que el flujo. Eso significa dos cosas: un andamio limpio y los comandos del flujo instalados. Sin eso, la IA puede generar código, pero no dentro de un proceso.

## **1. El andamio, desde Spring Initializr**

Genera el proyecto base en [**start.spring.io**](https://start.spring.io){:target="_blank"}. Un solo proyecto, no dos. Como Vaadin corre en la misma JVM que el backend, no hay app de frontend aparte.

Configúralo así:

- Project **Maven**, Language **Kotlin**, Spring Boot **4.1.x**, Java **25**, grupo `com.geovannycode`.
- Dependencias: **Vaadin**, **Spring Web**, **Spring Data JPA**, **Validation**, **Flyway Migration**, **PostgreSQL Driver**.

Descárgalo, ábrelo en IntelliJ y arráncalo una vez para confirmar que vive.

!!! warning "El andamio no lo crea la IA"
    El andamio sale de Spring Initializr, nunca de un asistente. El modelo tiene un corte de conocimiento y te arma el proyecto con versiones viejas. La IA entra después, para specs, diseño y código.

!!! warning "Cambios de Spring Boot 4 que te van a morder"
    El starter web ahora se llama `spring-boot-starter-webmvc`, no `-web`. Flyway va con `spring-boot-starter-flyway` más `flyway-database-postgresql`. Y en Boot 4 los starters de test están separados: `spring-boot-starter-webmvc-test`, `spring-boot-starter-data-jpa-test`, `spring-boot-starter-flyway-test`. El comando `/wire-harness` lo revisa por ti.

## **2. El `.claude`, el cerebro del flujo**

Copia la carpeta `.claude` a la raíz del proyecto. Son los comandos que armamos para este flujo, afinados a nuestro stack: Kotlin, Vaadin y arquitectura en capas.

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

En simple: cada archivo de `commands/` es una receta. Cuando escribes `/spec` en Claude Code, el agente abre `commands/spec.md` y sigue esa receta. No es magia ni un prompt secreto, es un instructivo que tú puedes leer y cambiar.

Y eso es lo bonito: si mañana tu equipo tiene otra convención, abres el archivo del comando y lo ajustas. El proceso vive con el código, no en la cabeza de alguien.

!!! info "Por qué los comandos están afinados a tu stack"
    Cada receta sabe que tu stack es Kotlin con Vaadin y arquitectura en capas, no Java con Angular ni hexagonal. Por eso el `/build` te genera una entidad JPA simple y un servicio con su interfaz e implementación, sin value objects ni puertos de por medio.

Abre el proyecto con Claude Code. Al escribir `/`, deberías ver aparecer los ocho comandos. Si los ves, el `.claude` quedó bien instalado.

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
  Kotlin 2.3.21 · Java 25 · Spring Boot 4.1.0 · Vaadin 25.2.0 · PostgreSQL 17

Huecos a resolver antes de construir:
  1. Migracion: Flyway por cablear
  2. Tests de integracion: falta Testcontainers
  3. Test de la UI: falta Karibu (v24)
  4. Guardrail: falta ArchUnit para el limite api/internal
```

En simple: `/onboard` es el maestro de obra que llega al lote, mira qué hay y te dice qué falta antes de empezar a construir.

## **4. `/wire-harness`, armar las barandas**

Este comando arma el harness: las dependencias y la configuración que después van a frenar el código que no cumple.

```bash title="Claude Code"
/wire-harness
```

Antes de tocar nada, hace un chequeo previo. Si falta una decisión, se detiene y pregunta en vez de asumir. Para este stack, deja cableado:

- Flyway con su starter de Boot 4 y el módulo de Postgres.
- El starter web correcto (`webmvc`) y los starters de test de Boot 4.
- Testcontainers para los tests de integración.
- Karibu `karibu-testing-v24` (versión 2.7.0) para testear la vista sin navegador.
- ArchUnit para el guardrail del límite api/internal.
- `ddl-auto: validate`, porque Flyway es el dueño del esquema.

!!! success "Checkpoint de la Fase 0"
    Deberías tener: el proyecto que arranca, la carpeta `.claude` con los ocho comandos visibles en Claude Code, y `.specs/` con `_stack.json` y `_onboarding.md`. Las barandas quedaron cableadas.

En la siguiente fase escribimos la especificación con `/spec`.
