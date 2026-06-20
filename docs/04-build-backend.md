# **Fase 3: El backend (`/build T-001` a `T-003`)**

## **¿Qué vamos a hacer y por qué?**

Ahora construimos, una tarea a la vez. El comando `/build` toma una tarea y la lleva por el ciclo de TDD: rojo, verde, refactor. Tú no escribes el código, lo dirige la spec; tú revisas cada bocado.

Con plastilina: en cada tarea, Claude primero escribe un test que falla (rojo), después el mínimo código para que pase (verde), y al final limpia sin romper nada (refactor). El test es el que manda.

!!! tip "Tu nombre va en el commit"
    La IA genera rápido, pero tú respondes por cada línea. Revisa cada tarea como si la hubieras escrito tú. "La IA lo hizo así" no es una respuesta válida en un code review. Por eso construimos en bocados: un diff chico se revisa de verdad.

## **1. `/build T-001`, la persistencia**

```bash title="Claude Code"
/build T-001
```

El agente pone `active_task: T-001` y `phase: red`, y escribe primero el test de persistencia, contra un Postgres real con Testcontainers. Lo corre, falla, porque todavía no hay entidad. Después genera el mínimo código:

```kotlin title="internal/Customer.kt (generado por /build)"
@Entity
@Table(name = "customer")
class Customer(
    @Column(name = "first_name", nullable = false, length = 100)
    var firstName: String,
    @Column(name = "email", nullable = false, length = 255)
    var email: String,
    // ... resto de campos
    @Id @Column(name = "id")
    var id: UUID = UUID.randomUUID(),
)
```

```kotlin title="internal/CustomerRepository.kt (generado por /build)"
interface CustomerRepository : JpaRepository<Customer, UUID> {
    fun existsByEmailIgnoreCase(email: String): Boolean
}
```

Y la migración de Flyway con el unique constraint del email. Corre el test de nuevo, verde, y marca la tarea `done`.

- **Fíjate en lo que generó**: una entidad JPA simple, sin value objects. `existsByEmailIgnoreCase` resuelve Q-002, el email único sin distinguir mayúsculas. Todo vive en `internal`.

!!! warning "Si el test de Testcontainers se queda colgado"
    Necesita Docker corriendo. Abre Docker Desktop antes. La primera vez se baja la imagen `postgres:17`, tómate un café.

## **2. `/build T-002`, el servicio**

```bash title="Claude Code"
/build T-002
```

Acá se ve el split api/internal en acción. El agente genera la interfaz en `api` y la implementación en `internal`:

```kotlin title="api/CustomerService.kt"
interface CustomerService {
    fun create(request: CustomerRequest): CustomerResponse
}
```

```kotlin title="internal/CustomerServiceImpl.kt"
@Service
class CustomerServiceImpl(private val repository: CustomerRepository) : CustomerService {
    @Transactional
    override fun create(request: CustomerRequest): CustomerResponse {
        val email = request.email.trim().lowercase()
        if (repository.existsByEmailIgnoreCase(email)) {
            throw DuplicateEmailException(email)
        }
        // ... construye el Customer y lo guarda
    }
}
```

El test del servicio usa el repositorio mockeado, sin base de datos. Cubre AC-008 (guarda) y AC-011 (el duplicado lanza la excepción y no guarda).

## **3. `/build T-003`, el HTTP**

```bash title="Claude Code"
/build T-003
```

Genera el controller y el manejo de errores con ProblemDetail (RFC 9457):

```kotlin title="api/CustomerController.kt"
@RestController
@RequestMapping("/api/customers")
class CustomerController(private val customerService: CustomerService) {
    @PostMapping
    fun create(@Valid @RequestBody request: CustomerRequest,
               uriBuilder: UriComponentsBuilder): ResponseEntity<CustomerResponse> {
        val response = customerService.create(request)
        // ... 201 + Location
    }
}
```

El `CustomerExceptionHandler` traduce las fallas a códigos: 409 para el email duplicado, 400 para validación, 500 para lo inesperado. Y va acotado con `assignableTypes`, para no interceptar otros controllers.

El test usa `@WebMvcTest` y `@MockitoBean` y verifica los códigos: 201, 400, 409.

- **Puntos Clave**
    1. **Cada tarea es un ciclo completo de TDD.** Rojo, verde, refactor, y un commit mental por tarea.
    2. **El contrato son los códigos.** Cada respuesta HTTP tiene su test.
    3. **El servicio es el dueño de la regla.** El controller traduce HTTP y delega.

!!! success "Checkpoint de la Fase 3"
    Las tres tareas del backend en `done` en el `.tdd-state.json`. Corre `./mvnw test` y deberías ver verde el repo, el servicio y el controller. El feature ya funciona por REST.

En la siguiente fase construimos la UI en Vaadin, que es la otra puerta al mismo servicio.
