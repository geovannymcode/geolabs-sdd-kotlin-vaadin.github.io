# **Fase 4: La UI en Vaadin (`/build T-004` a `T-006`)**

## **¿Qué vamos a hacer y por qué?**

Acá está el corazón de la charla. La UI es una clase Kotlin con Vaadin Flow. Y como corre en la misma JVM que el backend, llama al `CustomerService` directo. No hay app de frontend aparte, ni build de JavaScript, ni servicio HTTP del lado del cliente.

El frontend es una casa aparte que le grita al backend por la ventana. Con Vaadin, cocina y comedor están en la misma casa. La vista abre la puerta y le habla al servicio de frente.

## **1. `/build T-004`, el formulario**

```bash title="Claude Code"
/build T-004
```

El agente genera el layout, un modelo para el formulario y la vista con sus cinco campos. El modelo lleva la validación con Bean Validation, gemela de la del `CustomerRequest`:

```kotlin title="ui/CustomerFormModel.kt"
class CustomerFormModel {
    @field:NotBlank @field:Size(max = 100)
    @field:Pattern(regexp = "^[\\p{L} '-]*$")
    var firstName: String = ""
    @field:NotBlank @field:Email @field:Size(max = 255)
    var email: String = ""
    // ... resto
}
```

Esto cubre AC-001, los cinco campos en pantalla.

## **2. `/build T-005`, la validación inline**

```bash title="Claude Code"
/build T-005
```

Genera la vista con el `BeanValidationBinder`, que valida mientras el usuario escribe y prende el botón solo cuando todo es válido:

```kotlin title="ui/CustomerCreateView.kt"
@Route(value = "create-customer", layout = MainLayout::class)
class CustomerCreateView(private val customerService: CustomerService) : VerticalLayout() {
    private val binder = BeanValidationBinder(CustomerFormModel::class.java)
    init {
        binder.bindInstanceFields(this)
        save.isEnabled = false
        binder.addStatusChangeListener { save.isEnabled = binder.isValid }
        // ...
    }
}
```

El binder es un cable que conecta cada casilla del formulario con un campo del modelo. Cubre AC-002 a AC-007.

## **3. `/build T-006`, conectar al servicio**

```bash title="Claude Code"
/build T-006
```

Acá se cierra el círculo. El agente conecta el botón al `CustomerService`, el mismo del backend, y traduce la respuesta a la pantalla:

```kotlin title="ui/CustomerCreateView.kt (onSave)"
try {
    customerService.create(CustomerRequest(/* ... del modelo */))
    Notification.show("Cliente creado") // exito
    // limpia el form
} catch (ex: DuplicateEmailException) {
    email.isInvalid = true
    email.errorMessage = "Este email ya esta registrado"
}
```

- **Línea clave**: `customerService.create` es el mismo servicio que usa el REST. Cero lógica de negocio duplicada en la UI. Si mañana cambia una regla, cambia en un solo lugar.
- El email duplicado se marca en el campo, no en un cartel suelto. Cubre AC-011 desde la UI.

El test usa Karibu, que instancia la vista y la maneja en la JVM, sin navegador, en milisegundos.

```kotlin title="ui/CustomerCreateViewTest.kt"
@BeforeEach fun setup() = MockVaadin.setup()

@Test
fun `AC-002 el boton arranca apagado y se enciende al ser valido`() {
    val view = CustomerCreateView(FakeService { ok(it) })
    UI.getCurrent().add(view)
    val save = view._get<Button> { text = "Crear cliente" }
    assertThat(save.isEnabled).isFalse()
    // llena los campos y verifica que se prende
}
```

Karibu es un doble de riesgo del navegador. Hace todo lo que haría un usuario, pero en memoria. Por eso es rapidísimo.

!!! warning "Si un `_get` no compila"
    Karibu va al día con Vaadin, y los matchers (`label`, `text`) cambian entre versiones. Confirma que tu Karibu corresponde a tu Vaadin 25. Es el test más sensible a versión del proyecto.

!!! success "Checkpoint de la Fase 4"
    Las seis tareas en `done`. Tienes backend y UI. Falta correr todo junto y cerrar el loop.

En la última fase validamos y levantamos el proyecto.
