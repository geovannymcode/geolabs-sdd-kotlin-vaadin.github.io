# **Fase 5: Validar y correr (`/validate`, `/review`)**

## **¿Qué vamos a hacer y por qué?**

El feature está construido. Falta cerrar el loop: correr todo el harness de una, verificar que cada criterio quedó cubierto, hacer la auditoría final, y levantar el proyecto. Esto es lo que separa "tengo código" de "entregué un feature".

## **1. `/validate`, correr todo el harness**

```bash title="Claude Code"
/validate
```

El comando levanta Postgres, corre toda la suite y escribe el reporte. Deberías ver pasar las clases de test:

```text
CustomerRepositoryTest             persistencia contra Postgres real
CustomerServiceImplTest            la logica del servicio, repo mockeado
CustomerControllerTest             contrato HTTP 201/400/409
CustomerCreateViewTest             la vista tiene sus campos y su boton
CustomerCreateViewValidationTest   error inline por cada campo, sin notificacion
CustomerCreateViewSuccessTest      crea, notifica y limpia
MainLayoutTest                     la barra, el menu lateral y la ruta raiz
ArchitectureTest                   el limite api/internal protegido
```

## **2. El guardrail de arquitectura**

El `ArchitectureTest` es el que vuelve real la convención api/internal. Fíjate en su forma, con las anotaciones de ArchUnit:

```kotlin title="ArchitectureTest.kt"
@AnalyzeClasses(
    packages = ["com.geovannycode"],
    importOptions = [ImportOption.DoNotIncludeTests::class]
)
class ArchitectureTest {
    @ArchTest
    val `classes outside internal must not depend on internal classes`: ArchRule =
        noClasses()
            .that().resideOutsideOfPackages("com.geovannycode..internal..")
            .should().dependOnClassesThat().resideInAPackage("com.geovannycode..internal..")
            .allowEmptyShould(true)
}
```

En simple: Kotlin no te pone la baranda del package-private, así que la pones tú con este test. Como es un test, vive en el harness y corre en cada build.

## **3. La trazabilidad**

Cada criterio de aceptación apunta a su test y al código que lo cumple.

| AC | Tests | Código |
|----|-------|--------|
| AC-001 | CustomerCreateViewTest | CustomerCreateView |
| AC-002..006 | CustomerControllerTest, CustomerCreateViewValidationTest | CustomerRequest, CustomerFormModel |
| AC-007 | CustomerCreateViewValidationTest | CustomerCreateView |
| AC-008, AC-009 | CustomerCreateViewSuccessTest, CustomerServiceImplTest | CustomerCreateView, CustomerServiceImpl |
| AC-010 | CustomerControllerTest | CustomerController |
| AC-011 | CustomerControllerTest | CustomerExceptionHandler, CustomerRequest |
| AC-012 | CustomerControllerTest, CustomerServiceImplTest | CustomerExceptionHandler, CustomerServiceImpl |
| AC-013 | CustomerServiceImplTest, CustomerRepositoryTest | CustomerServiceImpl, Customer |

## **4. `/review`, la auditoría final**

```bash title="Claude Code"
/review
```

El agente audita el código contra la spec y devuelve un veredicto por gravedad. Sale **APPROVE**, y de paso te marca cosas menores para pulir, como acotar el handler con `assignableTypes` o no duplicar validación. Eso es lo que aporta el review: atrapar decisiones antes de que se vuelvan deuda.

## **5. Levantar el proyecto**

Y el momento que esperabas: lo pones a correr.

```bash title="terminal"
docker compose up -d
./mvnw spring-boot:run
```

Abre `http://localhost:8080/` o `http://localhost:8080/crear-cliente`, llena los cinco campos, y crea un cliente. Deberías ver la notificación verde y el formulario limpiarse. El REST sigue ahí para clientes externos:

```bash
curl -i -X POST http://localhost:8080/api/customers \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Juan García","apellido":"Pérez","email":"juan@example.com","direccion":"Calle 123 #45-67","telefono":"+573001234567"}'
```

El primero te da `201 Created` con el id y el Location. Repite el mismo email y verás `409 Conflict` con un ProblemDetail.

!!! success "Checkpoint final"
    El proyecto corriendo, el feature funcionando por UI y por REST, y en `.specs/` el rastro completo: spec, review, design, tasks, decisiones, validación y trazabilidad. Todo versionado al lado del código.

## **Cierre**

Construiste un feature completo siguiendo el flujo spec-driven, con Spring Boot en capas y UI en Vaadin. Sin JavaScript. Todo en la JVM. Y sin promptear: respondiste preguntas y diste contexto, y la estructura cargó el peso.

Eso es lo que nivela a un equipo, sin importar cuántos años lleve cada uno escribiendo código.

No es prompting. Es ingeniería.
