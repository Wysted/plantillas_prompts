# Clean Architecture - Plantilla Base Multi-Lenguaje

> Arquitectura genérica para iniciar proyectos desde cero - Backend, Frontend, Mobile, CLI, Desktop

Esta documentación describe la arquitectura base **independiente de cualquier proyecto específico**. Úsala como referencia para iniciar nuevos proyectos o para alimentar a un MCP/LLM con las convenciones arquitectónicas.

**Adaptable a**:
- **Lenguajes**: Go, Python, TypeScript, Java, C#, Kotlin, Swift, Dart, etc.
- **Tipos de Proyecto**: Backend APIs, Frontend SPAs, Mobile Apps, CLIs, Desktop Apps, Microservicios
- **Frameworks**: Gin, Django, FastAPI, NestJS, React, Angular, Vue, Flutter, Spring Boot, etc.

---

## 📋 Índice

1. [¿Qué es esta arquitectura?](#qué-es-esta-arquitectura)
2. [Estructura por tipo de proyecto](#estructura-por-tipo-de-proyecto)
3. [Las capas arquitectónicas](#las-capas-arquitectónicas)
4. [Patrones fundamentales](#patrones-fundamentales)
5. [Flujo de datos](#flujo-de-datos)
6. [Cómo implementar](#cómo-implementar)
7. [Reglas de oro](#reglas-de-oro)
8. [Ejemplos multi-lenguaje](#ejemplos-multi-lenguaje)

---

## 🎯 ¿Qué es esta arquitectura?

**Clean Architecture** aplicada a múltiples ecosistemas. Combina:

- **Layered Architecture** (arquitectura en capas)
- **Separation of Concerns** (separación de responsabilidades)
- **Dependency Inversion** (dependencias hacia abstracciones)
- **Testability** (facilidad para testing)

### Principios SOLID aplicados

| Principio | Aplicación |
|-----------|------------|
| **S**ingle Responsibility | Cada capa/módulo tiene una responsabilidad única |
| **O**pen/Closed | Extensible via interfaces/abstracciones |
| **L**iskov Substitution | Implementaciones son intercambiables |
| **I**nterface Segregation | Contratos pequeños y específicos |
| **D**ependency Inversion | Dependencias hacia abstracciones |

### Adaptación por Ecosistema

Esta arquitectura se adapta al ecosistema del lenguaje/framework:

| Ecosistema | Patrón Principal | Nomenclatura |
|------------|------------------|--------------|
| **Backend Go** | Repository + Service Layer | Interface/struct, NewX() constructors |
| **Backend Python** | Repository + Service Layer | Abstract classes, snake_case |
| **Backend TypeScript/Node** | Repository + Service Layer | Classes/interfaces, camelCase |
| **Frontend React** | Container/Presentational + Hooks | Components, useX() hooks |
| **Frontend Vue** | Composition API + Composables | Components, useX() composables |
| **Frontend Angular** | Services + Components | Decorators, PascalCase |
| **Mobile Flutter** | BLoC/MVVM | Widgets, ViewModels, Blocs |
| **Mobile React Native** | MVVM + Hooks | Components, ViewModels, hooks |

---

## 📁 Estructura por Tipo de Proyecto

### Backend (Go/Python/Node/Java/C#)

```
proyecto-backend/
├── cmd/                              # Punto de entrada
│   └── api/main.{go|py|ts|java}
│
├── internal/                         # Código privado
│   ├── config/                       # Configuración
│   │
│   ├── domain/                       # ⭐ Núcleo del negocio
│   │   └── [entity]/                 # Un módulo por entidad
│   │       ├── model.{ext}           # Entidad del dominio
│   │       ├── dto.{ext}             # Data Transfer Objects
│   │       ├── repository.{ext}      # Abstracción de persistencia
│   │       └── service.{ext}         # Lógica de negocio
│   │
│   └── delivery/                     # Adaptador de entrada
│       ├── http/                     # (REST API)
│       │   ├── handler/              # Controladores
│       │   ├── middleware/           # Auth, CORS, etc.
│       │   └── router.{ext}          # ⭐ Dependency Injection
│       ├── grpc/                     # (gRPC - opcional)
│       └── graphql/                  # (GraphQL - opcional)
│
├── pkg/                              # Código reutilizable
│   ├── database/
│   ├── logger/
│   └── jwt/
│
└── utils/                            # Utilidades generales
```

**Ejemplos de extensiones**:
- Go: `.go`
- Python: `.py`
- TypeScript: `.ts`
- Java: `.java`
- C#: `.cs`

---

### Frontend (React/Vue/Angular)

```
proyecto-frontend/
├── src/
│   ├── features/                     # ⭐ Organización por feature
│   │   └── [feature]/
│   │       ├── components/           # UI components
│   │       ├── hooks/                # Custom hooks (React)
│   │       ├── composables/          # Composables (Vue)
│   │       ├── services/             # Lógica de negocio
│   │       ├── store/                # Estado local
│   │       └── types/                # TypeScript types
│   │
│   ├── shared/                       # Código compartido
│   │   ├── components/               # Componentes reutilizables
│   │   ├── hooks/                    # Hooks globales
│   │   ├── services/                 # API clients
│   │   └── utils/                    # Helpers
│   │
│   ├── store/                        # Estado global
│   │   ├── slices/                   # Redux slices
│   │   └── index.{ts|js}
│   │
│   └── app/                          # Configuración app
│       ├── App.{tsx|jsx|vue}
│       └── router.{ts|js}
│
└── public/
```

**Variaciones**:
- **React**: `components/`, `hooks/`, `store/` (Redux/Zustand)
- **Vue**: `components/`, `composables/`, `stores/` (Pinia)
- **Angular**: `components/`, `services/`, `modules/`

---

### Mobile (Flutter/React Native)

```
proyecto-mobile/
├── lib/                              # Flutter
│   ├── features/                     # ⭐ Features
│   │   └── [feature]/
│   │       ├── presentation/         # Screens/Widgets
│   │       │   ├── screens/
│   │       │   ├── widgets/
│   │       │   └── bloc/             # BLoC (Flutter)
│   │       ├── domain/               # Entities, Use Cases
│   │       └── data/                 # Repositories, Data Sources
│   │
│   ├── core/                         # Shared
│   │   ├── theme/
│   │   ├── utils/
│   │   └── network/
│   │
│   └── main.dart
│
└── src/                              # React Native
    ├── screens/
    ├── components/
    ├── services/
    ├── viewmodels/
    └── navigation/
```

**Patrones**:
- **Flutter**: BLoC, Provider, Riverpod, MVVM
- **React Native**: MVVM, Hooks, Redux, MobX

---

### CLI (Go/Python/Node)

```
proyecto-cli/
├── cmd/
│   └── [app]/main.{go|py|ts}
│
├── internal/
│   ├── commands/                     # Command definitions
│   ├── handlers/                     # Command logic
│   ├── services/                     # Business logic
│   └── utils/                        # Formatters, helpers
│
└── pkg/
    ├── config/
    └── logger/
```

---

## 🏗️ Las Capas Arquitectónicas

### Backend Architecture

```
┌──────────────────────────────────────┐
│    1. DELIVERY LAYER                 │  ← HTTP/gRPC Handlers, Controllers
│    (Presentación)                    │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    2. APPLICATION LAYER              │  ← Services, Use Cases
│    (Lógica de Negocio)               │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    3. DOMAIN LAYER                   │  ← Entities, Models, DTOs
│    (Dominio)                         │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    4. DATA LAYER                     │  ← Repositories, DAL
│    (Persistencia)                    │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    5. INFRASTRUCTURE                 │  ← DB, External APIs, Cache
└──────────────────────────────────────┘
```

### Frontend Architecture

```
┌──────────────────────────────────────┐
│    1. PRESENTATION                   │  ← Components, Screens, Pages
│    (UI Layer)                        │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    2. STATE MANAGEMENT               │  ← Store, Context, Providers
│    (Estado)                          │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    3. BUSINESS LOGIC                 │  ← Services, Hooks, Composables
│    (Lógica)                          │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    4. DATA ACCESS                    │  ← API Clients, Repositories
│    (Datos)                           │
└──────────────────────────────────────┘
```

### Mobile Architecture (MVVM)

```
┌──────────────────────────────────────┐
│    1. VIEW                           │  ← Screens, Widgets, Components
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    2. VIEWMODEL / PRESENTER          │  ← UI Logic, State
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    3. USE CASES / INTERACTORS        │  ← Business Logic
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    4. REPOSITORIES                   │  ← Data Access
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    5. DATA SOURCES                   │  ← API, Local DB, Cache
└──────────────────────────────────────┘
```

---

## 🎨 Patrones Fundamentales

### 1. Repository Pattern (Backend/Mobile)

**Problema**: Lógica de datos mezclada con lógica de negocio

**Solución**: Abstraer acceso a datos

**Backend Go**:
```go
type Repository interface {
    Create(entity Entity) (*Entity, error)
    FindByID(id int64) (*Entity, error)
    FindAll(criteria *Criteria) ([]Entity, error)
}
```

**Backend Python**:
```python
from abc import ABC, abstractmethod

class Repository(ABC):
    @abstractmethod
    def create(self, entity: Entity) -> Entity:
        pass

    @abstractmethod
    def find_by_id(self, id: int) -> Optional[Entity]:
        pass
```

**Frontend TypeScript**:
```typescript
interface Repository<T> {
    create(data: CreateDTO): Promise<T>;
    findById(id: string): Promise<T>;
    findAll(filters?: Filters): Promise<T[]>;
}
```

**Mobile Flutter**:
```dart
abstract class Repository {
  Future<Entity> create(Entity entity);
  Future<Entity?> findById(int id);
  Future<List<Entity>> findAll(Criteria criteria);
}
```

---

### 2. Service Layer Pattern (Backend)

**Backend Go**:
```go
type Service interface {
    Create(dto EntityDTO) (*Entity, error)
    GetByID(id int64) (*Entity, error)
}

type service struct {
    repo Repository
}

func NewService(repo Repository) Service {
    return &service{repo: repo}
}
```

**Backend Python (Django-style)**:
```python
class EntityService:
    def __init__(self, repository: EntityRepository):
        self.repository = repository

    def create(self, dto: EntityDTO) -> Entity:
        # Validate
        if not self._validate_email(dto.email):
            raise ValidationError("Invalid email")

        # Business logic
        if self.repository.exists(email=dto.email):
            raise DuplicateError("Entity exists")

        # Persist
        return self.repository.create(dto.to_model())
```

**Backend TypeScript (NestJS-style)**:
```typescript
@Injectable()
export class EntityService {
    constructor(private repository: EntityRepository) {}

    async create(dto: CreateEntityDto): Promise<Entity> {
        // Validate
        this.validateEmail(dto.email);

        // Business logic
        const exists = await this.repository.exists({ email: dto.email });
        if (exists) throw new ConflictException('Entity exists');

        // Persist
        return this.repository.create(dto);
    }
}
```

---

### 3. Component Pattern (Frontend)

**React (Container/Presentational)**:
```typescript
// Container (logic)
const EntityListContainer: FC = () => {
    const { entities, loading } = useEntities();
    const handleSelect = (id: string) => navigate(`/entities/${id}`);

    return <EntityList entities={entities} onSelect={handleSelect} loading={loading} />;
};

// Presentational (UI)
interface Props {
    entities: Entity[];
    onSelect: (id: string) => void;
    loading: boolean;
}

const EntityList: FC<Props> = ({ entities, onSelect, loading }) => {
    if (loading) return <Spinner />;

    return (
        <div>
            {entities.map(entity => (
                <EntityCard key={entity.id} entity={entity} onClick={() => onSelect(entity.id)} />
            ))}
        </div>
    );
};
```

**Vue (Composition API)**:
```vue
<script setup lang="ts">
import { useEntities } from '@/features/entity/composables/useEntities';

const { entities, loading, selectEntity } = useEntities();
</script>

<template>
  <div v-if="loading">Loading...</div>
  <div v-else>
    <EntityCard
      v-for="entity in entities"
      :key="entity.id"
      :entity="entity"
      @select="selectEntity"
    />
  </div>
</template>
```

---

### 4. DTO Pattern (Multi-lenguaje)

**Backend Go**:
```go
type EntityDTO struct {
    Name  string `json:"name"`
    Email string `json:"email"`
}

func (dto *EntityDTO) ToModel() Entity {
    return Entity{
        Name:  dto.Name,
        Email: dto.Email,
    }
}
```

**Backend Python**:
```python
from dataclasses import dataclass

@dataclass
class EntityDTO:
    name: str
    email: str

    def to_model(self) -> Entity:
        return Entity(
            name=self.name,
            email=self.email
        )
```

**Frontend TypeScript**:
```typescript
interface EntityDTO {
    name: string;
    email: string;
}

class EntityMapper {
    static toModel(dto: EntityDTO): Entity {
        return {
            id: '',
            name: dto.name,
            email: dto.email,
            createdAt: new Date()
        };
    }
}
```

---

### 5. Dependency Injection

**Backend Go (Manual)**:
```go
// router.go
func NewRouter(db *sql.DB) *gin.Engine {
    // 1. Create repositories
    entityRepo := entity.NewRepository(db)

    // 2. Inject into services
    entityService := entity.NewService(entityRepo)

    // 3. Inject into handlers
    entityHandler := handler.NewEntityHandler(entityService)

    // 4. Register routes
    r := gin.New()
    r.POST("/entities", entityHandler.Create)

    return r
}
```

**Backend Python (Django-style)**:
```python
# views.py
class EntityViewSet(viewsets.ModelViewSet):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.repository = EntityRepository()
        self.service = EntityService(self.repository)
```

**Backend TypeScript (NestJS - Automatic)**:
```typescript
@Controller('entities')
export class EntityController {
    constructor(private readonly service: EntityService) {}

    @Post()
    create(@Body() dto: CreateEntityDto) {
        return this.service.create(dto);
    }
}
```

**Frontend React (Context)**:
```typescript
const EntityServiceContext = createContext<EntityService | null>(null);

export const EntityProvider: FC = ({ children }) => {
    const repository = new EntityRepository();
    const service = new EntityService(repository);

    return (
        <EntityServiceContext.Provider value={service}>
            {children}
        </EntityServiceContext.Provider>
    );
};

export const useEntityService = () => {
    const service = useContext(EntityServiceContext);
    if (!service) throw new Error('Provider missing');
    return service;
};
```

---

## 🔄 Flujo de Datos

### Backend API Flow

```
1. HTTP Request
   POST /api/entities
   Body: {"name": "Example", "email": "user@example.com"}
   ↓
2. Router → Encuentra ruta → EntityHandler.Create
   ↓
3. Middleware (Auth, Validation, etc.) → next()
   ↓
4. Handler
   - Parse JSON to DTO
   - Call service.Create(dto)
   ↓
5. Service
   - Validate data
   - Apply business rules
   - Call repository.Create()
   ↓
6. Repository
   - Build SQL/ORM query
   - Execute INSERT
   ↓
7. Database → Returns created entity
   ↓
   (Response flows back up)
   ↓
8. Handler → JSON response
   ↓
9. Client receives → {"status": 201, "data": {...}}
```

### Frontend SPA Flow

```
1. User Action → Click "Create Entity" button
   ↓
2. Component → Calls handleCreate()
   ↓
3. Service/Hook → entityService.create(data)
   ↓
4. API Client → POST /api/entities
   ↓
5. Backend processes...
   ↓
6. Response → {data: entity}
   ↓
7. State Update → Redux/Context updates store
   ↓
8. Component Re-renders → Shows new entity
```

### Mobile App Flow

```
1. User Interaction → Tap "Submit" button
   ↓
2. View/Screen → Calls viewModel.createEntity()
   ↓
3. ViewModel → Validates input, calls useCase.execute()
   ↓
4. Use Case → Business logic, calls repository.create()
   ↓
5. Repository → API call or local DB insert
   ↓
6. Data Source → Network/Database
   ↓
7. Response flows back
   ↓
8. ViewModel updates state → View rebuilds/re-renders
```

---

## ⚖️ Reglas de Oro

### ✅ DO (Universal)

1. **Separa responsabilidades** - Una clase/módulo = una responsabilidad
2. **Inyecta dependencias** - No instancies dentro, inyecta desde fuera
3. **Valida en la capa correcta** - Service/Use Case layer, no en UI
4. **Usa abstracciones** - Interfaces, abstract classes, contracts
5. **Define errores claros** - Errores específicos del dominio
6. **Mantén capas delgadas** - UI delgada, lógica en services
7. **Testea cada capa** - Unit tests, integration tests

### ❌ DON'T (Universal)

1. **No mezcles capas** - UI no debe hacer SQL directo
2. **No saltees capas** - UI → Service → Repository (no UI → Repository)
3. **No dupliques lógica** - DRY (Don't Repeat Yourself)
4. **No uses `any/dynamic`** - Tipado fuerte siempre que sea posible
5. **No ignores errores** - Manéjalos adecuadamente
6. **No hagas god objects** - Clases/módulos pequeños y enfocados

---

## 📚 Ejemplos Multi-Lenguaje

### Backend: Crear Entidad

<details>
<summary><b>Go Example</b></summary>

```go
// model.go
package entity

type Entity struct {
    ID        int64  `json:"id"`
    Name      string `json:"name"`
    Email     string `json:"email"`
    CreatedAt time.Time `json:"createdAt"`
}

// repository.go
type Repository interface {
    Create(Entity) (*Entity, error)
}

type repository struct {
    db *sql.DB
}

func NewRepository(db *sql.DB) Repository {
    return &repository{db: db}
}

// service.go
type Service interface {
    Create(EntityDTO) (*Entity, error)
}

type service struct {
    repo Repository
}

func NewService(repo Repository) Service {
    return &service{repo: repo}
}
```
</details>

<details>
<summary><b>Python Example</b></summary>

```python
# models.py
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Entity:
    id: int
    name: str
    email: str
    created_at: datetime

# repository.py
from abc import ABC, abstractmethod

class EntityRepository(ABC):
    @abstractmethod
    def create(self, entity: Entity) -> Entity:
        pass

class EntityRepositoryImpl(EntityRepository):
    def __init__(self, db):
        self.db = db

    def create(self, entity: Entity) -> Entity:
        # ORM logic
        pass

# service.py
class EntityService:
    def __init__(self, repository: EntityRepository):
        self.repository = repository

    def create(self, dto: EntityDTO) -> Entity:
        # Validate and create
        pass
```
</details>

<details>
<summary><b>TypeScript/NestJS Example</b></summary>

```typescript
// entity.model.ts
export class Entity {
    id: string;
    name: string;
    email: string;
    createdAt: Date;
}

// entity.repository.ts
export interface EntityRepository {
    create(entity: Entity): Promise<Entity>;
}

@Injectable()
export class EntityRepositoryImpl implements EntityRepository {
    constructor(@InjectRepository(Entity) private repo: Repository<Entity>) {}

    async create(entity: Entity): Promise<Entity> {
        return this.repo.save(entity);
    }
}

// entity.service.ts
@Injectable()
export class EntityService {
    constructor(private repository: EntityRepository) {}

    async create(dto: CreateEntityDto): Promise<Entity> {
        // Validate and create
    }
}
```
</details>

### Frontend: Component with State

<details>
<summary><b>React Example</b></summary>

```typescript
// useEntities.ts (hook)
export const useEntities = () => {
    const [entities, setEntities] = useState<Entity[]>([]);
    const [loading, setLoading] = useState(false);
    const service = useEntityService();

    const loadEntities = async () => {
        setLoading(true);
        const data = await service.getAll();
        setEntities(data);
        setLoading(false);
    };

    return { entities, loading, loadEntities };
};

// EntityList.tsx (component)
export const EntityList: FC = () => {
    const { entities, loading, loadEntities } = useEntities();

    useEffect(() => {
        loadEntities();
    }, []);

    if (loading) return <Spinner />;

    return (
        <div>
            {entities.map(entity => (
                <EntityCard key={entity.id} entity={entity} />
            ))}
        </div>
    );
};
```
</details>

<details>
<summary><b>Vue Example</b></summary>

```vue
<script setup lang="ts">
// useEntities.ts (composable)
import { ref, onMounted } from 'vue';
import { entityService } from '@/services/entity.service';

const entities = ref<Entity[]>([]);
const loading = ref(false);

const loadEntities = async () => {
    loading.value = true;
    entities.value = await entityService.getAll();
    loading.value = false;
};

onMounted(() => {
    loadEntities();
});
</script>

<template>
  <div v-if="loading">Loading...</div>
  <div v-else>
    <EntityCard
      v-for="entity in entities"
      :key="entity.id"
      :entity="entity"
    />
  </div>
</template>
```
</details>

### Mobile: MVVM Pattern

<details>
<summary><b>Flutter/BLoC Example</b></summary>

```dart
// entity.dart (model)
class Entity {
  final int id;
  final String name;
  final String email;

  Entity({required this.id, required this.name, required this.email});
}

// entity_repository.dart
abstract class EntityRepository {
  Future<Entity> create(Entity entity);
  Future<List<Entity>> getAll();
}

// entity_bloc.dart
class EntityBloc extends Bloc<EntityEvent, EntityState> {
  final EntityRepository repository;

  EntityBloc(this.repository) : super(EntityInitial()) {
    on<LoadEntities>(_onLoadEntities);
  }

  Future<void> _onLoadEntities(LoadEntities event, Emitter<EntityState> emit) async {
    emit(EntityLoading());
    try {
      final entities = await repository.getAll();
      emit(EntityLoaded(entities));
    } catch (e) {
      emit(EntityError(e.toString()));
    }
  }
}

// entity_screen.dart
class EntityScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<EntityBloc, EntityState>(
      builder: (context, state) {
        if (state is EntityLoading) {
          return CircularProgressIndicator();
        } else if (state is EntityLoaded) {
          return ListView.builder(
            itemCount: state.entities.length,
            itemBuilder: (context, index) {
              return EntityCard(entity: state.entities[index]);
            },
          );
        }
        return Container();
      },
    );
  }
}
```
</details>

---

## 🎓 Recursos por Ecosistema

### Backend

| Lenguaje | Framework | ORM | Validación |
|----------|-----------|-----|------------|
| **Go** | Gin, Fiber, Echo | GORM, sqlx, Bob | validator |
| **Python** | Django, FastAPI, Flask | Django ORM, SQLAlchemy | Pydantic, Marshmallow |
| **TypeScript** | NestJS, Express | TypeORM, Prisma | class-validator |
| **Java** | Spring Boot | Hibernate, JPA | Bean Validation |
| **C#** | ASP.NET Core | Entity Framework | Data Annotations |

### Frontend

| Framework | State Management | HTTP Client | Routing |
|-----------|-----------------|-------------|---------|
| **React** | Redux Toolkit, Zustand, Jotai | Axios, Fetch API | React Router |
| **Vue** | Pinia, Vuex | Axios | Vue Router |
| **Angular** | NgRx, Akita | HttpClient | Angular Router |
| **Svelte** | Svelte Store | Axios | SvelteKit |

### Mobile

| Platform | Architecture | State | Navigation |
|----------|--------------|-------|------------|
| **Flutter** | BLoC, Provider, Riverpod | flutter_bloc | go_router |
| **React Native** | MVVM, Redux | Redux, MobX | React Navigation |
| **Swift/iOS** | MVVM, VIPER | Combine | SwiftUI Navigation |
| **Kotlin/Android** | MVVM, MVI | ViewModel, Flow | Jetpack Navigation |

---

## 🆘 FAQ

### ¿Esta arquitectura funciona para cualquier lenguaje?

Sí, los **principios** son universales. La **implementación** se adapta al ecosistema (interfaces en Go/TS, abstract classes en Python, protocols en Swift).

### ¿Cuándo usar Repository Pattern?

En **Backend** y **Mobile**. En Frontend web es opcional (puedes usar API clients directamente en services).

### ¿Qué arquitectura uso en Frontend?

- **React/Vue**: Feature-based + Container/Presentational + Hooks/Composables
- **Angular**: Module-based + Services + Components

### ¿Qué arquitectura uso en Mobile?

- **Flutter**: BLoC o MVVM
- **React Native**: MVVM o Redux
- **Swift/Kotlin**: MVVM o VIPER/Clean

### ¿Dónde va la validación?

**Service/Use Case Layer**. La UI solo valida formato básico.

---

## 📝 Próximos Pasos

1. **Lee** `arquitectura_base.json` para detalles técnicos adaptados a tu lenguaje
2. **Revisa** `patrones_diseno.json` para ver todos los patrones aplicables
3. **Consulta** `ejemplos_practicos.md` para código real en tu ecosistema
4. **Usa** `PROMPT_GENERADOR.md` para generar código desde esta plantilla
5. **Adapta** según las convenciones de tu lenguaje/framework

---

**Versión**: 2.0 (Multi-lenguaje)
**Última actualización**: 2026-01-11
**Tipo**: Plantilla Genérica Universal
**Soporta**: Backend (Go, Python, TS, Java, C#), Frontend (React, Vue, Angular), Mobile (Flutter, RN), CLI
