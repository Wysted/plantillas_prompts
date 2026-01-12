# Ejemplos Prácticos de Patrones de Diseño

## 1. Constructor Pattern

### Ejemplo: UserRepository
```go
// Constructor que inyecta dependencias
func NewRepository(db *bob.DB) Repository {
    return &repository{db: db}
}

// Uso
userRepo := user.NewRepository(db)
```

**Ventajas:**
- Retorna la interfaz, no la implementación concreta
- Facilita testing (puedes inyectar mock)
- Centraliza la creación

---

## 2. Criteria Pattern (Specification)

### Ejemplo: Búsqueda Dinámica de Usuarios
```go
// Definición del Criteria
type Criteria struct {
    ID       int64
    Email    string
    Phone    string
    IsActive *bool
}

// Uso flexible
// Caso 1: Buscar por email
user, _ := repo.GetUser(&Criteria{Email: "user@example.com"})

// Caso 2: Buscar por ID
user, _ := repo.GetUser(&Criteria{ID: 123})

// Caso 3: Buscar usuarios activos por phone
user, _ := repo.GetUser(&Criteria{
    Phone: "+56912345678",
    IsActive: utils.Bool(true),
})

// Caso 4: Listar todos los usuarios inactivos
users, _ := repo.GetUsers(&Criteria{IsActive: utils.Bool(false)})
```

### Conversión a Query SQL
```go
func (r *repository) criteriaToWhere(criteria *Criteria) []bob.Mod[*dialect.SelectQuery] {
    mods := []bob.Mod[*dialect.SelectQuery]{}

    // Solo agrega condiciones si están definidas
    if criteria.Email != "" {
        mods = append(mods, sm.Where(psql.Quote("email").EQ(psql.Arg(criteria.Email))))
    }
    if criteria.ID != 0 {
        mods = append(mods, sm.Where(psql.Quote("id").EQ(psql.Arg(criteria.ID))))
    }
    if criteria.IsActive != nil {
        mods = append(mods, sm.Where(psql.Quote("isActive").EQ(psql.Arg(criteria.IsActive))))
    }

    return mods
}
```

**Ventajas:**
- Un solo método para múltiples tipos de búsqueda
- Type-safe (errores en compilación)
- Fácil agregar nuevos filtros
- No necesitas crear métodos como `FindByEmail()`, `FindByPhone()`, etc.

---

## 3. Builder Pattern (Fluent Interface)

### Ejemplo: FindOptions para Paginación y Ordenamiento
```go
// Definición
type FindOptions struct {
    limit *int64
    skip  *int64
    sort  *Sort
}

func NewFindOptions() *FindOptions {
    return &FindOptions{}
}

// Métodos encadenables (retornan *FindOptions)
func (f *FindOptions) Limit(limit int64) *FindOptions {
    f.limit = &limit
    return f
}

func (f *FindOptions) Skip(skip int64) *FindOptions {
    f.skip = &skip
    return f
}

func (f *FindOptions) Sort(sort Sort) *FindOptions {
    f.sort = &sort
    return f
}
```

### Uso Fluido
```go
// Ejemplo 1: Paginación simple
opts := NewFindOptions().Limit(10).Skip(20)
transactions, _ := repo.GetTransactions(nil, opts)

// Ejemplo 2: Con ordenamiento
opts := NewFindOptions().
    Limit(50).
    Skip(0).
    Sort(Sort{Date: DESC, Charge: ASC})

// Ejemplo 3: Sin opciones (listar todo)
transactions, _ := repo.GetTransactions(nil, nil)
```

**Ventajas:**
- Sintaxis legible y expresiva
- Evita constructores con muchos parámetros
- Flexible: puedes omitir cualquier opción

---

## 4. DTO Pattern + Mapper

### Ejemplo: UserDTO
```go
// DTO de entrada (desde HTTP)
type UserDTO struct {
    Email string `json:"email"`
    Phone string `json:"phone"`
}

// Mapper: DTO -> Domain Model
func (dto *UserDTO) ToModel() User {
    return User{
        Email: dto.Email,
        Phone: dto.Phone,
    }
}

// Flujo completo
// 1. Handler recibe JSON
var userDTO user.UserDTO
c.BindJSON(&userDTO)  // {"email": "test@example.com", "phone": "+56912345678"}

// 2. Service transforma a modelo de dominio
newUser := userDTO.ToModel()

// 3. Repository persiste
savedUser, _ := s.repo.InsertOne(newUser)
```

### Ejemplo: UpdateUserDTO (campos opcionales)
```go
// DTO con punteros para campos opcionales
type UpdateUserDTO struct {
    Email *string `json:"email,omitempty"`
    Phone *string `json:"phone,omitempty"`
}

func (dto *UpdateUserDTO) ToUpdateData() UpdateData {
    return UpdateData{
        Email: dto.Email,
        Phone: dto.Phone,
    }
}

// Uso
// Cliente envía: {"email": "new@example.com"}
// Phone queda como nil, no se actualiza
```

**Ventajas:**
- Desacopla representación HTTP de entidad de dominio
- Validación centralizada en service
- Versionado de API sin cambiar dominio

---

## 5. Repository Pattern

### Interfaz vs Implementación
```go
// Interfaz (contrato público)
type Repository interface {
    InsertOne(User) (*User, error)
    GetUser(*Criteria) (*User, error)
    GetUsers(*Criteria) ([]User, error)
    Update(*Criteria, UpdateData) error
    Delete(*Criteria) error
    Exists(*Criteria) (bool, error)
}

// Implementación privada (oculta detalles)
type repository struct {
    db *bob.DB  // Podría ser cualquier BD
}

// El service solo conoce la interfaz
type service struct {
    repo Repository  // ← Interfaz, no implementación
}
```

### Testing con Mocks
```go
// Mock para testing
type MockUserRepository struct {
    users []User
}

func (m *MockUserRepository) GetUser(c *Criteria) (*User, error) {
    // Implementación fake para tests
    for _, u := range m.users {
        if u.Email == c.Email {
            return &u, nil
        }
    }
    return nil, nil
}

// Usar en tests
mockRepo := &MockUserRepository{
    users: []User{{ID: 1, Email: "test@example.com"}},
}
service := user.NewService(mockRepo)  // ← Inyectar mock
```

---

## 6. Service Layer Pattern

### Ejemplo: Register con Validaciones
```go
func (s *service) Register(dto UserDTO) (*User, error) {
    // 1. Validar formato de email
    if err := s.validateEmail(dto.Email); err != nil {
        return nil, err  // ErrInvalidEmail
    }

    // 2. Validar formato de phone
    if err := s.validatePhone(dto.Phone); err != nil {
        return nil, err  // ErrInvalidPhone
    }

    // 3. Regla de negocio: email debe ser único
    exists, err := s.repo.Exists(&Criteria{Email: dto.Email})
    if err != nil {
        return nil, err
    }
    if exists {
        return nil, ErrUserAlreadyExists
    }

    // 4. Regla de negocio: phone debe ser único
    exists, err = s.repo.Exists(&Criteria{Phone: dto.Phone})
    if err != nil {
        return nil, err
    }
    if exists {
        return nil, ErrUserAlreadyExists
    }

    // 5. Persistir
    newUser := dto.ToModel()
    return s.repo.InsertOne(newUser)
}
```

**Responsabilidades:**
- ✅ Validar datos
- ✅ Aplicar reglas de negocio
- ✅ Orquestar repositorios
- ❌ NO conoce HTTP
- ❌ NO conoce SQL

---

## 7. Dependency Injection

### Ejemplo: router.go (Composición Manual)
```go
func NewRouter(db *bob.DB, cfg *config.Config) *gin.Engine {
    r := gin.New()

    // 1. Crear repositorios
    userRepo := user.NewRepository(db)
    bankAccountRepo := bank_account.NewRepository(db)
    transactionRepo := transaction.NewRepository(db)

    // 2. Crear servicios (inyectar repos)
    userService := user.NewService(userRepo)
    bankAccountService := bank_account.NewService(bankAccountRepo, cfg.EncryptionKey)
    transactionService := transaction.NewService(
        transactionRepo,
        bankAccountRepo,
        availableBalanceRepo,
    )

    // 3. Crear handlers (inyectar services)
    userHandler := handler.NewUserHandler(userService)
    transactionHandler := handler.NewTransactionHandler(transactionService)

    // 4. Registrar rutas
    user := r.Group("api/user")
    {
        user.POST("/register", userHandler.Register)
        user.GET("/me", middleware.JWTMiddleware(), userHandler.GetMe)
    }

    return r
}
```

**Flujo de Dependencias:**
```
DB → Repository → Service → Handler → Router
```

---

## 8. Unit of Work Pattern

### Ejemplo: InsertMany con Transacción
```go
func (r *repository) InsertMany(transactions []Transaction) error {
    ctx := context.Background()

    // 1. Iniciar transacción
    tx, err := r.db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("begin tx: %w", err)
    }

    // 2. Rollback automático en caso de error
    defer func() {
        if err != nil {
            _ = tx.Rollback(ctx)
        }
    }()

    // 3. Múltiples inserciones dentro de la transacción
    for _, t := range transactions {
        set := &models.TransactionSetter{
            OperationNumber: omit.From(t.OperationNumber),
            Charge:          omit.From(t.Charge),
            Payment:         omit.From(t.Payment),
        }
        if _, err = models.Transactions.Insert(set).One(ctx, tx); err != nil {
            return utils.ErrRepositoryFailed  // Activará rollback
        }
    }

    // 4. Commit si todo salió bien
    if err = tx.Commit(ctx); err != nil {
        return utils.ErrRepositoryFailed
    }

    return nil
}
```

**Atomicidad:**
- Si falla la inserción #5 de 10, todas se revierten
- Todo o nada

---

## 9. Adapter Pattern

### Ejemplo: HTTP Handler como Adaptador
```go
// Handler adapta HTTP → Service
func (h *UserHandler) Register(c *gin.Context) {
    // 1. Adaptar entrada HTTP a DTO
    var userDTO user.UserDTO
    if err := c.BindJSON(&userDTO); err != nil {
        c.AbortWithStatusJSON(400, ProblemDetails{
            Title: "Datos invalidos",
        })
        return
    }

    // 2. Llamar al service (capa de aplicación)
    user, err := h.service.Register(userDTO)
    if err != nil {
        // 3. Adaptar errores del dominio a HTTP
        c.AbortWithStatusJSON(500, ProblemDetails{
            Title: "Error interno",
        })
        return
    }

    // 4. Adaptar salida del dominio a HTTP JSON
    c.JSON(201, UserRes{
        Status: 201,
        Msg:    "Usuario creado",
        Body:   &user,
    })
}
```

---

## 10. Criteria + FindOptions Combinados

### Ejemplo Completo: Búsqueda Avanzada de Transacciones
```go
// Buscar transacciones de una cuenta bancaria
// Solo cargos (no abonos)
// Del último mes
// Ordenadas por fecha descendente
// Paginado: 10 resultados, página 2

criteria := &Criteria{
    BankAccountId: utils.Int64(123),
    OnlyCharge:    utils.Bool(true),
    DateFrom:      utils.String("2024-01-01"),
    DateTo:        utils.String("2024-01-31"),
}

opts := NewFindOptions().
    Limit(10).
    Skip(10).  // página 2 = skip primeros 10
    Sort(Sort{Date: DESC})

transactions, err := repo.GetTransactions(criteria, opts)

// SQL generado (aproximado):
// SELECT * FROM transactions
// WHERE bankAccountId = 123
//   AND charge > 0
//   AND date >= '2024-01-01'
//   AND date <= '2024-01-31'
// ORDER BY date DESC
// LIMIT 10 OFFSET 10
```

---

## Resumen de Beneficios

| Patrón | Problema que Resuelve |
|--------|----------------------|
| **Constructor** | Creación inconsistente de objetos |
| **Criteria** | Múltiples métodos `FindByX()` duplicados |
| **Builder** | Constructores con muchos parámetros |
| **DTO** | Acoplamiento entre HTTP y dominio |
| **Repository** | SQL esparcido por todo el código |
| **Service Layer** | Lógica de negocio mezclada con HTTP |
| **Dependency Injection** | Alto acoplamiento entre capas |
| **Unit of Work** | Inconsistencia en operaciones múltiples |
| **Adapter** | Cambiar frameworks sin tocar dominio |

---

## Anti-Patrones Evitados

❌ **God Object**: No hay clases que hagan todo
❌ **Anemic Domain Model**: El dominio tiene comportamiento (validaciones)
❌ **Magic Numbers**: Se usan constantes (`ASC`, `DESC`)
❌ **Hard-coded Dependencies**: Todo se inyecta
❌ **Fat Controllers**: Los handlers son delgados, el service tiene la lógica
❌ **Shotgun Surgery**: Cambios localizados por patrón Repository
