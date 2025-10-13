# Mismaiti Development Context

## Project Identity
- **Project Name**: Mismaiti
- **Base Package**: com.mismaiti
- **Target Platforms**: Android & iOS (Compose Multiplatform)

## Package Structure
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
│       └── local/
├── core/
│   ├── di/
│   ├── network/
│   ├── database/
│   └── util/
└── navigation/

## Architecture Patterns

### Clean Architecture Layers
1. **Presentation Layer**: UI components, ViewModels, MVI pattern
2. **Domain Layer**: Business logic, Use cases, Repository interfaces
3. **Data Layer**: Repository implementations, Data sources

### MVI Pattern
- Intent: User actions (sealed interface)
- State: UI state (data class)
- ViewModel: State management with StateFlow

## Technology Stack
- ✅ Ktor Client (networking)
- ✅ Kotlinx Coroutines (async)
- ✅ Kotlinx DateTime (date/time)
- ✅ Coil (image loading)
- ✅ Compose Navigation (navigation)
- ✅ Koin (dependency injection)
- ✅ Room (local database - multiplatform)

## Naming Conventions
- Screens: [FeatureName]Screen.kt
- ViewModels: [FeatureName]ViewModel.kt
- Use Cases: [Action][Feature]UseCase.kt
- Repositories: [Feature]Repository.kt / [Feature]RepositoryImpl.kt

## Code Generation Rules

### Repository Pattern
interface [Feature]Repository {
suspend fun getData(): ApiResult<List<Entity>>
}

### Use Case Pattern
class [Action][Feature]UseCase(private val repository: [Feature]Repository) {
suspend operator fun invoke(): ApiResult<ReturnType>
}

### ViewModel Pattern
class [Feature]ViewModel(private val useCase: UseCase) : ViewModel() {
private val _state = MutableStateFlow([Feature]State())
val state: StateFlow<[Feature]State> = _state.asStateFlow()

    fun handleIntent(intent: [Feature]Intent) { /* ... */ }
}

### DI Pattern
val [feature]Module = module {
viewModel { [Feature]ViewModel(get()) }
factory { [Feature]UseCase(get()) }
single<[Feature]Repository> { [Feature]RepositoryImpl(get()) }
}

### Navigation Pattern
NavHost(navController, startDestination = "feature_list") {
composable("feature_list") { [Feature]Screen(...) }
composable("feature_detail/{id}") { [Feature]DetailScreen(...) }
}

## Output Format
Always output valid JSON that can be parsed.
Always use PascalCase for class names and camelCase for function/variable names.
Always follow Kotlin coding conventions.