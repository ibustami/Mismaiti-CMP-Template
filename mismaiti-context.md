# Mismaiti Development Context

## Project Identity
- **Project Name**: Mismaiti
- **Base Package**: com.mismaiti
- **Target Platforms**: Android & iOS (Compose Multiplatform)

## Package Structure
```
com.mismaiti
├── [feature_name]
│   ├── presentation
│   │   ├── [FeatureName]Screen.kt
│   │   ├── [FeatureName]ViewModel.kt
│   │   ├── [FeatureName]Intent.kt
│   │   └── [FeatureName]State.kt
│   ├── domain
│   │   ├── model/
│   │   ├── usecase/
│   │   └── repository/
│   └── data
│       ├── repository/
│       ├── remote/
│       │   ├── dto/
│       │   └── [Feature]Service.kt
│       └── local/
│           ├── entity/
│           └── [Feature]Dao.kt
├── core/
│   ├── di/
│   ├── network/
│   ├── database/
│   └── util/
└── navigation/
```

## Technology Stack
### Dependencies Included in Template
- ✅ Ktor Client (networking)
- ✅ Kotlinx Coroutines (async operations)
- ✅ Kotlinx DateTime (date/time handling)
- ✅ Coil (image loading)
- ✅ Compose Navigation (screen navigation)
- ✅ Koin (dependency injection)
- ✅ Room (local database - multiplatform)

## Architecture Patterns

### Clean Architecture Layers
1. **Presentation Layer** (`presentation/`)
    - UI components (Composables)
    - ViewModels
    - MVI pattern (Intent, State, Effect)

2. **Domain Layer** (`domain/`)
    - Business logic
    - Use cases
    - Repository interfaces
    - Domain models

3. **Data Layer** (`data/`)
    - Repository implementations
    - Data sources (Remote/Local)
    - DTOs and mappers

### MVI Pattern Implementation
```kotlin
// Intent: User actions
sealed interface [Feature]Intent {
    data object Load : [Feature]Intent
    data class Submit(val data: String) : [Feature]Intent
    data class Delete(val id: String) : [Feature]Intent
}

// State: UI state
data class [Feature]State(
    val isLoading: Boolean = false,
    val data: List<Item> = emptyList(),
    val error: String? = null,
    val selectedItem: Item? = null
)

// ViewModel
class [Feature]ViewModel(
    private val useCase: [Feature]UseCase
) : ViewModel() {
    private val _state = MutableStateFlow([Feature]State())
    val state: StateFlow<[Feature]State> = _state.asStateFlow()
    
    fun handleIntent(intent: [Feature]Intent) {
        when (intent) {
            is [Feature]Intent.Load -> load()
            is [Feature]Intent.Submit -> submit(intent.data)
            is [Feature]Intent.Delete -> delete(intent.id)
        }
    }
    
    private fun load() {
        viewModelScope.launch {
            _state.update { it.copy(isLoading = true) }
            when (val result = useCase()) {
                is ApiResult.Success -> {
                    _state.update { 
                        it.copy(
                            isLoading = false,
                            data = result.data
                        )
                    }
                }
                is ApiResult.Error -> {
                    _state.update { 
                        it.copy(
                            isLoading = false,
                            error = result.exception.message
                        )
                    }
                }
            }
        }
    }
}
```

## Naming Conventions

### Files
- Screens: `[FeatureName]Screen.kt`
- ViewModels: `[FeatureName]ViewModel.kt`
- Use Cases: `[Action][Feature]UseCase.kt` (e.g., `GetUserProfileUseCase.kt`)
- Repositories: `[Feature]Repository.kt` / `[Feature]RepositoryImpl.kt`
- API Services: `[Feature]Service.kt`
- DTOs: `[Entity]Dto.kt`
- Room Entities: `[Entity]Entity.kt`
- DAOs: `[Entity]Dao.kt`

### Classes & Functions
- Classes: PascalCase (e.g., `UserProfile`)
- Functions: camelCase (e.g., `getUserProfile()`)
- Composables: PascalCase (e.g., `ProfileCard()`)
- Constants: UPPER_SNAKE_CASE (e.g., `MAX_RETRY_COUNT`)

## Code Generation Rules

### 1. Repository Pattern
```kotlin
// Domain layer interface
interface [Feature]Repository {
    suspend fun get[Data](): ApiResult<List<[Entity]>>
    fun observe[Data](): Flow<List<[Entity]>>
    suspend fun get[Entity]ById(id: String): ApiResult<[Entity]>
    suspend fun save[Entity](entity: [Entity]): ApiResult<Unit>
    suspend fun delete[Entity](id: String): ApiResult<Unit>
}

// Data layer implementation with offline support
class [Feature]RepositoryImpl(
    private val remoteDataSource: [Feature]Service,
    private val localDataSource: [Feature]Dao
) : [Feature]Repository {
    override suspend fun get[Data](): ApiResult<List<[Entity]>> {
        return try {
            // Fetch from remote
            val response = remoteDataSource.fetch[Data]()
            val entities = response.map { it.toDomain() }
            
            // Cache in local database
            entities.forEach { localDataSource.insert(it.toEntity()) }
            
            ApiResult.Success(entities)
        } catch (e: Exception) {
            // Fallback to cached data
            val cached = localDataSource.getAll().firstOrNull()
            if (cached?.isNotEmpty() == true) {
                ApiResult.Success(cached.map { it.toDomain() })
            } else {
                ApiResult.Error(e)
            }
        }
    }
    
    override fun observe[Data](): Flow<List<[Entity]>> {
        return localDataSource.getAll()
            .map { list -> list.map { it.toDomain() } }
    }
}
```

### 2. Use Case Pattern
```kotlin
class [Action][Feature]UseCase(
    private val repository: [Feature]Repository
) {
    suspend operator fun invoke([params]): ApiResult<[ReturnType]> {
        return repository.[method]([params])
    }
}

// Example with validation
class CreateSermonUseCase(
    private val repository: SermonRepository
) {
    suspend operator fun invoke(sermon: Sermon): ApiResult<Unit> {
        // Validation
        if (sermon.title.length < 3) {
            return ApiResult.Error(Exception("Title too short"))
        }
        
        return repository.saveSermon(sermon)
    }
}
```

### 3. ViewModel Pattern
```kotlin
class [Feature]ViewModel(
    private val get[Feature]UseCase: Get[Feature]UseCase,
    private val save[Feature]UseCase: Save[Feature]UseCase
) : ViewModel() {
    private val _state = MutableStateFlow([Feature]State())
    val state: StateFlow<[Feature]State> = _state.asStateFlow()
    
    init {
        handleIntent([Feature]Intent.Load)
    }
    
    fun handleIntent(intent: [Feature]Intent) {
        when (intent) {
            is [Feature]Intent.Load -> load()
            is [Feature]Intent.Submit -> submit(intent.data)
            is [Feature]Intent.Delete -> delete(intent.id)
        }
    }
    
    private fun load() {
        viewModelScope.launch {
            _state.update { it.copy(isLoading = true) }
            when (val result = get[Feature]UseCase()) {
                is ApiResult.Success -> {
                    _state.update { 
                        it.copy(
                            isLoading = false,
                            data = result.data,
                            error = null
                        )
                    }
                }
                is ApiResult.Error -> {
                    _state.update { 
                        it.copy(
                            isLoading = false,
                            error = result.exception.message
                        )
                    }
                }
            }
        }
    }
}
```

### 4. Composable Pattern
```kotlin
// Stateless composable
@Composable
fun [Feature]Content(
    state: [Feature]State,
    onIntent: ([Feature]Intent) -> Unit,
    modifier: Modifier = Modifier
) {
    Column(modifier = modifier.fillMaxSize()) {
        when {
            state.isLoading -> {
                CircularProgressIndicator()
            }
            state.error != null -> {
                ErrorMessage(
                    message = state.error,
                    onRetry = { onIntent([Feature]Intent.Load) }
                )
            }
            else -> {
                LazyColumn {
                    items(state.data) { item ->
                        [Feature]Item(
                            item = item,
                            onClick = { onIntent([Feature]Intent.Select(item.id)) }
                        )
                    }
                }
            }
        }
    }
}

// Stateful screen composable
@Composable
fun [Feature]Screen(
    viewModel: [Feature]ViewModel = koinViewModel(),
    onNavigateTo[Destination]: (String) -> Unit = {}
) {
    val state by viewModel.state.collectAsState()
    
    [Feature]Content(
        state = state,
        onIntent = viewModel::handleIntent
    )
}
```

### 5. Navigation Pattern
```kotlin
// navigation/NavGraph.kt
@Composable
fun AppNavGraph(navController: NavHostController) {
    NavHost(
        navController = navController, 
        startDestination = "[feature]_list"
    ) {
        composable("[feature]_list") {
            [Feature]Screen(
                onNavigateToDetail = { id ->
                    navController.navigate("[feature]_detail/$id")
                }
            )
        }
        
        composable(
            route = "[feature]_detail/{id}",
            arguments = listOf(
                navArgument("id") { 
                    type = NavType.StringType 
                }
            )
        ) { backStackEntry ->
            [Feature]DetailScreen(
                id = backStackEntry.arguments?.getString("id") ?: "",
                onNavigateBack = { navController.popBackStack() }
            )
        }
    }
}
```

### 6. Dependency Injection Pattern
```kotlin
// [feature]/di/[Feature]Module.kt
val [feature]Module = module {
    // ViewModel
    viewModel { [Feature]ViewModel(get(), get()) }
    
    // Use Cases
    factory { Get[Feature]UseCase(get()) }
    factory { Save[Feature]UseCase(get()) }
    factory { Delete[Feature]UseCase(get()) }
    
    // Repository
    single<[Feature]Repository> { 
        [Feature]RepositoryImpl(get(), get()) 
    }
    
    // Remote Data Source
    single { [Feature]Service(get()) }
    
    // Local Data Source (if needed)
    single { get<AppDatabase>().[feature]Dao() }
}

// Application initialization
fun initKoin() {
    startKoin {
        modules(
            coreModule,
            networkModule,
            databaseModule,
            [feature]Module,
            // Add more feature modules
        )
    }
}
```

### 7. Room Database Pattern
```kotlin
// core/database/AppDatabase.kt
@Database(
    entities = [
        [Entity]Entity::class,
        // Add more entities
    ],
    version = 1
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun [entity]Dao(): [Entity]Dao
}

// [feature]/data/local/[Entity]Entity.kt
@Entity(tableName = "[entity]")
data class [Entity]Entity(
    @PrimaryKey val id: String,
    @ColumnInfo(name = "name") val name: String,
    @ColumnInfo(name = "description") val description: String?,
    @ColumnInfo(name = "created_at") val createdAt: Long,
    @ColumnInfo(name = "updated_at") val updatedAt: Long
)

// [feature]/data/local/[Entity]Dao.kt
@Dao
interface [Entity]Dao {
    @Query("SELECT * FROM [entity] ORDER BY created_at DESC")
    fun getAll(): Flow<List<[Entity]Entity>>
    
    @Query("SELECT * FROM [entity] WHERE id = :id")
    suspend fun getById(id: String): [Entity]Entity?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(entity: [Entity]Entity)
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(entities: List<[Entity]Entity>)
    
    @Delete
    suspend fun delete(entity: [Entity]Entity)
    
    @Query("DELETE FROM [entity] WHERE id = :id")
    suspend fun deleteById(id: String)
    
    @Query("DELETE FROM [entity]")
    suspend fun deleteAll()
}
```

### 8. API Service Pattern (Ktor)
```kotlin
// [feature]/data/remote/[Feature]Service.kt
class [Feature]Service(private val client: HttpClient) {
    suspend fun fetch[Data](): List<[Entity]Dto> {
        return client.get("${ApiConfig.BASE_URL}/[endpoint]").body()
    }
    
    suspend fun get[Entity]ById(id: String): [Entity]Dto {
        return client.get("${ApiConfig.BASE_URL}/[endpoint]/$id").body()
    }
    
    suspend fun create[Entity](dto: [Entity]Dto): [Entity]Dto {
        return client.post("${ApiConfig.BASE_URL}/[endpoint]") {
            contentType(ContentType.Application.Json)
            setBody(dto)
        }.body()
    }
    
    suspend fun update[Entity](id: String, dto: [Entity]Dto): [Entity]Dto {
        return client.put("${ApiConfig.BASE_URL}/[endpoint]/$id") {
            contentType(ContentType.Application.Json)
            setBody(dto)
        }.body()
    }
    
    suspend fun delete[Entity](id: String) {
        client.delete("${ApiConfig.BASE_URL}/[endpoint]/$id")
    }
}
```

## Core Module Structure

### Network Module
```kotlin
// core/network/HttpClientFactory.kt
object HttpClientFactory {
    fun create(): HttpClient {
        return HttpClient {
            install(ContentNegotiation) {
                json(Json {
                    ignoreUnknownKeys = true
                    isLenient = true
                    prettyPrint = true
                })
            }
            install(Logging) {
                logger = Logger.DEFAULT
                level = LogLevel.ALL
            }
            install(HttpTimeout) {
                requestTimeoutMillis = 30000
                connectTimeoutMillis = 30000
            }
        }
    }
}

// core/network/ApiResult.kt
sealed class ApiResult<out T> {
    data class Success<T>(val data: T) : ApiResult<T>()
    data class Error(val exception: Throwable) : ApiResult<Nothing>()
}
```

### DI Core Module
```kotlin
// core/di/CoreModule.kt
val coreModule = module {
    single { HttpClientFactory.create() }
}

// core/di/DatabaseModule.kt
val databaseModule = module {
    single { get<DatabaseFactory>().createDatabase() }
    single { get<AppDatabase>().[entity]Dao() }
}

// core/di/NetworkModule.kt
val networkModule = module {
    single { get<HttpClient>() }
}
```

## Output Format Requirements
- Always output valid JSON that can be parsed
- Use PascalCase for class names
- Use camelCase for function/variable names
- Follow Kotlin coding conventions
- Include proper imports
- Add proper error handling
- Use coroutines for async operations
- Implement offline-first when applicable
- Add proper logging for debugging

## Feature Development Checklist
For each new feature, generate:
- [ ] Domain models (data classes)
- [ ] Repository interface (domain)
- [ ] Use cases (domain)
- [ ] DTOs and mappers (data/remote/dto)
- [ ] Repository implementation (data/repository)
- [ ] API service (data/remote)
- [ ] Room Entity & DAO (data/local) - if offline support needed
- [ ] Intent sealed interface (presentation)
- [ ] State data class (presentation)
- [ ] ViewModel (presentation)
- [ ] Screen composable (presentation)
- [ ] Component composables (presentation/components)
- [ ] DI module (di)
- [ ] Navigation routes (in NavGraph)
- [ ] Register DAO in DatabaseModule (if local storage used)