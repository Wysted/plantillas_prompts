# Prompt para Generar Código con Plantilla

> Usa este prompt para que un MCP/LLM genere código siguiendo la arquitectura base

---

## 🎯 Prompt para MCP/LLM

```
Eres un arquitecto de software experto en Go y Clean Architecture.

CONTEXTO:
Lee los siguientes archivos que definen la arquitectura base que debes seguir:

1. arquitectura/plantilla/arquitectura_base.json
2. arquitectura/plantilla/patrones_diseno.json
3. arquitectura/plantilla/guia_rapida.md

TAREA:
Genera un dominio completo llamado "[NOMBRE_DOMINIO]" siguiendo EXACTAMENTE los patrones y convenciones definidos en los archivos.

REQUISITOS:

1. Estructura de archivos requerida:
   - internal/domain/[NOMBRE_DOMINIO]/model.go
   - internal/domain/[NOMBRE_DOMINIO]/dto.go
   - internal/domain/[NOMBRE_DOMINIO]/errors.go
   - internal/domain/[NOMBRE_DOMINIO]/repository.go
   - internal/domain/[NOMBRE_DOMINIO]/service.go
   - internal/http/handler/[NOMBRE_DOMINIO]_handler.go

2. La entidad debe tener estos campos:
   [DESCRIBE LOS CAMPOS DE TU ENTIDAD]
   Ejemplo:
   - ID (int64)
   - Name (string)
   - Email (string)
   - CreatedAt (time.Time)

3. Casos de uso necesarios:
   [LISTA LOS CASOS DE USO]
   Ejemplo:
   - Create(dto) - Crear nueva entidad
   - GetByID(id) - Obtener por ID
   - GetAll(criteria) - Listar con filtros
   - Update(id, dto) - Actualizar
   - Delete(id) - Eliminar

4. Validaciones requeridas:
   [LISTA LAS VALIDACIONES]
   Ejemplo:
   - Email válido (regex)
   - Name no vacío
   - Email único

5. PATRONES OBLIGATORIOS A APLICAR:
   - ✅ Constructor Pattern (NewRepository, NewService, NewHandler)
   - ✅ Repository Pattern (interfaz + implementación privada)
   - ✅ Service Layer Pattern (interfaz + implementación privada)
   - ✅ Criteria Pattern (para búsquedas dinámicas)
   - ✅ DTO Pattern (con métodos ToModel, ToUpdateData)
   - ✅ Dependency Injection (manual en constructores)

6. REGLAS ESTRICTAS:
   - NUNCA retornar struct concreto desde constructores (retorna interfaz)
   - NUNCA poner SQL en service.go
   - NUNCA poner lógica de negocio en repository.go
   - SIEMPRE usar punteros para campos opcionales en Criteria
   - SIEMPRE validar en service layer, no en handlers
   - SIEMPRE usar errores custom definidos en errors.go

7. NAMING CONVENTIONS (obligatorio):
   - Interfaces: type Repository interface, type Service interface
   - Structs privados: type repository struct, type service struct
   - Constructores: func NewRepository(db *bob.DB) Repository
   - Métodos repo: InsertOne, GetByID, GetAll, Update, Delete, Exists
   - Métodos service: Create, GetByID, GetAll, Update, Delete

8. ESTRUCTURA DE CADA ARCHIVO:

   model.go:
   ```go
   package [NOMBRE_DOMINIO]

   import "time"

   type [Entidad] struct {
       ID        int64     `json:"id"`
       // ... campos
       CreatedAt time.Time `json:"createdAt"`
   }
   ```

   dto.go:
   ```go
   package [NOMBRE_DOMINIO]

   // DTO para crear
   type [Entidad]DTO struct {
       // campos sin ID ni CreatedAt
   }

   func (dto *[Entidad]DTO) ToModel() [Entidad] {
       return [Entidad]{
           // mapear campos
       }
   }

   // DTO para actualizar (campos opcionales)
   type Update[Entidad]DTO struct {
       Field1 *string `json:"field1,omitempty"`
       // usar punteros para opcionales
   }

   func (dto *Update[Entidad]DTO) ToUpdateData() UpdateData {
       return UpdateData{
           Field1: dto.Field1,
       }
   }
   ```

   errors.go:
   ```go
   package [NOMBRE_DOMINIO]

   import "errors"

   var (
       Err[Entidad]NotFound = errors.New("[entidad] not found")
       ErrInvalid[Campo] = errors.New("invalid [campo]")
       // más errores específicos
   )
   ```

   repository.go:
   ```go
   package [NOMBRE_DOMINIO]

   import (
       "context"
       "database/sql"
       "github.com/stephenafamo/bob"
       // imports necesarios
   )

   // INTERFAZ PÚBLICA
   type Repository interface {
       InsertOne([Entidad]) (*[Entidad], error)
       GetByID(int64) (*[Entidad], error)
       GetAll(*Criteria) ([][Entidad], error)
       Update(*Criteria, UpdateData) error
       Delete(*Criteria) error
       Exists(*Criteria) (bool, error)
   }

   // CRITERIA (búsquedas dinámicas)
   type Criteria struct {
       ID     int64
       Field1 string
       Field2 *bool  // puntero para opcional
   }

   // UPDATE DATA
   type UpdateData struct {
       Field1 *string  // punteros para campos opcionales
   }

   // STRUCT PRIVADO
   type repository struct {
       db *bob.DB
   }

   // CONSTRUCTOR
   func NewRepository(db *bob.DB) Repository {
       return &repository{db: db}
   }

   // MÉTODOS (implementar todos)
   func (r *repository) InsertOne([entidad] [Entidad]) (*[Entidad], error) {
       // implementación con Bob ORM
   }

   // Helper privado para convertir Criteria a WHERE
   func (r *repository) criteriaToWhere(criteria *Criteria) []bob.Mod[*dialect.SelectQuery] {
       if criteria == nil {
           return nil
       }
       mods := []bob.Mod[*dialect.SelectQuery]{}
       if criteria.Field1 != "" {
           mods = append(mods, sm.Where(psql.Quote("field1").EQ(psql.Arg(criteria.Field1))))
       }
       // más condiciones...
       return mods
   }
   ```

   service.go:
   ```go
   package [NOMBRE_DOMINIO]

   import (
       // imports
   )

   // INTERFAZ PÚBLICA
   type Service interface {
       Create([Entidad]DTO) (*[Entidad], error)
       GetByID(int64) (*[Entidad], error)
       GetAll() ([][Entidad], error)
       Update(int64, Update[Entidad]DTO) (*[Entidad], error)
       Delete(int64) error
   }

   // STRUCT PRIVADO
   type service struct {
       repo Repository
   }

   // CONSTRUCTOR
   func NewService(repo Repository) Service {
       return &service{repo: repo}
   }

   // MÉTODOS (con validaciones y lógica de negocio)
   func (s *service) Create(dto [Entidad]DTO) (*[Entidad], error) {
       // 1. Validar datos
       if err := s.validate[Campo](dto.[Campo]); err != nil {
           return nil, err
       }

       // 2. Reglas de negocio (ej: verificar duplicados)
       exists, _ := s.repo.Exists(&Criteria{Field: dto.Field})
       if exists {
           return nil, Err[Entidad]AlreadyExists
       }

       // 3. Persistir
       return s.repo.InsertOne(dto.ToModel())
   }

   // Métodos privados de validación
   func (s *service) validate[Campo](value string) error {
       if value == "" {
           return ErrInvalid[Campo]
       }
       // más validaciones
       return nil
   }
   ```

   handler/[NOMBRE_DOMINIO]_handler.go:
   ```go
   package handler

   import (
       "net/http"
       "strconv"
       "[proyecto]/internal/domain/[NOMBRE_DOMINIO]"
       "github.com/gin-gonic/gin"
   )

   type [Entidad]Handler struct {
       service [NOMBRE_DOMINIO].Service
   }

   func New[Entidad]Handler(service [NOMBRE_DOMINIO].Service) *[Entidad]Handler {
       return &[Entidad]Handler{service: service}
   }

   // @Summary Crear [entidad]
   // @Tags [NOMBRE_DOMINIO]
   // @Accept json
   // @Produce json
   // @Param dto body [NOMBRE_DOMINIO].[Entidad]DTO true "Datos"
   // @Success 201 {object} [Entidad]Response
   // @Failure 400 {object} ProblemDetails
   // @Router /api/[NOMBRE_DOMINIO] [post]
   func (h *[Entidad]Handler) Create(c *gin.Context) {
       var dto [NOMBRE_DOMINIO].[Entidad]DTO

       if err := c.BindJSON(&dto); err != nil {
           c.JSON(http.StatusBadRequest, ProblemDetails{
               Title: "Datos inválidos",
           })
           return
       }

       result, err := h.service.Create(dto)
       if err != nil {
           c.JSON(http.StatusInternalServerError, ProblemDetails{
               Title: "Error interno",
               Detail: err.Error(),
           })
           return
       }

       c.JSON(http.StatusCreated, [Entidad]Response{
           Status: 201,
           Msg:    "[Entidad] creado con éxito",
           Body:   result,
       })
   }

   // GetByID, GetAll, Update, Delete...
   ```

9. REGISTRO EN ROUTER:
   Genera también el código para agregar a router.go:
   ```go
   // En NewRouter()
   [NOMBRE_DOMINIO]Repo := [NOMBRE_DOMINIO].NewRepository(db)
   [NOMBRE_DOMINIO]Service := [NOMBRE_DOMINIO].NewService([NOMBRE_DOMINIO]Repo)
   [NOMBRE_DOMINIO]Handler := handler.New[Entidad]Handler([NOMBRE_DOMINIO]Service)

   [NOMBRE_DOMINIO]Group := r.Group("/api/[NOMBRE_DOMINIO]")
   {
       [NOMBRE_DOMINIO]Group.POST("", [NOMBRE_DOMINIO]Handler.Create)
       [NOMBRE_DOMINIO]Group.GET("/:id", [NOMBRE_DOMINIO]Handler.GetByID)
       [NOMBRE_DOMINIO]Group.GET("", [NOMBRE_DOMINIO]Handler.GetAll)
       [NOMBRE_DOMINIO]Group.PUT("/:id", [NOMBRE_DOMINIO]Handler.Update)
       [NOMBRE_DOMINIO]Group.DELETE("/:id", [NOMBRE_DOMINIO]Handler.Delete)
   }
   ```

OUTPUT ESPERADO:
Genera los 6 archivos completos con código funcional, siguiendo EXACTAMENTE
los patrones definidos en los archivos de arquitectura. El código debe estar
listo para copiar y pegar.

IMPORTANTE:
- NO agregues features extras no solicitadas
- NO uses librerías no mencionadas en arquitectura_base.json
- SÍ comenta el código donde sea necesario
- SÍ mantén consistencia con los ejemplos de arquitectura_base.json
```

---

## 📝 Ejemplo de Uso

### Caso 1: Generar dominio "Product"

```
[USA EL PROMPT DE ARRIBA Y COMPLETA:]

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
- Create(dto) - Crear producto
- GetByID(id) - Obtener por ID
- GetAll(criteria) - Listar con filtros (name, isActive)
- Update(id, dto) - Actualizar
- Delete(id) - Eliminar
- UpdateStock(id, quantity) - Actualizar stock

VALIDACIONES:
- Name no vacío (min 3 caracteres)
- Price >= 0
- Stock >= 0
- Name único
```

---

### Caso 2: Generar dominio "Order"

```
[USA EL PROMPT DE ARRIBA Y COMPLETA:]

TAREA:
Genera un dominio completo llamado "order"

ENTIDAD:
- ID (int64)
- UserID (int64)
- OrderNumber (string)
- TotalAmount (float64)
- Status (string) - "pending", "completed", "cancelled"
- CreatedAt (time.Time)

CASOS DE USO:
- Create(dto) - Crear orden
- GetByID(id) - Obtener por ID
- GetByUserID(userID) - Órdenes de un usuario
- GetAll(criteria) - Listar con filtros (status, userID, dateRange)
- UpdateStatus(id, status) - Cambiar estado
- Cancel(id) - Cancelar orden

VALIDACIONES:
- UserID > 0
- TotalAmount > 0
- Status válido (pending, completed, cancelled)
- OrderNumber único
- No se puede cancelar si status = "completed"
```

---

## 🎯 Variaciones del Prompt

### Para dominio con FindOptions (paginación)

Agrega después de "CASOS DE USO":

```
CARACTERÍSTICAS ADICIONALES:

1. Implementar Builder Pattern para paginación:
   - FindOptions con Limit(), Skip(), Sort()
   - Sort con múltiples campos ordenables

2. Estructura Sort:
   type Sort struct {
       ID        OrderMod
       Name      OrderMod
       CreatedAt OrderMod
   }

3. Uso en GetAll:
   GetAll(criteria *Criteria, opts *FindOptions) ([]Product, error)
```

---

### Para dominio con relaciones

Agrega después de "ENTIDAD":

```
RELACIONES:
- Pertenece a User (UserID int64)
- Tiene múltiples OrderItems (via OrderID)

CASOS DE USO ADICIONALES:
- GetWithUser(id) - Obtener con eager loading de User
- GetByUserID(userID) - Filtrar por usuario
```

---

### Para dominio sin Repository (como Auth)

```
NOTA ESPECIAL:
Este dominio NO tiene repository propio. Solo tiene service.go que
orquesta otros services.

ARCHIVOS A GENERAR:
- internal/domain/auth/dto.go
- internal/domain/auth/service.go
- internal/http/handler/auth_handler.go

SERVICE DEPENDENCIES:
type service struct {
    userService   user.Service
    codeService   code.Service
    sessionService session.Service
}
```

---

## 🔍 Checklist de Validación

Usa esto para verificar que el código generado cumple:

- [ ] ✅ Interfaces definidas y públicas
- [ ] ✅ Structs privados (lowercase)
- [ ] ✅ Constructores retornan interfaces
- [ ] ✅ Criteria con punteros para opcionales
- [ ] ✅ DTO con ToModel() y ToUpdateData()
- [ ] ✅ Errores custom en errors.go
- [ ] ✅ Validaciones en service, no handler
- [ ] ✅ SQL solo en repository
- [ ] ✅ Handler delgado (thin controller)
- [ ] ✅ Dependency Injection manual
- [ ] ✅ Tags Swagger en handlers
- [ ] ✅ ProblemDetails en errores HTTP
- [ ] ✅ Código comentado donde necesario

---

## 📚 Archivos de Referencia

El MCP debe leer estos archivos antes de generar:

1. `arquitectura_base.json` - Estructura y convenciones
2. `patrones_diseno.json` - Detalles de cada patrón
3. `guia_rapida.md` - Reglas DO's y DON'Ts

---

## 💡 Tips para Mejores Resultados

1. **Sé específico** con los campos de la entidad
2. **Lista todas las validaciones** necesarias
3. **Define casos de uso claros** (no ambiguos)
4. **Menciona relaciones** si las hay
5. **Indica si necesitas paginación** (FindOptions)
6. **Especifica autenticación** (JWT, API Key, ninguna)

---

## 🚀 Resultado Esperado

El MCP debe generar:

```
internal/domain/[NOMBRE]/
├── model.go          ✅ Entidad completa
├── dto.go            ✅ DTOs con métodos ToModel
├── errors.go         ✅ Errores específicos
├── repository.go     ✅ Interface + Impl + Criteria
└── service.go        ✅ Interface + Impl + Validaciones

internal/http/handler/
└── [NOMBRE]_handler.go  ✅ Handlers con Swagger

+ Código para router.go  ✅ Dependency Injection
```

Todo listo para copiar y pegar, siguiendo exactamente los patrones definidos.
