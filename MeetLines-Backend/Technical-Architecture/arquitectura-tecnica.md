# Arquitectura Técnica - MeetLines Backend

## Descripción General

MeetLines Backend es una aplicación **ASP.NET Core** que implementa **Clean Architecture** (Hexagonal Architecture) para proporcionar una API RESTful escalable y mantenible para el sistema de gestión de reuniones y citas.

## Principios de Arquitectura

### Clean Architecture (Hexagonal Architecture)

La arquitectura se divide en capas independientes que siguen el principio de inversión de dependencias:

```
┌─────────────────────────────────────────────┐
│  MeetLines.API (Controllers & Endpoints)    │ ← HTTP Entry Point
└──────────────┬──────────────────────────────┘
               │ (depends on)
┌──────────────▼──────────────────────────────┐
│  MeetLines.Application (UseCases & DTOs)    │ ← Business Logic Orchestration
└──────────────┬──────────────────────────────┘
               │ (depends on)
┌──────────────▼──────────────────────────────┐
│  MeetLines.Domain (Entities & Rules)        │ ← Pure Domain Logic
└──────────────────────────────────────────────┘
       ▲              │              ▲
       │ (implements) │ (depends on) │ (implements)
       │              ▼              │
    ┌──────────────────────────────────────┐
    │  MeetLines.Infrastructure            │ ← Concrete Implementations
    │  (Repos, DB, Services, Config)       │
    └──────────────────────────────────────┘
       ▲
       │ (uses)
       │
┌──────────────────────────────────────────┐
│  SharedKernel (Common Utilities)         │
└──────────────────────────────────────────┘
```

## Capas de la Aplicación

### 1. **API Layer** (`MeetLines.API`)

**Responsabilidades:**
- Exponer endpoints HTTP
- Validar solicitudes
- Mapear DTOs a entidades de dominio
- Manejar autenticación/autorización
- Devolver respuestas HTTP apropiadas

**Componentes principales:**
```
MeetLines.API/
├── Controllers/           # Endpoints REST
├── Filters/              # Filtros globales (exceptions, validation)
├── Middleware/           # Pipeline HTTP (auth, CORS, logging)
├── GeoIp/               # Servicios de geolocalización
├── Jobs/                # Scheduled tasks
├── DTOs/                # Data Transfer Objects
├── appsettings.json     # Configuración
└── Program.cs           # Punto de entrada
```

**Ejemplo de flujo en un controlador:**
```csharp
[ApiController]
[Route("api/[controller]")]
public class AppointmentsController : ControllerBase
{
    private readonly ICreateAppointmentUseCase _createUseCase;
    
    public AppointmentsController(ICreateAppointmentUseCase createUseCase)
    {
        _createUseCase = createUseCase;
    }
    
    [HttpPost]
    public async Task<IActionResult> Create(CreateAppointmentDto dto)
    {
        var result = await _createUseCase.Execute(dto);
        return Ok(result);
    }
}
```

### 2. **Application Layer** (`MeetLines.Application`)

**Responsabilidades:**
- Definir casos de uso (use cases)
- Orquestar la lógica de negocio
- Definir interfaces de repositorios
- Mapear entre DTOs y entidades de dominio
- Manejar transacciones

**Componentes principales:**
```
MeetLines.Application/
├── UseCases/            # Casos de uso (create, update, delete)
├── Services/            # Servicios de aplicación
├── DTOs/                # Modelos de transferencia de datos
├── Validators/          # Validadores de entrada
├── Common/              # Utilidades compartidas
└── Interfaces/          # Contratos de repositorios
```

**Estructura de un caso de uso:**
```csharp
public class CreateAppointmentUseCase : ICreateAppointmentUseCase
{
    private readonly IAppointmentRepository _repository;
    private readonly INotificationService _notificationService;
    
    public async Task<AppointmentDto> Execute(CreateAppointmentRequest request)
    {
        // 1. Validar entrada
        var validator = new CreateAppointmentValidator();
        var validation = await validator.ValidateAsync(request);
        
        if (!validation.IsValid)
            throw new ValidationException(validation.Errors);
        
        // 2. Crear entidad de dominio
        var appointment = Appointment.Create(
            request.ServiceId,
            request.CustomerPhone,
            request.ScheduledTime
        );
        
        // 3. Persistir
        await _repository.Add(appointment);
        await _repository.UnitOfWork.SaveChangesAsync();
        
        // 4. Notificar
        await _notificationService.SendConfirmation(appointment);
        
        // 5. Devolver DTO
        return new AppointmentDto { /* mapping */ };
    }
}
```

### 3. **Domain Layer** (`MeetLines.Domain`)

**Responsabilidades:**
- Definir entidades de dominio
- Implementar reglas de negocio
- Definir especificaciones y agregados
- Definir eventos de dominio
- **NO** tiene dependencias externas

**Componentes principales:**
```
MeetLines.Domain/
├── Entities/            # Agregados raíz (Project, Appointment, Service)
├── ValueObjects/        # Objetos de valor (Money, Schedule, etc.)
├── Aggregates/          # Agregados y sus relaciones
├── Enums/               # Enumeraciones del dominio
├── Repositories/        # Interfaces de repositorios
├── Events/              # Eventos de dominio
└── Specifications/      # Especificaciones de query
```

**Ejemplo de entidad de dominio:**
```csharp
public class Appointment : AggregateRoot
{
    public Guid Id { get; private set; }
    public Guid ProjectId { get; private set; }
    public string CustomerPhone { get; private set; }
    public DateTime ScheduledTime { get; private set; }
    public AppointmentStatus Status { get; private set; }
    
    public static Appointment Create(
        Guid projectId,
        string customerPhone,
        DateTime scheduledTime)
    {
        // Validaciones de dominio
        if (string.IsNullOrWhiteSpace(customerPhone))
            throw new DomainException("Phone cannot be empty");
        
        if (scheduledTime < DateTime.UtcNow)
            throw new DomainException("Cannot schedule in the past");
        
        var appointment = new Appointment
        {
            Id = Guid.NewGuid(),
            ProjectId = projectId,
            CustomerPhone = customerPhone,
            ScheduledTime = scheduledTime,
            Status = AppointmentStatus.Pending
        };
        
        // Publicar evento de dominio
        appointment.AddDomainEvent(new AppointmentCreatedDomainEvent(appointment));
        
        return appointment;
    }
}
```

### 4. **Infrastructure Layer** (`MeetLines.Infrastructure`)

**Responsabilidades:**
- Implementar repositorios (acceso a datos)
- Integrar con base de datos (Entity Framework)
- Implementar servicios externos (Email, SMS, WhatsApp)
- Configurar inyección de dependencias
- Manejar caché, logging, etc.

**Componentes principales:**
```
MeetLines.Infrastructure/
├── Data/                # DbContext, migrations, seeds
├── Repositories/        # Implementaciones de repositorios
├── Services/            # Servicios externos
│   ├── NotificationService  # Email, SMS
│   ├── WhatsappService      # Integración WhatsApp
│   ├── PaymentService       # Procesamiento de pagos
│   └── AuthService          # OAuth, JWT
├── IoC/                 # Configuración de inyección de dependencias
└── Configuration/       # Configuración de Entity Framework
```

**Ejemplo de repositorio:**
```csharp
public class AppointmentRepository : IAppointmentRepository
{
    private readonly MeetLinesDbContext _context;
    
    public async Task<Appointment> GetByIdAsync(Guid id)
    {
        return await _context.Appointments
            .Include(a => a.Service)
            .Include(a => a.Customer)
            .FirstOrDefaultAsync(a => a.Id == id);
    }
    
    public async Task<IEnumerable<Appointment>> GetByProjectAsync(Guid projectId)
    {
        return await _context.Appointments
            .Where(a => a.ProjectId == projectId)
            .ToListAsync();
    }
    
    public async Task Add(Appointment appointment)
    {
        await _context.Appointments.AddAsync(appointment);
    }
    
    public void Update(Appointment appointment)
    {
        _context.Appointments.Update(appointment);
    }
}
```

### 5. **Shared Kernel** (`SharedKernel`)

**Responsabilidades:**
- Utilidades comunes reutilizables
- Clases base para entidades y agregados
- Manejo de eventos
- Extensiones

```csharp
public abstract class AggregateRoot : Entity
{
    private List<DomainEvent> _domainEvents = new();
    
    public IReadOnlyList<DomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    public void AddDomainEvent(DomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }
    
    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
}
```

## Patrones Clave

### Unit of Work Pattern

Coordina las operaciones de base de datos:

```csharp
public interface IUnitOfWork
{
    IAppointmentRepository Appointments { get; }
    IServiceRepository Services { get; }
    IProjectRepository Projects { get; }
    
    Task SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitAsync();
    Task RollbackAsync();
}
```

### Repository Pattern

Abstrae el acceso a datos:

```csharp
public interface IAppointmentRepository : IRepository<Appointment>
{
    Task<Appointment> GetByIdAsync(Guid id);
    Task<IEnumerable<Appointment>> GetByProjectAsync(Guid projectId);
    Task<IEnumerable<Appointment>> GetAvailableSlotsAsync(Guid serviceId);
}
```

### Dependency Injection

Configuración en `Program.cs`:

```csharp
builder.Services
    .AddApplicationServices()        // UseCases, Validators
    .AddInfrastructureServices()     // Repositories, Services
    .AddDomainServices();            // Domain logic services
```

### CQRS (Command Query Responsibility Segregation)

Separación de operaciones de lectura y escritura:

```csharp
// Command (Write)
public class CreateAppointmentCommand : IRequest<AppointmentDto>
{
    public Guid ProjectId { get; set; }
    public string CustomerPhone { get; set; }
    public DateTime ScheduledTime { get; set; }
}

// Query (Read)
public class GetAppointmentsQuery : IRequest<IEnumerable<AppointmentDto>>
{
    public Guid ProjectId { get; set; }
}
```

## Tecnologías Clave

| Componente | Tecnología | Propósito |
|-----------|-----------|----------|
| Framework | ASP.NET Core 8 | API web |
| ORM | Entity Framework Core | Acceso a datos |
| Base de Datos | PostgreSQL | Almacenamiento persistente |
| Validación | FluentValidation | Validación de modelos |
| Mapeo | AutoMapper | Mapeo de DTOs ↔ Entidades |
| Autenticación | JWT + OAuth2 | Seguridad |
| Logging | Serilog | Registro de eventos |
| Testing | xUnit + Moq | Pruebas automatizadas |

## Flujo de Solicitud Típico

```
HTTP Request
    ↓
[Middleware] → Auth, CORS, Logging
    ↓
[Controller] → Validar entrada
    ↓
[UseCase] → Orquestar lógica
    ↓
[Domain] → Aplicar reglas de negocio
    ↓
[Repository] → Acceder/persistir datos
    ↓
[Infrastructure] → Base de datos, servicios externos
    ↓
[Response] → DTO + Status Code
    ↓
[Middleware] → Logging, Compression
    ↓
HTTP Response
```

## Ventajas de esta Arquitectura

✅ **Testabilidad**: Cada capa puede probarse independientemente  
✅ **Mantenibilidad**: Cambios localizados no afectan otras capas  
✅ **Escalabilidad**: Fácil agregar nuevas funcionalidades  
✅ **Flexibilidad**: Cambiar implementaciones sin afectar la lógica de negocio  
✅ **Independencia de frameworks**: El dominio no depende de tecnologías específicas  

## Referencias

- [Clean Architecture - Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Domain-Driven Design - Eric Evans](https://www.domainlanguage.com/ddd/)
- [ASP.NET Core Documentation](https://learn.microsoft.com/en-us/aspnet/core/)
