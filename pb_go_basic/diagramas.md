# Diagramas de Arquitectura

## 1. Estructura de Capas

```
┌─────────────────────────────────────────────────────────────┐
│                    CAPA DE PRESENTACIÓN                      │
│                    (internal/http/)                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Handlers   │  │ Middleware  │  │   Router    │         │
│  │   (HTTP)    │  │  (Auth)     │  │  (Rutas)    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└───────────────────────────┬─────────────────────────────────┘
                            │ DTOs
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  CAPA DE APLICACIÓN                          │
│                  (internal/domain/*/service.go)              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Service Layer (Lógica de Negocio)                   │   │
│  │  - Validaciones                                       │   │
│  │  - Reglas de negocio                                  │   │
│  │  - Orquestación                                       │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────┘
                            │ Models
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                     CAPA DE DOMINIO                          │
│                  (internal/domain/)                          │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐              │
│  │  Models   │  │   DTOs    │  │  Errors   │              │
│  │ (User,    │  │ (UserDTO) │  │ (Custom)  │              │
│  │  Transac) │  │           │  │           │              │
│  └───────────┘  └───────────┘  └───────────┘              │
└───────────────────────────┬─────────────────────────────────┘
                            │ Repository Interface
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  CAPA DE PERSISTENCIA                        │
│              (internal/domain/*/repository.go)               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Repository Pattern                                   │   │
│  │  - Criteria (filtros)                                 │   │
│  │  - FindOptions (paginación)                           │   │
│  │  - CRUD operations                                    │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────┘
                            │ Bob ORM
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                 CAPA DE INFRAESTRUCTURA                      │
│                        (pkg/)                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │    DB    │  │  Logger  │  │   JWT    │  │   WAHA   │   │
│  │(Postgres)│  │          │  │          │  │ (WhatsApp)│   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Flujo de Request Completo

```
┌─────────────────────────────────────────────────────────────┐
│  1. Cliente HTTP                                             │
│     POST /api/user/register                                  │
│     Body: {"email": "user@example.com", "phone": "+56..."}  │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  2. Gin Router (router.go)                                   │
│     Encuentra ruta: user.POST("/register", handler.Register) │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  3. Middleware (si aplica)                                   │
│     - APIKeyMiddleware() → Valida X-API-Key                  │
│     - JWTMiddleware() → Valida Bearer token                  │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  4. Handler (user_handler.go)                                │
│     func (h *UserHandler) Register(c *gin.Context)          │
│     - c.BindJSON(&userDTO)                                   │
│     - h.service.Register(userDTO)                            │
└─────────────────┬───────────────────────────────────────────┘
                  │ userDTO
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  5. Service (user/service.go)                                │
│     func (s *service) Register(dto UserDTO) (*User, error)  │
│     - validateEmail(dto.Email)                               │
│     - validatePhone(dto.Phone)                               │
│     - s.repo.Exists(&Criteria{Email: dto.Email})            │
│     - s.repo.InsertOne(dto.ToModel())                        │
└─────────────────┬───────────────────────────────────────────┘
                  │ Criteria / User
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  6. Repository (user/repository.go)                          │
│     func (r *repository) InsertOne(user User) (*User, error)│
│     - criteriaToWhere() → Construye WHERE clause             │
│     - models.Users.Insert().One()                            │
└─────────────────┬───────────────────────────────────────────┘
                  │ Bob ORM Query
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  7. Bob ORM (pkg/db/)                                        │
│     INSERT INTO users (email, phone) VALUES ($1, $2)         │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  8. PostgreSQL                                               │
│     Ejecuta INSERT y retorna ID generado                     │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ↓ (Response flow - mismas capas en reversa)
┌─────────────────────────────────────────────────────────────┐
│  9. Handler → JSON Response                                  │
│     c.JSON(201, UserRes{Status: 201, Body: &user})          │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  10. Cliente recibe                                          │
│      {"status": 201, "msg": "Usuario creado", "body": {...}}│
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Dependency Injection Graph

```
                    ┌──────────────┐
                    │   main.go    │
                    │  (cmd/api/)  │
                    └──────┬───────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ↓                     ↓
        ┌─────────────┐      ┌─────────────┐
        │   Config    │      │   Postgres  │
        │   Load()    │      │   NewDB()   │
        └─────────────┘      └──────┬──────┘
                                    │
                          ┌─────────┼─────────┐
                          │         │         │
                          ↓         ↓         ↓
                    ┌─────────┬─────────┬─────────┐
                    │UserRepo │BankRepo │TransRepo│
                    └────┬────┴────┬────┴────┬────┘
                         │         │         │
                         ↓         ↓         ↓
                    ┌─────────┬─────────┬─────────┐
                    │UserServ │BankServ │TransServ│
                    └────┬────┴────┬────┴────┬────┘
                         │         │         │
                         ↓         ↓         ↓
                    ┌─────────┬─────────┬─────────┐
                    │UserHand │BankHand │TransHand│
                    └────┬────┴────┬────┴────┬────┘
                         │         │         │
                         └─────────┼─────────┘
                                   │
                                   ↓
                           ┌─────────────┐
                           │   Router    │
                           │  (Gin)      │
                           └─────────────┘
```

---

## 4. Patrón Repository Detallado

```
┌─────────────────────────────────────────────────────────────┐
│                     SERVICE LAYER                            │
│                                                              │
│  service.Register(dto UserDTO) {                             │
│      repo.Exists(&Criteria{Email: dto.Email})               │
│      repo.InsertOne(dto.ToModel())                           │
│  }                                                           │
│                                                              │
│  Conoce: Repository Interface                                │
│  No conoce: SQL, Bob ORM, PostgreSQL                         │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          │ Llama a interfaz
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                  REPOSITORY INTERFACE                        │
│                                                              │
│  type Repository interface {                                 │
│      InsertOne(User) (*User, error)                          │
│      GetUser(*Criteria) (*User, error)                       │
│      Exists(*Criteria) (bool, error)                         │
│      Update(*Criteria, UpdateData) error                     │
│      Delete(*Criteria) error                                 │
│  }                                                           │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          │ Implementado por
                          ↓
┌─────────────────────────────────────────────────────────────┐
│              REPOSITORY IMPLEMENTATION                       │
│                                                              │
│  type repository struct {                                    │
│      db *bob.DB                                              │
│  }                                                           │
│                                                              │
│  func (r *repository) InsertOne(user User) (*User, error) { │
│      set := &models.UserSetter{...}                          │
│      return models.Users.Insert(set).One(ctx, r.db)         │
│  }                                                           │
│                                                              │
│  Conoce: Bob ORM, SQL, PostgreSQL                            │
│  No expuesto: Detalles de implementación privados            │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ↓
                  ┌──────────────┐
                  │  PostgreSQL  │
                  └──────────────┘
```

---

## 5. Criteria Pattern en Acción

```
┌─────────────────────────────────────────────────────────────┐
│  CASO 1: Buscar por Email                                    │
│                                                              │
│  criteria := &Criteria{Email: "user@example.com"}           │
│  repo.GetUser(criteria)                                      │
│                                                              │
│  → WHERE email = 'user@example.com'                          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  CASO 2: Buscar usuarios activos                             │
│                                                              │
│  criteria := &Criteria{IsActive: utils.Bool(true)}          │
│  repo.GetUsers(criteria)                                     │
│                                                              │
│  → WHERE isActive = true                                     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  CASO 3: Combinación múltiple                                │
│                                                              │
│  criteria := &Criteria{                                      │
│      Email: "user@example.com",                              │
│      IsActive: utils.Bool(true),                             │
│  }                                                           │
│  repo.GetUser(criteria)                                      │
│                                                              │
│  → WHERE email = 'user@example.com' AND isActive = true      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  CONVERSIÓN INTERNA (criteriaToWhere)                        │
│                                                              │
│  func criteriaToWhere(c *Criteria) []bob.Mod[...] {         │
│      mods := []bob.Mod[...]{}                                │
│                                                              │
│      if c.Email != "" {                                      │
│          mods = append(mods, WHERE email = $1)               │
│      }                                                       │
│      if c.IsActive != nil {                                  │
│          mods = append(mods, WHERE isActive = $2)            │
│      }                                                       │
│                                                              │
│      return mods  // Bob ORM los combina con AND             │
│  }                                                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Builder Pattern (FindOptions)

```
┌─────────────────────────────────────────────────────────────┐
│  CONSTRUCCIÓN FLUIDA                                         │
│                                                              │
│  opts := NewFindOptions()                                    │
│            ↓                                                 │
│          .Limit(10)                                          │
│            ↓                                                 │
│          .Skip(20)                                           │
│            ↓                                                 │
│          .Sort(Sort{Date: DESC})                             │
│                                                              │
│  // Resultado: {limit: 10, skip: 20, sort: {Date: DESC}}    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  CONVERSIÓN A SQL                                            │
│                                                              │
│  repo.GetTransactions(criteria, opts)                        │
│                                                              │
│  ↓                                                           │
│                                                              │
│  SELECT * FROM transactions                                  │
│  WHERE ... (criteria)                                        │
│  ORDER BY date DESC                                          │
│  LIMIT 10                                                    │
│  OFFSET 20                                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Estructura de Directorios con Patrones

```
backend-wsp/
│
├── cmd/api/
│   └── main.go                    ← Constructor Pattern (inicializa todo)
│
├── internal/
│   ├── config/
│   │   └── config.go              ← Config Pattern
│   │
│   ├── http/
│   │   ├── handler/
│   │   │   ├── user_handler.go    ← Adapter Pattern (HTTP → Service)
│   │   │   ├── transaction_handler.go
│   │   │   └── responses.go       ← DTO Pattern (Response DTOs)
│   │   │
│   │   ├── middleware/
│   │   │   ├── auth.go            ← Chain of Responsibility
│   │   │   └── api_key.go
│   │   │
│   │   └── router.go              ← Dependency Injection (manual)
│   │
│   └── domain/
│       ├── user/
│       │   ├── model.go           ← Domain Model
│       │   ├── dto.go             ← DTO Pattern + Mapper Pattern
│       │   ├── errors.go          ← Custom Error Pattern
│       │   ├── service.go         ← Service Layer Pattern
│       │   └── repository.go      ← Repository + Criteria + Constructor
│       │
│       └── transaction/
│           ├── model.go
│           ├── dto.go
│           ├── errors.go
│           ├── service.go
│           └── repository.go      ← + Builder Pattern (FindOptions)
│
├── pkg/
│   ├── db/
│   │   ├── postgres.go            ← Adapter Pattern (Bob → Postgres)
│   │   ├── models/                ← ORM Generated Models
│   │   └── factory/               ← Factory Pattern
│   │
│   ├── jwt/
│   │   └── jwt.go                 ← Strategy Pattern (auth strategy)
│   │
│   └── waha/
│       └── client.go              ← Adapter Pattern (External API)
│
└── utils/
    └── errors.go                  ← Custom Error Pattern
```

---

## 8. Comparación: Con vs Sin Patrones

### ❌ Sin Patrones (Anti-patrón)
```go
// Handler hace TODO
func Register(c *gin.Context) {
    var data map[string]string
    c.BindJSON(&data)

    // SQL directo en el handler
    _, err := db.Exec(
        "INSERT INTO users (email, phone) VALUES ($1, $2)",
        data["email"],
        data["phone"],
    )

    if err != nil {
        c.JSON(500, "Error")
        return
    }

    c.JSON(201, "OK")
}

// Problemas:
// - SQL mezclado con HTTP
// - Sin validaciones
// - Imposible testear
// - Cambiar BD afecta handlers
```

### ✅ Con Patrones (Tu Proyecto)
```go
// 1. Handler (solo adapta HTTP)
func (h *UserHandler) Register(c *gin.Context) {
    var userDTO user.UserDTO
    c.BindJSON(&userDTO)

    user, err := h.service.Register(userDTO)
    if err != nil {
        c.JSON(500, ProblemDetails{...})
        return
    }

    c.JSON(201, UserRes{Body: &user})
}

// 2. Service (lógica de negocio)
func (s *service) Register(dto UserDTO) (*User, error) {
    s.validateEmail(dto.Email)
    s.repo.Exists(&Criteria{Email: dto.Email})
    return s.repo.InsertOne(dto.ToModel())
}

// 3. Repository (persistencia)
func (r *repository) InsertOne(user User) (*User, error) {
    return models.Users.Insert(...).One(ctx, r.db)
}

// Ventajas:
// ✅ Separación de responsabilidades
// ✅ Testeable (mocks)
// ✅ Reutilizable
// ✅ Mantenible
```

---

## 9. Mapa Mental de Patrones

```
                    ┌─────────────────┐
                    │  TU PROYECTO    │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ↓                    ↓                    ↓
  ┌──────────┐        ┌──────────┐        ┌──────────┐
  │Creación  │        │Estructura│        │Comportam.│
  └────┬─────┘        └────┬─────┘        └────┬─────┘
       │                   │                    │
       ├─Constructor       ├─Repository         ├─Strategy
       ├─Factory           ├─Adapter            ├─Criteria
       └─Builder           ├─DTO                ├─Template
                           └─Mapper             ├─Chain
                                                └─UnitOfWork
```
