# **Fase 5: Validar y correr (`/validate`, `/review`)**

## **¿Qué vamos a hacer y por qué?**

El feature está construido. Falta cerrar el loop: correr todo el harness de una, verificar que cada criterio quedó cubierto, hacer la auditoría final, y levantar el proyecto. Esto es lo que separa "tengo código" de "entregué un feature".

## **1. `/validate`, correr todo el harness**

```bash title="Claude Code"
/validate
```

El comando levanta Postgres, corre toda la suite, y escribe el reporte. Deberías ver pasar las cinco clases de test:

```text
CustomerRepositoryIT     persistencia contra Postgres real
CustomerServiceTest      la logica del servicio, repo mockeado
CustomerControllerTest   contrato HTTP 201/400/409
CustomerCreateViewTest   la vista de Vaadin, browserless
ArchitectureTest         el limite api/internal protegido
```

Y deja dos artefactos: `07-validation-report.md` con el resultado, y `07a-traceability.md` con la trazabilidad.

## **2. La trazabilidad, lo más bonito de enseñar**

La tabla de trazabilidad conecta cada criterio de aceptación con su test y con el código que lo cumple.

| AC | Tests | Código de producción |
|----|-------|----------------------|
| AC-001, AC-002 | CustomerCreateViewTest | CustomerCreateView, CustomerFormModel |
| AC-008 | CustomerServiceTest, CustomerRepositoryIT, CustomerControllerTest | Customer, CustomerRepository, CustomerServiceImpl, CustomerController |
| AC-011 | CustomerServiceTest, CustomerControllerTest, CustomerCreateViewTest | CustomerServiceImpl, CustomerExceptionHandler, CustomerCreateView |

Con plastilina: es el hilo que conecta lo que pedimos, lo que probamos y lo que construimos. Si alguien pregunta si la regla del email único está cubierta, no abres el código, miras la tabla.

## **3. `/review`, la auditoría final**

```bash title="Claude Code"
/review
```

El agente audita el código contra la spec y devuelve un veredicto por gravedad. En este feature sale **APPROVE**, con un par de cosas menores que vale la pena atender:

```text
F-R-001 (should-fix): el @RestControllerAdvice debe ir acotado con
  assignableTypes, para no interceptar otros controllers.
F-R-002 (should-fix): no duplicar la validacion entre el controller y el servicio.
```

Eso es lo que aporta el review: no es ceremonia, es atrapar decisiones antes de que se vuelvan deuda.

## **4. Levantar el proyecto**

Y el momento que esperabas: lo pones a correr.

```bash title="terminal"
docker compose up -d
./mvnw spring-boot:run
```

Abre `http://localhost:8080/create-customer`, llena el formulario y crea un cliente. Deberías ver la notificación verde. Repite el mismo email y verás el error en el campo. Y el REST sigue ahí para clientes externos:

```bash
curl -i -X POST http://localhost:8080/api/customers \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Ada","lastName":"Lovelace","email":"ada@example.com"}'
```

El primero te da `201 Created`; el segundo, `409 Conflict`.

!!! success "Checkpoint final"
    El proyecto corriendo, el feature funcionando por UI y por REST, y en `.specs/` el rastro completo: spec, review, design, tasks, decisiones, validación y trazabilidad. Todo versionado al lado del código.

## **Cierre**

Construiste un feature completo siguiendo el flujo spec-driven, con Spring Boot en capas y UI en Vaadin. Sin JavaScript. Todo en la JVM. Y sin promptear: respondiste preguntas y diste contexto, y la estructura cargó el peso.

Eso es lo que nivela a un equipo, sin importar cuántos años lleve cada uno escribiendo código.

No es prompting. Es ingeniería.
