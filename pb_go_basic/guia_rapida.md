# Guía Rápida de Referencia

## 🎯 Cheat Sheet de Patrones

### Constructor Pattern
```go
// ✅ Correcto
func NewRepository(db *bob.DB) Repository {
    return &repository{db: db}
}

// ❌ Incorrecto
func NewRepository(db *bob.DB) *repository {  // No retornar implementación
    return &repository{db: db}
}
```

---

### Criteria Pattern
```go
// Búsqueda simple
repo.GetUser(&Criteria{Email: "user@example.com"})

// Búsqueda combinada
repo.GetUsers(&Criteria{
    IsActive: utils.Bool(true),
    Phone: "+56912345678",
})

// Listar todo
repo.GetUsers(nil)
```

---

### Builder Pattern
```go
// Encadenar métodos
opts := NewFindOptions().
    Limit(10).
    Skip(20).
    Sort(Sort{Date: DESC})

repo.GetTransactions(criteria, opts)
```

---

### DTO to Model
```go
// En dto.go
func (dto *UserDTO) ToModel() User {
    return User{
        Email: dto.Email,
        Phone: dto.Phone,
    }
}

// En service.go
newUser := dto.ToModel()
repo.InsertOne(newUser)
```

---

### Service Layer Validation
```go
func (s *service) Register(dto UserDTO) (*User, error) {
    // 1. Validar formato
    if err := s.validateEmail(dto.Email); err != nil {
        return nil, err
    }

    // 2. Reglas de negocio
    exists, _ := s.repo.Exists(&Criteria{Email: dto.Email})
    if exists {
        return nil, ErrUserAlreadyExists
    }

    // 3. Persistir
    return s.repo.InsertOne(dto.ToModel())
}
```

---

### Error Handling
```go
// En errors.go
var (
    ErrUserNotFound = errors.New("user not found")
    ErrInvalidEmail = errors.New("invalid email")
)

// En service.go
if user == nil {
    return nil, ErrUserNotFound
}

// En handler.go
user, err := h.service.GetUser(email)
if err != nil {
    if errors.Is(err, user.ErrUserNotFound) {
        c.JSON(404, ProblemDetails{...})
        return
    }
    c.JSON(500, ProblemDetails{...})
    return
}
```

---

## 📁 Dónde Va Cada Cosa

| ¿Qué es? | ¿Dónde va? | Ejemplo |
|----------|-----------|---------|
| Entidad del dominio | `internal/domain/*/model.go` | `type User struct {...}` |
| DTO de entrada | `internal/domain/*/dto.go` | `type UserDTO struct {...}` |
| Lógica de negocio | `internal/domain/*/service.go` | `func (s *service) Register()` |
| Query SQL | `internal/domain/*/repository.go` | `func (r *repository) GetUser()` |
| Endpoint HTTP | `internal/http/handler/*.go` | `func (h *Handler) Register()` |
| Middleware | `internal/http/middleware/*.go` | `func JWTMiddleware()` |
| Configuración | `internal/config/config.go` | `type Config struct {...}` |
| Utilidades DB | `pkg/db/` | `func NewPostgres()` |
| Cliente externo | `pkg/waha/`, `pkg/jwt/` | `type Client struct {...}` |
| Utilidades generales | `utils/` | `func Bool()`, `func MapNoError()` |
| Errores custom | `internal/domain/*/errors.go` | `var ErrUserNotFound` |

---

## 🔄 Flujo de Implementación

### Agregar Nuevo Endpoint

1. **Define el modelo** (`model.go`)
```go
type Product struct {
    ID    int64
    Name  string
    Price float64
}
```

2. **Crea el DTO** (`dto.go`)
```go
type ProductDTO struct {
    Name  string `json:"name"`
    Price float64 `json:"price"`
}

func (dto *ProductDTO) ToModel() Product {
    return Product{Name: dto.Name, Price: dto.Price}
}
```

3. **Define errores** (`errors.go`)
```go
var ErrProductNotFound = errors.New("product not found")
```

4. **Crea interfaz y repo** (`repository.go`)
```go
type Repository interface {
    InsertOne(Product) (*Product, error)
    GetByID(int64) (*Product, error)
}

type repository struct { db *bob.DB }
func NewRepository(db *bob.DB) Repository { ... }
```

5. **Crea el service** (`service.go`)
```go
type Service interface {
    Create(ProductDTO) (*Product, error)
    GetByID(int64) (*Product, error)
}

type service struct { repo Repository }
func NewService(repo Repository) Service { ... }
```

6. **Crea el handler** (`handler/product_handler.go`)
```go
type ProductHandler struct { service product.Service }
func NewProductHandler(service product.Service) *ProductHandler { ... }

func (h *ProductHandler) Create(c *gin.Context) {
    var dto product.ProductDTO
    c.BindJSON(&dto)
    result, err := h.service.Create(dto)
    c.JSON(201, result)
}
```

7. **Registra en router** (`router.go`)
```go
productRepo := product.NewRepository(db)
productService := product.NewService(productRepo)
productHandler := handler.NewProductHandler(productService)

products := r.Group("api/products")
{
    products.POST("", productHandler.Create)
    products.GET("/:id", productHandler.GetByID)
}
```

---

## 🧪 Testing

### Mock de Repository
```go
type MockProductRepository struct {
    products []Product
}

func (m *MockProductRepository) GetByID(id int64) (*Product, error) {
    for _, p := range m.products {
        if p.ID == id {
            return &p, nil
        }
    }
    return nil, ErrProductNotFound
}
```

### Test de Service
```go
func TestCreate(t *testing.T) {
    mockRepo := &MockProductRepository{}
    service := NewService(mockRepo)

    dto := ProductDTO{Name: "Test", Price: 100}
    product, err := service.Create(dto)

    assert.NoError(t, err)
    assert.Equal(t, "Test", product.Name)
}
```

---

## 💡 Snippets Útiles

### Puntero opcional en Criteria
```go
// En Criteria
type Criteria struct {
    IsActive *bool  // nil = no filtrar
}

// Uso
utils.Bool(true)  // helper para crear *bool
```

### Transaction (Unit of Work)
```go
tx, _ := r.db.BeginTx(ctx, nil)
defer func() {
    if err != nil {
        tx.Rollback(ctx)
    }
}()

// operaciones...

tx.Commit(ctx)
```

### Mapper SQL → Domain
```go
func (r *repository) sqlToModel(m *models.User) *User {
    return &User{
        ID:    m.ID,
        Email: m.Email,
        Phone: m.Phone,
    }
}
```

### Validación Regex
```go
var emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$`)

func (s *service) validateEmail(email string) error {
    if !emailRegex.MatchString(email) {
        return ErrInvalidEmail
    }
    return nil
}
```

---

## 🚦 Reglas de Oro

### 1. Separación de Responsabilidades
- **Handler**: Solo adapta HTTP ↔ Service
- **Service**: Solo lógica de negocio
- **Repository**: Solo persistencia

### 2. Dependencias
- Handler depende de Service (interfaz)
- Service depende de Repository (interfaz)
- Repository depende de DB

### 3. Interfaces
- Siempre define interfaz antes de implementación
- Retorna interfaz, no struct concreto
- Usa interfaces para inyección de dependencias

### 4. Errores
- Errores custom en `errors.go`
- No usar strings genéricos
- Propagar errores hacia arriba
- Adaptar en handler a HTTP status

### 5. DTOs
- Request DTOs en `dto.go`
- Siempre método `ToModel()` o `ToUpdateData()`
- No exponer modelos de BD en HTTP

---

## 📊 Decisiones Rápidas

### ¿Cuándo usar Criteria?
- ✅ Búsquedas con múltiples filtros opcionales
- ✅ Queries dinámicas
- ❌ Búsqueda simple por ID (usa método directo)

### ¿Cuándo usar Builder?
- ✅ Paginación + Ordenamiento + Filtros
- ✅ Opciones complejas
- ❌ Queries sin opciones

### ¿Cuándo crear nuevo DTO?
- ✅ Request diferente a la entidad
- ✅ Campos opcionales (update)
- ❌ Si es idéntico al modelo (usar modelo)

### ¿Cuándo crear nuevo Service?
- ✅ Por cada agregado/dominio
- ❌ Por cada endpoint

### ¿Cuándo usar Transaction?
- ✅ Múltiples inserts/updates relacionados
- ✅ Operaciones que deben ser atómicas
- ❌ Queries de solo lectura

---

## 🎨 Convenciones de Código

### Nombres
```go
// Repository
type Repository interface { ... }          // Interfaz pública
type repository struct { ... }             // Struct privado
func NewRepository() Repository { ... }    // Constructor

// Service
type Service interface { ... }             // Interfaz pública
type service struct { ... }                // Struct privado
func NewService() Service { ... }          // Constructor

// Handler
type UserHandler struct { ... }            // Struct público
func NewUserHandler() *UserHandler { ... } // Constructor
```

### Métodos Repository
```go
InsertOne(entity)      // Insertar uno
InsertMany([]entity)   // Insertar múltiples
GetByID(id)            // Obtener por ID
GetAll(criteria)       // Obtener con filtros
Update(criteria, data) // Actualizar
Delete(criteria)       // Eliminar
Exists(criteria)       // Verificar existencia
Count(criteria)        // Contar
```

### Métodos Service
```go
Create(dto)            // Crear (con validaciones)
GetByID(id)            // Obtener por ID
GetByEmail(email)      // Obtener por campo único
GetAll()               // Listar todos
Update(id, dto)        // Actualizar
Delete(id)             // Eliminar
```

### Métodos Handler
```go
func (h *Handler) Create(c *gin.Context)   // POST
func (h *Handler) GetByID(c *gin.Context)  // GET /:id
func (h *Handler) GetAll(c *gin.Context)   // GET /
func (h *Handler) Update(c *gin.Context)   // PUT/PATCH /:id
func (h *Handler) Delete(c *gin.Context)   // DELETE /:id
```

---

## 🔍 Debugging

### Logs Útiles
```go
// En service
logger.Infof("Creating user: %s", dto.Email)

// En repository
logger.Debugf("Query: %+v", criteria)

// En handler
logger.Errorf("Error: %v", err)
```

### Ver Query SQL Generado
```go
// En repository (temporal para debug)
query := models.Users.Query(mods...)
sql, args := query.Build()
fmt.Printf("SQL: %s\nArgs: %v\n", sql, args)
```

---

## 📦 Utilidades Comunes

### utils/ptr.go
```go
func Bool(b bool) *bool { return &b }
func Int64(i int64) *int64 { return &i }
func String(s string) *string { return &s }
```

### utils/func.go
```go
func MapNoError[T, R any](items []T, fn func(T) R) []R {
    result := make([]R, len(items))
    for i, item := range items {
        result[i] = fn(item)
    }
    return result
}
```

---

## ⚡ Performance Tips

1. **Usa Preload para joins**
```go
models.Transactions.Query(
    models.Preload.Transaction.BankAccountIdBankAccount(),
)
```

2. **Limita resultados por defecto**
```go
opts := NewFindOptions().Limit(100)  // max 100
```

3. **Usa índices en BD**
```sql
CREATE INDEX idx_users_email ON users(email);
```

4. **Cache en service (si es necesario)**
```go
type service struct {
    repo  Repository
    cache map[string]*User  // Para datos que no cambian
}
```

---

## 🆘 Errores Comunes

### ❌ Handler llama directamente a Repository
```go
// MAL
func (h *Handler) Create(c *gin.Context) {
    h.repo.InsertOne(...)  // ❌ Sin validaciones
}

// BIEN
func (h *Handler) Create(c *gin.Context) {
    h.service.Create(dto)  // ✅ Con validaciones
}
```

### ❌ Service hace SQL
```go
// MAL
func (s *service) GetUser(id int64) {
    db.Query("SELECT * FROM users WHERE id = $1", id)  // ❌
}

// BIEN
func (s *service) GetUser(id int64) {
    return s.repo.GetByID(id)  // ✅
}
```

### ❌ Retornar struct privado
```go
// MAL
func NewService() *service { ... }  // ❌ No testeable

// BIEN
func NewService() Service { ... }   // ✅ Retorna interfaz
```

### ❌ No validar en service
```go
// MAL
func (s *service) Register(dto UserDTO) {
    return s.repo.InsertOne(dto.ToModel())  // ❌ Sin validar
}

// BIEN
func (s *service) Register(dto UserDTO) {
    if err := s.validateEmail(dto.Email); err != nil {
        return nil, err
    }
    // ...validaciones...
    return s.repo.InsertOne(dto.ToModel())  // ✅
}
```

---

## 📚 Recursos

- [Documentación completa](./README.md)
- [Patrones aplicados](./patrones_diseno.json)
- [Ejemplos prácticos](./ejemplos_practicos.md)
- [Diagramas](./diagramas.md)
