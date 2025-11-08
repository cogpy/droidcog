# Now in Android - Technical Architecture Overview

## Executive Summary

**Now in Android** is a production-grade Android application demonstrating Google's recommended architecture patterns. The application follows an offline-first, reactive architecture built on modern Android technologies including Jetpack Compose, Room, Retrofit, Hilt, and WorkManager.

**Primary Function**: Deliver curated Android development news and articles to users, enabling them to follow topics of interest and bookmark content.

**Architecture Pattern**: Clean Architecture with MVVM, Unidirectional Data Flow (UDF), and Repository Pattern

---

## System Architecture Overview

```mermaid
graph TB
    subgraph "UI Layer"
        UI[Jetpack Compose UI]
        VM[ViewModels]
    end
    
    subgraph "Domain Layer"
        UC[Use Cases]
    end
    
    subgraph "Data Layer"
        REPO[Repositories]
        LOCAL[Local Data Sources]
        REMOTE[Remote Data Sources]
    end
    
    subgraph "Local Storage"
        ROOM[(Room Database)]
        DS[(Proto DataStore)]
    end
    
    subgraph "Remote Services"
        API[REST API]
    end
    
    subgraph "Background Processing"
        WM[WorkManager]
        SYNC[Sync Workers]
    end
    
    UI --> VM
    VM --> UC
    UC --> REPO
    REPO --> LOCAL
    REPO --> REMOTE
    LOCAL --> ROOM
    LOCAL --> DS
    REMOTE --> API
    WM --> SYNC
    SYNC --> REPO
    
    style UI fill:#e1f5fe
    style VM fill:#b3e5fc
    style UC fill:#81d4fa
    style REPO fill:#4fc3f7
    style LOCAL fill:#29b6f6
    style REMOTE fill:#03a9f4
    style ROOM fill:#039be5
    style DS fill:#0288d1
    style API fill:#0277bd
    style WM fill:#01579b
    style SYNC fill:#01579b
```

---

## Layer-by-Layer Architecture

### 1. UI Layer (Presentation)

**Technology Stack**: Jetpack Compose, Material 3, Navigation Compose

**Components**:
- **Composable Functions**: Declarative UI components
- **ViewModels**: State holders exposing UI state streams
- **UI State Models**: Immutable data classes representing screen state

**Responsibilities**:
- Render UI based on state
- Capture user interactions
- Navigate between screens
- Handle configuration changes

**Key Patterns**:
- Unidirectional Data Flow (UDF)
- State hoisting
- Single Activity architecture

```mermaid
graph LR
    User((User)) --> Compose[Composable UI]
    Compose --> VM[ViewModel]
    VM --> |UI State Stream| Compose
    VM --> UC[Use Cases]
    
    style Compose fill:#e1f5fe
    style VM fill:#b3e5fc
```

### 2. Domain Layer (Business Logic)

**Technology Stack**: Kotlin Coroutines, Flow

**Components**:
- **Use Cases**: Single-responsibility business logic operations

**Example Use Cases**:
- `GetFollowableTopicsUseCase`: Combines topic data with user follow status
- `GetUserNewsResourcesUseCase`: Merges news resources with user bookmark/view state
- `GetSearchContentsUseCase`: Provides search functionality

**Responsibilities**:
- Combine data from multiple repositories
- Transform data for UI consumption
- Implement complex business rules

**Pattern**: Each use case has a single `invoke()` operator function

### 3. Data Layer (Data Management)

**Technology Stack**: Room, Proto DataStore, Retrofit, OkHttp

**Components**:

#### 3.1 Repositories (Interfaces)

- **NewsRepository**: Manages news resource data
- **TopicsRepository**: Manages topic data
- **UserDataRepository**: Manages user preferences and state
- **SearchContentsRepository**: Manages search functionality
- **RecentSearchRepository**: Manages search history

#### 3.2 Repository Implementations

All repositories follow the **Offline-First** pattern:

```mermaid
sequenceDiagram
    participant Client
    participant Repo as Repository
    participant Local as Local Storage
    participant Remote as Remote API
    participant Sync as Sync Worker
    
    Client->>Repo: Request Data
    Repo->>Local: Read from Local
    Local-->>Repo: Return Cached Data
    Repo-->>Client: Emit Data Stream
    
    Note over Sync: Background Process
    Sync->>Remote: Fetch Updates
    Remote-->>Sync: Return New Data
    Sync->>Local: Update Local Storage
    Local-->>Repo: Emit Updated Data
    Repo-->>Client: Emit Updated Data
```

**Key Principles**:
1. **Single Source of Truth**: Local database is the source of truth
2. **Reactive Streams**: Data exposed as Kotlin Flows
3. **Background Sync**: WorkManager handles periodic synchronization
4. **Error Resilience**: Exponential backoff for sync failures

#### 3.3 Data Sources

**Local Data Sources**:

| Data Source | Technology | Purpose |
|-------------|-----------|---------|
| `NiaDatabase` | Room (SQLite) | Relational data (news, topics) |
| `NiaPreferencesDataSource` | Proto DataStore | User preferences and state |
| Full-Text Search | Room FTS | Search functionality |

**Remote Data Sources**:

| Data Source | Technology | Purpose |
|-------------|-----------|---------|
| `NiaNetworkDataSource` | Retrofit + OkHttp | REST API client |
| Network API | JSON over HTTPS | News and topic content |

---

## Data Model Architecture

### Core Entities

```mermaid
erDiagram
    NewsResourceEntity ||--o{ NewsResourceTopicCrossRef : "has"
    TopicEntity ||--o{ NewsResourceTopicCrossRef : "belongs to"
    
    NewsResourceEntity {
        string id PK
        string title
        string content
        string url
        string headerImageUrl
        instant publishDate
        string type
    }
    
    TopicEntity {
        string id PK
        string name
        string shortDescription
        string longDescription
        string url
        string imageUrl
    }
    
    NewsResourceTopicCrossRef {
        string newsResourceId FK
        string topicId FK
    }
    
    UserData {
        set bookmarkedNewsResources
        set viewedNewsResources
        set followedTopics
        themeBrand themeBrand
        darkThemeConfig darkThemeConfig
        boolean useDynamicColor
        boolean shouldHideOnboarding
    }
```

### Entity Relationships

**Many-to-Many Relationship**:
- NewsResource ↔ Topic (via NewsResourceTopicCrossRef)
- Foreign key constraints with CASCADE delete

**User State** (Proto DataStore):
- Bookmarked news resource IDs
- Viewed news resource IDs
- Followed topic IDs
- UI preferences (theme, dark mode, etc.)

---

## Data Flow Architecture

### Read Operation Flow

```mermaid
sequenceDiagram
    participant UI as Composable UI
    participant VM as ViewModel
    participant UC as Use Case
    participant Repo as Repository
    participant DAO as Room DAO
    participant DB as SQLite Database
    
    UI->>VM: Observe State
    VM->>UC: Invoke Use Case
    UC->>Repo: Get Data Stream
    Repo->>DAO: Query Data
    DAO->>DB: SELECT
    DB-->>DAO: Result Set
    DAO-->>Repo: Flow<Entity>
    Repo-->>UC: Flow<Model>
    UC-->>VM: Flow<DomainModel>
    VM-->>UI: Flow<UIState>
    
    Note over DB: Data Change Event
    DB-->>DAO: Notify Change
    DAO-->>Repo: Emit New Data
    Repo-->>UC: Transform
    UC-->>VM: Update
    VM-->>UI: Recompose
```

### Write Operation Flow

```mermaid
sequenceDiagram
    participant UI as Composable UI
    participant VM as ViewModel
    participant Repo as Repository
    participant DS as DataStore
    participant DAO as Room DAO
    
    UI->>VM: User Action (e.g., Bookmark)
    VM->>Repo: Update State
    Repo->>DS: Write Preference
    DS-->>Repo: Success
    Repo-->>VM: Complete
    
    Note over DS: Preference Change
    DS->>Repo: Emit New UserData
    Repo-->>VM: Update State
    VM-->>UI: Recompose with New State
```

### Synchronization Flow

```mermaid
sequenceDiagram
    participant App as Application
    participant WM as WorkManager
    participant SW as SyncWorker
    participant Repo as Repository
    participant API as Remote API
    participant DAO as Room DAO
    
    App->>WM: Schedule Periodic Sync
    
    loop Periodic Execution
        WM->>SW: Execute Work
        SW->>Repo: syncWith(changeList)
        Repo->>API: getNewsResources()
        API-->>Repo: NetworkNewsResource[]
        Repo->>API: getTopics()
        API-->>Repo: NetworkTopic[]
        Repo->>DAO: upsert(entities)
        DAO-->>Repo: Success
        Repo-->>SW: Success
        SW-->>WM: Work Complete
    end
    
    Note over SW,DAO: On Failure: Exponential Backoff
```

---

## Module Architecture

### Module Dependency Graph

```mermaid
graph TD
    APP[app]
    
    subgraph "Feature Modules"
        FORYOU[feature:foryou]
        INTERESTS[feature:interests]
        BOOKMARKS[feature:bookmarks]
        TOPIC[feature:topic]
        SEARCH[feature:search]
        SETTINGS[feature:settings]
    end
    
    subgraph "Core Modules"
        DATA[core:data]
        DATABASE[core:database]
        DATASTORE[core:datastore]
        NETWORK[core:network]
        MODEL[core:model]
        DOMAIN[core:domain]
        UI[core:ui]
        COMMON[core:common]
        DESIGNSYS[core:designsystem]
    end
    
    subgraph "Sync Module"
        SYNCWORK[sync:work]
    end
    
    APP --> FORYOU
    APP --> INTERESTS
    APP --> BOOKMARKS
    APP --> TOPIC
    APP --> SEARCH
    APP --> SETTINGS
    APP --> SYNCWORK
    
    FORYOU --> DATA
    INTERESTS --> DATA
    BOOKMARKS --> DATA
    TOPIC --> DATA
    SEARCH --> DATA
    SETTINGS --> DATA
    
    FORYOU --> DOMAIN
    INTERESTS --> DOMAIN
    SEARCH --> DOMAIN
    
    DATA --> DATABASE
    DATA --> DATASTORE
    DATA --> NETWORK
    DATA --> MODEL
    
    SYNCWORK --> DATA
    
    FORYOU --> UI
    INTERESTS --> UI
    BOOKMARKS --> UI
    TOPIC --> UI
    SEARCH --> UI
    SETTINGS --> UI
    
    UI --> DESIGNSYS
    UI --> MODEL
    
    DATABASE --> MODEL
    NETWORK --> MODEL
    DATASTORE --> MODEL
    DOMAIN --> DATA
    
    style APP fill:#ff6b6b
    style FORYOU fill:#4ecdc4
    style INTERESTS fill:#4ecdc4
    style BOOKMARKS fill:#4ecdc4
    style TOPIC fill:#4ecdc4
    style SEARCH fill:#4ecdc4
    style SETTINGS fill:#4ecdc4
    style DATA fill:#ffe66d
    style DATABASE fill:#a8e6cf
    style NETWORK fill:#a8e6cf
    style MODEL fill:#dcedc1
    style DOMAIN fill:#ffd3b6
```

**Module Types**:
1. **App Module**: Main application module with dependency injection setup
2. **Feature Modules**: Self-contained UI features with navigation
3. **Core Modules**: Shared business logic, data access, and UI components
4. **Sync Module**: Background synchronization workers

---

## Integration Architecture

### External API Integration

**API Specification**:
- **Protocol**: REST over HTTPS
- **Format**: JSON
- **Authentication**: None (public API)

**Endpoints**:

| Endpoint | Method | Purpose | Response |
|----------|--------|---------|----------|
| `/topics` | GET | Retrieve topics | `List<NetworkTopic>` |
| `/newsresources` | GET | Retrieve news | `List<NetworkNewsResource>` |
| `/changelists/topics` | GET | Get topic changes | `List<NetworkChangeList>` |
| `/changelists/newsresources` | GET | Get news changes | `List<NetworkChangeList>` |

**Network Model Transformation**:

```mermaid
graph LR
    NET[NetworkNewsResource] --> |map| ENTITY[NewsResourceEntity]
    ENTITY --> |Room| DB[(Database)]
    DB --> |Flow| ENTITY2[NewsResourceEntity]
    ENTITY2 --> |asExternalModel| MODEL[NewsResource]
    MODEL --> |combine with UserData| USER[UserNewsResource]
    
    style NET fill:#e3f2fd
    style ENTITY fill:#bbdefb
    style DB fill:#90caf9
    style MODEL fill:#64b5f6
    style USER fill:#42a5f5
```

### Build Variants & Product Flavors

**Build Types**:
- `debug`: Development builds with logging
- `release`: Optimized production builds
- `benchmark`: Performance testing builds

**Product Flavors**:
- `demo`: Uses local static data (no backend required)
- `prod`: Connects to production backend API

**Active Configurations**:
- `demoDebug`: Primary development configuration
- `demoRelease`: Performance testing
- `prodRelease`: Production builds (published to Play Store)

---

## Dependency Injection Architecture

**Framework**: Hilt (Dagger-based)

**Injection Scopes**:

| Scope | Lifecycle | Purpose |
|-------|-----------|---------|
| `@Singleton` | Application | App-wide single instances |
| `@ActivityRetainedScoped` | Activity (survives config changes) | ViewModels |
| `@ActivityScoped` | Activity | Activity-specific dependencies |

**Key Modules**:

```mermaid
graph TB
    subgraph "Hilt Modules"
        DM[DataModule]
        NM[NetworkModule]
        DBM[DatabaseModule]
        DSM[DataStoreModule]
        DAOM[DaosModule]
    end
    
    DM --> |provides| REPO[Repositories]
    NM --> |provides| NET[Network Client]
    DBM --> |provides| DB[Room Database]
    DSM --> |provides| DS[DataStore]
    DAOM --> |provides| DAO[DAOs]
    
    DB --> DAO
    
    style DM fill:#e1bee7
    style NM fill:#ce93d8
    style DBM fill:#ba68c8
    style DSM fill:#ab47bc
    style DAOM fill:#9c27b0
```

---

## State Management Architecture

### UI State Pattern

**State Container**: ViewModel

**State Representation**: Sealed interfaces/classes

**Example**:

```kotlin
sealed interface NewsFeedUiState {
    data object Loading : NewsFeedUiState
    data class Success(val feed: List<UserNewsResource>) : NewsFeedUiState
}
```

**Benefits**:
- Exhaustive when expressions
- Type-safe state handling
- Clear loading/success/error states

### State Transformation

```mermaid
graph TB
    DS[DataStore UserData] --> UC[Use Case]
    DB[Room NewsResources] --> UC
    UC --> |combine| VM[ViewModel]
    VM --> |map to UI state| UI[UI State]
    
    UI --> COMP[Composable]
    
    USER[User Event] --> COMP
    COMP --> VM
    VM --> REPO[Repository]
    REPO --> DS
    
    style DS fill:#fff9c4
    style DB fill:#fff59d
    style UC fill:#fff176
    style VM fill:#ffee58
    style UI fill:#ffeb3b
    style COMP fill:#fdd835
```

---

## Background Processing Architecture

**Framework**: WorkManager

**Worker Types**:
1. **SyncWorker**: Periodic data synchronization
2. **DelegatingWorker**: Delegates to other workers dynamically

**Sync Strategy**:

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Enqueued: App Launch
    Enqueued --> Running: WorkManager Trigger
    Running --> Success: Sync Complete
    Running --> Failed: Network Error
    Success --> Idle
    Failed --> Retry: Backoff Delay
    Retry --> Running
    
    note right of Failed
        Exponential Backoff:
        1st: 30s
        2nd: 60s
        3rd: 120s
        ...
    end note
```

**Constraints**:
- Network connectivity required
- Battery not low (optional)
- Device idle (optional for non-urgent sync)

---

## Testing Architecture

### Test Doubles Strategy

**Philosophy**: No mocking frameworks

**Approach**: Test implementations of interfaces

**Test Repository Pattern**:

```mermaid
graph LR
    PROD[Production Repository] --> |implements| IFACE[Repository Interface]
    TEST[Test Repository] --> |implements| IFACE
    
    PROD --> |uses| REAL[Real Data Sources]
    TEST --> |uses| MEMORY[In-Memory Data]
    
    VM[ViewModel] --> IFACE
    
    UNITTEST[Unit Tests] --> TEST
    UNITTEST --> VM
    
    INSTRTEST[Instrumented Tests] --> PROD
    INSTRTEST --> |temp storage| TEMPDS[(Temp DataStore)]
    
    style PROD fill:#c8e6c9
    style TEST fill:#a5d6a7
    style UNITTEST fill:#81c784
    style INSTRTEST fill:#66bb6a
```

**Test Types**:

| Test Type | Framework | Scope |
|-----------|-----------|-------|
| Unit Tests | JUnit + Truth + Turbine | ViewModels, Use Cases, Repositories |
| Screenshot Tests | Roborazzi | UI Components |
| Instrumented Tests | AndroidX Test | Full app integration |
| Benchmark Tests | Macrobenchmark | Performance |

---

## Security & Privacy

**Authentication**: Not required (public content)

**Data Protection**:
- User preferences stored locally (Proto DataStore)
- No sensitive data transmitted
- HTTPS for all network calls

**Privacy**:
- No user tracking
- No analytics in demo flavor
- Firebase analytics in prod flavor (opt-in)

---

## Performance Optimizations

### Baseline Profiles
- AOT compilation for critical paths
- Reduced app startup time
- Stored in `app/src/main/baseline-prof.txt`

### Lazy Loading
- Paging for large lists
- Lazy column/grid composables
- On-demand image loading (Coil)

### Database Optimization
- Indexed columns for fast queries
- Full-Text Search for search functionality
- Foreign key constraints for referential integrity

### Network Optimization
- Response caching (OkHttp)
- Conditional requests (ETags)
- Change lists to minimize data transfer

---

## Summary of Architectural Characteristics

| Characteristic | Implementation |
|----------------|----------------|
| **Scalability** | Modular architecture enables parallel development |
| **Maintainability** | Clear separation of concerns across layers |
| **Testability** | Dependency injection + test doubles |
| **Reliability** | Offline-first ensures app works without network |
| **Performance** | Baseline profiles, lazy loading, efficient queries |
| **Security** | HTTPS, local data storage, no sensitive data |
| **Extensibility** | Plugin-based feature modules |
| **Observability** | Structured logging, Firebase analytics (prod) |

---

## Technology Stack Summary

| Layer | Technologies |
|-------|-------------|
| **UI** | Jetpack Compose, Material 3, Navigation Compose, Coil |
| **State Management** | ViewModel, Kotlin Flow, Coroutines |
| **Dependency Injection** | Hilt (Dagger) |
| **Data Persistence** | Room (SQLite), Proto DataStore |
| **Networking** | Retrofit, OkHttp, Kotlin Serialization |
| **Background Processing** | WorkManager |
| **Testing** | JUnit, Truth, Turbine, AndroidX Test, Roborazzi |
| **Build System** | Gradle (Kotlin DSL), Convention Plugins |
| **Performance** | Baseline Profiles, Macrobenchmark |

---

## Key Design Decisions

1. **Offline-First**: Local database is source of truth
   - **Rationale**: Better UX, works without network, reduces latency

2. **Reactive Streams (Flow)**: Data exposed as Kotlin Flows
   - **Rationale**: Automatic UI updates, reactive programming model

3. **Unidirectional Data Flow**: Events down, data up
   - **Rationale**: Predictable state management, easier debugging

4. **Single Activity**: One activity with Compose navigation
   - **Rationale**: Simplified navigation, better transitions, modern pattern

5. **Modularization**: Feature-based and layer-based modules
   - **Rationale**: Parallel development, build time optimization, clear boundaries

6. **No Mocking in Tests**: Test doubles instead of mocks
   - **Rationale**: More realistic tests, less brittle, exercises production code

7. **Proto DataStore**: Structured preferences storage
   - **Rationale**: Type-safe, efficient, supports migrations

---

*This document provides a comprehensive technical overview of the Now in Android application architecture. For formal specifications of the system behavior, see the accompanying Z++ specification files.*
