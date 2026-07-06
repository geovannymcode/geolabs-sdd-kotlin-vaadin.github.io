# **Fase 4: La UI en Vaadin (`/build T-004` a `T-006`)**

## **¿Qué vamos a hacer y por qué?**

Acá está el corazón de la charla. La UI es una clase Kotlin con Vaadin Flow. Y como corre en la misma JVM que el backend, llama al `CustomerService` directo. No hay app de frontend aparte, ni build de JavaScript, ni servicio HTTP del lado del cliente.

En simple: en el ejemplo original con Angular, el frontend es una casa aparte que le grita al backend por la ventana. Con Vaadin, cocina y comedor están en la misma casa. La vista abre la puerta y le habla al servicio de frente.

## **1. `/build T-004`, el layout y el formulario**

```bash title="Claude Code"
/build T-004
```

El agente genera el layout con su barra, su título y un menú lateral, la configuración del shell con el tema Lumo, y el modelo del formulario.

```kotlin title="ui/MainLayout.kt"
@Layout
class MainLayout : AppLayout() {
    init {
        val toggle = DrawerToggle()
        val title = H1("Gestión de clientes")
        title.style.set("font-size", "1.125rem").set("margin", "0")

        val nav = SideNav()
        nav.addItem(SideNavItem("Crear cliente", CustomerCreateView::class.java, VaadinIcon.USER.create()))

        addToDrawer(Scroller(nav))
        addToNavbar(toggle, title)
    }
}
```

```kotlin title="ui/AppShellConfig.kt"
@Theme(themeClass = Lumo::class)
class AppShellConfig : AppShellConfigurator
```

El modelo del formulario lleva la misma validación que el `CustomerRequest`, con los cinco campos:

```kotlin title="ui/CustomerFormModel.kt"
data class CustomerFormModel(
    @field:NotBlank @field:Size(max = 255) var nombre: String = "",
    @field:NotBlank @field:Size(max = 255) var apellido: String = "",
    @field:NotBlank @field:Email @field:Size(max = 254) var email: String = "",
    @field:NotBlank @field:Size(max = 255) var direccion: String = "",
    @field:NotBlank @field:Size(max = 20)
    @field:Pattern(regexp = "^\\+?[0-9]+$") var telefono: String = ""
)
```

## **2. `/build T-005`, la vista y la validación inline**

```bash title="Claude Code"
/build T-005
```

La vista se registra en una ruta, recibe el servicio por el constructor y conecta cada campo con el modelo a través del binder:

```kotlin title="ui/CustomerCreateView.kt"
@Route(value = "crear-cliente", layout = MainLayout::class)
@RouteAlias(value = "", layout = MainLayout::class)
class CustomerCreateView(
    private val customerService: CustomerService
) : VerticalLayout() {

    internal val nombreField = TextField("Nombre")
    internal val apellidoField = TextField("Apellido")
    internal val emailField = TextField("Email")
    internal val direccionField = TextField("Dirección")
    internal val telefonoField = TextField("Teléfono")
    internal val submitButton = Button("Crear cliente")

    private val binder = BeanValidationBinder(CustomerFormModel::class.java)

    init {
        binder.forField(nombreField).bind("nombre")
        binder.forField(apellidoField).bind("apellido")
        binder.forField(emailField).bind("email")
        binder.forField(direccionField).bind("direccion")
        binder.forField(telefonoField).bind("telefono")
        // ...
        add(nombreField, apellidoField, emailField, direccionField, telefonoField, submitButton)
    }
}
```

- **`@Route("crear-cliente")` más `@RouteAlias("")`**: la vista responde en `/crear-cliente` y también en la raíz.
- **`internal val` en los campos**: los deja visibles para los tests, sin exponerlos afuera del módulo.
- **`binder.forField(...).bind("nombre")`**: en simple, conecta cada casilla con el campo del modelo del mismo nombre. El binder usa las anotaciones del modelo para validar mientras el usuario escribe.

## **3. `/build T-006`, conectar al servicio**

```bash title="Claude Code"
/build T-006
```

El clic valida, llama al servicio, muestra la notificación de éxito y limpia el formulario:

```kotlin title="ui/CustomerCreateView.kt (submit)"
submitButton.addClickListener {
    val model = CustomerFormModel()
    if (binder.writeBeanIfValid(model)) {
        customerService.create(
            CustomerRequest(model.nombre, model.email, model.telefono, model.apellido, model.direccion)
        )
        Notification.show("Cliente creado correctamente", 3000, Notification.Position.TOP_CENTER)
            .addThemeVariants(NotificationVariant.LUMO_SUCCESS)
        binder.readBean(CustomerFormModel())
    }
}
```

- **`writeBeanIfValid`**: si el formulario no es válido, no hace nada y los errores inline ya están a la vista. Cubre AC-007.
- **`customerService.create`**: la UI llama al mismo servicio del backend. Cero lógica de negocio duplicada acá. Cubre AC-008.
- **`binder.readBean(CustomerFormModel())`**: limpia el formulario tras crear. Cubre AC-009.

## **4. Los tests browserless con Karibu**

Karibu instancia la vista y la maneja en la JVM, sin navegador, en milisegundos. El proyecto los organiza por tema:

```kotlin title="ui/CustomerCreateViewTest.kt (los campos existen)"
@Test
fun `view has nombre email telefono fields and submit button`() {
    val view = CustomerCreateView(customerService)
    view._get<TextField> { label = "Nombre" }
    view._get<TextField> { label = "Email" }
    view._get<TextField> { label = "Teléfono" }
    view._get<Button> { text = "Crear cliente" }
}
```

- `CustomerCreateViewValidationTest`: envía con cada campo inválido (nombre, apellido, dirección, email, teléfono) y verifica el error inline, que no hay notificación y que el servicio no se llamó (`verifyNoInteractions`).
- `CustomerCreateViewSuccessTest`: envía válido y verifica que se llama al servicio, aparece la notificación, se autocierra y el formulario se limpia.
- `MainLayoutTest`: verifica la barra con su título "Gestión de clientes", el menú lateral con "Crear cliente", y que la raíz navega a la vista de crear.

En simple: Karibu es un doble de riesgo del navegador. Hace todo lo que haría un usuario, pero en memoria. Por eso es rapidísimo.

!!! warning "Karibu y la versión de Vaadin"
    Este proyecto usa `karibu-testing-v24` en la versión 2.7.0, que corresponde a Vaadin 25. Si algún `_get` no compila, revisa que la versión de Karibu case con tu Vaadin. Es el punto más sensible a versión del proyecto.

!!! success "Checkpoint de la Fase 4"
    Las seis tareas en `done`. Tienes backend y UI. Falta correr todo junto y cerrar el loop.

En la última fase validamos y levantamos el proyecto.
