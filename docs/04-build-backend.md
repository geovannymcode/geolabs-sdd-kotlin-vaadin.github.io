# **Fase 3: El backend (`/build T-001` a `T-003`)**

## **¿Qué vamos a hacer y por qué?**

Ahora construimos, una tarea a la vez. El comando `/build` toma una tarea y la lleva por el ciclo de TDD: rojo, verde, refactor. Tú no escribes el código, lo dirige la spec; tú revisas cada bocado.

En simple: en cada tarea, Claude primero escribe un test que falla (rojo), después el mínimo código para que pase (verde), y al final limpia sin romper nada (refactor). El test es el que manda.

!!! tip "Tu nombre va en el commit"
    La IA genera rápido, pero tú respondes por cada línea. Revisa cada tarea como si la hubieras escrito tú. "La IA lo hizo así" no es una respuesta válida en un code review. Por eso construimos en bocados: un diff chico se revisa de verdad.

## **1. `/build T-001`, la persistencia**

```bash title="Claude Code"
/build T-001
```

El agente escribe primero el test de persistencia contra un Postgres real, lo corre, falla, y después genera la entidad en `internal`, con los cinco campos del cliente:

```kotlin title="internal/Customer.kt (generado por /build)"
@Entity
@Table(
    name = "customers",
    uniqueConstraints = [UniqueConstraint(name = "uq_customers_email", columnNames = ["email"])]
)
class Customer(
    @Column(nullable = false, length = 255)
    val nombre: String,
    @Column(nullable = false, length = 254)
    val email: String,
    @Column(nullable = false, length = 20)
    val telefono: String,
    @Column(nullable = false, length = 255)
    val apellido: String = "",
    @Column(nullable = false, length = 255)
    val direccion: String = "",
    @Id
    val id: UUID = UUID.randomUUID()
)
```

- **Fíjate**: es una entidad JPA simple, sin value objects. Campos `val`, inmutables. El `id` se autogenera. El plugin `kotlin-jpa` del `pom.xml` le pone por detrás el constructor sin argumentos que JPA exige.

El repositorio, también en `internal`:

```kotlin title="internal/CustomerRepository.kt"
interface CustomerRepository : JpaRepository<Customer, UUID> {
    fun existsByEmailIgnoreCase(email: String): Boolean
}
```

Las migraciones de Flyway. La `V1` es la baseline, y la `V2` crea la tabla con el unique del email:

```sql title="db/migration/V2__create_customers.sql"
CREATE TABLE customers (
    id        UUID         NOT NULL,
    nombre    VARCHAR(255) NOT NULL,
    apellido  VARCHAR(255) NOT NULL,
    email     VARCHAR(254) NOT NULL,
    telefono  VARCHAR(20)  NOT NULL,
    direccion VARCHAR(255) NOT NULL,
    CONSTRAINT pk_customers PRIMARY KEY (id),
    CONSTRAINT uq_customers_email UNIQUE (email)
);
```

El test corre contra Postgres real con Testcontainers, usando una configuración compartida:

```kotlin title="TestcontainersConfiguration.kt"
@TestConfiguration(proxyBeanMethods = false)
class TestcontainersConfiguration {
    @Bean @ServiceConnection
    fun postgresContainer(): PostgreSQLContainer<*> =
        PostgreSQLContainer(DockerImageName.parse("postgres:17"))
}
```

```kotlin title="internal/CustomerRepositoryTest.kt"
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Import(TestcontainersConfiguration::class)
class CustomerRepositoryTest {
    @Autowired private lateinit var repository: CustomerRepository
    // guarda con todos los campos, busca por email sin distinguir mayusculas,
    // y rechaza el email duplicado a nivel de base (DataIntegrityViolationException)
}
```

!!! warning "Cambio de paquetes de test en Spring Boot 4"
    Ojo con los imports. En Boot 4 se movieron: `@DataJpaTest` viene de `org.springframework.boot.data.jpa.test.autoconfigure`, y `@AutoConfigureTestDatabase` de `org.springframework.boot.jdbc.test.autoconfigure`. Si copias un tutorial de Boot 3, no compilan.

## **2. `/build T-002`, el servicio**

```bash title="Claude Code"
/build T-002
```

La interfaz y los DTOs van en `api`; la implementación, en `internal`. El request lleva los cinco campos con Bean Validation:

```kotlin title="api/CustomerRequest.kt"
data class CustomerRequest(
    @field:NotBlank @field:Size(max = 255)
    val nombre: String,
    @field:NotBlank @field:Email @field:Size(max = 254)
    val email: String,
    @field:NotBlank @field:Size(max = 20)
    @field:Pattern(regexp = "^\\+?[0-9]+$")
    val telefono: String,
    @field:NotBlank @field:Size(max = 255)
    val apellido: String = "",
    @field:NotBlank @field:Size(max = 255)
    val direccion: String = ""
)
```

- **El `@field:`**: en Kotlin la anotación de validación va sobre el campo. Sin ese prefijo, la validación no corre, en silencio. Es el error más común al validar DTOs en Kotlin.
- El `CustomerResponse` es mínimo, solo el id: `data class CustomerResponse(val id: UUID)`.

La implementación normaliza el email, chequea unicidad y guarda con los cinco campos:

```kotlin title="internal/CustomerServiceImpl.kt"
@Service
class CustomerServiceImpl(private val repository: CustomerRepository) : CustomerService {
    override fun create(request: CustomerRequest): CustomerResponse {
        val normalizedEmail = request.email.lowercase()
        if (repository.existsByEmailIgnoreCase(normalizedEmail)) {
            throw DuplicateEmailException(normalizedEmail)
        }
        val customer = Customer(
            nombre = request.nombre, email = normalizedEmail, telefono = request.telefono,
            apellido = request.apellido, direccion = request.direccion
        )
        return CustomerResponse(id = repository.save(customer).id)
    }
}
```

- **`lowercase()`**: normaliza el email antes de comparar y guardar. Resuelve Q-002.
- El test usa mockito-kotlin con el repositorio mockeado, y un `argumentCaptor` para confirmar que el email se guardó en minúsculas. Cubre AC-008 y AC-013.

## **3. `/build T-003`, el HTTP**

```bash title="Claude Code"
/build T-003
```

El controller traduce HTTP al servicio y devuelve 201 con Location:

```kotlin title="api/CustomerController.kt"
@RestController
@RequestMapping("/api/customers")
class CustomerController(private val customerService: CustomerService) {
    @PostMapping
    fun create(@Valid @RequestBody request: CustomerRequest): ResponseEntity<CustomerResponse> {
        val response = customerService.create(request)
        val location = URI.create("/api/customers/${response.id}")
        return ResponseEntity.created(location).body(response)
    }
}
```

Y el manejo de errores con ProblemDetail (RFC 9457), acotado a este controller:

```kotlin title="api/CustomerExceptionHandler.kt"
@RestControllerAdvice(assignableTypes = [CustomerController::class])
class CustomerExceptionHandler {
    @ExceptionHandler(DuplicateEmailException::class)
    fun handleDuplicateEmail(ex: DuplicateEmailException): ResponseEntity<ProblemDetail> {
        val problem = ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, ex.message ?: "Duplicate email")
        problem.title = "Email Already Registered"
        return ResponseEntity.status(HttpStatus.CONFLICT).body(problem)
    }
}
```

- **`assignableTypes`**: el handler solo atiende a este controller. La validación 400 la maneja Spring por defecto, también en formato ProblemDetail.
- El test usa `@WebMvcTest` con `@MockitoBean` y cubre los códigos: 201 con id y Location, varios 400 por validación (uno por cada campo), y 409 por duplicado, verificando el `application/problem+json`.

!!! info "Otro cambio de import de Boot 4"
    `@WebMvcTest` ahora viene de `org.springframework.boot.webmvc.test.autoconfigure`.

- **Puntos Clave**
    1. **Cada tarea es un ciclo completo de TDD.** Rojo, verde, refactor, y un commit por tarea.
    2. **El contrato son los códigos.** 201, 400, 409, cada uno con su test.
    3. **El servicio es el dueño de la regla.** El controller traduce HTTP y delega.

!!! success "Checkpoint de la Fase 3"
    Las tres tareas del backend en `done`. Corre `./mvnw test` y deberías ver verde el repo, el servicio y el controller. El feature ya funciona por REST.

En la siguiente fase construimos la UI en Vaadin, la otra puerta al mismo servicio.
