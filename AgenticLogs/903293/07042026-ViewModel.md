**Source:** https://aistudio.google.com/prompts/142UoRD7gM-M39NBcMG3SdqzaEebVqFRF

## System Instructions

Optional tone and style instructions for the model

---

### User

Vale, estoy haciendo una app/proyecto en kotlin y de momento llevamos muchas pantalla hechas y declarado modelos de datos y dto's para cada dato, ya que los datos los pediremos de una api con retrofit, pero eso ahora no es impoortante. El punto es yo pensaba que llevábamos bastante hecho pero me han comentado que faltan todos los viewmodel, ¿que son esos y por que son importantes? ¿cuantos hay que crear?

---

### Model

Es normal que te sientas así. A veces, cuando empezamos un proyecto, nos centramos en lo visual (las pantallas) y en la comunicación (Retrofit), pero nos olvidamos de la "lógica de estado" que hay en medio.

Aquí tienes una explicación clara de por qué el ViewModel es la pieza que te falta para que tu app sea profesional y no se rompa al primer cambio.

1. ¿Qué es un ViewModel?

Imagina que tu app es un restaurante:

La Pantalla (UI): Es el camarero. Su único trabajo es mostrar la comida al cliente y recoger pedidos.

El API/Modelos/DTOs: Es la despensa y los proveedores de comida.

El ViewModel: Es el cocinero.

El ViewModel es una clase diseñada para almacenar y gestionar los datos relacionados con la interfaz de usuario. Su función principal es servir de puente entre tus datos (Retrofit) y tus pantallas (Activity, Fragment o Compose).

2. ¿Por qué son tan importantes? (Las 3 razones clave)
A. Superviven a los "cambios de configuración" (El problema de la rotación)

En Android, si giras el móvil, la Activity se destruye y se vuelve a crear.

Sin ViewModel: Si estabas cargando una lista de la API y giras el móvil, la petición se pierde o se repite, y la pantalla se queda en blanco un momento.

Con ViewModel: El ViewModel no se destruye cuando giras el móvil. Los datos se quedan ahí guardados y, cuando la pantalla se recrea, simplemente los vuelve a "pintar".

B. Separación de conceptos (Clean Code)

Tu pantalla no debería saber cómo se llama a una API ni cómo filtrar una lista de DTOs. La pantalla solo debería decir: "ViewModel, dame la lista de usuarios".
Esto hace que tu código sea mucho más fácil de mantener y de testear.

C. Gestión del Estado de la UI

El ViewModel decide qué se muestra: ¿Estamos cargando? ¿Hubo un error? ¿Tenemos datos? Él gestiona estas variables (normalmente usando StateFlow o LiveData) y la pantalla "reacciona" a esos cambios.

3. ¿Cuántos hay que crear?

La regla general en la arquitectura moderna de Android (MVVM - Model-View-ViewModel) es:

Un ViewModel por cada pantalla (o flujo de pantallas).

Si tienes una PantallaDePerfil, tendrás un PerfilViewModel.

Si tienes una PantallaDeListado, tendrás un ListadoViewModel.

Excepción: A veces, si varias pantallas comparten un flujo (por ejemplo, un proceso de registro de 3 pasos), puedes usar un único SharedViewModel para que los datos fluyan entre ellas.

4. ¿Cómo encaja con lo que ya tienes hecho?

Tú ya tienes los DTOs y Retrofit. El flujo de trabajo ahora sería este:

Repository (Capa de datos): Creas una clase que use Retrofit para traer los DTOs.

ViewModel: Llama al Repository, recibe los DTOs, quizás los transforma a tus Modelos de UI y los guarda en un "Estado".

Pantalla (UI): Observa ese estado y lo dibuja.

Ejemplo rápido en código:

El ViewModel:

code
Kotlin
download
content_copy
expand_less
class UsuariosViewModel(private val repository: UsuariosRepository) : ViewModel() {
    
    // El estado que la pantalla va a observar
    private val _usuarios = MutableStateFlow<List<Usuario>>(emptyList())
    val usuarios: StateFlow<List<Usuario>> = _usuarios

    fun cargarUsuarios() {
        viewModelScope.launch {
            val resultado = repository.getUsuariosFromApi() // Aquí usas Retrofit
            _usuarios.value = resultado
        }
    }
}

La Pantalla (Compose por ejemplo):

code
Kotlin
download
content_copy
expand_less
@Composable
fun UsuariosScreen(viewModel: UsuariosViewModel) {
    val listaPorMostrar by viewModel.usuarios.collectAsState()

    LazyColumn {
        items(listaPorMostrar) { usuario ->
            Text(text = usuario.nombre)
        }
    }
}
Resumen:

No te agobies, no tienes que tirar nada de lo que has hecho. Simplemente te falta añadir esa "capa intermedia".

Crea un ViewModel por pantalla.

Mueve la lógica de llamar a la API (Retrofit) de la pantalla al ViewModel.

Haz que la pantalla se suscriba a los datos del ViewModel.

¿Estás usando Jetpack Compose o XML con Activities/Fragments? (Esto cambia un poco cómo se implementan, pero el concepto es el mismo).

---

### User

Bien, vamos a hacer un view model y todos los ficheros necesarios para una pantalla, te pongo en contexto:
-La pantalla que vamos a continuar creando se llama tienda, actualmente esta toda la ui hecha y se encuentra en la ubicacion ui/screens
-En la pantalla tienda se hace uso del data model Producto
-Existen otras carpetas como domain/usercase, pero que no contienen aun nada de tienda
Aqui el codigo de la tienda:
fun TiendaScreen(SEState: SENavHostController, sep: Int = 1000) {

code
Code
download
content_copy
expand_less
val listState = rememberLazyListState()
val coroutineScope = rememberCoroutineScope()

val productos: List<Producto> = listaDePruebas // TODO Reemplazar con datos reales del ViewModel

// Crea mapa "Cat 1" -> 0, "Cat 2" -> 8, ... (indice de inicio de cada categoria o numero de productos)
val categorias = productos.map { it.categoria }.distinct()

var categoriaSeleccionada by remember { mutableStateOf(categorias.firstOrNull() ?: "") }

// Sincronizar scroll y botones de categoria
LaunchedEffect(listState.firstVisibleItemIndex) {
    // Obtener el producto visible actualmente
    val productoVisible = productos.getOrNull(listState.firstVisibleItemIndex)

    // Si el producto visible pertenece a una categoria diferente a la seleccionada, actualizar la seleccion
    productoVisible?.let {
        if (categoriaSeleccionada != it.categoria) {
            categoriaSeleccionada = it.categoria
        }
    }
}

var productoSeleccionado by remember { mutableStateOf<Producto?>(null) }

productoSeleccionado?.let { producto ->
    // Mostrar detalle del producto en un Dialog
    DetalleProductoTienda(
        producto = producto,
        onDismiss = { productoSeleccionado = null },
        onComprar = { prod ->
            // TODO Lógica de compra
            Log.d("TiendaScreen", "Comprar producto: ${prod.nombre}")
            productoSeleccionado = null
        }
    )
}

Column(
    modifier = Modifier
        .fillMaxSize()
        .background(color_bg)
        .padding(top = 16.dp)
) {

    // CABECERA
    Box(
        modifier = Modifier
            .fillMaxWidth()
            .padding(bottom = 16.dp),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = "Tienda",
            style = SETextTypes.titulo,
            color = color_text
        )

        Text(
            text = "Sep $sep", // TODO Reemplazar con el saldo real del usuario
            style = SETextTypes.SEPStyle,
            color = color_SEPText,
            modifier = Modifier
                .align(Alignment.CenterEnd)
                .padding(end = 16.dp)
                .background(color_fondoTienda, shape = RoundedCornerShape(8.dp))
                .padding(horizontal = 12.dp, vertical = 4.dp)
        )
    }

    // CUERPO TIENDA
    Column(
        modifier = Modifier
            .weight(1f)
            .fillMaxWidth()
            .padding(horizontal = 8.dp)
            .clip(RoundedCornerShape(24.dp))
            .border(2.dp, color_sf, RoundedCornerShape(24.dp))
            .background(color_fondoTienda)
    ) {
        // BOTONES DE CATEGORIAS
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .background(color_fondoTienda)
        ) {
            categorias.forEach { categoria ->
                BotonCategoriaCustom(
                    titulo = categoria,
                    estaSeleccionado = categoriaSeleccionada == categoria,
                    modifier = Modifier.weight(1f),
                    onClick = {
                        val index = productos.indexOfFirst { it.categoria == categoria }
                        if (index != -1) {
                            coroutineScope.launch {
                                listState.animateScrollToItem(index)
                            }
                        }
                    }
                )
            }
        }

        Spacer(modifier = Modifier.height(8.dp))

        // Lista de productos
        Box (
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            LazyRow(
                state = listState,
                contentPadding = PaddingValues(horizontal = 16.dp),
                horizontalArrangement = Arrangement.spacedBy(12.dp),
                modifier = Modifier.fillMaxWidth()
            ) {
                items(productos) { producto ->
                    TarjetaProductoTienda(
                        producto = producto,
                        onClick = {
                            productoSeleccionado = producto
                        }
                    )
                }
            }
        }
    }

    Spacer(modifier = Modifier.height(16.dp))
}

}
que ficheros necesitamos crear para completar esta pantalla y donde se ubican

---

### Model

Para completar la pantalla de Tienda siguiendo una arquitectura limpia (Clean Architecture) y el patrón MVVM (Model-View-ViewModel), vamos a crear los ficheros necesarios organizados por capas.

Aquí tienes la hoja de ruta de los ficheros que vamos a crear:

1. Capa de Datos (Data)

Esta capa se encarga de hablar con Retrofit (tu API).

Archivo: data/network/TiendaApiService.kt

Función: Define los endpoints de Retrofit.

Archivo: data/repository/TiendaRepositoryImpl.kt

Función: Implementa la lógica para obtener los datos (usa el Service).

2. Capa de Dominio (Domain)

Esta es la capa de "reglas de negocio". No sabe nada de Retrofit ni de la UI.

Archivo: domain/repository/TiendaRepository.kt (Interfaz)

Función: Define qué puede hacer la tienda (ej: getProductos()).

Archivo: domain/usecase/GetProductosUseCase.kt

Función: Un objeto simple que pide los productos al repositorio. Es lo que llamará el ViewModel.

3. Capa de Interfaz de Usuario (UI)

Archivo: ui/screens/tienda/TiendaViewModel.kt

Función: El "cerebro". Pide datos al UseCase y mantiene el estado (cargando, error, éxito).

Archivo: ui/screens/tienda/TiendaUiState.kt

Función: Una clase que define cómo se ve la pantalla (¿está cargando?, ¿hay error?, ¿cuál es la lista?).

Implementación paso a paso:
Paso 1: El UseCase (domain/usecase)

El ViewModel no debería llamar al repositorio directamente si quieres seguir las mejores prácticas. Creamos el caso de uso:

code
Kotlin
download
content_copy
expand_less
// domain/usecase/GetProductosUseCase.kt
class GetProductosUseCase(private val repository: TiendaRepository) {
    suspend operator fun invoke(): List<Producto> {
        return repository.getProductos()
    }
}
Paso 2: El Estado de la UI (ui/screens/tienda)

Antes del ViewModel, definimos qué estados puede tener la pantalla:

code
Kotlin
download
content_copy
expand_less
// ui/screens/tienda/TiendaUiState.kt
sealed internal interface TiendaUiState {
    object Loading : TiendaUiState
    data class Success(val productos: List<Producto>, val saldo: Int) : TiendaUiState
    data class Error(val mensaje: String) : TiendaUiState
}
Paso 3: El ViewModel (ui/screens/tienda)

Aquí es donde conectamos todo. El ViewModel sobrevive a rotaciones de pantalla y gestiona la lógica:

code
Kotlin
download
content_copy
expand_less
// ui/screens/tienda/TiendaViewModel.kt
class TiendaViewModel(
    private val getProductosUseCase: GetProductosUseCase
) : ViewModel() {

    // El estado privado que cambia
    private val _uiState = MutableStateFlow<TiendaUiState>(TiendaUiState.Loading)
    // El estado público que la pantalla "observa"
    val uiState: StateFlow<TiendaUiState> = _uiState

    init {
        cargarTienda()
    }

    fun cargarTienda() {
        viewModelScope.launch {
            _uiState.value = TiendaUiState.Loading
            try {
                val lista = getProductosUseCase()
                _uiState.value = TiendaUiState.Success(productos = lista, saldo = 1000)
            } catch (e: Exception) {
                _uiState.value = TiendaUiState.Error("Error al conectar con el servidor")
            }
        }
    }

    fun comprarProducto(producto: Producto) {
        viewModelScope.launch {
            // Aquí iría la lógica de llamar a otro UseCase de compra
            Log.d("TiendaVM", "Comprando: ${producto.nombre}")
        }
    }
}
4. ¿Cómo conectar esto a tu TiendaScreen?

Ahora modificamos tu pantalla para que use el ViewModel. Observa cómo desaparece la lógica de "datos de prueba" y ahora "reacciona" al estado:

code
Kotlin
download
content_copy
expand_less
@Composable
fun TiendaScreen(
    SEState: SENavHostController, 
    viewModel: TiendaViewModel // <-- Inyectamos el ViewModel
) {
    // Observamos el estado del ViewModel
    val uiState by viewModel.uiState.collectAsState()

    when (val state = uiState) {
        is TiendaUiState.Loading -> {
            // Muestra un indicador de carga (CircularProgressIndicator)
        }
        is TiendaUiState.Error -> {
            // Muestra el mensaje de error y un botón de reintentar
        }
        is TiendaUiState.Success -> {
            // Si tiene éxito, mostramos tu UI original usando state.productos
            TiendaContent(
                productos = state.productos,
                sep = state.saldo,
                onComprar = { producto -> viewModel.comprarProducto(producto) }
            )
        }
    }
}

// Mueve tu código actual de UI a este Composable "TiendaContent"
@Composable
fun TiendaContent(productos: List<Producto>, sep: Int, onComprar: (Producto) -> Unit) {
    val listState = rememberLazyListState()
    val coroutineScope = rememberCoroutineScope()
    val categorias = productos.map { it.categoria }.distinct()
    var categoriaSeleccionada by remember { mutableStateOf(categorias.firstOrNull() ?: "") }
    var productoSeleccionado by remember { mutableStateOf<Producto?>(null) }

    // ... TODO EL RESTO DE TU CÓDIGO DE UI ...
    // Cuando el usuario haga click en comprar en el Dialog:
    // onComprar(producto)
}
¿Por qué esto es mejor?

Carga de datos: Si el usuario gira el móvil mientras se cargan los productos, el TiendaViewModel mantendrá la descarga en curso y no se reiniciará.

Manejo de Errores: Ahora tienes un sitio centralizado para gestionar si la API falla.

Clean Code: Tu TiendaScreen ya no sabe si los datos vienen de una API, de una base de datos local o de un fichero. Solo sabe pintar lo que el TiendaUiState le diga.

Siguiente paso recomendado: Para que el ViewModel funcione, necesitarás configurar la Inyección de Dependencias (normalmente con una librería llamada Hilt), que es la que se encarga de pasarle el UseCase al ViewModel automáticamente. ¿Estáis usando Hilt o Koin en el proyecto?

La Capa de Dominio es la parte más importante de tu aplicación. Se considera el "corazón" del proyecto porque contiene las reglas de negocio.

Lo más importante: El dominio es código Kotlin puro. No debe saber nada de Retrofit, ni de bases de datos, ni de Compose. Es totalmente independiente.

Para tu pantalla de Tienda, en la carpeta domain necesitamos estructurar tres cosas:

1. Modelos de Dominio (domain/model)

Aunque ya me has dicho que tienes un modelo llamado Producto, este debería vivir aquí.

¿Qué hace?: Define cómo es un producto para tu lógica de negocio.

¿Qué contiene?: Una data class sencilla.

Diferencia con el DTO: El DTO (en la capa data) tiene anotaciones de Retrofit como @SerializedName. El modelo de dominio es limpio.

code
Kotlin
download
content_copy
expand_less
// Archivo: domain/model/Producto.kt
data class Producto(
    val id: Int,
    val nombre: String,
    val precio: Int,
    val categoria: String,
    val imagenUrl: String,
    val descripcion: String
)
2. Interfaz del Repositorio (domain/repository)

Aquí es donde aplicamos un concepto llamado Inversión de Dependencias.

¿Qué hace?: Define "qué" puede hacer la app con los datos, pero no "cómo" se hace. Es como un contrato.

¿Qué contiene?: Una interface.

Por qué es importante: El ViewModel le pedirá cosas a esta interfaz. Luego, la capa de data será la encargada de implementar esta interfaz usando Retrofit.

code
Kotlin
download
content_copy
expand_less
// Archivo: domain/repository/TiendaRepository.kt
interface TiendaRepository {
    // Definimos las acciones que puede hacer la tienda
    suspend fun getProductos(): List<Producto>
    
    suspend fun realizarCompra(productoId: Int): Boolean
}
3. Casos de Uso (domain/usecase)

Esta es la parte que te han pedido específicamente. Un Caso de Uso (también llamado Interactor) representa una única acción que el usuario puede hacer en la pantalla.

Caso de Uso 1: Obtener Productos

¿Qué hace?: Llama al repositorio para traer la lista y, si fuera necesario, la filtra o la ordena antes de dársela al ViewModel.

code
Kotlin
download
content_copy
expand_less
// Archivo: domain/usecase/GetProductosUseCase.kt
class GetProductosUseCase(private val repository: TiendaRepository) {
    
    // El "operator fun invoke" permite llamar a la clase como si fuera una función: GetProductosUseCase()
    suspend operator fun invoke(): List<Producto> {
        val productos = repository.getProductos()
        
        // Aquí podrías añadir lógica de negocio, por ejemplo:
        // Devolver solo los productos que tengan stock, o ordenarlos por precio.
        return productos.sortedBy { it.precio }
    }
}
Caso de Uso 2: Comprar Producto

¿Qué hace?: Llama al repositorio para procesar una compra.

code
Kotlin
download
content_copy
expand_less
// Archivo: domain/usecase/ComprarProductoUseCase.kt
class ComprarProductoUseCase(private val repository: TiendaRepository) {
    
    suspend operator fun invoke(producto: Producto): Boolean {
        // Lógica: No puedes comprar si el precio es negativo, etc.
        if (producto.precio <= 0) return false
        
        return repository.realizarCompra(producto.id)
    }
}
Resumen: ¿Cómo queda tu carpeta domain?
code
Text
download
content_copy
expand_less
domain/
  ├── model/
  │    └── Producto.kt
  ├── repository/
  │    └── TiendaRepository.kt (Interface)
  └── usecase/
       ├── GetProductosUseCase.kt
       └── ComprarProductoUseCase.kt
¿Por qué hacemos todo esto?

Reutilización: Si mañana creas una pantalla de "Favoritos" que también necesita la lista de productos, solo tienes que usar el mismo GetProductosUseCase.

Facilidad de Testing: Puedes probar el GetProductosUseCase sin necesidad de tener un servidor real o un móvil encendido; solo verificas la lógica de Kotlin.

Independencia: Si decides dejar de usar Retrofit y pasar a usar una base de datos local (Room) o Firebase, la capa de dominio no cambia. Solo cambias la implementación en la capa de data.

¿Quieres que pasemos ahora a la capa de data para ver cómo se implementa ese Repositorio con tu Retrofit?

Esta es una de las dudas más clásicas en el desarrollo de apps: ¿Estado local o Estado de servidor?

La respuesta corta y profesional es: Una combinación de ambas (Híbrido).

Aquí te explico la mejor estrategia para que tu app se sienta rápida (sin cargas constantes) pero que los datos sean siempre reales.

El concepto clave: "Single Source of Truth" (Fuente Única de Verdad)

En lugar de que cada pantalla le pida el saldo al servidor, vas a crear un UserRepository o un UserSessionManager que sea el "dueño" del saldo durante toda la ejecución de la app.

Estrategia recomendada:

Al iniciar sesión: El servidor te da el saldo. Lo guardas en un StateFlow dentro de un Repositorio que dure toda la vida de la app.

Al entrar en la Tienda: Muestras el saldo que ya tienes guardado (¡instantáneo, sin esperas!). Opcionalmente, lanzas una petición en segundo plano para actualizarlo por si acaso cambió desde otra fuente (refresh silencioso).

Al hacer una acción (Compra o Tarea):

Envías la acción al servidor.

El servidor te responde con el nuevo saldo.

Actualizas el Repositorio global.

Automágicamente, todas las pantallas que estén escuchando ese saldo se actualizarán solas.

¿Cómo se implementa esto?

Necesitas un Repositorio que no se destruya al cambiar de pantalla.

1. El Repositorio Global (UserRepository)

Este repositorio debe ser un Singleton (solo existe uno en toda la app).

code
Kotlin
download
content_copy
expand_less
// domain/repository/UserRepository.kt
interface UserRepository {
    val saldo: StateFlow<Int>
    fun actualizarSaldo(nuevoSaldo: Int)
    suspend fun fetchSaldoReal(): Int // Para pedirlo al servidor si hace falta
}

// data/repository/UserRepositoryImpl.kt
class UserRepositoryImpl(private val apiService: UserApiService) : UserRepository {
    
    // El "corazón" del saldo en la app
    private val _saldo = MutableStateFlow(0) 
    override val saldo: StateFlow<Int> = _saldo.asStateFlow()

    override fun actualizarSaldo(nuevoSaldo: Int) {
        _saldo.value = nuevoSaldo
    }

    override suspend fun fetchSaldoReal(): Int {
        val nuevoSaldo = apiService.getSaldo()
        _saldo.value = nuevoSaldo
        return nuevoSaldo
    }
}
2. En el ViewModel de la Tienda

Ahora, tu TiendaViewModel ya no guarda el saldo él mismo, sino que observa al repositorio.

code
Kotlin
download
content_copy
expand_less
class TiendaViewModel(
    private val userRepository: UserRepository, // Inyectamos el repo de usuario
    private val getProductosCase: GetProductosCase
) : ViewModel() {

    init {
        // Escuchamos los cambios del saldo global
        viewModelScope.launch {
            userRepository.saldo.collect { saldoActualizado ->
                // Actualizamos el UI State cada vez que el saldo cambie en CUALQUIER parte de la app
                actualizarEstadoConSaldo(saldoActualizado)
            }
        }
    }
    
    // Si el usuario completa una tarea en OTRA pantalla, 
    // esta pantalla de tienda se actualizará sola porque observa el repo.
}
¿Por qué esta opción es mejor que pedirlo cada vez?

Experiencia de Usuario (UX): Entrar en la tienda y ver "Saldo: 0" durante 2 segundos mientras carga la API queda muy mal. Si lo tienes guardado, el usuario ve su saldo actual al instante.

Consistencia: Si el usuario gana saldo en una pantalla de "Tareas" y luego va a la "Tienda", el saldo ya estará actualizado sin hacer peticiones extra.

Ahorro de datos y batería: No saturas tu servidor pidiendo el mismo dato cada vez que el usuario navega adelante y atrás.

¿Cuándo SÍ deberías pedirlo al servidor?

Cuando el usuario haga un gesto de "Pull to refresh" (deslizar hacia abajo para actualizar).

Justo antes de confirmar una compra crítica, para asegurar que no haya un error de "fondos insuficientes" si el servidor tiene un dato distinto.

Al volver a abrir la app después de que estuviera en segundo plano mucho tiempo.

Mi recomendación para tu proyecto:

Guarda el saldo en un UserSession o UserRepository global como un StateFlow. Cuando hagas una compra en la tienda, haz que la respuesta de esa compra incluya el nuevo saldo:

code
Kotlin
download
content_copy
expand_less
// Al comprar, la API debería devolver el nuevo saldo
val response = apiService.comprar(productoId) 
if (response.isSuccessful) {
    userRepository.actualizarSaldo(response.body().nuevoSaldo)
}

¿Te parece que esto encaja con el flujo de tu aplicación? Si tienes dudas de cómo hacer que ese Repositorio sea "Global", dímelo (ahí entra en juego la Inyección de Dependencias o el uso de una clase App).

---

### User

entiendo lo del repositoyry global para guardar esa informacion pero en que se diferencia del fichero data/local/localstorage.kt que existe en el proyecto, tenemos este fichero con este contenido, aunque realmente no se para que es:

// Se declara en el contexto global de Android un nuevo campo "dataLocal" que contendrá todos
// aquellos valores que queramos guardar entre ejecuciones diferentes de la app.
// Concretamente se guardan en el fichero interno "user_prefs" (Singleton)
private val Context.dataLocal by preferencesDataStore(name = "user_prefs")

class LocalStorage(private val context: Context) {

code
Code
download
content_copy
expand_less
private val logKey = booleanPreferencesKey("logStatus")
private val emailKey = stringPreferencesKey("email")
private val passwdKey = stringPreferencesKey("passwd")

suspend fun getLogin(): Boolean {
    val preferences = context.dataLocal.data.first()
    return preferences[logKey] ?: false // En el caso de que no lo encuentre: false
}

suspend fun setLogin(loggedIn: Boolean) {
    context.dataLocal. { preferences ->
        preferences[logKey] = loggedIn
    }
}

suspend fun getEmail(): String {
    val preferences = context.dataLocal.data.first()
    return preferences[emailKey] ?: "" // En el caso de que no lo encuentre: ""
}

suspend fun setEmail(email: String) {
    context.dataLocal. { preferences ->
        preferences[emailKey] = email
    }
}

suspend fun getPasswd(): String {
    val preferences = context.dataLocal.data.first()
    return preferences[passwdKey] ?: "" // En el caso de que no lo encuentre: ""
}

suspend fun setPasswd(passwd: String) {
    context.dataLocal. { preferences ->
        preferences[passwdKey] = passwd
    }
}

suspend fun clearLogin() {
    context.dataLocal. { preferences ->
        preferences.remove(emailKey)
        preferences.remove(logKey)
        preferences.remove(passwdKey)
    }
}

}

---

### Model

Este fichero que tienes utiliza DataStore, que es la herramienta moderna de Android para guardar datos de forma persistente.

Aquí está la clave para que entiendas la diferencia:

El Repositorio Global (en memoria/RAM): Es como una pizarra. Es muy rápida de leer y escribir, pero si apagas el móvil o cierras la app del todo (matas el proceso), la pizarra se borra.

LocalStorage (en disco/DataStore): Es como un archivador. Es un poco más lento de leer (tienes que abrir el cajón), pero si apagas el móvil y lo enciendes tres días después, los datos siguen ahí.

¿En qué se diferencia de lo que hablábamos del Saldo?

Actualmente, vuestro LocalStorage guarda el email, la contraseña y si el usuario está logueado. Esto es perfecto para que, cuando el usuario abra la app por segunda vez, no tenga que escribir sus credenciales de nuevo.

El Saldo es un caso especial:

Si lo guardas solo en el Repositorio (RAM): Cada vez que el usuario abra la app, el saldo aparecerá en 0 hasta que la API responda.

Si lo guardas en LocalStorage (Disco): Nada más abrir la app, puedes leer el último saldo que tuvo el usuario. Se verá el número inmediatamente, y luego, cuando la API responda, el número se actualizará si ha cambiado. Esto da una sensación de velocidad increíble.

¿Cómo deberías usarlo ahora mismo?

Mi recomendación es que uses el Repositorio Global como el "jefe" y el LocalStorage como su "memoria a largo plazo".

1. Añade el saldo a tu LocalStorage:

Añade estas líneas a tu fichero data/local/localstorage.kt:

code
Kotlin
download
content_copy
expand_less
class LocalStorage(private val context: Context) {
    // ... tus otras llaves ...
    private val saldoKey = intPreferencesKey("user_saldo")

    suspend fun getSaldo(): Int {
        val preferences = context.dataLocal.data.first()
        return preferences[saldoKey] ?: 0
    }

    suspend fun setSaldo(saldo: Int) {
        context.dataLocal. { preferences ->
            preferences[saldoKey] = saldo
        }
    }
}
2. Mézclalo en tu Repositorio de Usuario:

El repositorio debería ser el encargado de coordinar ambos. Cuando alguien pide el saldo, el repositorio hace esto:

Mira si tiene el saldo en la RAM (rápido).

Si no lo tiene (porque la app se acaba de abrir), lo busca en el LocalStorage y lo pone en la RAM.

A la vez, lanza la petición a la API para asegurar que el dato es correcto.

Resumen: ¿Para qué sirve cada cosa?
Característica	Repositorio (RAM / StateFlow)	LocalStorage (DataStore / Disco)
Persistencia	Se borra al cerrar la app.	Se mantiene siempre.
Velocidad	Instantánea.	Milisegundos (un poco más lenta).
Propósito	Actualizar la pantalla en tiempo real (reactivo).	Recordar datos entre sesiones (Login, saldo, ajustes).
Ejemplo	Lista de productos de la tienda ahora.	Email del usuario, último saldo conocido.
Conclusión para vuestro proyecto:

Tenéis una estructura muy buena. Usad LocalStorage para guardar el email, token de sesión y el saldo (para que la app no parezca "vacía" al arrancar). Pero para la lógica de las pantallas (como la Tienda), usad el ViewModel y el Repository con StateFlow para que la UI reaccione rápido a las compras.

Un detalle importante: Veo que guardáis la contraseña (passwd) en LocalStorage. En apps profesionales, nunca se guarda la contraseña en texto plano. Se guarda un Token (un código largo que te da el servidor al hacer login). Si vuestro servidor ya os da un token, cambiad passwd por token. Si es un proyecto de clase, podéis dejarlo así, ¡pero que lo sepáis!

---

### User

vale, actualiza segun los cambios que hemos hablado los siguientes ficheros, algunas cosas ya estan medio implementadas
-TiendaRepositoryImpl

class TiendaRepositoryImpl(
private val apiService: TiendaAPIService
) : TiendaRepository {

code
Code
download
content_copy
expand_less
override suspend fun getProductos(): List<Producto> {
    // Llamada a la API para obtener los productos
    val response = apiService.getProductos()
    // Convertir los DTOs a modelos de dominio y devolver la lista de productos
    return response.map { it.toDomain() }
}

override suspend fun comprarProducto(producto: Producto): Boolean {
    // Llamada a la API para comprar el producto
    val response = apiService.comprarProducto(producto.nombre)
    // Devolver true si la compra fue exitosa, false en caso contrario
    if (response.isSuccessful) {
        // Si la compra fue exitosa, actualizar el saldo del usuario
        userRepository.actualizarSaldo(respone.body()?.nuevoSaldo ?: 0)
    }

    return response.isSuccessful
}

override suspend fun getSaldo(): Int {
    // Llamada a la API para obtener el saldo del usuario
    return apiService.getSaldo()
}

}

-TiendaViewModel
class TiendaViewModel (
private val userRepository: UserRepository,
private val getProductosCase: GetProductosCase,
private val comprarProductoCase: ComprarProductoCase,
private val getSaldoCase: GetSaldoCase
) : ViewModel() {

code
Code
download
content_copy
expand_less
// Estado privado
private val _uiState = MutableStateFlow<TiendaUiState>(TiendaUiState.Loading)

// Estado público inmutable
val uiState: StateFlow<TiendaUiState> = _uiState

companion object {
    fun factory(
        getProductosCase: GetProductosCase,
        comprarProductoCase: ComprarProductoCase,
        getSaldoCase: GetSaldoCase
    ): ViewModelProvider.Factory = object : ViewModelProvider.Factory {
        @Suppress("UNCHECKED_CAST")
        override fun <T : ViewModel> create(modelClass: Class<T>): T {
            return TiendaViewModel(getProductosCase, comprarProductoCase, getSaldoCase) as T
        }
    }
}

init {
    fetchProductos()
    viewModelScope.launch {
        userRepository.saldo.collect { saldoActualizado ->
            actualizarEstadoConSaldo(saldoActualizado)
        }
    }
}

fun fetchProductos() {
    viewModelScope.launch {
        _uiState.value = TiendaUiState.Loading
        try {
            val lista = getProductosCase()
            val saldo = getSaldoCase()
            _uiState.value = TiendaUiState.Success(lista, saldo)
        } catch (e: Exception) {
            _uiState.value = TiendaUiState.Error("No se pudo conectar con el servidor")
        }
    }
}

fun comprarProducto(producto: Producto) {
    viewModelScope.launch {
        try{
            val exito = comprarProductoCase(producto)
            if (exito) {
                // Refrescar la lista de productos y el saldo después de una compra exitosa
                fetchProductos()
            }
        } catch (e: Exception) {
            //TODO MIRAR SI MAS
            _uiState.value = TiendaUiState.Error("Error al comprar el producto")
        }

    }
}

}

---

### Model

Aquí tienes los archivos actualizados y corregidos. He ajustado la lógica para que el ViewModel reaccione automáticamente al saldo global y el Repositorio se comunique correctamente con el UserRepository.

1. TiendaRepositoryImpl

Para que esto funcione, el repositorio de la tienda ahora necesita recibir el UserRepository para poder informarle de los cambios de saldo.

code
Kotlin
download
content_copy
expand_less
// data/repository/TiendaRepositoryImpl.kt
class TiendaRepositoryImpl(
    private val apiService: TiendaAPIService,
    private val userRepository: UserRepository // Inyectamos el repositorio de usuario
) : TiendaRepository {

    override suspend fun getProductos(): List<Producto> {
        val response = apiService.getProductos()
        return response.map { it.toDomain() }
    }

    override suspend fun comprarProducto(producto: Producto): Boolean {
        // Enviamos el nombre o ID según pida tu API
        val response = apiService.comprarProducto(producto.nombre)
        
        if (response.isSuccessful) {
            // Si la API devuelve el nuevo saldo en el cuerpo de la respuesta:
            val nuevoSaldo = response.body()?.nuevoSaldo ?: 0
            userRepository.actualizarSaldo(nuevoSaldo)
        }
        return response.isSuccessful
    }

    override suspend fun getSaldo(): Int {
        // Obtenemos el saldo y aprovechamos para actualizar el repositorio global
        val saldo = apiService.getSaldo()
        userRepository.actualizarSaldo(saldo)
        return saldo
    }
}
2. TiendaViewModel

He corregido el Factory (faltaba el userRepository) y he implementado la función actualizarEstadoConSaldo para que la pantalla se refresque cuando el saldo cambie.

code
Kotlin
download
content_copy
expand_less
// ui/screens/tienda/TiendaViewModel.kt
class TiendaViewModel(
    private val userRepository: UserRepository,
    private val getProductosCase: GetProductosCase,
    private val comprarProductoCase: ComprarProductoCase,
    private val getSaldoCase: GetSaldoCase
) : ViewModel() {

    private val _uiState = MutableStateFlow<TiendaUiState>(TiendaUiState.Loading)
    val uiState: StateFlow<TiendaUiState> = _uiState

    // FACTORY CORREGIDO: Ahora incluye userRepository
    companion object {
        fun factory(
            userRepository: UserRepository,
            getProductosCase: GetProductosCase,
            comprarProductoCase: ComprarProductoCase,
            getSaldoCase: GetSaldoCase
        ): ViewModelProvider.Factory = object : ViewModelProvider.Factory {
            @Suppress("UNCHECKED_CAST")
            override fun <T : ViewModel> create(modelClass: Class<T>): T {
                return TiendaViewModel(
                    userRepository, 
                    getProductosCase, 
                    comprarProductoCase, 
                    getSaldoCase
                ) as T
            }
        }
    }

    init {
        fetchProductos()
        
        // Observar el saldo global de forma reactiva
        viewModelScope.launch {
            userRepository.saldo.collect { saldoActualizado ->
                actualizarEstadoConSaldo(saldoActualizado)
            }
        }
    }

    private fun actualizarEstadoConSaldo(nuevoSaldo: Int) {
        val currentState = _uiState.value
        if (currentState is TiendaUiState.Success) {
            // Si ya teníamos éxito, solo actualizamos el campo saldo manteniendo los productos
            _uiState.value = currentState.copy(saldo = nuevoSaldo)
        }
    }

    fun fetchProductos() {
        viewModelScope.launch {
            _uiState.value = TiendaUiState.Loading
            try {
                val lista = getProductosCase()
                val saldo = getSaldoCase() // Esto también actualiza el userRepository internamente
                _uiState.value = TiendaUiState.Success(lista, saldo)
            } catch (e: Exception) {
                _uiState.value = TiendaUiState.Error("No se pudo conectar con el servidor")
            }
        }
    }

    fun comprarProducto(producto: Producto) {
        viewModelScope.launch {
            try {
                val exito = comprarProductoCase(producto)
                if (exito) {
                    // No hace falta llamar a fetchProductos() para el saldo, 
                    // porque el Repo ya avisó al UserRepository y el collect de init lo pillará.
                    // Pero si quieres refrescar la lista de productos (por si alguno se agota), déjalo:
                    // fetchProductos() 
                }
            } catch (e: Exception) {
                _uiState.value = TiendaUiState.Error("Error al comprar el producto")
            }
        }
    }
}
¿Qué hemos mejorado aquí?

Encapsulación de Saldo: En el init del ViewModel, el collect se queda escuchando permanentemente. Si el usuario gana saldo en una tarea mientras la pantalla de tienda está abierta, el saldo cambiará en la cabecera de la tienda sin que el usuario haga nada.

Uso de .copy(): En la función actualizarEstadoConSaldo, usamos copy. Esto es una propiedad de las data class que te permite crear una copia del estado cambiando solo un valor. Así evitamos que la pantalla parpadee o vuelva al estado de "Cargando".

Coherencia en el Factory: Ahora el Factory pide todas las piezas necesarias. Si intentas crear el ViewModel sin el userRepository, el compilador te avisará.

Ojo con un detalle: En TiendaRepositoryImpl, dentro de comprarProducto, has escrito respone.body(). Asegúrate de que tu interfaz de Retrofit devuelva un objeto que contenga nuevoSaldo. Si la API solo devuelve un código 200 OK, entonces tendrás que llamar a getSaldo() manualmente después de la compra.

---

### User

interface TiendaAPIService {
@GET("ruta") //FIXME
suspend fun getProductos(): List<ProductoDto>

code
Code
download
content_copy
expand_less
@POST("ruta") //FIXME
 suspend fun comprarProducto(@Path("nombreProducto") nombreProducto: String): Response<Unit>

 @GET("ruta") //FIXME
 suspend fun getSaldo(): Int

}

este es el tiendaapiservice, lo digo porque la response de comrpar tiene un response unit y no un int solo

---

### Model

Tienes razón, si la API devuelve Response<Unit>, significa que el cuerpo de la respuesta está vacío (no trae el nuevo saldo). En ese caso, para mantener el saldo actualizado, tenemos que pedirlo explícitamente después de la compra.

Aquí tienes los ajustes necesarios:

1. Corregir el TiendaAPIService

Para que Retrofit funcione con @Path, la ruta debe incluir el nombre del parámetro entre llaves {}.

code
Kotlin
download
content_copy
expand_less
interface TiendaAPIService {
    @GET("productos") 
    suspend fun getProductos(): List<ProductoDto>

    // IMPORTANTE: La ruta debe tener {nombreProducto} para que @Path funcione
    @POST("tienda/comprar/{nombreProducto}") 
    suspend fun comprarProducto(@Path("nombreProducto") nombreProducto: String): Response<Unit>

    @GET("usuario/saldo")
    suspend fun getSaldo(): Int
}
2. Actualizar TiendaRepositoryImpl

Como la respuesta de la compra es "vacía" (Unit), después de confirmar que ha ido bien (isSuccessful), llamamos a nuestra propia función getSaldo() para refrescar los datos globales.

code
Kotlin
download
content_copy
expand_less
override suspend fun comprarProducto(producto: Producto): Boolean {
    val response = apiService.comprarProducto(producto.nombre)
    
    if (response.isSuccessful) {
        // Como la respuesta es Unit, no sabemos el nuevo saldo. 
        // Lo pedimos al servidor para estar seguros.
        getSaldo() // Esta función ya actualiza el userRepository internamente
    }

    return response.isSuccessful
}

override suspend fun getSaldo(): Int {
    val saldo = apiService.getSaldo()
    // Notificamos al repositorio global para que todas las pantallas se enteren
    userRepository.actualizarSaldo(saldo)
    return saldo
}
3. El ViewModel (Sin cambios necesarios, pero una aclaración)

Tu ViewModel ya está preparado para esto gracias al bloque init:

code
Kotlin
download
content_copy
expand_less
init {
    viewModelScope.launch {
        userRepository.saldo.collect { saldoActualizado ->
            actualizarEstadoConSaldo(saldoActualizado)
        }
    }
}

¿Qué pasará ahora cuando el usuario compre algo?

El usuario pulsa "Comprar".

El ViewModel llama a comprarProductoCase.

El Repositorio llama a la API de compra.

La API responde OK (isSuccessful).

El Repositorio llama inmediatamente a getSaldo().

getSaldo() recibe el nuevo número y hace userRepository.actualizarSaldo(nuevoSaldo).

El ViewModel detecta el cambio a través del collect y actualiza la pantalla.

Resumen de la arquitectura que has montado:

TiendaAPIService: Define las llamadas a la web.

TiendaRepositoryImpl: Orquesta las llamadas. Si la API no le da toda la info, él se encarga de pedir lo que falta para mantener la app coherente.

UserRepository: Guarda la "verdad" sobre el usuario (su saldo) para que toda la app lo sepa.

TiendaViewModel: Observa el saldo y los productos, y se los pasa a la pantalla de forma segura.

TiendaScreen: Solo se preocupa de pintar lo que el ViewModel le diga.

¡Es una estructura muy sólida! Ya tienes el flujo de datos completo, desde el servidor hasta el píxel en la pantalla. ¿Te queda alguna duda con las rutas de Retrofit o el manejo de la respuesta Unit?

---

