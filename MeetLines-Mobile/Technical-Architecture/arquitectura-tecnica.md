# Arquitectura Técnica - MeetLines Mobile

## Descripción General

MeetLines Mobile es una aplicación Android nativa construida con **Kotlin** y **Jetpack Compose** que permite a usuarios finales buscar negocios, agendar servicios y gestionar sus citas.

## Stack Tecnológico

| Tecnología | Propósito |
|-----------|----------|
| Kotlin | Lenguaje de programación |
| Jetpack Compose | UI declarativa |
| Jetpack Architecture (MVVM) | Patrón de arquitectura |
| Room Database | Base de datos local |
| Retrofit | Cliente HTTP |
| Hilt | Inyección de dependencias |
| Coroutines | Programación asincrónica |
| Navigation Compose | Navegación |

## Estructura del Proyecto

```
MeetLines-Mobile/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/Gula/MeetLines/
│   │   │   │   ├── ui/                  # Interfaz de usuario
│   │   │   │   │   ├── screens/         # Pantallas principales
│   │   │   │   │   │   ├── HomeScreen.kt
│   │   │   │   │   │   ├── SearchScreen.kt
│   │   │   │   │   │   ├── BookingScreen.kt
│   │   │   │   │   │   ├── AppointmentsScreen.kt
│   │   │   │   │   │   ├── ProfileScreen.kt
│   │   │   │   │   │   └── LoginScreen.kt
│   │   │   │   │   ├── components/      # Componentes reutilizables
│   │   │   │   │   │   ├── ServiceCard.kt
│   │   │   │   │   │   ├── AppointmentItem.kt
│   │   │   │   │   │   └── BottomNavigation.kt
│   │   │   │   │   ├── navigation/      # Navegación
│   │   │   │   │   └── theme/           # Tema y estilos
│   │   │   │   ├── viewmodel/           # ViewModels
│   │   │   │   │   ├── HomeViewModel.kt
│   │   │   │   │   ├── SearchViewModel.kt
│   │   │   │   │   ├── BookingViewModel.kt
│   │   │   │   │   └── AuthViewModel.kt
│   │   │   │   ├── model/               # Modelos de datos
│   │   │   │   │   ├── Project.kt
│   │   │   │   │   ├── Service.kt
│   │   │   │   │   ├── Appointment.kt
│   │   │   │   │   └── User.kt
│   │   │   │   ├── data/                # Capa de datos
│   │   │   │   │   ├── repository/      # Repositorios
│   │   │   │   │   │   ├── ProjectRepository.kt
│   │   │   │   │   │   ├── AppointmentRepository.kt
│   │   │   │   │   │   └── AuthRepository.kt
│   │   │   │   │   ├── local/           # Base de datos local
│   │   │   │   │   │   ├── AppDatabase.kt
│   │   │   │   │   │   └── dao/
│   │   │   │   │   │       ├── AppointmentDao.kt
│   │   │   │   │   │       └── UserDao.kt
│   │   │   │   │   └── remote/          # API remota
│   │   │   │   │       ├── ApiService.kt
│   │   │   │   │       └── interceptor/
│   │   │   │   ├── di/                  # Inyección de dependencias
│   │   │   │   │   ├── NetworkModule.kt
│   │   │   │   │   ├── DatabaseModule.kt
│   │   │   │   │   └── RepositoryModule.kt
│   │   │   │   ├── util/                # Utilidades
│   │   │   │   │   ├── Constants.kt
│   │   │   │   │   ├── DateFormatter.kt
│   │   │   │   │   └── SharedPreferencesManager.kt
│   │   │   │   └── MainActivity.kt
│   │   │   └── res/
│   │   │       ├── drawable/            # Imágenes y vectores
│   │   │       ├── layout/              # Layouts XML (si aplica)
│   │   │       ├── values/
│   │   │       │   ├── strings.xml
│   │   │       │   ├── colors.xml
│   │   │       │   └── styles.xml
│   │   │       └── AndroidManifest.xml
│   │   └── test/
│   │       ├── java/.../                # Tests unitarios
│   │       └── resources/
│   └── build.gradle.kts                 # Configuración Gradle
├── gradle/
│   └── wrapper/
├── settings.gradle.kts
├── build.gradle.kts
├── .gitignore
└── README.md
```

## Arquitectura MVVM

### ViewModel Ejemplo

```kotlin
@HiltViewModel
class SearchViewModel @Inject constructor(
    private val projectRepository: ProjectRepository,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
    
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    private val _searchQuery = MutableStateFlow("")
    val searchQuery: StateFlow<String> = _searchQuery.asStateFlow()
    
    private val _projects = MutableStateFlow<List<Project>>(emptyList())
    val projects: StateFlow<List<Project>> = _projects.asStateFlow()
    
    fun searchProjects(query: String) {
        _searchQuery.value = query
        viewModelScope.launch {
            try {
                _uiState.value = UiState.Loading
                val results = projectRepository.searchProjects(query)
                _projects.value = results
                _uiState.value = UiState.Success
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}

sealed class UiState {
    object Loading : UiState()
    object Success : UiState()
    data class Error(val message: String) : UiState()
}
```

### Screen con Compose

```kotlin
@Composable
fun SearchScreen(
    viewModel: SearchViewModel = hiltViewModel(),
    onProjectClick: (Project) -> Unit
) {
    val searchQuery by viewModel.searchQuery.collectAsStateWithLifecycle()
    val projects by viewModel.projects.collectAsStateWithLifecycle()
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    
    Column(modifier = Modifier.fillMaxSize()) {
        // Search bar
        SearchBar(
            value = searchQuery,
            onValueChange = { viewModel.searchProjects(it) },
            placeholder = "Buscar negocios..."
        )
        
        // Content
        when (uiState) {
            is UiState.Loading -> {
                CircularProgressIndicator(modifier = Modifier.align(Alignment.CenterHorizontally))
            }
            is UiState.Error -> {
                Text("Error: ${(uiState as UiState.Error).message}")
            }
            is UiState.Success -> {
                LazyColumn {
                    items(projects) { project ->
                        ProjectCard(
                            project = project,
                            onClick = { onProjectClick(project) }
                        )
                    }
                }
            }
        }
    }
}
```

## Modelo de Datos

```kotlin
@Entity(tableName = "appointments")
data class Appointment(
    @PrimaryKey val id: String,
    val projectId: String,
    val serviceName: String,
    val businessName: String,
    val scheduledTime: Long,
    val duration: Int,
    val status: AppointmentStatus,
    val createdAt: Long
)

enum class AppointmentStatus {
    PENDING,
    CONFIRMED,
    COMPLETED,
    CANCELLED
}

data class Project(
    val id: String,
    val name: String,
    val description: String,
    val businessType: String,
    val rating: Float,
    val reviewCount: Int,
    val profilePhotoUrl: String,
    val services: List<Service>
)

data class Service(
    val id: String,
    val name: String,
    val description: String,
    val price: Float,
    val duration: Int,
    val capacity: Int
)
```

## Repositorio Ejemplo

```kotlin
@Inject
class AppointmentRepository(
    private val apiService: ApiService,
    private val appointmentDao: AppointmentDao
) {
    
    fun getMyAppointments(): Flow<List<Appointment>> {
        return appointmentDao.getAllAppointments()
    }
    
    suspend fun createAppointment(request: CreateAppointmentRequest): Appointment {
        return withContext(Dispatchers.IO) {
            try {
                val response = apiService.createAppointment(request)
                val appointment = response.toEntity()
                appointmentDao.insert(appointment)
                appointment
            } catch (e: Exception) {
                throw Exception("Failed to create appointment: ${e.message}")
            }
        }
    }
    
    suspend fun cancelAppointment(appointmentId: String) {
        return withContext(Dispatchers.IO) {
            apiService.cancelAppointment(appointmentId)
            appointmentDao.delete(appointmentId)
        }
    }
}
```

## Inyección de Dependencias (Hilt)

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.API_BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .client(httpClient)
            .build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "meetlines_db"
        ).build()
    }
    
    @Provides
    fun provideAppointmentDao(database: AppDatabase): AppointmentDao {
        return database.appointmentDao()
    }
}
```

## Navegación

```kotlin
sealed class Screen(val route: String) {
    object Home : Screen("home")
    object Search : Screen("search")
    object Booking : Screen("booking/{projectId}") {
        fun createRoute(projectId: String) = "booking/$projectId"
    }
    object Appointments : Screen("appointments")
    object Profile : Screen("profile")
    object Login : Screen("login")
}

@Composable
fun Navigation() {
    val navController = rememberNavController()
    
    NavHost(navController = navController, startDestination = Screen.Home.route) {
        composable(Screen.Home.route) {
            HomeScreen(onNavigate = { navController.navigate(it) })
        }
        composable(Screen.Search.route) {
            SearchScreen(onProjectClick = {
                navController.navigate(Screen.Booking.createRoute(it.id))
            })
        }
        composable(Screen.Booking.route) { backStackEntry ->
            val projectId = backStackEntry.arguments?.getString("projectId") ?: ""
            BookingScreen(projectId = projectId)
        }
    }
}
```

## Conclusión

MeetLines Mobile proporciona:

✅ Interfaz moderna con Jetpack Compose  
✅ Arquitectura MVVM limpia y escalable  
✅ Sincronización offline con Room  
✅ Manejo seguro de tokens JWT  
✅ Corrutinas para operaciones asincrónicas  
✅ Inyección de dependencias con Hilt  

## Referencias

- [Android Jetpack Compose](https://developer.android.com/jetpack/compose)
- [Kotlin Coroutines](https://kotlinlang.org/docs/coroutines-overview.html)
- [Room Database](https://developer.android.com/training/data-storage/room)
- [Hilt Dependency Injection](https://developer.android.com/training/dependency-injection/hilt-android)
- [MVVM Architecture](https://developer.android.com/jetpack/guide)
