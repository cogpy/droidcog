# Now in Android - Formal Specifications

This directory contains comprehensive technical architecture documentation and formal Z++ specifications for the **Now in Android** application.

## Overview

The formal specifications provide a rigorous mathematical model of the Now in Android application's architecture, data model, system state, operations, and external integrations. These specifications serve as:

- **Reference Documentation**: Complete technical documentation of the application architecture
- **Formal Verification Basis**: Mathematical foundation for verifying system properties
- **Design Contract**: Precise specification of system behavior and invariants
- **Onboarding Resource**: Comprehensive guide for understanding the application

## Contents

### 1. Architecture Overview (`architecture_overview.md`)

**Comprehensive technical architecture documentation with Mermaid diagrams**

Topics covered:
- System architecture overview with component diagrams
- Layer-by-layer architecture breakdown (UI, Domain, Data)
- Data model entity-relationship diagrams
- Data flow diagrams (read operations, write operations, synchronization)
- Module dependency graph
- Integration architecture
- State management patterns
- Background processing architecture
- Dependency injection configuration
- Testing architecture
- Performance optimizations
- Technology stack summary

**Start here** if you want to understand the overall architecture and design decisions.

### 2. Data Model Specification (`data_model.zpp`)

**Z++ formal specification of the data layer**

Components specified:
- **Core Entities**: `Topic`, `NewsResource`, `UserData`, `RecentSearchQuery`
- **Relationship Entities**: `NewsResourceTopicCrossRef` (many-to-many relationship)
- **Derived Entities**: `UserNewsResource`, `FollowableTopic` (computed views)
- **Full-Text Search Entities**: `TopicFtsEntity`, `NewsResourceFtsEntity`
- **Network DTOs**: `NetworkTopic`, `NetworkNewsResource`, `NetworkChangeList`
- **Global Invariants**: Uniqueness, referential integrity, ordering constraints
- **Transformation Functions**: DTO-to-entity mapping, query execution
- **Query Specifications**: Filtering and full-text search operations

**Use this** to understand the data model structure, relationships, and constraints.

### 3. System State Specification (`system_state.zpp`)

**Z++ formal specification of the complete application state**

State components specified:
- **DatabaseState**: Room database persistent storage (SQLite)
- **DataStoreState**: Proto DataStore user preferences and metadata
- **NetworkState**: Runtime network connectivity and synchronization status
- **SyncWorkState**: WorkManager background job state
- **NiaApplicationState**: Complete unified system state with cross-component invariants
- **InitialState**: First-launch state specification
- **State Observations**: Read-only (`ΞNiaApplicationState`) and mutating (`ΔNiaApplicationState`) operations
- **Derived Queries**: `GetAllUserNewsResources`, `GetFollowableTopics`, etc.
- **State Invariants**: Temporal consistency, referential integrity, sync consistency
- **Theorems**: Formal properties guaranteed to hold for all valid states

**Use this** to understand how application state is structured and managed.

### 4. Operations Specification (`operations.zpp`)

**Z++ formal specification of all system operations**

Operation categories:

#### User Data Operations
- `SetTopicFollowed`: Follow/unfollow topics
- `SetNewsResourceBookmarked`: Bookmark/unbookmark news resources
- `SetNewsResourceViewed`: Mark news resources as viewed
- `SetThemePreferences`: Update theme settings
- `CompleteOnboarding`: Mark onboarding as complete

#### Read Operations
- `GetUserNewsResources`: Query news resources with user state
- `GetFollowableTopics`: Get topics with follow status
- `GetTopic`: Retrieve specific topic by ID
- `SearchContent`: Full-text search across news and topics
- `GetRecentSearchQueries`: Retrieve search history

#### Search History Operations
- `InsertRecentSearchQuery`: Add query to search history
- `ClearRecentSearchQueries`: Clear search history

#### Synchronization Operations
- `StartSync`: Initiate background synchronization
- `SyncTopics`: Synchronize topics from API
- `SyncNewsResources`: Synchronize news resources from API
- `CompleteSyncSuccess`: Mark sync as successful
- `CompleteSyncError`: Handle sync failure with retry logic

#### Network Operations
- `UpdateNetworkStatus`: Track network connectivity changes

Each operation specifies:
- **Preconditions**: Required state before operation
- **Postconditions**: Guaranteed state after operation
- **Invariants**: Properties maintained across operations
- **Error Handling**: Failure scenarios and recovery

**Use this** to understand how the system responds to user actions and state changes.

### 5. External Integrations Specification (`integrations.zpp`)

**Z++ formal specification of external API integrations**

Integration components:

#### Network Protocol Specifications
- `HttpRequest` and `HttpResponse` schemas
- HTTP method and status code definitions
- Content-Type handling

#### REST API Contract Specifications
- `API_GetTopics`: Retrieve topics from backend
- `API_GetNewsResources`: Retrieve news resources from backend
- `API_GetTopicChangeList`: Get incremental topic changes
- `API_GetNewsResourceChangeList`: Get incremental news changes

#### Error Handling & Retry Logic
- `NetworkError` classification (timeout, connection, HTTP errors)
- `RetryPolicy` with exponential backoff
- Retryable vs non-retryable error determination

#### Incremental Synchronization Protocol
- Complete sync workflow specification
- Change list processing
- Version management
- Conflict resolution

#### Caching & Offline Strategy
- Offline-first cache policy
- Stale-while-revalidate pattern
- Cache invalidation rules

#### Rate Limiting & Throttling
- Client-side rate limiting
- Burst request handling

#### Data Validation & Sanitization
- API response validation
- HTML content sanitization (XSS prevention)
- URL validation

#### Security
- HTTPS enforcement
- Certificate validation
- Response sanitization

**Use this** to understand how the application integrates with external services.

## Z++ Notation Guide

Z++ is a formal specification language based on Z notation with extensions for object-oriented and component-based systems. Key notational elements:

### Schema Notation

```z
┌ SchemaName ────────────────────────────────────────────────────────────┐
│ field1: Type1                                                          │
│ field2: Type2                                                          │
├────────────────────────────────────────────────────────────────────────┤
│ constraint1                  (* Comment *)                             │
│ constraint2                                                            │
└────────────────────────────────────────────────────────────────────────┘
```

### Common Symbols

| Symbol | Meaning |
|--------|---------|
| `ℤ` | Integers |
| `ℕ` | Natural numbers (non-negative integers) |
| `BOOL` | Boolean values (true/false) |
| `STRING` | Unicode text |
| `ℙ T` | Power set (set of all subsets of T) |
| `seq T` | Sequence of elements of type T |
| `T ↦ U` | Mapping from T to U |
| `T ∪ U` | Union of types T and U |
| `T ∩ U` | Intersection of types T and U |
| `T \ U` | Set difference (elements in T but not in U) |
| `∀` | For all (universal quantification) |
| `∃` | There exists (existential quantification) |
| `∃!` | There exists exactly one |
| `⇒` | Implies |
| `⇔` | If and only if |
| `∧` | Logical AND |
| `∨` | Logical OR |
| `¬` | Logical NOT |
| `⟨⟩` | Empty sequence |
| `⁀` | Sequence concatenation |
| `ran s` | Range of sequence s |
| `dom s` | Domain of sequence s |
| `#s` | Size/cardinality of collection s |
| `∈` | Element of (membership) |
| `∉` | Not an element of |
| `⊆` | Subset or equal |
| `⊂` | Proper subset |
| `Ξ` | State observation (read-only) |
| `Δ` | State change (mutation) |

### Type Definitions

```z
TypeName ::= value1 | value2 | value3
(* Defines an enumeration type *)
```

### Function Definitions

```z
FunctionName: InputType → OutputType
∀ input: InputType •
  FunctionName(input) = expression
```

## How to Read the Specifications

### For Software Engineers

1. Start with `architecture_overview.md` to understand the system design
2. Read `data_model.zpp` to understand entities and relationships
3. Review `operations.zpp` to see how user actions are processed
4. Reference `system_state.zpp` when you need to understand state management
5. Consult `integrations.zpp` for external API details

### For Formal Methods Practitioners

1. Review the type definitions in each specification
2. Examine the schemas and their invariants
3. Study the operation specifications (preconditions, postconditions)
4. Verify the theorems using the provided invariants
5. Check consistency across specifications

### For Product Managers

1. Read `architecture_overview.md` for high-level architecture
2. Use the Mermaid diagrams to understand data flows
3. Reference `operations.zpp` to understand feature capabilities
4. Consult `integrations.zpp` for external dependencies

### For QA Engineers

1. Use `operations.zpp` preconditions as test setup requirements
2. Use postconditions as expected outcomes for test assertions
3. Reference invariants to create negative tests
4. Use `integrations.zpp` error scenarios for error handling tests

## Benefits of Formal Specifications

### 1. Precision
- Unambiguous specification of system behavior
- Eliminates interpretation gaps between documentation and implementation

### 2. Completeness
- Comprehensive coverage of all system states and operations
- Explicit specification of edge cases and error conditions

### 3. Consistency
- Mathematical rigor ensures internal consistency
- Cross-references between specifications validated

### 4. Verification
- Formal properties can be mechanically verified
- Invariants serve as correctness criteria

### 5. Documentation
- Self-documenting through explicit constraints and comments
- Permanent record of design decisions

### 6. Communication
- Common language for developers, testers, and stakeholders
- Reduces misunderstandings in requirements

## Verification Properties

The specifications define several key properties that should hold for all valid system states:

### Referential Integrity
- All foreign key references point to existing entities
- Deleting an entity cascades to remove dependent references

### Temporal Consistency
- All timestamps respect causality (no future dates)
- Event ordering is preserved

### User Data Consistency
- Bookmarked items are always viewed
- User preferences reference valid entities

### Synchronization Consistency
- Version numbers are monotonically increasing
- Successful sync ensures data completeness

### Offline-First Guarantee
- Application remains functional without network
- Local data is the source of truth

## Relationship to Implementation

These formal specifications describe the **intended behavior** of the Now in Android application. The actual Kotlin/Java implementation in this repository should conform to these specifications.

### Mapping to Code

| Specification | Implementation |
|---------------|----------------|
| `Topic` entity | `core/model/src/main/kotlin/.../Topic.kt` |
| `NewsResource` entity | `core/model/src/main/kotlin/.../NewsResource.kt` |
| `UserData` entity | `core/model/src/main/kotlin/.../UserData.kt` |
| `DatabaseState` | `core/database/src/main/kotlin/.../NiaDatabase.kt` |
| `DataStoreState` | `core/datastore/src/main/kotlin/.../NiaPreferencesDataSource.kt` |
| Operations | `core/data/src/main/kotlin/.../repository/*Repository.kt` |
| API contracts | `core/network/src/main/kotlin/.../NiaNetworkDataSource.kt` |

## Maintenance

These specifications should be updated when:
- Adding new entities or fields to the data model
- Introducing new user-facing operations
- Changing synchronization or caching strategies
- Modifying external API contracts
- Adding new invariants or constraints

## Tools for Working with Z++ Specifications

While these specifications are written in a formal notation, they can be read and understood with a basic understanding of mathematical logic and set theory. For formal verification:

- **Z/EVES**: Theorem prover for Z specifications
- **ProofPower**: Formal verification toolset supporting Z
- **Community Z Tools (CZT)**: Open-source Z toolset
- **Manual Review**: Peer review by formal methods experts

## Contributing

When contributing to these specifications:

1. **Maintain Consistency**: Ensure new specifications align with existing patterns
2. **Add Comments**: Use `(* ... *)` comments to explain complex constraints
3. **Specify Invariants**: Explicitly state properties that must hold
4. **Define Operations Completely**: Include pre/postconditions for all operations
5. **Cross-Reference**: Link related specifications across files

## Questions and Support

For questions about these specifications:
- File an issue in the GitHub repository
- Contact the architecture team
- Consult formal methods experts in your organization

## License

These formal specifications are part of the Now in Android project and are licensed under the Apache License, Version 2.0.

---

*These formal specifications provide a rigorous mathematical foundation for understanding, implementing, and verifying the Now in Android application architecture.*
