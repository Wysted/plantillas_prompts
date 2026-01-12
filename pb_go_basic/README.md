# Clean Architecture - Plantilla Base para Go

> Arquitectura genérica para iniciar proyectos backend desde cero

Esta documentación describe la arquitectura base **independiente de cualquier proyecto específico**. Úsala como referencia para iniciar nuevos proyectos o para alimentar a un MCP/LLM con las convenciones arquitectónicas.

---

## 📋 Índice

1. [¿Qué es esta arquitectura?](#qué-es-esta-arquitectura)
2. [Estructura de directorios](#estructura-de-directorios)
3. [Las 5 capas](#las-5-capas)
4. [Patrones fundamentales](#patrones-fundamentales)
5. [Flujo de datos](#flujo-de-datos)
6. [Cómo implementar](#cómo-implementar)
7. [Reglas de oro](#reglas-de-oro)
8. [Ejemplo práctico](#ejemplo-práctico)

---

## 🎯 ¿Qué es esta arquitectura?

**Clean Architecture** aplicada a Go con enfoque en **API REST**. Combina:

- **Layered Architecture** (arquitectura en capas)
- **Repository Pattern** (abstracción de datos)
- **Service Layer Pattern** (lógica de negocio)
- **Dependency Injection** (bajo acoplamiento)

### Principios SOLID aplicados

| Principio | Aplicación |
|-----------|------------|
| **S**ingle Responsibility | Cada capa tiene una responsabilidad única |
| **O**pen/Closed | Extensible via interfaces |
| **L**iskov Substitution | Repositorios son intercambiables |
| **I**nterface Segregation | Interfaces pequeñas y específicas |
| **D**ependency Inversion | Dependencias hacia abstracciones |

---

## 📁 Estructura de Directorios

```
proyecto/
├── cmd/
│   └── api/
│       └── main.go                 # Punto de entrada
│
├── internal/                       # Código privado
│   ├── config/
│   │   └── config.go               # Configuración
│   │
│   ├── domain/                     # ⭐ Núcleo del negocio
│   │   └── [entidad]/              # Un paquete por entidad
│   │       ├── model.go            # Entidad del dominio
│   │       ├── dto.go              # Data Transfer Objects
│   │       ├── errors.go           # Errores del dominio
│   │       ├── repository.go       # Interface + Impl de persistencia
│   │       └── service.go          # Interface + Impl de lógica negocio
│   │
│   └── http/                       # Adaptador HTTP
│       ├── handler/
│       │   └── [entidad]_handler.go # Controladores
│       ├── middleware/
│       │   ├── auth.go             # Autenticación
│       │   └── api_key.go          # Validación API Key
│       ├── utils/
│       │   └── claims.go           # Utilidades HTTP
│       └── router.go               # ⭐ Inyección dependencias
│
├── pkg/                            # Código reutilizable
│   ├── db/
│   │   └── postgres.go             # Conexión BD
│   ├── logger/
│   │   └── logger.go               # Sistema logging
│   └── jwt/
│       └── jwt.go                  # Manejo JWT
│
├── utils/                          # Utilidades generales
│   ├── errors.go
│   ├── ptr.go                      # Helpers de punteros
│   └── func.go                     # Map, Filter, etc.
│
├── go.mod
└── .env
```

---

## 🏗️ Las 5 Capas

```
┌──────────────────────────────────────┐
│    1. PRESENTACIÓN (HTTP)            │  ← Handlers, Middleware
│    internal/http/                    │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    2. APLICACIÓN (Services)          │  ← Lógica de negocio
│    internal/domain/*/service.go      │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    3. DOMINIO (Entities)             │  ← Models, DTOs, Errors
│    internal/domain/*/model.go        │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    4. PERSISTENCIA (Repositories)    │  ← Abstracción de datos
│    internal/domain/*/repository.go   │
└────────────┬─────────────────────────┘
             ↓
┌──────────────────────────────────────┐
│    5. INFRAESTRUCTURA (DB, APIs)     │  ← pkg/, utils/
└──────────────────────────────────────┘
```

### 1. Capa de Presentación

**Ubicación**: `internal/http/`

**Responsabilidad**: Adaptar HTTP ↔ Aplicación

**Contiene**:
- `handler/` - Controladores (reciben requests, retornan responses)
- `middleware/` - Autenticación, validación, CORS
- `router.go` - Configuración de rutas + Dependency Injection

**Qué hace**:
- ✅ Parsear JSON a DTOs
- ✅ Llamar al service correspondiente
- ✅ Convertir respuestas a JSON
- ✅ Manejar códigos HTTP

**Qué NO hace**:
- ❌ Validar reglas de negocio
- ❌ Acceder a la BD directamente
- ❌ Contener lógica de negocio

---

### 2. Capa de Aplicación

**Ubicación**: `internal/domain/*/service.go`

**Responsabilidad**: Orquestar la lógica de negocio

**Contiene**:
- `Service` interface
- `service` struct (implementación privada)
- Casos de uso (Create, Update, Delete, etc.)

**Qué hace**:
- ✅ Validar datos (formato, reglas)
- ✅ Aplicar reglas de negocio
- ✅ Orquestar llamadas a repositorios
- ✅ Coordinar transacciones

**Qué NO hace**:
- ❌ Conocer HTTP (no sabe de requests)
- ❌ Hacer SQL directo
- ❌ Conocer qué BD se usa

---

### 3. Capa de Dominio

**Ubicación**: `internal/domain/*/model.go`, `dto.go`, `errors.go`

**Responsabilidad**: Representar conceptos del negocio

**Contiene**:
- `model.go` - Entidades (User, Product, Order)
- `dto.go` - Objetos de transferencia
- `errors.go` - Errores específicos

**Qué contiene**:
- ✅ Estructuras de datos
- ✅ Métodos de transformación (ToModel, ToDTO)
- ✅ Errores semánticos

**Qué NO contiene**:
- ❌ Lógica de persistencia
- ❌ Lógica HTTP
- ❌ Detalles de frameworks

---

### 4. Capa de Persistencia

**Ubicación**: `internal/domain/*/repository.go`

**Responsabilidad**: Abstraer acceso a datos

**Contiene**:
- `Repository` interface
- `Criteria` struct (filtros de búsqueda)
- `repository` struct (implementación privada)
- Métodos CRUD

**Qué hace**:
- ✅ Definir contratos de persistencia
- ✅ Ejecutar queries SQL (via ORM)
- ✅ Mapear modelos BD ↔ dominio
- ✅ Manejar transacciones

**Qué NO hace**:
- ❌ Validar reglas de negocio
- ❌ Conocer HTTP
- ❌ Exponer SQL al service

---

### 5. Capa de Infraestructura

**Ubicación**: `pkg/`, `utils/`

**Responsabilidad**: Implementaciones técnicas y servicios externos

**Contiene**:
- `pkg/db/` - Conexión a BD
- `pkg/logger/` - Sistema de logs
- `pkg/jwt/` - Manejo de tokens
- `utils/` - Helpers generales

---

## 🎨 Patrones Fundamentales

### 1. Repository Pattern

**Problema**: SQL mezclado en la lógica de negocio

**Solución**: Abstraer persistencia con interfaces

```go
// Interface (contrato)
type Repository interface {
    InsertOne(User) (*User, error)
    GetByID(int64) (*User, error)
    GetAll(*Criteria) ([]User, error)
    Update(*Criteria, UpdateData) error
    Delete(*Criteria) error
}

// Implementación privada
type repository struct {
    db *sql.DB
}

func NewRepository(db *sql.DB) Repository {
    return &repository{db: db}
}
```

**Ventaja**: Cambiar de PostgreSQL a MongoDB solo afecta el repository

---

### 2. Service Layer Pattern

**Problema**: Lógica de negocio esparcida

**Solución**: Centralizar en services

```go
// Interface
type Service interface {
    Create(dto UserDTO) (*User, error)
    GetByID(int64) (*User, error)
}

// Implementación
type service struct {
    repo Repository
}

func NewService(repo Repository) Service {
    return &service{repo: repo}
}

func (s *service) Create(dto UserDTO) (*User, error) {
    // 1. Validar
    if err := s.validateEmail(dto.Email); err != nil {
        return nil, err
    }

    // 2. Regla de negocio
    exists, _ := s.repo.Exists(&Criteria{Email: dto.Email})
    if exists {
        return nil, ErrUserAlreadyExists
    }

    // 3. Persistir
    return s.repo.InsertOne(dto.ToModel())
}
```

---

### 3. Criteria Pattern

**Problema**: Necesitas múltiples métodos FindByX()

**Solución**: Queries dinámicas con un solo método

```go
type Criteria struct {
    ID       int64
    Email    string
    IsActive *bool  // nil = no filtrar
}

// Un solo método para múltiples búsquedas
repo.GetAll(&Criteria{Email: "user@example.com"})
repo.GetAll(&Criteria{IsActive: utils.Bool(true)})
repo.GetAll(&Criteria{Email: "user@example.com", IsActive: utils.Bool(true)})
```

---

### 4. DTO Pattern

**Problema**: Acoplar HTTP con dominio

**Solución**: Objetos de transferencia

```go
// DTO (capa HTTP)
type UserDTO struct {
    Email string `json:"email"`
    Phone string `json:"phone"`
}

func (dto *UserDTO) ToModel() User {
    return User{
        Email: dto.Email,
        Phone: dto.Phone,
    }
}

// Uso en handler
var dto UserDTO
c.BindJSON(&dto)
user, _ := h.service.Create(dto)
```

---

### 5. Dependency Injection

**Problema**: Alto acoplamiento entre componentes

**Solución**: Inyectar dependencias en constructores

```go
// router.go
func NewRouter(db *sql.DB) *gin.Engine {
    // 1. Crear repositorios
    userRepo := user.NewRepository(db)

    // 2. Inyectar en services
    userService := user.NewService(userRepo)

    // 3. Inyectar en handlers
    userHandler := handler.NewUserHandler(userService)

    // 4. Registrar rutas
    r := gin.New()
    r.POST("/users", userHandler.Create)

    return r
}
```

**Ventaja**: Testing con mocks, bajo acoplamiento

---

## 🔄 Flujo de Datos

```
1. HTTP Request
   POST /api/users
   Body: {"email": "user@example.com", "phone": "+56..."}
   ↓
2. Router
   Encuentra ruta → UserHandler.Create
   ↓
3. Middleware (si aplica)
   JWTMiddleware() → Valida token → next()
   ↓
4. Handler
   - c.BindJSON(&dto)
   - h.service.Create(dto)
   ↓
5. Service
   - validateEmail()
   - repo.Exists() → verificar duplicados
   - repo.InsertOne()
   ↓
6. Repository
   - Construye query SQL
   - db.Exec("INSERT INTO...")
   ↓
7. Database
   - Ejecuta INSERT
   - Retorna ID generado
   ↓
   (Respuesta sube por el mismo camino)
   ↓
8. Handler
   - c.JSON(201, UserResponse{...})
   ↓
9. Cliente recibe
   {"status": 201, "data": {...}}
```

---

## 🚀 Cómo Implementar

### Paso 1: Setup Inicial

```bash
# Crear proyecto
mkdir mi-proyecto && cd mi-proyecto
go mod init github.com/usuario/mi-proyecto

# Crear estructura
mkdir -p cmd/api
mkdir -p internal/{config,domain,http/{handler,middleware,utils}}
mkdir -p pkg/{db,logger,jwt}
mkdir -p utils
```

### Paso 2: Implementar un Dominio (ej: User)

#### 2.1. Model (`internal/domain/user/model.go`)

```go
package user

import "time"

type User struct {
    ID        int64     `json:"id"`
    Email     string    `json:"email"`
    Phone     string    `json:"phone"`
    IsActive  bool      `json:"isActive"`
    CreatedAt time.Time `json:"createdAt"`
}
```

#### 2.2. DTO (`internal/domain/user/dto.go`)

```go
package user

type UserDTO struct {
    Email string `json:"email"`
    Phone string `json:"phone"`
}

func (dto *UserDTO) ToModel() User {
    return User{
        Email: dto.Email,
        Phone: dto.Phone,
    }
}
```

#### 2.3. Errors (`internal/domain/user/errors.go`)

```go
package user

import "errors"

var (
    ErrUserNotFound      = errors.New("user not found")
    ErrUserAlreadyExists = errors.New("user already exists")
    ErrInvalidEmail      = errors.New("invalid email format")
)
```

#### 2.4. Repository (`internal/domain/user/repository.go`)

```go
package user

import "database/sql"

// Interface (pública)
type Repository interface {
    InsertOne(User) (*User, error)
    GetByID(int64) (*User, error)
    GetAll(*Criteria) ([]User, error)
    Exists(*Criteria) (bool, error)
}

// Criteria para búsquedas dinámicas
type Criteria struct {
    ID       int64
    Email    string
    IsActive *bool
}

// Implementación (privada)
type repository struct {
    db *sql.DB
}

// Constructor
func NewRepository(db *sql.DB) Repository {
    return &repository{db: db}
}

// Implementar métodos...
func (r *repository) InsertOne(user User) (*User, error) {
    query := `INSERT INTO users (email, phone) VALUES ($1, $2) RETURNING id, created_at`
    err := r.db.QueryRow(query, user.Email, user.Phone).Scan(&user.ID, &user.CreatedAt)
    if err != nil {
        return nil, err
    }
    return &user, nil
}

// ... más métodos
```

#### 2.5. Service (`internal/domain/user/service.go`)

```go
package user

import "regexp"

// Interface (pública)
type Service interface {
    Create(UserDTO) (*User, error)
    GetByID(int64) (*User, error)
    GetAll() ([]User, error)
}

// Implementación (privada)
type service struct {
    repo Repository
}

// Constructor
func NewService(repo Repository) Service {
    return &service{repo: repo}
}

// Casos de uso
func (s *service) Create(dto UserDTO) (*User, error) {
    // 1. Validar
    if !isValidEmail(dto.Email) {
        return nil, ErrInvalidEmail
    }

    // 2. Regla de negocio: no duplicados
    exists, _ := s.repo.Exists(&Criteria{Email: dto.Email})
    if exists {
        return nil, ErrUserAlreadyExists
    }

    // 3. Persistir
    return s.repo.InsertOne(dto.ToModel())
}

var emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$`)

func isValidEmail(email string) bool {
    return emailRegex.MatchString(email)
}
```

#### 2.6. Handler (`internal/http/handler/user_handler.go`)

```go
package handler

import (
    "net/http"
    "mi-proyecto/internal/domain/user"
    "github.com/gin-gonic/gin"
)

type UserHandler struct {
    service user.Service
}

func NewUserHandler(service user.Service) *UserHandler {
    return &UserHandler{service: service}
}

func (h *UserHandler) Create(c *gin.Context) {
    var dto user.UserDTO

    // 1. Parsear request
    if err := c.BindJSON(&dto); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid request"})
        return
    }

    // 2. Llamar service
    result, err := h.service.Create(dto)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    // 3. Retornar response
    c.JSON(http.StatusCreated, result)
}
```

#### 2.7. Router (`internal/http/router.go`)

```go
package http

import (
    "database/sql"
    "mi-proyecto/internal/domain/user"
    "mi-proyecto/internal/http/handler"
    "github.com/gin-gonic/gin"
)

func NewRouter(db *sql.DB) *gin.Engine {
    r := gin.Default()

    // Dependency Injection
    userRepo := user.NewRepository(db)
    userService := user.NewService(userRepo)
    userHandler := handler.NewUserHandler(userService)

    // Rutas
    users := r.Group("/api/users")
    {
        users.POST("", userHandler.Create)
        users.GET("/:id", userHandler.GetByID)
        users.GET("", userHandler.GetAll)
    }

    return r
}
```

#### 2.8. Main (`cmd/api/main.go`)

```go
package main

import (
    "database/sql"
    "log"
    "mi-proyecto/internal/http"
    _ "github.com/lib/pq"
)

func main() {
    // 1. Conectar BD
    db, err := sql.Open("postgres", "postgres://user:pass@localhost/dbname?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // 2. Crear router
    router := http.NewRouter(db)

    // 3. Iniciar servidor
    log.Println("Server running on :8080")
    router.Run(":8080")
}
```

---

## ⚖️ Reglas de Oro

### ✅ DO

1. **Usa interfaces** para Services y Repositories
2. **Inyecta dependencias** via constructores
3. **Valida en Service Layer**, no en handlers
4. **Usa DTOs** para desacoplar HTTP del dominio
5. **Define errores** específicos en `errors.go`
6. **Retorna interfaces** desde constructores, no structs
7. **Usa Criteria** para queries dinámicas
8. **Mantén handlers delgados** (thin controllers)

### ❌ DON'T

1. **No pongas SQL en handlers**
2. **No pongas lógica de negocio en repositories**
3. **No saltees capas** (handler → repository directo)
4. **No uses `panic()`** para errores esperados
5. **No retornes modelos de BD** directamente en HTTP
6. **No mezcles responsabilidades** de capas

---

## 📚 Ejemplo Práctico: Crear Entidad "Product"

### Checklist

- [ ] Crear `internal/domain/product/`
- [ ] Implementar `model.go` → `type Product struct`
- [ ] Implementar `dto.go` → `type ProductDTO struct` + `ToModel()`
- [ ] Implementar `errors.go` → `var ErrProductNotFound`
- [ ] Implementar `repository.go` → Interface + Impl + Constructor
- [ ] Implementar `service.go` → Interface + Impl + Constructor
- [ ] Crear `handler/product_handler.go` → Handler + Constructor
- [ ] Registrar en `router.go` → DI + Rutas

### Estructura Final

```
internal/domain/product/
├── model.go         ✅
├── dto.go           ✅
├── errors.go        ✅
├── repository.go    ✅
└── service.go       ✅

internal/http/handler/
└── product_handler.go ✅

internal/http/
└── router.go        ✅ (agregar DI y rutas)
```

---

## 🎓 Recursos Adicionales

### Lecturas Recomendadas

- **Clean Architecture** - Robert C. Martin
- **Domain-Driven Design** - Eric Evans
- **Patterns of Enterprise Application Architecture** - Martin Fowler
- **Effective Go** - Go Documentation

### Tecnologías Populares

| Componente | Opciones |
|------------|----------|
| HTTP Framework | gin, fiber, echo, chi |
| ORM | GORM, sqlx, ent, bob |
| Validación | validator, ozzo-validation |
| Testing | testify, gomock |
| Config | viper, godotenv |

---

## 🆘 FAQ

### ¿Cuándo crear un nuevo Service?

Por cada **entidad/agregado** del dominio (User, Product, Order).

### ¿Cuándo usar Criteria?

Cuando necesitas **múltiples filtros opcionales** en una búsqueda.

### ¿Cuándo usar Transaction?

Para operaciones que deben ser **atómicas** (múltiples inserts/updates relacionados).

### ¿Dónde va la validación?

**Service Layer**. Los handlers solo validan que el JSON sea válido.

### ¿Puedo usar esta arquitectura sin ORM?

Sí, con `database/sql` puro o `sqlx`. El repository abstrae la implementación.

---

## 📝 Próximos Pasos

1. Lee `arquitectura_base.json` para detalles técnicos
2. Revisa los patrones aplicados
3. Implementa tu primer dominio siguiendo el ejemplo
4. Adapta según necesidades específicas

---

**Versión**: 1.0
**Última actualización**: 2026-01-11
**Tipo**: Plantilla Genérica
