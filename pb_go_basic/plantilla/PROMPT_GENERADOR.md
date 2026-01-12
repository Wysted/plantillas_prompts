# Prompt para Generar Código con Plantilla Multi-Lenguaje

> Usa este prompt para que un MCP/LLM genere código siguiendo la arquitectura base en cualquier lenguaje/framework

---

## 🎯 Prompt Universal para MCP/LLM

```
Eres un arquitecto de software experto en Clean Architecture y desarrollo multi-lenguaje.

CONTEXTO:
Lee los siguientes archivos que definen la arquitectura base que debes seguir:

1. arquitectura/plantilla/arquitectura_base.json
2. arquitectura/plantilla/patrones_diseno.json
3. arquitectura/plantilla/guia_rapida.md

INFORMACIÓN DEL PROYECTO:
Lenguaje: [Go / Python / TypeScript / Java / C# / Dart / etc.]
Tipo: [Backend API / Frontend SPA / Mobile App / CLI / Desktop]
Framework: [Gin / Django / NestJS / React / Flutter / etc.]

TAREA:
Genera un módulo/feature completo llamado "[NOMBRE_MODULO]" siguiendo EXACTAMENTE
los patrones y convenciones del lenguaje/framework especificado.

REQUISITOS:

1. ESTRUCTURA DE ARCHIVOS (adaptada al ecosistema):

   Backend Go:
   - internal/domain/[NOMBRE]/model.go
   - internal/domain/[NOMBRE]/dto.go
   - internal/domain/[NOMBRE]/errors.go
   - internal/domain/[NOMBRE]/repository.go
   - internal/domain/[NOMBRE]/service.go
   - internal/http/handler/[NOMBRE]_handler.go

   Backend Python:
   - app/domain/[NOMBRE]/models.py
   - app/domain/[NOMBRE]/schemas.py
   - app/domain/[NOMBRE]/repository.py
   - app/domain/[NOMBRE]/service.py
   - app/api/[NOMBRE]/views.py (Django) o routes.py (FastAPI)

   Backend TypeScript/NestJS:
   - src/[NOMBRE]/entities/[NOMBRE].entity.ts
   - src/[NOMBRE]/dto/create-[NOMBRE].dto.ts
   - src/[NOMBRE]/[NOMBRE].repository.ts
   - src/[NOMBRE]/[NOMBRE].service.ts
   - src/[NOMBRE]/[NOMBRE].controller.ts
   - src/[NOMBRE]/[NOMBRE].module.ts

   Frontend React:
   - src/features/[NOMBRE]/components/[Nombre]List.tsx
   - src/features/[NOMBRE]/components/[Nombre]Form.tsx
   - src/features/[NOMBRE]/hooks/use[Nombre].ts
   - src/features/[NOMBRE]/services/[nombre].service.ts
   - src/features/[NOMBRE]/types/[nombre].types.ts

   Frontend Vue:
   - src/features/[nombre]/components/[Nombre]List.vue
   - src/features/[nombre]/composables/use[Nombre].ts
   - src/features/[nombre]/services/[nombre].service.ts
   - src/features/[nombre]/types/[nombre].types.ts

   Mobile Flutter:
   - lib/features/[nombre]/presentation/screens/[nombre]_screen.dart
   - lib/features/[nombre]/presentation/widgets/[nombre]_list.dart
   - lib/features/[nombre]/presentation/bloc/[nombre]_bloc.dart
   - lib/features/[nombre]/domain/entities/[nombre].dart
   - lib/features/[nombre]/domain/usecases/get_[nombre].dart
   - lib/features/[nombre]/data/repositories/[nombre]_repository.dart
   - lib/features/[nombre]/data/datasources/[nombre]_remote_datasource.dart

2. DEFINICIÓN DE LA ENTIDAD/MODELO:
   [DESCRIBE LOS CAMPOS]
   Ejemplo (genérico):
   - id (ID único)
   - name (string/texto)
   - email (string/texto)
   - isActive (boolean)
   - createdAt (timestamp/datetime)

3. CASOS DE USO NECESARIOS:
   [LISTA LOS CASOS DE USO]
   Ejemplo:
   - Create - Crear nueva entidad
   - GetByID - Obtener por ID
   - GetAll - Listar (con filtros opcionales)
   - Update - Actualizar
   - Delete - Eliminar

4. VALIDACIONES REQUERIDAS:
   [LISTA LAS VALIDACIONES]
   Ejemplo:
   - Email válido (regex/formato)
   - Name no vacío (mínimo X caracteres)
   - Email único (no duplicados)

5. PATRONES OBLIGATORIOS A APLICAR (según ecosistema):

   Backend (todos los lenguajes):
   - ✅ Constructor/Factory Pattern
   - ✅ Repository Pattern (interface/abstract + implementación)
   - ✅ Service Layer Pattern (interface/abstract + implementación)
   - ✅ Criteria/Filter Pattern (búsquedas dinámicas)
   - ✅ DTO/Schema Pattern (desacoplamiento)
   - ✅ Dependency Injection

   Frontend:
   - ✅ Container/Presentational Pattern (React/Vue)
   - ✅ Custom Hooks/Composables (React/Vue)
   - ✅ Service Pattern (API calls)
   - ✅ Type Safety (TypeScript)

   Mobile:
   - ✅ BLoC/MVVM Pattern
   - ✅ Repository Pattern
   - ✅ Use Case Pattern
   - ✅ Entity Pattern

6. REGLAS ESTRICTAS (universales):
   - NUNCA mezclar capas (UI no debe hacer SQL directo)
   - NUNCA saltarse capas (UI → Service → Repository)
   - NUNCA poner lógica de negocio en UI/Controllers
   - NUNCA poner lógica de negocio en Repositories
   - SIEMPRE validar en service/use case layer
   - SIEMPRE usar abstracciones (interfaces, abstract classes)
   - SIEMPRE seguir naming conventions del lenguaje
   - SIEMPRE manejar errores apropiadamente

7. NAMING CONVENTIONS (según lenguaje):

   Go:
   - Interfaces: type Repository interface, type Service interface
   - Structs privados: type repository struct, type service struct
   - Constructores: func NewRepository() Repository
   - Funciones públicas: PascalCase, privadas: camelCase

   Python:
   - Clases: PascalCase (class UserService)
   - Funciones/métodos: snake_case
   - Constantes: UPPER_SNAKE_CASE
   - Archivos: snake_case

   TypeScript:
   - Clases/Interfaces: PascalCase
   - Funciones/variables: camelCase
   - Archivos: kebab-case (user-service.ts)
   - Decoradores: @Injectable(), @Controller()

   Dart/Flutter:
   - Clases: PascalCase
   - Funciones/variables: camelCase
   - Archivos: snake_case
   - Widgets: PascalCase

8. ESTRUCTURA DE CADA ARCHIVO (ejemplos por lenguaje):

   === BACKEND GO ===

   model.go:
   ```go
   package [NOMBRE]

   import "time"

   type [Entity] struct {
       ID        int64     `json:"id"`
       Name      string    `json:"name"`
       Email     string    `json:"email"`
       IsActive  bool      `json:"isActive"`
       CreatedAt time.Time `json:"createdAt"`
   }
   ```

   repository.go:
   ```go
   package [NOMBRE]

   // Interface pública
   type Repository interface {
       Create(entity [Entity]) (*[Entity], error)
       GetByID(id int64) (*[Entity], error)
       GetAll(criteria *Criteria) ([][Entity], error)
       Update(id int64, data UpdateData) error
       Delete(id int64) error
   }

   type Criteria struct {
       Email    string
       IsActive *bool  // puntero para opcional
   }

   type UpdateData struct {
       Name  *string
       Email *string
   }

   // Struct privado
   type repository struct {
       db *sql.DB
   }

   // Constructor
   func NewRepository(db *sql.DB) Repository {
       return &repository{db: db}
   }

   func (r *repository) Create(entity [Entity]) (*[Entity], error) {
       // Implementación con ORM/SQL
   }
   ```

   service.go:
   ```go
   package [NOMBRE]

   type Service interface {
       Create(dto [Entity]DTO) (*[Entity], error)
       GetByID(id int64) (*[Entity], error)
   }

   type service struct {
       repo Repository
   }

   func NewService(repo Repository) Service {
       return &service{repo: repo}
   }

   func (s *service) Create(dto [Entity]DTO) (*[Entity], error) {
       // 1. Validar
       if err := s.validateEmail(dto.Email); err != nil {
           return nil, err
       }

       // 2. Regla de negocio
       exists, _ := s.repo.GetAll(&Criteria{Email: dto.Email})
       if len(exists) > 0 {
           return nil, Err[Entity]AlreadyExists
       }

       // 3. Persistir
       return s.repo.Create(dto.ToModel())
   }
   ```

   === BACKEND PYTHON (Django/FastAPI) ===

   models.py:
   ```python
   from dataclasses import dataclass
   from datetime import datetime

   @dataclass
   class Entity:
       id: int
       name: str
       email: str
       is_active: bool
       created_at: datetime
   ```

   schemas.py (DTOs):
   ```python
   from pydantic import BaseModel, EmailStr

   class EntityCreateDTO(BaseModel):
       name: str
       email: EmailStr

       def to_model(self) -> Entity:
           return Entity(
               name=self.name,
               email=self.email
           )

   class EntityUpdateDTO(BaseModel):
       name: Optional[str] = None
       email: Optional[EmailStr] = None
   ```

   repository.py:
   ```python
   from abc import ABC, abstractmethod
   from typing import List, Optional

   class EntityRepository(ABC):
       @abstractmethod
       def create(self, entity: Entity) -> Entity:
           pass

       @abstractmethod
       def get_by_id(self, id: int) -> Optional[Entity]:
           pass

       @abstractmethod
       def get_all(self, filters: dict = None) -> List[Entity]:
           pass

   class EntityRepositoryImpl(EntityRepository):
       def __init__(self, db):
           self.db = db

       def create(self, entity: Entity) -> Entity:
           # Implementación con ORM
           pass
   ```

   service.py:
   ```python
   class EntityService:
       def __init__(self, repository: EntityRepository):
           self.repository = repository

       def create(self, dto: EntityCreateDTO) -> Entity:
           # Validar
           if not self._validate_email(dto.email):
               raise ValidationError("Invalid email")

           # Regla de negocio
           existing = self.repository.get_all({"email": dto.email})
           if existing:
               raise DuplicateError("Entity exists")

           # Persistir
           return self.repository.create(dto.to_model())

       def _validate_email(self, email: str) -> bool:
           # Validación
           return True
   ```

   === BACKEND TYPESCRIPT (NestJS) ===

   [nombre].entity.ts:
   ```typescript
   export class Entity {
       id: string;
       name: string;
       email: string;
       isActive: boolean;
       createdAt: Date;
   }
   ```

   create-[nombre].dto.ts:
   ```typescript
   import { IsEmail, IsNotEmpty } from 'class-validator';

   export class CreateEntityDto {
       @IsNotEmpty()
       name: string;

       @IsEmail()
       email: string;
   }
   ```

   [nombre].repository.ts:
   ```typescript
   export interface EntityRepository {
       create(entity: Entity): Promise<Entity>;
       findById(id: string): Promise<Entity>;
       findAll(filters?: any): Promise<Entity[]>;
   }

   @Injectable()
   export class EntityRepositoryImpl implements EntityRepository {
       constructor(
           @InjectRepository(Entity)
           private repository: Repository<Entity>
       ) {}

       async create(entity: Entity): Promise<Entity> {
           return this.repository.save(entity);
       }
   }
   ```

   [nombre].service.ts:
   ```typescript
   @Injectable()
   export class EntityService {
       constructor(private repository: EntityRepository) {}

       async create(dto: CreateEntityDto): Promise<Entity> {
           // Validar (class-validator ya validó formato)

           // Regla de negocio
           const existing = await this.repository.findAll({ email: dto.email });
           if (existing.length > 0) {
               throw new ConflictException('Entity already exists');
           }

           // Persistir
           return this.repository.create(dto as Entity);
       }
   }
   ```

   === FRONTEND REACT ===

   types/[nombre].types.ts:
   ```typescript
   export interface Entity {
       id: string;
       name: string;
       email: string;
       isActive: boolean;
       createdAt: Date;
   }

   export interface CreateEntityDto {
       name: string;
       email: string;
   }
   ```

   services/[nombre].service.ts:
   ```typescript
   import axios from 'axios';

   export class EntityService {
       private baseUrl = '/api/entities';

       async create(dto: CreateEntityDto): Promise<Entity> {
           const response = await axios.post(this.baseUrl, dto);
           return response.data;
       }

       async getAll(): Promise<Entity[]> {
           const response = await axios.get(this.baseUrl);
           return response.data;
       }

       async getById(id: string): Promise<Entity> {
           const response = await axios.get(`${this.baseUrl}/${id}`);
           return response.data;
       }
   }
   ```

   hooks/use[Nombre].ts:
   ```typescript
   import { useState, useEffect } from 'react';
   import { EntityService } from '../services/entity.service';

   const entityService = new EntityService();

   export const useEntities = () => {
       const [entities, setEntities] = useState<Entity[]>([]);
       const [loading, setLoading] = useState(false);
       const [error, setError] = useState<string | null>(null);

       const loadEntities = async () => {
           setLoading(true);
           try {
               const data = await entityService.getAll();
               setEntities(data);
           } catch (err) {
               setError(err.message);
           } finally {
               setLoading(false);
           }
       };

       const createEntity = async (dto: CreateEntityDto) => {
           const created = await entityService.create(dto);
           setEntities([...entities, created]);
           return created;
       };

       return { entities, loading, error, loadEntities, createEntity };
   };
   ```

   components/[Nombre]List.tsx:
   ```typescript
   import React, { useEffect } from 'react';
   import { useEntities } from '../hooks/useEntities';

   export const EntityList: React.FC = () => {
       const { entities, loading, loadEntities } = useEntities();

       useEffect(() => {
           loadEntities();
       }, []);

       if (loading) return <div>Loading...</div>;

       return (
           <div>
               {entities.map(entity => (
                   <div key={entity.id}>
                       <h3>{entity.name}</h3>
                       <p>{entity.email}</p>
                   </div>
               ))}
           </div>
       );
   };
   ```

   === MOBILE FLUTTER (BLoC) ===

   domain/entities/[nombre].dart:
   ```dart
   class Entity {
     final int id;
     final String name;
     final String email;
     final bool isActive;
     final DateTime createdAt;

     Entity({
       required this.id,
       required this.name,
       required this.email,
       required this.isActive,
       required this.createdAt,
     });
   }
   ```

   data/repositories/[nombre]_repository.dart:
   ```dart
   abstract class EntityRepository {
     Future<Entity> create(Entity entity);
     Future<Entity?> getById(int id);
     Future<List<Entity>> getAll();
   }

   class EntityRepositoryImpl implements EntityRepository {
     final EntityRemoteDataSource remoteDataSource;

     EntityRepositoryImpl(this.remoteDataSource);

     @override
     Future<Entity> create(Entity entity) async {
       return await remoteDataSource.create(entity);
     }

     @override
     Future<List<Entity>> getAll() async {
       return await remoteDataSource.getAll();
     }
   }
   ```

   presentation/bloc/[nombre]_bloc.dart:
   ```dart
   class EntityBloc extends Bloc<EntityEvent, EntityState> {
     final EntityRepository repository;

     EntityBloc(this.repository) : super(EntityInitial()) {
       on<LoadEntities>(_onLoadEntities);
       on<CreateEntity>(_onCreateEntity);
     }

     Future<void> _onLoadEntities(
       LoadEntities event,
       Emitter<EntityState> emit,
     ) async {
       emit(EntityLoading());
       try {
         final entities = await repository.getAll();
         emit(EntityLoaded(entities));
       } catch (e) {
         emit(EntityError(e.toString()));
       }
     }
   }
   ```

   presentation/screens/[nombre]_screen.dart:
   ```dart
   class EntityScreen extends StatelessWidget {
     @override
     Widget build(BuildContext context) {
       return BlocBuilder<EntityBloc, EntityState>(
         builder: (context, state) {
           if (state is EntityLoading) {
             return Center(child: CircularProgressIndicator());
           } else if (state is EntityLoaded) {
             return ListView.builder(
               itemCount: state.entities.length,
               itemBuilder: (context, index) {
                 final entity = state.entities[index];
                 return ListTile(
                   title: Text(entity.name),
                   subtitle: Text(entity.email),
                 );
               },
             );
           } else if (state is EntityError) {
             return Center(child: Text('Error: ${state.message}'));
           }
           return Container();
         },
       );
     }
   }
   ```

9. CONFIGURACIÓN DE DEPENDENCIAS:

   Backend Go (router.go):
   ```go
   [nombre]Repo := [nombre].NewRepository(db)
   [nombre]Service := [nombre].NewService([nombre]Repo)
   [nombre]Handler := handler.New[Entity]Handler([nombre]Service)

   r.POST("/api/[nombre]", [nombre]Handler.Create)
   ```

   Backend Python (Django urls.py):
   ```python
   from .views import EntityViewSet
   router.register(r'entities', EntityViewSet, basename='entity')
   ```

   Backend TypeScript (NestJS module):
   ```typescript
   @Module({
     imports: [TypeOrmModule.forFeature([Entity])],
     controllers: [EntityController],
     providers: [EntityService, EntityRepositoryImpl],
   })
   export class EntityModule {}
   ```

   Frontend React (App.tsx):
   ```typescript
   <EntityProvider>
     <EntityList />
   </EntityProvider>
   ```

   Flutter (main.dart):
   ```dart
   BlocProvider(
     create: (context) => EntityBloc(
       EntityRepositoryImpl(EntityRemoteDataSource())
     ),
     child: EntityScreen(),
   )
   ```

OUTPUT ESPERADO:
Genera todos los archivos necesarios con código funcional y completo, siguiendo
EXACTAMENTE los patrones y convenciones del lenguaje/framework especificado.
El código debe estar listo para copiar y pegar.

IMPORTANTE:
- NO agregues features extras no solicitadas
- NO uses librerías no estándar sin justificación
- SÍ comenta el código donde sea necesario
- SÍ mantén consistencia con las convenciones del lenguaje
- SÍ respeta la estructura de archivos del framework
```

---

## 📝 Ejemplos de Uso por Ecosistema

### Backend Go: Dominio "Product"

```
[USA EL PROMPT DE ARRIBA Y COMPLETA:]

Lenguaje: Go
Tipo: Backend API
Framework: Gin

TAREA:
Genera un dominio completo llamado "product"

ENTIDAD:
- ID (int64)
- Name (string)
- Description (string)
- Price (float64)
- Stock (int)
- IsActive (bool)
- CreatedAt (time.Time)

CASOS DE USO:
- Create - Crear producto
- GetByID - Obtener por ID
- GetAll - Listar con filtros (name, isActive)
- Update - Actualizar
- Delete - Eliminar
- UpdateStock - Actualizar stock

VALIDACIONES:
- Name no vacío (min 3 caracteres)
- Price >= 0
- Stock >= 0
- Name único
```

---

### Backend Python: Feature "User"

```
[USA EL PROMPT DE ARRIBA Y COMPLETA:]

Lenguaje: Python
Tipo: Backend API
Framework: FastAPI

TAREA:
Genera un módulo completo llamado "user"

ENTIDAD:
- id (int)
- username (str)
- email (str)
- password_hash (str)
- is_active (bool)
- created_at (datetime)

CASOS DE USO:
- create - Crear usuario
- get_by_id - Obtener por ID
- get_by_email - Buscar por email
- get_all - Listar usuarios
- update - Actualizar
- deactivate - Desactivar usuario

VALIDACIONES:
- Email válido (formato)
- Username único
- Email único
- Password mínimo 8 caracteres
```

---

### Frontend React: Feature "Tasks"

```
[USA EL PROMPT DE ARRIBA Y COMPLETA:]

Lenguaje: TypeScript
Tipo: Frontend SPA
Framework: React

TAREA:
Genera un feature completo llamado "tasks"

ENTIDAD:
- id (string)
- title (string)
- description (string)
- status ('pending' | 'in_progress' | 'completed')
- priority ('low' | 'medium' | 'high')
- dueDate (Date)

CASOS DE USO:
- Create - Crear tarea
- GetAll - Listar tareas (con filtros por status)
- GetById - Obtener una tarea
- Update - Actualizar tarea
- UpdateStatus - Cambiar estado
- Delete - Eliminar tarea

COMPONENTES NECESARIOS:
- TaskList - Lista de tareas
- TaskForm - Formulario crear/editar
- TaskCard - Tarjeta individual
- TaskFilters - Filtros

VALIDACIONES:
- Title no vacío (min 3 caracteres)
- Status válido
- Priority válida
- DueDate no puede ser pasada
```

---

### Mobile Flutter: Feature "Orders"

```
[USA EL PROMPT DE ARRIBA Y COMPLETA:]

Lenguaje: Dart
Tipo: Mobile App
Framework: Flutter (BLoC)

TAREA:
Genera un feature completo llamado "orders"

ENTIDAD:
- id (int)
- orderNumber (String)
- userId (int)
- totalAmount (double)
- status (String) - 'pending', 'confirmed', 'delivered', 'cancelled'
- createdAt (DateTime)

CASOS DE USO:
- CreateOrder - Crear orden
- GetOrders - Listar órdenes
- GetOrderById - Obtener orden específica
- UpdateOrderStatus - Actualizar estado
- CancelOrder - Cancelar orden

SCREENS:
- OrderListScreen - Lista de órdenes
- OrderDetailScreen - Detalle de orden
- CreateOrderScreen - Crear orden

VALIDACIONES:
- TotalAmount > 0
- Status válido
- No se puede cancelar si status = 'delivered'
```

---

## 🔍 Checklist de Validación (Universal)

- [ ] ✅ Abstracciones definidas (interfaces/abstract classes)
- [ ] ✅ Implementaciones privadas/concretas separadas
- [ ] ✅ Constructores/Factories retornan abstracciones
- [ ] ✅ DTOs/Schemas con métodos de transformación
- [ ] ✅ Errores específicos del dominio
- [ ] ✅ Validaciones en service/use case layer
- [ ] ✅ Acceso a datos solo en repository/data layer
- [ ] ✅ UI/Controllers delgados (thin)
- [ ] ✅ Dependency Injection implementado
- [ ] ✅ Naming conventions del lenguaje respetadas
- [ ] ✅ Código comentado donde necesario
- [ ] ✅ Type safety (cuando aplique)

---

## 📚 Archivos de Referencia

El MCP debe leer estos archivos antes de generar:

1. `arquitectura_base.json` - Estructura y convenciones multi-lenguaje
2. `patrones_diseno.json` - Detalles de cada patrón
3. `guia_rapida.md` - Reglas universales

---

## 💡 Tips para Mejores Resultados

1. **Especifica el lenguaje/framework** claramente
2. **Sé específico** con los campos de la entidad
3. **Lista todas las validaciones** necesarias
4. **Define casos de uso claros** (no ambiguos)
5. **Menciona relaciones** si las hay
6. **Indica características especiales** (paginación, filtros, auth)
7. **Respeta las convenciones** del ecosistema elegido

---

## 🚀 Resultado Esperado

El MCP debe generar código completo y funcional adaptado al ecosistema especificado, listo para copiar y pegar, siguiendo exactamente los patrones definidos en la arquitectura base.
