# Prompt para Extraer Plantilla Genérica

> Usa este prompt para que un MCP extraiga una plantilla arquitectónica genérica desde **cualquier proyecto** (cualquier lenguaje, cualquier tipo)

---

## 🎯 Prompt Universal

```
Eres un arquitecto de software experto en abstracción de patrones arquitectónicos multi-lenguaje.

CONTEXTO:
Tengo un proyecto de software bien estructurado del cual quiero extraer su arquitectura
como plantilla genérica reutilizable para futuros proyectos similares.

INFORMACIÓN DEL PROYECTO:
Lenguaje: [Go / Python / TypeScript / Java / C# / Kotlin / Swift / etc.]
Tipo: [Backend API / Frontend SPA / Mobile App / CLI / Desktop / Microservicio / etc.]
Framework: [Gin, Django, React, Angular, Spring Boot, Flutter, etc.]
Arquitectura aparente: [Clean, MVC, MVVM, Hexagonal, Layered, Feature-based, etc.]

RUTA DEL PROYECTO:
[ESPECIFICA LA RUTA]

TAREA:
Analiza el proyecto y genera una PLANTILLA ARQUITECTÓNICA GENÉRICA.

PASOS DE ANÁLISIS:

1. IDENTIFICAR ARQUITECTURA
   - Tipo de arquitectura (Clean, MVC, MVVM, MVP, Hexagonal, Layered, DDD, etc.)
   - Principios aplicados (SOLID, DRY, KISS, YAGNI, etc.)
   - Capas / Módulos / Layers identificadas
   - Separación de responsabilidades

2. EXTRAER ESTRUCTURA
   Genera estructura GENÉRICA adaptada al ecosistema:

   **Para Backend**:
   - Capa de entrada (controllers, handlers, routes, endpoints)
   - Capa de lógica (services, use cases, business logic)
   - Capa de dominio (models, entities, domain objects)
   - Capa de datos (repositories, DAL, ORM, queries)
   - Infraestructura (config, external services, adapters)

   **Para Frontend**:
   - Presentación (components, views, pages, screens)
   - Lógica (services, hooks, composables, view models)
   - Estado (store, context, state management)
   - Data (API calls, data access, repositories)
   - UI (design system, shared components, layouts)

   **Para Mobile**:
   - Screens / Views / Pages
   - ViewModels / Presenters / Controllers
   - Services / Use Cases / Interactors
   - Models / Entities / Domain
   - Data / Repositories / Network

   **Para CLI**:
   - Commands (command definitions)
   - Handlers (command logic)
   - Services (business logic)
   - Utils (helpers, formatters)

   USA nombres genéricos: [Módulo], [Entidad], [Feature], [Componente], etc.

3. IDENTIFICAR PATRONES
   Busca estos patrones (si existen):

   **Creacionales**:
   - Factory, Builder, Singleton, Prototype, Abstract Factory

   **Estructurales**:
   - Repository, Adapter, Facade, Decorator, Proxy, Composite

   **Comportamiento**:
   - Strategy, Observer, Command, Template Method, State, Chain of Responsibility

   **Arquitectónicos**:
   - MVC, MVVM, MVP, Clean Architecture, Hexagonal
   - Service Layer, Repository Pattern, DTO
   - Dependency Injection, Inversion of Control

   **Específicos del ecosistema**:
   - Hooks (React), Composables (Vue), Decorators (Angular/NestJS)
   - Context Providers, Redux Pattern, BLoC (Flutter)
   - Middleware, Guards, Interceptors

4. EXTRAER CONVENCIONES
   Documenta:
   - Naming conventions (PascalCase, camelCase, kebab-case, snake_case)
   - Estructura de archivos (por feature, por tipo, por capa)
   - Organización de imports/exports
   - Manejo de errores
   - Testing patterns
   - Configuración (env, constants, config files)

5. FLUJO DE DATOS GENÉRICO
   Describe el flujo SIN lógica específica:

   **Backend**:
   Request → [Delivery Layer] → [Application Layer] → [Domain Layer] → [Data Layer] → Response

   **Frontend**:
   User Action → [UI Component] → [State/Service] → [Data Layer] → State Update → Re-render

   **Mobile**:
   User Interaction → [View] → [ViewModel/Presenter] → [Use Case] → [Repository] → Update UI

6. TECNOLOGÍAS COMO RECOMENDACIONES
   Lista tech stack como opciones, NO requisitos:
   - "Framework recomendado: X (alternativas: Y, Z)"
   - "ORM sugerido: A (alternativas: B, C)"
   - "State management: Redux (alternativas: Context API, MobX, Zustand)"

7. REGLAS Y MEJORES PRÁCTICAS
   Extrae reglas implícitas:
   - Qué va en cada capa/módulo
   - Qué puede/no puede hacer cada componente
   - Flujo de dependencias (quién depende de quién)
   - Separación de concerns
   - Testing strategies

OUTPUT FORMATO:

Genera archivo JSON estructurado:

{
  "nombre": "[Tipo de Arquitectura] para [Lenguaje/Framework]",
  "descripcion": "Plantilla arquitectónica para proyectos [tipo]",
  "ecosistema": {
    "lenguaje": "[Lenguaje principal]",
    "tipo_proyecto": "[Backend/Frontend/Mobile/CLI/Desktop/etc]",
    "frameworks_recomendados": ["...", "..."],
    "alternativas": ["...", "..."]
  },

  "arquitectura": {
    "tipo": "Clean Architecture / MVC / MVVM / etc",
    "principios": ["SOLID", "DRY", "..."],
    "capas": [
      {
        "nombre": "[Capa 1]",
        "ubicacion": "[ruta genérica]",
        "responsabilidad": "...",
        "ejemplos": ["..."],
        "que_contiene": ["..."],
        "que_NO_contiene": ["..."]
      }
      // más capas
    ]
  },

  "estructura_directorios": {
    // Estructura adaptada al ecosistema del proyecto
    // CON placeholders genéricos
  },

  "patrones_identificados": {
    "patron_X": {
      "categoria": "Creacional/Estructural/Comportamiento/Arquitectónico",
      "ubicacion": "[donde se aplica]",
      "implementacion": "...",
      "proposito": "...",
      "ventajas": ["...", "..."]
    }
    // más patrones
  },

  "convenciones": {
    "nombres": {
      "archivos": "...",
      "clases": "...",
      "funciones": "...",
      "variables": "..."
    },
    "organizacion": {
      "por_feature": true/false,
      "por_capa": true/false,
      "por_tipo": true/false
    },
    "imports": "...",
    "exports": "...",
    "errores": "..."
  },

  "flujo_datos": {
    "descripcion": "Flujo genérico de datos",
    "pasos": ["Paso 1", "Paso 2", "..."],
    "diagrama": "..."
  },

  "stack_tecnologico": {
    "recomendaciones": {
      "categoria_1": {
        "recomendado": "...",
        "alternativas": ["...", "..."],
        "razon": "..."
      }
    }
  },

  "mejores_practicas": {
    "hacer": ["...", "..."],
    "no_hacer": ["...", "..."],
    "testing": ["...", "..."],
    "seguridad": ["...", "..."]
  },

  "ejemplo_minimo": {
    "descripcion": "Estructura mínima para empezar",
    "archivos": ["...", "..."]
  }
}

INSTRUCCIONES CRÍTICAS:

1. **AGNÓSTICO DEL LENGUAJE**:
   - Adapta terminología al ecosistema (class/struct/component/widget/etc)
   - Respeta convenciones del lenguaje (PascalCase en C#, snake_case en Python)
   - Usa patrones idiomáticos del lenguaje

2. **GENERALIZACIÓN TOTAL**:
   - NO menciones entidades de negocio específicas (user, product, order)
   - USA: [Entity], [Feature], [Module], [Component], [Resource]
   - NO incluyas lógica de negocio específica

3. **ABSTRACCIÓN DE PATRONES**:
   Backend:
   - `UserRepository` → `[Entity]Repository`
   - `CreateUser()` → `Create[Entity]()`

   Frontend:
   - `UserList.tsx` → `[Entity]List.tsx`
   - `useUser()` → `use[Entity]()`

   Mobile:
   - `UserScreen` → `[Entity]Screen`
   - `UserViewModel` → `[Entity]ViewModel`

4. **ESTRUCTURA vs CONTENIDO**:
   ✅ BIEN: "Validar datos en capa de aplicación"
   ❌ MAL: "Validar que email sea único"

   ✅ BIEN: "DTO con método de transformación"
   ❌ MAL: "UserDTO con email y password"

5. **TECNOLOGÍAS FLEXIBLES**:
   ✅ "ORM recomendado: TypeORM (alt: Prisma, Sequelize)"
   ❌ "Debes usar TypeORM obligatoriamente"

   ✅ "State: Redux Toolkit (alt: Zustand, Jotai, Context)"
   ❌ "Solo Redux"

6. **EJEMPLOS GENÉRICOS**:

   Backend:
   ```
   // ✅ Genérico
   class EntityService {
       create(data: CreateEntityDTO): Entity
       findById(id: ID): Entity
   }

   // ❌ Específico
   class UserService {
       createUser(email, password)
   }
   ```

   Frontend:
   ```
   // ✅ Genérico
   function EntityList({ entities, onSelect }) {
       return entities.map(entity => <EntityCard key={entity.id} />)
   }

   // ❌ Específico
   function UserList({ users }) {
       return users.map(user => <div>{user.email}</div>)
   }
   ```

7. **ADAPTACIÓN POR TIPO**:

   **Si es Backend**:
   - Documenta capas: Delivery, Application, Domain, Data
   - Patrones: Repository, Service Layer, DTO, Dependency Injection

   **Si es Frontend**:
   - Documenta: Components, State, Services, Routing
   - Patrones: Container/Presentational, Custom Hooks, State Management

   **Si es Mobile**:
   - Documenta: Screens, ViewModels, Navigation, State
   - Patrones: MVVM, BLoC, Clean Architecture adaptado

VALIDACIÓN:

La plantilla debe ser:
- ✅ 100% genérica (sin negocio específico)
- ✅ Agnóstica del dominio
- ✅ Adaptada al ecosistema del lenguaje
- ✅ Con ejemplos abstractos
- ✅ Tecnologías como opciones
- ✅ Aplicable a proyectos similares
- ✅ Idiomática al lenguaje/framework

PREGUNTAS CLAVE:

1. ¿Qué convenciones del lenguaje/framework se siguen?
2. ¿Qué patrones son UNIVERSALES vs ESPECÍFICOS del proyecto?
3. ¿Qué es ESENCIAL de la arquitectura?
4. ¿Qué tecnologías son INTERCAMBIABLES?
5. ¿Cómo se adaptaría esto a un proyecto similar pero diferente negocio?

OUTPUT FINAL:

Un archivo JSON estructurado, genérico, agnóstico del dominio de negocio,
adaptado al ecosistema del lenguaje, con patrones abstraídos y reutilizable
para cualquier proyecto del mismo tipo.
```

---

## 📝 Ejemplos por Tipo de Proyecto

### Backend (Go/Python/Node/Java)

```
Lenguaje: Go
Tipo: Backend REST API
Framework: Gin
Ruta: /backend-project

→ Output: Clean Architecture para Go APIs
  - Capas: Delivery (HTTP), Application (Services), Domain, Data (Repositories)
  - Patrones: Repository, Service Layer, Dependency Injection
  - Convenciones: PascalCase types, camelCase funcs, package por feature
```

### Frontend (React/Vue/Angular)

```
Lenguaje: TypeScript
Tipo: Frontend SPA
Framework: React
Ruta: /frontend-app

→ Output: Feature-based Architecture para React
  - Estructura: features/, components/, services/, hooks/, store/
  - Patrones: Custom Hooks, Context Providers, Component Composition
  - State: Redux Toolkit pattern
```

### Mobile (React Native/Flutter)

```
Lenguaje: Dart
Tipo: Mobile App
Framework: Flutter
Ruta: /mobile-app

→ Output: BLoC Architecture para Flutter
  - Capas: Presentation (Screens/Widgets), Business Logic (BLoCs), Data (Repositories)
  - Patrones: BLoC, Repository, Dependency Injection (get_it)
  - Convenciones: snake_case, barrel exports
```

---

## 🎯 Comparación: Específico vs Genérico

| Específico (❌) | Genérico (✅) |
|----------------|--------------|
| UserService | [Entity]Service |
| createUser(email, pwd) | create[Entity](data) |
| UserList.tsx | [Entity]List.tsx |
| user_controller.py | [entity]_controller.py |
| validateEmail() | validate[Field]() |
| fetchUsers() | fetch[Entities]() |

---

## 💡 Tips

1. **Analiza 3+ módulos** similares para identificar patrones
2. **Respeta idioms** del lenguaje (no forces Go patterns en Python)
3. **Abstrae dominio** pero mantén convenciones técnicas
4. **Documenta razones** de decisiones arquitectónicas
5. **Incluye trade-offs** de cada patrón

---

## 🔍 Checklist

- [ ] Arquitectura identificada correctamente
- [ ] Estructura genérica (sin entidades específicas)
- [ ] Patrones abstraídos
- [ ] Convenciones del lenguaje respetadas
- [ ] Tecnologías como opciones, no requisitos
- [ ] Ejemplos con placeholders
- [ ] Aplicable a otros proyectos del mismo tipo
- [ ] Idiomático al ecosistema

---

**Úsalo para**: Backend APIs, Frontend SPAs, Mobile Apps, CLIs, Desktop Apps, Microservicios, Librerías, y cualquier proyecto con arquitectura definida.
